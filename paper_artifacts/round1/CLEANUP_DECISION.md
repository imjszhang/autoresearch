# Round 1 Cleanup Decision

> Date: 2026-04-17

## Decision

No destructive cleanup was performed as part of the paper archive step.

The archive is now complete enough that cleanup is safe in principle, but the experiment environment is being left intact by default to preserve immediate reproducibility and avoid deleting anything the user may still want to inspect.

## Safe Cleanup Candidates Identified

From the WSL experiment workspace:

- `run_*.log`: 348 files, about 12.8 MB
- helper shell scripts `_*.sh`: 12 files, about 2.6 KB
- helper Python scripts `_*.py`: 8 files, about 10.3 KB
- `*.pid`: 27 files, about 138 B
- `.venv/`: about 7.41 GB

## Assets Explicitly Kept

- `.git/`
- `experiment_data/`
- `work_dir/`
- branch history already pushed to `origin`
- archive branch `archive/round1-artifacts`
- offline package `paper_artifacts/autoresearch-round1-artifacts-2026-04-17-v1.zip`

## Recommended Next Action

If the goal is only to reclaim disk space, the safest order is:

1. delete transient WSL logs / helper files
2. verify the archive zip still opens and the archive branch is on `origin`
3. remove `.venv/` only if `uv sync` rebuild time is acceptable

This note records the decision that archival and cleanup are separate operations, and only the archival side has been executed in this run.
