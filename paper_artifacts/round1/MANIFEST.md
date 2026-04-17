# Round 1 Artifact Manifest

> Date: 2026-04-17
> Scope: `baseline-*`, `static-ct-*`, `adaptive-ct-*` for Round 1

## Purpose

This directory is the archival payload for the completed Round 1 paper-facing experiment set.
It preserves the primary evidence chain needed for result verification:

- per-run `results_*.tsv` tables
- run-level best commit / best `val_bpb` references
- minimal reproduction instructions

The canonical narrative summary remains in `work_dir/journal/2026-04-17/round1-experiment-summary.md`.

## Included Result Tables

| Run | File | Best Commit | Best `val_bpb` |
| ---- | ---- | ---- | ---- |
| `baseline-1` | `experiment_data/results_baseline-1.tsv` | `87faa61` | `1.084914` |
| `baseline-2` | `experiment_data/results_baseline-2.tsv` | `dc65330` | `1.106892` |
| `baseline-3` | `experiment_data/results_baseline-3.tsv` | `4376f7b` | `1.071198` |
| `static-ct-1` | `experiment_data/results_static-ct-1.tsv` | `bc556a4` | `1.070910` |
| `static-ct-2` | `experiment_data/results_static-ct-2.tsv` | `33467ae` | `1.076708` |
| `static-ct-3` | `experiment_data/results_static-ct-3.tsv` | `826c359` | `1.080227` |
| `adaptive-ct-1` | `experiment_data/results_adaptive-ct-1.tsv` | `f019769` | `1.088484` |
| `adaptive-ct-2` | `experiment_data/results_adaptive-ct-2.tsv` | `d1fad32` | `1.078147` |
| `adaptive-ct-3` | `experiment_data/results_adaptive-ct-3.tsv` | `be4e9c4` | `1.066517` |

## Branches Backed By Remote Archive

The experiment branch family is expected to exist on `origin`:

- `autoresearch/baseline`
- `autoresearch/baseline-1`
- `autoresearch/baseline-2`
- `autoresearch/baseline-3`
- `autoresearch/baseline-pilot-1`
- `autoresearch/static-ct`
- `autoresearch/static-ct-1`
- `autoresearch/static-ct-2`
- `autoresearch/static-ct-3`
- `autoresearch/static-ct-pilot-1`
- `autoresearch/adaptive-ct`
- `autoresearch/adaptive-ct-1`
- `autoresearch/adaptive-ct-2`
- `autoresearch/adaptive-ct-3`
- `autoresearch/adaptive-ct-pilot-1`

## Evidence Hierarchy

Use the following priority order when verifying the paper:

1. `paper_artifacts/round1/experiment_data/results_*.tsv`
2. `work_dir/experiment/logs/round_state.json`
3. `work_dir/journal/2026-04-17/round1-experiment-summary.md`
4. per-run agent logs under `work_dir/experiment/logs/`
5. branch tips and intermediate commits on `origin`

## Notes

- `results.tsv` at repository root is not the archival source of truth.
- WSL temporary files such as `run_*.log`, `*.pid`, `nohup*.out`, and helper scripts are intentionally excluded from this archive branch.
- The WSL cache `~/.cache/autoresearch/` is useful for convenience, but not required if `prepare.py` can regenerate the data deterministically.
