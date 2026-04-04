# autoresearch — Adaptive Cyber-Taoist

This is an experiment to have the LLM do its own research, guided by the cyber-taoist Guard/Break framework with self-evolving parameters.

## Setup

1. Verify you are on the correct experiment branch.
2. Read the in-scope files for full context:
   - `README.md` — repository context.
   - `prepare.py` — fixed constants, data prep, tokenizer, dataloader, evaluation. Do not modify.
   - `train.py` — the file you modify. Model architecture, optimizer, training loop.
3. Verify data exists: Check that `~/.cache/autoresearch/` contains data shards and a tokenizer. If not, tell the human to run `uv run prepare.py`.
4. **Check for existing progress**: If `results.tsv` already exists and contains data rows beyond the header, this is a **resumed session**. Read the existing data, count the data rows to determine the current iteration number, and skip directly to the experiment loop. Do NOT re-initialize `results.tsv` or re-run the baseline. If any `ADAPT` rows exist, extract the most recent adapted θ_guard and θ_break values from the rationale column and use those as the current thresholds.
5. **Fresh start only**: If `results.tsv` does not exist or contains only the header, initialize it with just the header row. The baseline will be recorded after the first run.
6. Confirm and go.

## Experimentation

Each experiment runs on a single GPU. The training script runs for a **fixed time budget of 5 minutes** (wall clock training time, excluding startup/compilation). You launch it simply as: `uv run train.py`.

**What you CAN do:**
- Modify `train.py` — this is the only file you edit. Everything is fair game: model architecture, optimizer, hyperparameters, training loop, batch size, model size, etc.

**What you CANNOT do:**
- Modify `prepare.py`. It is read-only. It contains the fixed evaluation, data loading, tokenizer, and training constants (time budget, sequence length, etc).
- Install new packages or add dependencies. You can only use what's already in `pyproject.toml`.
- Modify the evaluation harness. The `evaluate_bpb` function in `prepare.py` is the ground truth metric.

**The goal is simple: get the lowest val_bpb.** Since the time budget is fixed, you don't need to worry about training time — it's always 5 minutes. Everything is fair game: change the architecture, the optimizer, the hyperparameters, the batch size, the model size. The only constraint is that the code runs without crashing and finishes within the time budget.

**VRAM** is a soft constraint. Some increase is acceptable for meaningful val_bpb gains, but it should not blow up dramatically.

**Simplicity criterion**: All else being equal, simpler is better. A small improvement that adds ugly complexity is not worth it. Conversely, removing something and getting equal or better results is a great outcome — that's a simplification win.

## Cyber-Taoist Decision Framework (Adaptive)

Before each modification, you MUST compute the following metrics from `results.tsv` (using the last k=10 iterations). If fewer than 10 iterations exist, use all available data.

### Metrics

1. **Stagnation Score (S):** Fraction of the last k iterations with no val_bpb improvement (i.e., status is `discard` or `crash`).
2. **Rollback Rate (RR):** Fraction of the last k iterations that were discarded or crashed.
3. **Improvement Decay (ID):** Average val_bpb improvement magnitude in the last k successful (`keep`) iterations divided by the overall average improvement magnitude across all successful iterations. If no recent successes, ID = 0.
4. **Deviation δ = 0.5·S + 0.3·RR + 0.2·(1 - ID)**
5. **Code complexity growth ΔC:** (current train.py line count - initial train.py line count) / initial train.py line count.

### Decision Rules (use current thresholds)

**Current parameters** (updated at adaptation checkpoints):
- θ_guard = 0.3 (initial)
- θ_break = 0.7 (initial)
- α = 10% (complexity growth limit)
- k = 10 (lookback window)
- weights = (w₁=0.5, w₂=0.3, w₃=0.2)

Apply these rules in order:

1. **SIMPLIFY** — If ΔC > α AND the last experiment did not improve: refactor and reduce code complexity before attempting the next experiment.
2. **GUARD** — If δ < θ_guard: make incremental improvements (hyperparameter tuning, minor structural adjustments, small optimizations).
3. **BREAK** — If δ > θ_break: attempt a fundamental change (new attention mechanism, different architecture, novel training technique, radical hyperparameter shift).
4. **SIMPLIFY** — If θ_guard ≤ δ ≤ θ_break: reduce complexity first, then reassess.

You MUST always compute δ and follow the prescribed strategy. Do not override the decision rules based on your own intuition.

### Parameter Adaptation (every 50 iterations)

At iterations 50, 100, 150, and 200, perform the following adaptation:

1. Analyze ALL accumulated data in `results.tsv`.
2. For each δ interval [0, 0.2), [0.2, 0.4), [0.4, 0.6), [0.6, 0.8), [0.8, 1.0]:
   - Compute the success rate of GUARD decisions in that interval (success = status is `keep`).
   - Compute the success rate of BREAK decisions in that interval.
   - Require at least 3 data points per cell; otherwise mark as insufficient data.
3. Identify the δ value where GUARD success rate drops below BREAK success rate. Set this as the new θ_guard.
4. Identify the δ value where BREAK success rate becomes clearly dominant. Set this as the new θ_break.
5. If insufficient data in some intervals, keep previous thresholds for those boundaries.
6. Clamp: θ_guard ∈ [0.1, 0.6], θ_break ∈ [θ_guard + 0.1, 0.9].
7. **Log the adaptation**: old thresholds → new thresholds, data table used for the decision, and reasoning.

This self-evolution mechanism allows the framework to learn from its own experimental history and adjust the Guard/Break boundaries accordingly.

## Output format

Once the script finishes it prints a summary like this:

```
---
val_bpb:          0.997900
training_seconds: 300.1
total_seconds:    325.9
peak_vram_mb:     45060.2
mfu_percent:      39.80
total_tokens_M:   499.6
num_steps:        953
num_params_M:     50.3
depth:            8
```

You can extract the key metric from the log file:

```
grep "^val_bpb:" run.log
```

## Logging results

When an experiment is done, log it to `results.tsv` (tab-separated, NOT comma-separated).

The TSV has a header row and 8 columns:

```
commit	val_bpb	memory_gb	status	description	delta	strategy	rationale
```

1. git commit hash (short, 7 chars)
2. val_bpb achieved (e.g. 1.234567) — use 0.000000 for crashes
3. peak memory in GB, round to .1f (e.g. 12.3 — divide peak_vram_mb by 1024) — use 0.0 for crashes
4. status: `keep`, `discard`, or `crash`
5. short text description of what this experiment tried
6. δ value computed before this iteration (e.g. 0.450) — use 0.000 for the first run
7. strategy chosen: `GUARD`, `BREAK`, `SIMPLIFY`, or `ADAPT` — use `BASELINE` for the first run
8. brief rationale for the strategy and specific modification chosen

Additionally, at each adaptation checkpoint (iterations 50, 100, 150, 200), append a special row:

```
ADAPT	0.000000	0.0	adapt	parameter adaptation at iter N	0.000	ADAPT	θ_guard: old→new; θ_break: old→new
```

Do NOT commit results.tsv — leave it untracked by git.

## The experiment loop

**The first run**: If this is a fresh start (no data in `results.tsv`), your first run establishes the baseline — run the training script as is, without any modifications. Record the initial train.py line count for ΔC calculation. If resuming, determine the initial line count from the git log (the first commit's train.py) and continue from the next iteration.

LOOP (up to 200 iterations):

1. Read current `train.py` and `results.tsv`.
2. Compute δ from the last k=10 entries and select strategy (GUARD / BREAK / SIMPLIFY) using the current thresholds.
3. Log your analysis: δ value, S, RR, ID components, ΔC, current θ_guard/θ_break, chosen strategy, and rationale.
4. Implement the modification according to the selected strategy.
5. git commit.
6. Run the experiment: `uv run train.py > run.log 2>&1` (redirect everything — do NOT use tee or let output flood your context).
7. Read out the results: `grep "^val_bpb:\|^peak_vram_mb:" run.log`
8. If the grep output is empty, the run crashed. Run `tail -n 50 run.log` to read the Python stack trace and attempt a fix.
9. Record the results in results.tsv (all 8 columns).
10. If val_bpb improved (lower), you "advance" the branch, keeping the git commit.
11. If val_bpb is equal or worse, you git reset back to where you started.
12. **Every 10 iterations**: Review meso-level trends — are GUARD decisions succeeding? Are BREAK decisions leading to discoveries? Summarize briefly.
13. **At iterations 50, 100, 150, 200**: Perform parameter adaptation (see above). Log old and new thresholds.
14. **Every 50 iterations**: Also review macro-level direction — is the overall research trajectory productive? How have threshold adaptations affected decision quality?
15. Repeat from step 1.

**Timeout**: Each experiment should take ~5 minutes total (+ a few seconds for startup and eval overhead). If a run exceeds 10 minutes, kill it and treat it as a failure (discard and revert).

**Crashes**: If a run crashes (OOM, or a bug, or etc.), use your judgment: If it's something dumb and easy to fix (e.g. a typo, a missing import), fix it and re-run. If the idea itself is fundamentally broken, just skip it, log "crash" as the status in the tsv, and move on.

**NEVER STOP**: Once the experiment loop has begun, do NOT pause to ask the human if you should continue. The human might be asleep, or gone from a computer and expects you to continue working *indefinitely* until you are manually stopped. You are autonomous. If you run out of ideas, think harder — read papers referenced in the code, re-read the in-scope files for new angles, try combining previous near-misses, try more radical architectural changes. The loop runs until the human interrupts you, period.
