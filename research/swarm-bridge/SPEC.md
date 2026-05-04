# PR-R3: Agent-Swarm Bridge

## Problem
Feynman is a single-planner deep-research agent. Open-source swarms (ruvnet/ruflo, claude_code_agent_farm, claude-swarm, openclaw) explore the design space of *parallel multi-agent epistemics*. We want to know whether and when swarms beat a single planner on long-horizon memos, controlling for tokens.

## Adapter contract
All adapters implement `SwarmAdapter`:
```
class SwarmAdapter(Protocol):
    def submit(self, subtask: SubTask, budget: Budget) -> SwarmHandle: ...
    def poll(self, handle: SwarmHandle) -> SwarmResult | NotReady: ...
    def cancel(self, handle: SwarmHandle) -> None: ...
    name: str
    capabilities: set[Capability]   # {RESEARCH, CRITIQUE, CODE, SYNTHESIS}
```
Feynman planner sees swarms as virtual skills; manifest (PR-R1) records swarm name + commit SHA.

## Topology ablations
- **Star** - planner fans out to N parallel researchers, then merges.
- **Mesh** - researchers exchange intermediate findings before submission.
- **Hierarchical** - sub-planners spawn their own swarms (depth=2).
- **Debate-then-judge** - 2+ swarms argue, third LLM adjudicates.

## Token-controlled comparison
Baseline single planner is given the same total token budget as the swarm so wins are not just "more compute".

## Acceptance
- Paired bootstrap (alpha=0.05, n=30 prompts) shows >=1 topology with positive lift on (faithfulness * quality) composite.
- No regression of >2 pts on faithfulness vs. baseline.
- Per-swarm cost-per-quality reported.
