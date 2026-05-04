# PhD Research Roadmap

Live ordering of research PRs against this fork. Each PR is *small, reviewable, and reversible*. We prefer many narrow PRs over a few large ones so that ablations stay legible.

## Dependency graph

```
PR-R1 (repro)  --->  PR-R2 (faithfulness)  --->  PR-R4 (autoresearch loops)
        \                                              /
         \---> PR-R3 (swarm-bridge) -------------------
                          \
                           \---> PR-R5 (DealLens cross-pollination)
```

PR-R1 is the foundation: every later experiment must emit a repro manifest, otherwise its results are not citable.

## PR sequence

### PR-R1 - Reproducibility manifest + replay harness
- **Goal:** every Feynman run emits `run.manifest.json` (model id, sampling params, tool versions, retrieval cache hash, seed, git SHA, env hash).
- **Deliverables:** `research/repro/SPEC.md`, `research/repro/manifest.schema.json`, `research/repro/replay.py` (loads manifest, re-executes deterministically against cached retrieval).
- **Success metric:** >= 95% byte-identical memo replay on a 20-run regression set when retrieval is cached; >= 0.9 ROUGE-L when retrieval is live.
- **Risk:** non-determinism in upstream LLM APIs. Mitigation: pin temperature=0 + capture logprobs; treat live-mode as a separate metric.

### PR-R2 - Source-grounding faithfulness eval
- **Goal:** automated entailment scoring of every cited claim in a Feynman memo.
- **Deliverables:** `research/faithfulness/rubric.md`, `judge_prompts/`, `scorer.py` (NLI + LLM-judge ensemble), `eval_sets/feynman-fa-v0.jsonl` (200 hand-labeled claim/source pairs).
- **Success metric:** judge agreement with human labels Cohen's kappa >= 0.7 on held-out set.
- **Connects to upstream:** results feed back into Feynman's existing eval harness as a new metric column.

### PR-R3 - Agent-swarm bridge
- **Goal:** pluggable adapters that let Feynman planner delegate sub-tasks to external swarms.
- **Adapters (one file each, stub first):**
  - `swarm-bridge/adapters/ruflo_adapter.py`        (ruvnet/ruflo)
  - `swarm-bridge/adapters/agent_farm_adapter.py`   (claude_code_agent_farm)
  - `swarm-bridge/adapters/claude_swarm_adapter.py` (claude-swarm)
  - `swarm-bridge/adapters/openclaw_adapter.py`     (openclaw)
- **Topology ablations:** star, mesh, hierarchical, debate-then-judge.
- **Success metric:** statistically significant improvement (paired bootstrap, alpha=0.05) over single-planner baseline on the faithfulness + memo-quality composite.

### PR-R4 - Autoresearch self-improvement loops
- **Goal:** iterative memo refinement loop with (a) memo-quality scorer, (b) thesis-discovery proposer, (c) gating against reward hacking.
- **Deliverables:** `autoresearch/controller.py`, `autoresearch/scoring_model.md`, `autoresearch/anti_reward_hacking.md`.
- **Success metric:** monotone non-decreasing quality across N=5 rounds on a held-out prompt set, with no faithfulness regression > 2 pts.

### PR-R5 - DealLens cross-pollination
- **Goal:** wire gstack-style deal-state machine + paperclip-style evidence graph into Feynman as a `deallens` skill pack; benchmark vs. vanilla deep-research on a VC memo eval set.
- **Deliverables:** `deallens/skill_pack.yaml`, `deallens/state_machine.md`, `deallens/evidence_graph.md`, `deallens/benchmark/`.
- **Success metric:** human-rated memo quality lift >= 15% on a 30-deal benchmark, with no faithfulness regression.

## Cadence

- One PR open at a time per pillar; lab-notebook entry in `research/notes/` every Friday.
- All experiments tracked in `research/benchmarks/leaderboard.md`.
- Anything promoted to upstream goes through a separate `upstream/` branch and a clean cherry-pick.
