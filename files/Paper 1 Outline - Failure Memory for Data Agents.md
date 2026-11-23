# Paper #1 Outline — Failure Memory for Data Agents
Author: Mogana Kumaran Sivaraman (Mohan), Gap Inc.
Decision date: 2026-11-22 — Paper #1 is centred on failure memory for data agents, using compute-fair evaluation as the method backbone.
Status: Outline v1, for review before literature review and experiment design in detail.

## Working titles (pick 1)
1. Does Failure Memory Make Data Agents Reliable? A Compute-Fair Evaluation Under Drift and Repeated Failures
2. From Incident Memory to Agent Memory: Failure Fingerprinting for Reliable Data Agents
3. Remembering How Data Work Fails: Evaluating Failure-Memory Agents on Text-to-SQL, Analytics, and ELT Tasks

## Core claim / thesis
Data agents fail repeatedly in recognisable ways — wrong schema links, stale metric definitions, broken joins, unsafe writes, failed transformations — yet standard agents start every task with no memory of prior failures, and standard benchmarks measure only one-shot outcome accuracy. A failure-memory layer for agent trajectories — fingerprinting failures, storing them as episodes, and retrieving them on recurrence — should improve reliability (pass^k, recovery) more than it improves one-shot accuracy, and stale memory should measurably harm performance under schema/data drift. No existing data-agent benchmark tests this.
Scope note (author instruction, 2026-11-22): The author's previous papers are background reference only and are NOT used in this paper — not as method, taxonomy, positioning, or citations. This outline stands on the public literature and the paper's own experiments.

## Why now (positioning)
- The author brings 19 years of data engineering practice (per author) to the problem selection and failure analysis — experience, not prior publications, is the positioning.
- Gap analysis (verified redo, 2026-11-22) found no data-specific benchmark or controlled study for agent failure memory, schema/data drift at agent level, compute-fair single vs multi-agent for data, or pass^k reliability for data agents.

## Problem formulation
Given data tasks T, agent architecture A (single-agent, single + self-debug, multi-agent verifier/centralised), memory condition M, budget B (tokens / $ / latency, fixed ex ante), and environment state E (original vs drifted schema/data):
- M ∈ {NoMemory, ExactFingerprint, EmbeddingRetrieval (contrastive), FullTraceRAG, StaleMemory (pre-drift), OracleMemory}
- Measure outcome, trajectory, reliability, safety, and cost jointly, over n ≥ 5 repeated runs.
Research questions:
- RQ1: Does failure memory improve pass^1, pass^3/pass^k, and recovery rate — at equal budget?
- RQ2: Which memory representation wins: exact fingerprint, embedding retrieval, or full-trace RAG? Where does each fail?
- RQ3: When does memory hurt? Stale-memory harm under drift: stale-schema hallucination rate, re-exploration rate.
- RQ4: Is any multi-agent advantage over single-agent preserved once memory and budget are both controlled?

Hypotheses (pre-register):
- H1: Memory improves reliability (pass^k) and recovery more than pass^1.
- H2: Embedding retrieval > exact fingerprint on paraphrased / mutated failures; exact > embedding on identical recurrences (precision).
- H3: Stale memory under drift performs worse than no memory on drifted tasks (negative transfer), unless the agent re-explores.
- H4: At iso-budget, MAS advantage (if any) shrinks once SAS gets the same memory — memory, not agent count, is the active ingredient for repeated-failure tasks.

## Proposed system / method to evaluate
Failure Memory Layer for agents (adaptation, not new platform):
1. Capture: after each run, encode the failure episode — task intent, schema elements touched, tool/SQL trajectory, error type, execution state diff, successful remediation.
2. Fingerprint: normalised deterministic fingerprint, defined in this paper for agent traces, + embedding of the trajectory for similarity retrieval.
3. Store: dual-index registry (fingerprint + vector), with timestamp / schema-version tagging (critical for drift experiments).
4. Retrieve & surface: at task start and on first failure, retrieve top-k similar failures + remediations; inject as concise memory, not full traces.
5. Forget / refresh: schema-version mismatch triggers re-exploration prompt / memory invalidation — the mechanism H3 tests.

## Experiments
Benchmarks (public, corrected where available):
- Text-to-SQL: BIRD Mini-Dev corrected subset (Jin et al. 2026 audit), Spider 2.0-lite
- Analytics pipelines: KramaBench subset
- Data engineering: ELT-Bench subset or data-eng-bench subset (Pass^3 precedent)
- Drift suite: programmatic operators on BIRD/Spider DBs — rename, split, re-code values, metric redefinition, table deprecation — at graded severity (from Drift-and-Destroy design in gap analysis)
Design:
- Repeated-failure task sequences: Session 1 tasks plant / encounter failure classes; Session 2+ tasks recur those classes (exact, mutated, drifted) — this session-chained design is what makes memory testable at all.
- Baselines: SAS, SAS+self-debug, MAS-centralised verifier, MAS-independent — all × memory conditions, iso-token and iso-$ arms.
- Models: 1 frontier API model + 1-2 open models.
- Power: n ≥ 5 runs on a powered subset, CIs, power analysis following On Randomness in Agentic Evals.

Metrics:
- Outcome: execution accuracy / verifier pass, semantic correctness (FLEX-style where EX is unreliable)
- Reliability: pass^1, pass^3, pass^k, variance decomposition (sampling vs exploration path vs memory retrieval)
- Cost-efficiency: tokens, $, latency, cost per reliable success (cost / pass^3)
- Trajectory (TrajData-style, programmatic where possible): schema exploration recall before first SQL, recovery rate (detects → diagnoses → fixes), redundant exploration, steps
- Safety: destructive-op count via sandbox state diff, read-before-write rate
- Memory-specific: retrieval precision/recall for the true failure class, stale-hallucination rate, re-exploration rate after drift

## Risks / reviewer objections (and rebuttals built into design)
- "Memory is just RAG" → show fingerprint + embedding + version-aware invalidation beats full-trace RAG at equal budget; ablate each.
- "Synthetic drift / planted failures aren't real" → derive failure classes and drift operators from documented real warehouse/ELT failures in the public literature and from pilot dry-run traces; validate a small set qualitatively.
- "Kim et al. / compute-fair already done" → they did no data tasks, no memory condition, no write-bearing/safety tasks.
- "Benchmark labels are wrong" → use corrected subsets, report label-audit method, semantic grading for ambiguous items.
- Cost of repeated runs → powered subsets + sequential stopping; report cost per reliable success as a finding, not just a budget line.

## Target venues (in order)
1. VLDB / PVLDB — strongest data fit for agent/data-systems work of this kind
2. CIDR — benchmark / system-proposal version if experiments need narrowing
3. NeurIPS Datasets & Benchmarks — if the drift + repeated-failure suite is released as the main artefact
Avoid: a pure BIRD audit angle (saturated, per gap analysis).

## PIVOT — adopted 2026-11-22 (Protocol Amendment A9; supersedes the framing above where they conflict)
Stage 1 of the pilot tested the memory hypothesis at full power and found no gain (A1 − A0 = −1.8 pts, 95% CI −5.7…+2.1; acquisition rate 0.565). Per Protocol §9, Paper #1 is reframed as a **compute-fair, corrected-label negative-result study**: title direction — "Failure Memory Does Not Help Data Agents (Yet): A Compute-Fair, Corrected-Label Evaluation on Repeated Text-to-SQL Failures". Contributions become: (1) the pre-registered null result with tight CI; (2) the strict no-gold memory-acquisition methodology and its measured 56.5% rate; (3) the compute-fairness accounting (memory cost +22% tokens at equal attempts); (4) corrected-label dual reporting incl. a newly verified gold error (bird-1006, Amendment M1); (5) Stage 2 bounds — generate-and-select at matched tokens (A3), stale memory under drift (A4), generic vs fingerprinted memory (A2). The sections below remain as the record of the original plan; drafting follows this pivot note.

## Immediate next steps
1. Literature review: agent memory (LongMemEval-V2, MemoryAgentBench), compute-fair MAS (Kim et al., Tran & Kiela), reliability (On Randomness, tau-bench pass^k), drift (StructHallu-Drift). Public literature only — the author's previous papers are reference only and are not used here.
2. Failure-class taxonomy: build agent trajectory failure classes from pilot dry-run traces and public benchmark error analyses.
3. Prototype harness: session-chained task construction from BIRD + drift operators; memory layer v1 (fingerprint + embedding).
4. Pilot: small powered run (1 model, SAS only, NoMemory vs Embedding vs Stale) to test H1/H3 before scaling.
