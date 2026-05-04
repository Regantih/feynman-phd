# PR-R4: Autoresearch Self-Improvement Loops

## Problem
A single Feynman pass yields fixed-quality output. Autoresearch-style loops (propose -> critique -> rewrite -> score) can iteratively raise quality, but naive loops drift, hallucinate, or game the scorer.

## Loop architecture
```
[draft memo] --> [scorer (quality + faithfulness)] --> [thesis-discovery proposer]
                          ^                                       |
                          |                                       v
                  [reward-hacking gate] <--- [rewriter w/ new evidence requests]
```

State persists in `runs/<id>/rounds/{0..N}/` with PR-R1 manifests per round.

## Components
- **Scoring model:** weighted sum of (faithfulness from PR-R2, structural completeness, novelty, citation density). Weights frozen per experiment.
- **Thesis-discovery proposer:** generates falsifiable sub-theses missing from the current memo; biased toward high-information-gain queries.
- **Anti-reward-hacking gate:** rejects rounds where score went up but (a) faithfulness dropped, (b) citation reuse exceeds threshold, (c) structural diversity dropped (n-gram self-BLEU spike).

## Acceptance
- Monotone non-decreasing composite score over N=5 rounds on 30 prompts (>=80% of prompts).
- Faithfulness regression <=2 pts vs. round 0.
- Reward-hacking gate fires correctly on injected adversarial rounds (>=90% recall).
