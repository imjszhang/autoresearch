# Round 1 Reproducibility Guide

> Date: 2026-04-17
> Audience: paper authors, reviewers, and future reruns

## What Must Be Preserved

The minimum artifact set for Round 1 consists of:

1. `paper_artifacts/round1/experiment_data/results_*.tsv`
2. `work_dir/experiment/programs/baseline.md`
3. `work_dir/experiment/programs/static_ct.md`
4. `work_dir/experiment/programs/adaptive_ct.md`
5. `work_dir/experiment/docs/expected_results.md`
6. `work_dir/experiment/logs/round_state.json`
7. `work_dir/journal/2026-04-17/round1-experiment-summary.md`
8. git history for all `autoresearch/*` branches
9. environment lock files: `pyproject.toml`, `uv.lock`, `.python-version`

## What Is Optional

These are convenient but not required for paper verification:

- `~/.cache/autoresearch/` if `prepare.py` can rebuild the dataset/tokenizer deterministically
- local WSL `.venv/`
- WSL temporary runner files such as `run_*.log`, `*.pid`, `_*.sh`, `_*.py`

## How To Reconstruct The Environment

1. Clone the repository.
2. Check out the archival branch or unpack the Round 1 archive.
3. Recreate the Python environment with the locked dependencies:
   - `uv sync`
4. Rebuild cached data if needed:
   - `uv run prepare.py`
5. Use the archived `results_*.tsv` files as the source of truth for statistics and plots.

## How To Rebuild Paper Figures And Tables

Use `paper_artifacts/round1/experiment_data/results_*.tsv` as the primary input.

Recommended procedure:

1. Compute per-run best `val_bpb` by taking the minimum `val_bpb` among rows with `status=keep`.
2. Group runs into:
   - `baseline-*`
   - `static-ct-*`
   - `adaptive-ct-*`
3. Recompute the metrics described in `work_dir/experiment/docs/expected_results.md`:
   - final / best `val_bpb`
   - keep rate
   - crash count
   - learning curves / AUC
   - adaptive threshold trajectory from `ADAPT` rows
4. Cross-check the final summary against `work_dir/journal/2026-04-17/round1-experiment-summary.md`.

## Interpretation Rules

- Trust `results_*.tsv` over root-level `results.tsv`.
- Trust archived run summaries and `round_state.json` over transient WSL stdout logs.
- When a git HEAD temporarily moved ahead of a logged result, the archival state was reconciled back to the last valid logged keep.

## Round 1 Final Reference Point

- Best overall run: `adaptive-ct-3`
- Best commit: `be4e9c4`
- Best `val_bpb`: `1.066517`

## Excluded On Purpose

The following were intentionally not made part of the paper archive branch:

- transient WSL process-management files
- interrupted nohup / screen / MCP helper logs
- ad hoc debugging helpers created during recovery

They are excluded because they do not change the scientific conclusions once `results_*.tsv`, journals, and branch history are preserved.
