# Feynman PhD Research Program

This fork extends [getcompanion-ai/feynman](https://github.com/getcompanion-ai/feynman) into a PhD-grade research testbed for **agentic research systems**. The Feynman codebase already provides a strong substrate (deep-research orchestration, source-grounding, skill packs, evaluation harness), so we treat it as the *vehicle* for a coordinated set of experiments rather than rebuilding from scratch.

## Why Feynman as a research platform

- **Source-grounded generation** with explicit citations -> direct hook for faithfulness / hallucination metrics.
- **Skill packs + planner** -> a clean place to inject new agent topologies (DealLens VC analyst, autoresearch loops, paperclip-style deal-state machines).
- **Experiments harness** (`experiments/`) already wired -> we extend it with reproducibility manifests, evidence graphs, and self-improvement loops.
- **Open MIT license** -> publishable artifacts, datasets, and code without IP friction.

## Research pillars (PRs map 1:1 to pillars)

| Pillar | Question | Artifacts | Tracking PR |
|---|---|---|---|
| P1. Reproducibility of agentic research | Can we deterministically replay a deep-research run given a manifest of (model, tools, seeds, retrieved docs)? | `research/repro/` manifest spec, replay harness | PR-R1 |
| P2. Source-grounding faithfulness | What fraction of generated claims are *entailed* by cited sources, and how does it degrade with planner depth? | `research/faithfulness/` rubric, judge prompts, eval set | PR-R2 |
| P3. Multi-agent epistemics | Do swarms (ruvnet/ruflo, claude_code_agent_farm, claude-swarm, openclaw) outperform single-planner Feynman on long-horizon memos? Under what topology? | `research/swarm-bridge/` adapters + ablations | PR-R3 |
| P4. Self-improving research loops | Can autoresearch-style scoring + thesis-discovery loops raise memo quality monotonically across N rounds without reward hacking? | `research/autoresearch/` controllers, scoring models | PR-R4 |
| P5. DealLens cross-pollination | Does plugging gstack/paperclip deal-state + evidence-graph plumbing into Feynman improve VC memo quality vs. vanilla deep-research? | `research/deallens/` skill pack + benchmark | PR-R5 |

## Repository layout (additive, non-breaking)

```
research/
  README.md              <- this file
  roadmap.md             <- PR sequence + dependencies
  repro/                 <- P1: reproducibility manifests + replay
  faithfulness/          <- P2: grounding eval harness
  swarm-bridge/          <- P3: agent-swarm adapters
  autoresearch/          <- P4: self-improvement loops
  deallens/              <- P5: VC analyst skill pack
  benchmarks/            <- shared eval datasets + leaderboards
  notes/                 <- lit review, design memos, lab notebook
```

Nothing under `research/` ships with the upstream product; it is strictly an experimental overlay. Production code paths in `.feynman/`, `experiments/`, `extensions/` remain authoritative until a research pillar is promoted.

## Upstream relationship

- We **sync** from `getcompanion-ai/feynman:main` regularly.
- Research-only changes stay in this fork.
- Generally useful improvements (bug fixes, skill-pack APIs, eval harness upgrades) are proposed back upstream as small PRs.

## Status

Scaffold in progress. See `research/roadmap.md` for the live PR plan.
