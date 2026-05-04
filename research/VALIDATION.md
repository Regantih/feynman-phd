# PR1-PR5 Validation: Five-Axis Stress Test

Validates the PR-R1..R5 build against the dimensions a Marketlogic / Sequoia-grade analyst (and a PhD committee) would actually press on. For each axis: how the build addresses it, what would invalidate it, and the operational metric.

Legend: R1=Repro, R2=Faithfulness, R3=Swarm-Bridge, R4=Autoresearch, R5=DealLens.

---

## Axis 1 - Industry & Cross-Industry Coverage

**Why it matters.** A research agent that only works on SaaS memos is a toy. PhD-grade claims require evidence that the system generalizes across vertical structures (regulatory regimes, evidence types, jargon).

**How the build addresses it**
- R2 eval set v0 spans 5 domains (VC, biomedical, legal, climate, geopolitics) so faithfulness is not a single-vertical artifact.
- R5 30-deal benchmark is *cross-industry by construction*: enterprise SaaS, biotech, defense-tech, climate, fintech, robotics. Stage-gating is industry-agnostic; sub-skills (MarketSizer, MoatAnalyst, RiskCartographer) are industry-aware via parameterized prompts.
- R3 swarm topologies are evaluated *per industry slice* on the same prompts to expose where parallelism helps (e.g. biotech's evidence sprawl) vs. hurts (terms-heavy memos).
- R4 scoring weights are industry-conditioned (citation density matters more in biotech/legal than in early-stage SaaS).

**Cross-industry transfer test**
- Train autoresearch (R4) scorer weights on 4 industries, hold one out, measure quality lift. Pass if held-out lift >= 60% of in-distribution lift.
- DealLens evidence-graph schema must accept all 6 industries with <=10% custom-typing per industry.

**Invalidator.** If R5 lift concentrates in 1-2 industries and is flat or negative elsewhere, the cross-industry claim fails and we reframe as a *vertical* contribution.

---

## Axis 2 - Customer Pain-Point Amplification

**Why it matters.** Memos that only restate decks miss the asymmetric upside: surfacing pain that founders *under-state* and customers *over-pay to solve*. This is where alpha lives.

**How the build addresses it**
- R5 sub-skill `RiskCartographer` is paired with a `PainAmplifier` step (added to skill_pack.yaml) that: (a) extracts pains from primary-source customer interviews / G2 / Reddit / earnings transcripts via R3 swarm, (b) ranks by severity x frequency x willingness-to-pay proxies, (c) cross-checks against the deck's stated pain.
- R2 faithfulness gate prevents fabricated pains: every amplified pain must cite >= 2 independent primary sources.
- R4 thesis-discovery proposer is biased toward *missing pain hypotheses* ("what pain is consistent with the traction curve but not stated?").

**Operational metric**
- Pain-Discovery Recall: of N partner-flagged "real pains" per deal, fraction surfaced by DealLens. Target >= 0.7.
- Pain-Severity Calibration: rank correlation (Spearman) between system pain-severity scores and partner ratings >= 0.5.
- Pain Novelty: fraction of surfaced pains *not* present in the founder deck >= 0.3.

**Invalidator.** If novelty < 0.1 across the benchmark, system is parroting decks; we cut PainAmplifier or rebuild it on different primary-source mix.

---

## Axis 3 - Customer Spending Potential

**Why it matters.** TAM theater is the #1 VC memo failure mode. Real spend potential = (segment headcount) x (budget authority) x (replacement willingness) x (cycle time) - and it is industry-shaped.

**How the build addresses it**
- R5 schema mandates a `spend_model` block per deal: bottoms-up (account count x ACV x logo-velocity) AND tops-down (BLS/IBIS/Statista cross-check) AND budget-line attribution (which existing line item gets displaced).
- R2 faithfulness scorer treats every quantitative spend claim as a load-bearing claim: must cite primary numeric sources, NLI-checked at the figure level (not just the sentence).
- R3 ComparablesFinder swarm pulls public-comp ACVs, NRR, payback, and segment penetration in parallel; the spend model must reconcile vs. >=3 comps.
- R4 anti-reward-hacking gate explicitly flags monotone TAM inflation across rounds (a known failure mode).

**Operational metric**
- Spend-Model Faithfulness (R2 sub-score) >= 0.9 on numeric claims.
- Bottoms-up vs. tops-down delta documented; |delta|/mean <= 0.4 or memo must explain the gap.
- Budget-line attribution present in 100% of IC-stage memos (stage-gating).

**Invalidator.** If spend models pass our checks but partners reject them as "directionally wrong" >40% of the time, our scorer is mis-specified; iterate scorer + add partner-feedback loop into R4.

---

## Axis 4 - Geographic Coverage

**Why it matters.** A US-only research agent fails on EU privacy regimes, India payments rails, MENA capital flow, China dual-circulation, LatAm FX. Geography changes evidence sources, regulatory exposure, and even acceptable comparable sets.

**How the build addresses it**
- R5 schema fields `geo.hq`, `geo.customer_geo_mix`, `geo.regulatory_regime`, `geo.currency_exposure` are mandatory at IC stage.
- R3 swarms can be configured with region-specific retrieval pools (EU-hosted endpoints for GDPR-sensitive corpora; India-hosted for RBI filings; etc.). Adapter capability tags include `REGION:{US,EU,UK,IN,SG,AE,BR,JP,CN-ex}`.
- R2 eval set v1 (post-PR-R2) extends with non-US claim/source pairs (target: >= 30% non-US sources) so faithfulness scoring isn't anchored to English-language US sources.
- R1 manifests record retrieval-pool region so geographic provenance is auditable.

**Operational metric**
- Geo-Faithfulness Parity: faithfulness on non-US deals within 5 pts of US deals on the R5 benchmark.
- Source-Region Diversity: for any non-US-HQ deal, >= 40% of citations from in-region sources.
- Currency / regulatory normalization present in 100% of cross-border memos.

**Invalidator.** Faithfulness gap >10 pts US vs. non-US -> retrieval coverage problem; expand R3 region pools and re-baseline before publishing.

---

## Axis 5 - Sovereignty

**Why it matters.** Regulated capital (gov LPs, sovereign wealth, defense-aligned funds) and regulated customers (gov, healthcare, finance) impose data-residency, export-control, dual-use, and model-provenance constraints. "It works" is not enough; "it works without crossing a sovereign boundary" is the real bar.

**How the build addresses it**
- R1 manifest records: model provider, model region, retrieval-cache region, tool endpoints region, skill-pack provenance hash. Any cross-boundary call is loggable and auditable per run.
- R5 schema fields: `sovereignty.data_residency`, `sovereignty.export_controls` (EAR/ITAR/EU dual-use), `sovereignty.gov_customer_share`, `sovereignty.dual_use_risk`, `sovereignty.model_provenance_constraints`.
- R3 swarm-bridge supports *sovereign-mode* runs that restrict adapters to in-region providers and refuse cross-border tool calls; the bridge fails closed.
- R2 faithfulness pipeline can run on-prem (deberta NLI + local LLM judge) so sensitive memos never require external judging.
- R4 autoresearch loops can be pinned to a single sovereign stack; reward-hacking gate blocks rounds that introduce out-of-region citations when sovereign-mode is set.

**Operational metric**
- Sovereign-Mode Conformance: 0 cross-boundary calls when sovereign-mode flag is set, verified by R1 manifest audit on 100% of sovereign runs.
- Export-Control Flagging Recall: on a 20-deal injected-risk set, system flags ITAR/EAR/EU-dual-use exposure with >= 0.9 recall, <= 0.2 false-positive rate.
- Model-Provenance Disclosure present in every memo footer.

**Invalidator.** Any cross-boundary leak in sovereign-mode is a hard fail; ship a fix and re-run audit before any further experiments are accepted.

---

## Roll-up acceptance for the PhD program

The PR1-PR5 build is considered *validated* only when ALL five axes meet their operational metrics on the R5 30-deal benchmark, replayable from R1 manifests, with R2 faithfulness >= 0.85 memo-level, R3 showing >=1 topology with statistically significant lift, and R4 showing monotone non-decreasing quality across 5 rounds without reward-hacking-gate breaches.

Failure on any single axis does not invalidate the program; it scopes the publishable claim. We track each axis as an independent hypothesis with its own go/no-go.
