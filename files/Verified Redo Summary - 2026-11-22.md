# Verified Redo — Summary for Delivery (2026-11-22)

Prepared 2026-11-22 (America/Los_Angeles) from the two redo reports already completed
in this run's research notes. This file is a delivery summary only; the reports are
the records of verification.

## What was redone, in Mohan's order
1. **Step 1 v2 — verified initial briefing:** `workspace/research_notes/verified-initial-briefing-agentic-data-v-20261122-2049/report.md`
2. **Step 2 v2 — verified gap analysis, using ONLY Step 1 v2 as baseline:** `workspace/research_notes/verified-gap-analysis-failure-memory-v2-20261122-2051/report.md`

Both were done under Mohan's standing instruction (2026-11-22): deep research mode,
primary sources preferred, every key number double-confirmed, discrepancies reported
not silently resolved, unverified items labelled plainly.

## Headline verdict (Step 2 v2)
- **Paper #1 decision is kept: centre on failure memory for data agents, evaluated
  compute-fairly under schema/data drift and repeated failures.**
- But only **as a gap claim, not as an effectiveness claim**: no primary source
  verified in either pass shows failure/episodic memory improves a data agent on a
  public data benchmark at matched compute. That absence is precisely the gap.
- **Paper #1 must be empirical, not another architecture/framework paper.** His
  CJAST 2026 fingerprinting paper self-describes as a prospective framework with
  empirical evaluation deferred to future work.
- Recommended next step: a **~100-task pilot** — arms: no-memory single agent,
  episodic (raw, no LLM-consolidation) memory, CHASE-style generate-and-select at
  the *same total token budget*, generic-episodic ablation, stale-memory (pre-drift
  memory on post-drift tasks). Headline metric **pass^k (k=4 or 8)**. If memory does
  not beat no-memory on recurring failures in the pilot, stop and pivot to a
  compute-fair negative-result paper (G2/G5).

## Corrections the redo made to the v1 work
- **Publications (publisher-verified only, 5 of 7):** JISEM 2022 Historical Incident
  Reuse; IJISAE 2023 Contrastive Failure Embeddings; JISEM 2024 Incident Memory
  Layer (full title incl. subtitle); CJAST 2026 Reliability Gap / Failure
  Fingerprinting (DOI 10.9734/cjast/2026/v45i34673); CJAST 2026 PV-Aware Net-Load
  Forecasting (published 2026-11-17; results hedged in its own abstract — 6.82%
  lower QS7 on original 8 horizons, 0.50% worse across all 48, "no statistically
  conclusive difference" — do not summarise as a clean win).
- **NOT a published paper:** Semantic Data Quality for Agentic DataOps (ICIEEE
  2026) — verified only as a conference registration listing, Paper ID 519.
- **Could not verify on publisher site:** IJCNIS 2020 "From Observability to
  Action" — full text found only on Academia.edu, not on ijcnis.org.
- **Google Scholar:** no profile for Mogana Kumaran Sivaraman could be found or
  accessed; no citation counts are verified anywhere.
- **Spider 2.0 version labels (corrected in Step 2 v2):** arXiv v1 = **17.0%**,
  arXiv v2 / ICLR 2025 = **21.3%** for the o1-preview code agent — Step 1 v2 had
  these reversed. Always quote with version.
- **DAB:** v1 best **38% pass@1** (54 queries); later / EMNLP 2026 version best
  **57% pass@1** (104 queries). Quote with version.
- **Jin et al. annotation audit:** arXiv v3 (2601.08778) verified — **52.8% BIRD
  Mini-Dev / 62.8% Spider 2.0-Snow** annotation errors, ranks shifting −9 to +9.
  A UIUC experts listing for the CIDR 2026 version gives **66.1%** for Spider
  2.0-Snow — discrepancy unresolved, likely a version difference; check the
  CIDR camera-ready before quoting in a paper.
- **SWE-bench "1.96% → 78.4%" trajectory from the v1 briefing was dropped** in
  the redo: not verified as a clean trajectory (independent spot-check in this
  run found secondary sources attributing 78.4% to different models and a higher
  April-2026 top score of 87.6%).

## Independent spot-checks done in this proactivity run (2026-11-22)
- **SQLAgent IS real — corrects the v1 gap analysis.** Jiang et al., "SQLAgent:
  Learning to Explore Before Generating as a Data Engineer", ACL 2026 Findings,
  pp. 39311–39331; arXiv:2602.01952. Two-stage: MCTS-inspired exploration builds
  a database-specific knowledge base, then a dual-agent system generates SQL.
  The v1 claim that it "could not be located as a primary paper" is now
  superseded. Paper #1 related work must include it; it strengthens the
  exploration-failure class.
- **CMC Text-to-SQL study verified (primary journal):** Borovčak et al.,
  CMC 2026;88(2):80 — Qwen2.5-Coder single-agent **72.44%** execution accuracy
  on Spider dev vs **66.83%** multi-agent; latency 0.52s vs 2.49s; tokens
  421.7 vs 1117.08. Scope: open models only, Spider dev — not a general
  single-beats-multi claim (CHASE-SQL, a multi-agent system, leads BIRD at 73.0%).
- **Tran & Kiela (arXiv:2604.02460, Apr 2026) corroborated** via independent
  secondary summaries: single agent matches/outperforms multi-agent at equal
  thinking-token budgets on multi-hop reasoning, 3 model families, 5 architectures.
  General-agent evidence [G], not data-specific.

## Gaps scored in Step 2 v2 (novelty / feasibility / data / venue)
- G1 Failure memory, repeated failures [D, open]: 5 / 4 / 5 / VLDB 5 — Paper #1 core.
- G2 Compute-fair single vs multi [D, open]: 4 / 5 / 5 / 4 — cheapest gap, fairness protocol.
- G3 pass^k for data agents [D, open]: 4 / 4 / 5 / 3–4 — headline metric for Paper #1.
- G4 Drift / stale memory [D, open]: 5 / 3 / 2 / VLDB 5 — must build + release the drift split.
- G5 Corrected evaluation [D, partially addressed]: 3 / 3 / 4 / VLDB 5 — use Jin's corrected subset, report original + corrected labels.
- G6 Data failure taxonomy / debuggability: 4 / 3 / 3 / 4 — cheap add-on from collected traces.
- G7 Latency/cost in data MAS: 3 / 5 / 5 / 4 — near-free if combined with G2.
- G8 Lazy-agent / data-agent safety: **not scored** — openness not verified to the required standard; needs a dedicated systematic search. Keep out of Paper #1 motivation.

## What still needs Mohan
1. Confirm he has seen the redo verdict and the pilot as the next step (vs. going straight to a full literature review).
2. His Google Scholar link is still not shared — the publication list still cannot be confirmed complete against Scholar.
3. Daily briefing: next run Mon 2026-11-23 08:23:45 PT, already re-aimed at failure memory / stale memory / drift and already carrying the verification standard in its cron definition. No change needed.
