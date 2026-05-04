# PR-R2: Source-Grounding Faithfulness Eval

## Problem
Feynman emits citations, but no system measures whether each generated claim is *entailed* by its cited source. Without that metric we cannot defend memos in regulated industries (finance, healthcare, public sector).

## Pipeline
1. **Claim extraction:** parse memo into atomic claims `(claim_text, citation_ids)`.
2. **Evidence retrieval:** fetch the cited passages from the run's retrieval cache (per PR-R1 manifest).
3. **Entailment scoring:** ensemble of (a) NLI model (deberta-v3-large-mnli), (b) LLM-judge with rubric below, (c) abstention class for ambiguous claims.
4. **Aggregation:** memo-level faithfulness = supported_claims / total_claims; weighted by claim load-bearingness.

## Rubric (judge prompt anchors)
- `SUPPORTED` - claim is directly entailed by passage.
- `PARTIALLY_SUPPORTED` - core fact entailed, but quantifiers/dates/modifiers diverge.
- `UNSUPPORTED` - claim not entailed.
- `CONTRADICTED` - passage says opposite.
- `NOT_IN_SOURCE` - hallucinated citation.

## Eval set v0
200 hand-labeled (claim, source) pairs sampled across 5 domains (VC memos, biomedical, legal, climate, geopolitics). Stored in `eval_sets/feynman-fa-v0.jsonl`.

## Acceptance
- Cohen's kappa(judge, human) >= 0.7 on held-out 50.
- Reports per-depth degradation curve (claim faithfulness vs. planner depth).
- Becomes a required column in PR-R3/R4/R5 leaderboard rows.
