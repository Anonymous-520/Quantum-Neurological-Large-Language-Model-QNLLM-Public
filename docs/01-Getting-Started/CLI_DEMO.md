# QNLLM On-Device CLI Demo

Run QNLLM locally with no servers. Uses TBRH, provenance graphs, and deterministic replay.

## Commands

```bash
qnllm run task.json            # Execute task spec, store replayable record + snapshot (requires valid snapshot hash)
qnllm explain task.json        # Run then summarize (shortcut)
qnllm explain <output_id>      # Summarize stored output and hashes (validates snapshot)
qnllm replay <output_id|hash>  # Regenerate deterministically and verify hash (validates snapshot)
qnllm audit <output_id|hash>   # Show provenance graph + snapshot metadata (validates snapshot)
```

## Task Spec (task.json)
```json
{
  "task_id": "demo_task_1",
  "task": "explain",
  "memory_ids": [1, 2],
  "confidence": 0.82,
  "task_params": {"action": "stabilize", "reason": "demo"}
}
```

## Snapshot Guard
- Default snapshot: `data/snapshot_v2.5.qnllm`
- Expected SHA256: `533fde88c7e07e63b0a7071104887cb8c3fd7efdc86663b5da67882eb72eee2c`
- Override path: env `QNLLM_SNAPSHOT_PATH`
- CLI refuses to run if snapshot is missing or hash mismatches.

## What Gets Stored
- `tbrh_output.text` + `replay_hash`
- Provenance graph hash (Invariant 15)
- Task spec for deterministic replay
- Frozen memory snapshot metadata (hash + path)

Default storage: `~/.qnllm/runs` (override with `QNLLM_RUNS_DIR`).
