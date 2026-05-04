# PR-R1: Reproducibility Manifest + Replay Harness

## Problem
Agentic deep-research runs are non-deterministic black boxes: outputs depend on model build, sampling, retrieval ordering, tool versions, and time of day. Without a manifest, *no* downstream metric (faithfulness, swarm uplift, autoresearch monotonicity) is auditable.

## Manifest schema (v0)
Every Feynman run writes `runs/<run_id>/run.manifest.json`:

```json
{
  "run_id": "uuid",
  "created_at": "ISO-8601",
  "git": {"sha": "...", "dirty": false, "upstream_sha": "..."},
  "env": {"python": "3.11.x", "os": "...", "hash": "sha256(lockfile)"},
  "model": {"provider": "anthropic", "id": "claude-...", "sampling": {"temperature": 0, "top_p": 1, "seed": 42}},
  "tools": [{"name": "web_search", "version": "...", "config_hash": "..."}],
  "retrieval": {"cache_dir": "caches/<hash>", "cache_hash": "sha256", "live": false},
  "prompt": {"id": "...", "hash": "sha256"},
  "skills": [{"id": "deallens", "version": "0.1.0", "hash": "..."}],
  "outputs": {"memo_path": "...", "trace_path": "...", "claims_path": "..."},
  "metrics": {"faithfulness": null, "quality": null}
}
```

## Replay contract
`replay.py <manifest>`:
1. Verifies git SHA + lockfile hash match (else aborts unless `--force`).
2. Restores retrieval cache from `cache_hash`.
3. Re-runs planner with identical sampling + seed.
4. Diffs output bytes; emits `replay.report.json`.

## Acceptance gates
- Cached-mode replay: >= 95% byte-identical on 20-run regression set.
- Live-mode replay: >= 0.9 ROUGE-L vs. original memo.
- Manifest required by every later PR (R2-R5) before metrics are accepted.

## Failure modes tracked
- Provider stochasticity even at temp=0 -> log logprobs, flag drift.
- Tool API drift -> pin versions; warn on minor bumps, fail on major.
- Retrieval cache misses -> hard-fail in cached mode.
