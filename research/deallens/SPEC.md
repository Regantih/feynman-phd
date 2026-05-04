# PR-R5: DealLens Cross-Pollination

## Problem
VC memo generation has structure that vanilla deep-research ignores: deals progress through stages (sourcing -> screening -> diligence -> IC -> term sheet), and each stage has required evidence categories (team, market, traction, moat, terms). gstack's deal-state machine and paperclip's evidence-graph encode this structure cleanly. We port both into Feynman as a `deallens` skill pack.

## Components
- **Deal-state machine** (`state_machine.md`): nodes = stages; transitions guarded by required evidence completeness.
- **Evidence graph** (`evidence_graph.md`): typed nodes (Claim, Source, Metric, Team, Risk, Comparable); edges (supports, contradicts, derives_from). Backs both citation and faithfulness scoring.
- **Skill pack** (`skill_pack.yaml`): wires sub-skills (TeamProfiler, MarketSizer, TractionAuditor, MoatAnalyst, RiskCartographer, ComparablesFinder, TermSheetReviewer, FundFitScorer).
- **Variable schema** for deal evaluation (mandatory at IC stage):
  - team: founder-market-fit score, prior-exit count, key-hire risk
  - stage: pre-seed/seed/A/B; capital efficiency; runway
  - market: TAM/SAM/SOM with source-grounded derivation; growth rate; cyclicality
  - traction: revenue/usage curve, retention cohort, NRR
  - moat: data, distribution, regulatory, network effects
  - terms: valuation, dilution, pro-rata, governance
  - geo: HQ, customer geo mix, regulatory regime, currency exposure
  - sovereignty: data residency, export controls, dual-use risk, gov customer share

## Benchmark
30-deal benchmark spanning enterprise SaaS, biotech, defense-tech, climate, fintech, robotics. Each deal has a partner-graded reference memo.

## Acceptance
- Human-rated memo quality lift >= 15% vs. vanilla Feynman deep-research.
- Faithfulness (PR-R2) non-regression.
- Stage-gating: planner refuses to emit IC memo if evidence-graph completeness < 0.85 across required categories.
