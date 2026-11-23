# Failure Memory Does Not Help Data Agents: A Compute-Fair, Corrected-Label Evaluation

**Draft v1 — 2026-11-22.** Author: Mogana Kumaran Sivaraman. Status: working draft under the pivot framing adopted in Protocol Amendment A9. All experiments were pre-registered in a frozen protocol (Pilot Protocol v2, with dated Amendments A1–A11) before any pilot run; numbers below are as-run. The open-model replication (Amendment A12, OpenRouter Qwen2.5-Coder-32B-Instruct) is complete and incorporated in §5.5 and §7.

## Abstract

Episodic memory of past failures is widely assumed to help LLM agents, and memory mechanisms are a standard component of agent architectures. We test that assumption for data agents — Text-to-SQL agents working over relational databases — under the evaluation conditions the claim actually requires: a pre-registered protocol, paired comparisons at matched token budgets, and scoring against both original benchmark labels and independently audited corrected labels. On a purpose-built split of 100 recurring-failure task pairs from BIRD, neither of two memory designs improves pass@1 over a no-memory baseline: failure-fingerprinted episodic memory scores −1.8 points (95% CI −5.7…+2.1) and generic episodic memory −4.6 points (−9.5…0.0), at equal or greater token cost. Under a purpose-built drift split (60 tasks, 55% silent semantic drift), memory frozen before the drift — the hypothesised stale-memory hazard — is exactly neutral on silent drift (0.0 points), while drift itself halves baseline accuracy (53.9% → 30.8%). The one positive result in the study is diagnostic: generate-and-select beats no-memory by +9.8 points on original labels but by exactly 0.0 on corrected labels, while running 17% over its nominal token budget — a worked example of how label noise and loose compute accounting can jointly manufacture an apparent method win. We release the evaluation design, the drift split construction, and a strict no-gold memory-acquisition methodology (56.5% acquisition rate) as reusable artefacts. Our conclusion is narrow and, we argue, useful: on this substrate, at matched compute, failure memory is not where the gains are — and evaluations that do not control labels and tokens cannot be trusted to say where they are.

## 1. Introduction

Agent memory is having its framework moment. Long-term episodic stores, reflection buffers, and failure libraries are built into most serious agent stacks, and the intuition behind them is appealing: an agent that has seen a failure before should not repeat it. For data agents — systems that answer questions by generating and executing SQL over enterprise schemas — the intuition seems especially strong, because failures recur: the same schema quirks, value formats, and join traps appear across questions on the same database.

Yet the evidence base for failure memory in data work is thin in a specific way. The strongest published multi-agent result on Text-to-SQL, CHASE-SQL, is a generate-and-select method whose gains come at explicitly higher test-time compute; it says nothing about memory. General-domain agent research offers a cautionary signal — consolidated LLM memory has been shown to degrade performance below a no-memory baseline in some settings — but no published study, to our knowledge, tests failure memory for data agents at matched compute against audited labels. Meanwhile the labels themselves are suspect: a recent audit of BIRD and Spider 2.0 found annotation error rates above 50% on audited subsets, with leaderboard rank shifts of up to nine places after correction. An evaluation that ignores both problems — unmatched compute and noisy labels — can manufacture wins that do not exist.

This paper is the report of a pre-registered attempt to find out whether failure memory helps, designed so that a negative answer would be as informative as a positive one. Our protocol, hypotheses, exclusion rules, and go/no-go margins were frozen before any pilot run and amended only through dated, public amendments. The answer, across two memory designs and a drift condition, is no — and the study's one positive arm turns out to be an object lesson in why the controls mattered.

**Contributions.**

1. **A pre-registered, compute-fair test of failure memory for data agents.** Two memory designs (failure-fingerprinted and generic episodic), a no-memory baseline, and a generate-and-select arm, compared pairwise on identical task pairs with paired bootstrap intervals, at measured token budgets.
2. **Dual-label evaluation as a first-class control.** Every contrast is reported on original BIRD labels and on the corrected labels of Jin et al.'s audit (498 tasks). The study's headline methodological finding — a +9.8-point effect that is exactly 0.0 under corrected labels — is only visible because both views were pre-committed.
3. **A drift split for data agents, and a stale-memory measurement.** Sixty execution-verified drifted database variants (55% silent semantic drift) let us measure negative transfer directly: stale memory is neutral, including exactly 0.0 on silent drift.
4. **A strict memory-acquisition methodology.** Memory episodes are acquired only from agent-discovered, execution-verified remediations — gold SQL never touches the store — with the acquisition rate (56.5%) reported as a headline statistic, so a null result cannot be dismissed as an empty store.
5. **A new verified benchmark error.** The study's own runs surfaced a BIRD gold error (question 1006) outside the existing audit's coverage, verified by direct execution and logged through a dated manifest amendment.

## 2. Related work

**Benchmarks and the label problem.** BIRD (Li et al., NeurIPS 2023 Datasets & Benchmarks) is the standard large-scale Text-to-SQL benchmark: 12,751 question–SQL pairs over 95 databases, where GPT-4 reached 54.89% execution accuracy against 92.96% human performance. Spider 2.0 (Lei et al., ICLR 2025) moves to enterprise workflows, where a leading agent scored 17.0% (arXiv v1) to 21.3% (ICLR 2025 version) — a reminder that version matters when quoting results. Jin et al. (2026) audited both benchmarks and reported annotation error rates of 52.8% on BIRD Mini-Dev and 62.8% on Spider 2.0-Snow, with corrected-label rank shifts of −9 to +9 places; we use their corrected labels for 498 BIRD Dev tasks and treat label status as an explicit variable (§3.1).

**Data agents.** CHASE-SQL (Pourreza et al., ICLR 2025) is, to our knowledge, the only verified data-specific multi-agent win on BIRD (73.01% dev execution accuracy), achieved by generating diverse candidates and selecting among them at additional test-time compute — the design our A3 arm mirrors under a budget cap. SQLAgent (Jiang et al., Findings of ACL 2026) takes a different route, learning to explore the database before generating SQL; exploration targets exactly the schema-grounding failures that dominate our error taxonomy (§3.4). Broader data-analysis benchmarks (DSBench; DABstep; the DataAgentBench line of work) report best-agent accuracies in the 14–57% range depending on version and subset, underscoring how unsettled data-agent evaluation remains.

**Agent memory.** MemGPT (Packer et al., 2023) popularised tiered, OS-style memory for LLM agents in general domains. Evidence that memory helps is mixed even there: Zhang et al. (2026) show that *consolidated* LLM memory degrades with repeated updating and can fall below a no-memory baseline, while an episodic-only control retaining raw trajectories remains competitive — in general domains. Whether raw episodic memory itself helps is untested for data agents; that is the cell this study fills. No published work we could find isolates failure memory for data agents under matched compute; the gap analysis underlying this study (two scoped passes over arXiv and major venues, November 2026) found the cell empty. Our A1/A2 arms are, to our knowledge, the first controlled test.

**Reliability measurement.** τ-bench (Yao et al., 2024) introduced pass^k — the probability that all k independent runs succeed — as a reliability metric for agents, showing steep degradation (GPT-4o below 25% at pass^8 in retail). We adopt run-mean pass@1 for the pilot gate (with pass^k reserved for the full study) and report intervals throughout, following the same reliability-first stance. Failure taxonomies for multi-agent systems (e.g., MAST, Cemri et al., NeurIPS 2025 Datasets & Benchmarks) are general-domain; our taxonomy (§3.4) is derived blind from data-agent traces.

## 3. Method

### 3.1 Substrate and label policy

The substrate is classic BIRD Dev (dev_20240627, pinned): 1,534 tasks over 11 databases. Because label quality is a variable in this study, every task carries a label status: **L1** corrected label available from Jin et al.'s audit (498 tasks; 220 with SQL differing from the original gold); **L2** original label only (1,035 tasks); **L3** disputed, no correction — excluded from all primary metrics and logged (1 task: question 1006, a gold error discovered by this study's dry runs, in which the gold's string-parsing ordering returns a 2:00.000 lap as "fastest" while the milliseconds column's minimum is 1:07.411; verified by direct execution and corroborated by two independent models); **L4** ambiguous (0 tasks). All results are reported dually, on original labels for the full primary set and on corrected labels for the L1 subset. The manifest, splits, and all file hashes were frozen before the pilot; the single post-freeze change (the L3 relabelling of q1006) was made by dated amendment with the original hash preserved.

### 3.2 Splits

**Repeated-failure split.** One hundred Session-1/Session-2 task pairs, clustered by database and identical table-set signature, balanced 9–10 pairs per database. Pair similarity is a structural distance, 1 − [0.5·Jaccard(table sets) + 0.3·Jaccard(SQL skeleton tokens) + 0.2·Jaccard(question tokens)]; observed distances run 0.086–0.307 (median 0.209). Pairs below distance 0.10 (3 pairs) are flagged recall-control and excluded from the primary metric, leaving 97 primary Session-2 tasks; results are also stratified by distance tertile. Session 1 is where memory is acquired; Session 2 is where it is tested. Memory for a task comes only from strictly earlier sessions; a violation invalidates the run (none occurred).

**Drift split.** Sixty tasks, all L1, each with a drifted copy of its database built by one of seven operators (column rename 23, value recode 10, category merge 9, unit change 9, type change 4, temporal grain 4, null semantics 1). Thirty-three variants (55%) are *silent* — the meaning changes under unchanged column names — and 27 are overt renames, which serve as the control condition. Every variant was verified by execution before freezing. Figure 2 shows the composition by operator. Scoring uses the drift-adjusted original gold executed on the drifted database; drifted corrected gold was not constructed, a limitation we record.

**Figure 2.** Drift-split composition: 60 execution-verified variants by operator, split into silent (33) and overt (27). *(File: "Paper 1 Figure 2 - Drift Split.png".)*

### 3.3 Arms

All arms share one base agent: schema + evidence + question → SQL generation, read-only execution with a 10-second limit, retry on execution error with the error fed back (maximum 3 attempts), and a self-verification step in which the agent's own verifier accepts or revises the result. Prompts, tools, sandbox, and step limits are identical across arms; only the memory/selection mechanism varies.

- **A0 — no memory.** The base agent, stateless across sessions.
- **A1 — failure-fingerprinted episodic memory.** Episodes are raw traces keyed by a deterministic fingerprint over (failure class, error signature, schema elements, schema version). Retrieval is exact fingerprint match, else top-k (k ≤ 3) embedding retrieval over raw traces. No LLM consolidation is applied to stored episodes.
- **A2 — generic episodic memory.** The same store and retrieval budget without fingerprint keying (similarity retrieval only), isolating the contribution of the fingerprint design.
- **A3 — generate-and-select.** Three candidates are generated and executed; a selector call picks among those that executed, in the style of CHASE-SQL. Its token budget is capped at A1's measured average tokens per task (3,506) — see §5.2 for how that cap behaved as implemented, and §7 for the fix applied in the replication.
- **A4 — stale memory (drift split only).** A1's mechanism with the store frozen at its pre-drift state and never updated during the drift phase.

### 3.4 Memory acquisition policy

The acquisition rule is the methodological core of the study, because a memory evaluation is only meaningful if the memory could plausibly have been built in deployment. An episode enters the store **only** when the agent itself recovers from a failure during a task — an execution error or a verifier rejection — and the remediation is verified by execution and accepted by the agent's own verifier. Gold SQL, gold labels, and human fixes never create, complete, or repair an episode. Failures the agent cannot resolve go to a separate unresolved log. The **acquisition rate** — episodes acquired divided by failures observed — is reported as a headline statistic alongside accuracy, so that a null accuracy result can be checked against the possibility that the store was simply empty. The rule is enforced in harness code, not by convention.

### 3.5 Failure taxonomy

Failure classes were derived by blind open coding of dry-run traces (two annotators, 100% per-task primary-defect agreement on the local-model run; seed classes admitted only at reconciliation), then frozen before the pilot: F1 wrong schema link, F2 stale schema assumption, F3 wrong join/fan-out, F4 aggregation/grouping incl. temporal-snapshot semantics, F5 value-format mishandling, F6 unrecovered execution error, F7 correct content in the wrong answer shape, F8 unsafe/destructive (log-only), F9 no-change repair loop (a new class from open coding: the agent resubmits byte-identical SQL after error feedback — observed in 8 of 8 retries of a 3B local model, absent at frontier scale). Gold-label errors are not failure classes; they are handled by the label policy in §3.1.

## 4. Experimental setup

**Pre-registration and power.** The protocol — claim, arms, splits, exclusion rules, analysis plan — was frozen before any pilot run. A simulation-based power analysis from dry-run base rates (per-task latent success probabilities, 4 runs per task per arm, paired test on run-mean pass@1) found power nearly invariant to base rate in [0.30, 0.55], with a minimum detectable effect at 80% power of 10–12 points at n = 100. The pre-committed GO margin for the memory hypothesis was therefore +12 points of paired pass@1 (A1 − A0) with the 95% CI excluding zero; the protocol's §9 pre-committed that a null would pivot this paper to the compute-fair negative-result framing rather than quietly change the claim. Split sizes follow from the same analysis: 100 repeated-failure pairs, 60 drift tasks.

**Model and harness.** All pilot runs use a single pinned snapshot, gpt-5-mini-2025-08-07, via API, temperature at the provider default for this model family. Four runs per task per arm. Scoring is execution accuracy: the predicted result set is compared with the gold result set as a multiset, order-sensitive when the gold query has ORDER BY. Total pilot cost was $14.91 across 3,760 trajectories (2,800 repeated-split, 960 drift), inside the pre-set budget caps; the full amendment and budget trail (A1–A11) is part of the study record.

**Analysis.** Primary contrasts are paired per-task differences in run-mean pass@1 with task-level bootstrap 95% CIs (10,000 resamples, fixed seed). Token fairness is reported dually: attempt-matched (same step limits) and token-measured (actual mean tokens per trajectory per arm), with token-matching treated openly as an approximation.

## 5. Results

Figure 1 summarises every primary contrast in the study, by split and label view, including the open-model replication (§5.5); the subsections below give the numbers in full.

**Figure 1.** Paired pass@1 differences (percentage points) with bootstrap 95% CIs, for all arm contrasts on the repeated-failure split (original and corrected labels), the drift split, and the Qwen2.5-Coder-32B replication (original and corrected labels). The dashed line marks the pre-committed +12-point GO margin. *(File: "Paper 1 Figure 1 - Paired Differences.png".)*

### 5.1 Failure memory does not help (primary contrast)

On the 97 primary Session-2 tasks, the no-memory baseline A0 scores 0.539 pass@1. Failure-fingerprinted memory (A1) scores 0.521: a paired difference of **−1.8 points (95% CI −5.7…+2.1)**. The pre-committed GO criterion (+12 points, CI excluding zero) fails by a wide margin, and the interval rules out any true gain larger than ≈2 points in this regime — an informative null, not merely an underpowered one. On the 49 primary tasks with corrected labels the difference is −1.0 points (−6.1…+4.1). Stratified by pair-similarity tertile, no stratum is positive (near −3.8, mid +0.8, far −2.3): the null is not hiding among the most similar task pairs.

The store was not empty. In Session 1, A1 observed 177 failures and acquired **100 verified episodes (acquisition rate 0.565)** under the no-gold policy of §3.4; 77 failures were unresolved and logged. A1's Session-2 trajectories used a mean of 3,506 tokens against A0's 2,862 (+22%, the injected memory context) at essentially identical attempt counts (1.69 vs 1.68). Memory cost more compute and returned nothing.

### 5.2 Generate-and-select: a label-dependent win, over budget

A3 is the study's only positive arm on original labels: 0.637 pass@1, beating A0 by **+9.8 points (+4.1…+16.0)** and A1 by **+11.6 (+5.7…+18.0)**. On the 49 corrected-label tasks, however, both differences collapse — **0.0 points vs A0 (−8.2…+8.2)** and +1.0 vs A1 (−7.7…+10.2). Two readings remain live: the original-label win is a label-noise artefact (generate-and-select is better at matching buggy golds), or the 49-task subset is underpowered (±8 points) and cannot exclude a real effect. The data as they stand cannot distinguish these, and we report both views rather than claim the win.

A3 also exposes a fairness failure in our own harness: its mean cost was **4,102 tokens per trajectory, 17% over the 3,506 cap**, because the cap check ran before each candidate-generation call while the selector call sat outside it. A nominally token-matched arm was therefore not token-matched. We report A3 as-run with this caveat; the replication (§7) enforces the cap over the whole trajectory, selector included.

### 5.3 Generic memory is also null

A2 scores 0.492 pass@1: **−4.6 points vs A0 (−9.5…0.0)** and −2.8 vs A1 (−7.7…+1.6) on original labels; −4.6 and −3.6 respectively on corrected labels. Unlike A3, A2 was genuinely token-matched (3,463 mean tokens vs A1's 3,506; 1.70 attempts). The memory null therefore spans both designs at equal compute: it is not an artefact of fingerprint keying.

### 5.4 Drift: stale memory does not hurt; live memory does not help

On the 60 drift tasks, A0 falls to 0.308 pass@1 (from 0.539 undrifted) — drift itself is the dominant effect, with silent drift (0.326) only slightly easier than overt (0.287). Against that baseline:

| Contrast | Original labels | Corrected labels |
|---|---|---|
| A1 − A0 (live memory) | −4.6 (−9.6…+0.4) | −0.8 (−5.0…+3.3) |
| A4 − A0 (stale memory) | −1.7 (−6.3…+2.9) | +1.3 (−2.9…+5.4) |
| A4 − A1 (stale vs live) | +2.9 (−2.1…+7.9) | +2.1 (−1.7…+6.7) |

Stale memory (A4) is within ±2 points of no-memory overall and **exactly 0.0 on the 33 silent-drift tasks** (−5.3…+4.6) — the condition where pre-drift episodes were hypothesised to mislead most. Live memory (A1) is likewise null under drift on corrected labels. With 60 tasks the intervals are ±5–12 points, so a small stale-memory harm cannot be excluded; the hypothesised *direction* of harm, however, is not observed anywhere in the split.

### 5.5 Replication on an open-weight model: the memory null is not a single-model artefact

We replicated the repeated-failure split (arms A0/A1/A3; A2 and drift not replicated) on **Qwen2.5-Coder-32B-Instruct via OpenRouter** — same substrate, splits, prompts, harness, scoring, and analysis, at a total cost of $2.37. The replication's A3 cap was A1's measured mean in the replication itself (1,614 tokens) and, unlike the mini run, was enforced over the whole trajectory including the selector call.

The headline result is a second, independent null for failure memory: A0 and A1 both score **0.583 pass@1**, a paired difference of **exactly 0.0 points (95% CI −4.6…+4.6)** on the 97 primary tasks. A3 is also null in the replication (+0.8 vs A0, −6.4…+8.3). On corrected labels (49 tasks), A1−A0 is +3.1 points (0.0…+8.2) — the interval touches zero and is not a significant effect — and A3−A0 is +4.1 (−4.1…+12.2).

Two caveats qualify the replication, and we report both. First, Qwen's memory store was thin: its Session-1 acquisition rate was **0.132** (27 verified episodes from 237 failures), against mini's 0.565, because Qwen rarely produced an execution-verified remediation — in 22% of multi-attempt Session-1 trajectories it resubmitted byte-identical SQL after error feedback (the F9 no-change repair loop). Mini's null is "memory, well stocked, did not help"; Qwen's null is partly "memory, thinly stocked, did not help." Retrieval still occurred (184/400 Session-2 trajectories retrieved at least one episode) with no benefit signal (0.560 with retrieval vs 0.588 without). Second, the enforced whole-trajectory cap *truncated* A3 rather than matching it: 89% of A3 trajectories hit the cap fallback, only 72/400 ran the full 3-candidate procedure, and 22% relied on a single capped "mercy" call. Mini's A3 ran the full procedure 17% over budget; the replication's ran a truncated procedure within budget. The replication's A3 null therefore cannot be read as a clean refutation of mini's original-label A3 win — but the corrected-label A3 contrast is null in both models (0.0 mini, +4.1 replication), so the label-sensitivity finding is consistent across models.

Taken together: the failure-memory null now spans two models, two token regimes (A1 at 3,506 vs 1,614 mean tokens), and two very different acquisition rates. Model identity moves the baseline level (Qwen's no-memory 0.583 vs mini's 0.539) but does not move the memory contrast. The single-model limitation in §7 is retired; the single-benchmark limitation remains.

## 6. Discussion

Three conclusions survive both label views. First, **episodic failure memory, in either of two designs, does not improve data-agent accuracy at matched compute on recurring failures** — despite a healthy acquisition rate and a store built under deployment-realistic rules. Whatever limits this agent on Session-2 tasks, it is not a lack of remembered failures. Candidate mechanisms — retrieved episodes acting as distractors (+22% tokens for no gain), Session-2 failures dominated by classes memory cannot fix (e.g., F7 answer-shape errors), pair similarity too low for transfer (median distance 0.209) — are hypotheses our data suggests but does not adjudicate; we list them as such.

Second, **evaluation hygiene is not a formality; it changes the answer.** The same study, scored two ways, produces a +9.8-point method win and a 0.0 null. Combined with a 17% budget overshoot inside a nominally capped arm, the A3 result is a concrete demonstration that label audits and whole-trajectory token accounting are load-bearing for agent evaluation claims — including, we note, claims made for memory systems evaluated without them.

Third, **drift is an agent-robustness problem, not a memory-poisoning problem.** Drift halved baseline accuracy while stale memory was neutral even under silent semantic change. Effort spent on stale-memory invalidation for this class of agent may be misallocated relative to effort spent on drift detection and re-exploration — a prioritisation claim our data supports directionally, at this scale.

## 7. Limitations and threats to validity

- **Single benchmark, two models.** All results use BIRD Dev. The single-model limitation is retired: the failure-memory null now holds on two models — gpt-5-mini (A1−A0 −1.8 points) and Qwen2.5-Coder-32B-Instruct (exactly 0.0) — at different token levels and acquisition rates (§5.5). A2 and drift were not replicated on the open model, and no second benchmark was run, so generality beyond BIRD is still unestablished.
- **Corrected-label coverage.** Corrected labels exist for 498 of 1,534 tasks (about one-third audited); corrected-label contrasts run on 49 primary tasks with ±8-point intervals. The A3 label-flip is therefore reported as "does not survive corrected labels," not as proof the original-label effect is spurious.
- **Drift power.** Sixty drift tasks yield ±5–12-point intervals; small stale-memory effects in either direction are not excluded.
- **Token-matching is approximate.** Even with per-call caps, input-token growth from injected context makes exact equality across arms impossible; we report measured tokens for every arm so readers can re-normalise.
- **Drift scoring.** Drifted tasks are scored against drift-adjusted original gold; drifted corrected gold was not constructed.
- **Regime.** Pair distances (median 0.209) represent moderate recurrence. Memory might behave differently under near-duplicate recurrence (our 3 recall-control pairs are excluded from the primary metric by design) or in multi-session deployments longer than two sessions.

## 8. Conclusion

Under a frozen protocol, at matched compute, against both original and audited labels, failure memory did not help a data agent — in two designs, on recurring failures, and under drift, where stale memory also did not hurt. The study's one apparent win dissolved under corrected labels and an honest token accounting. We read this less as a verdict on memory in principle than as a measurement of where the burden of proof now sits: claims that agent memory helps — and agent evaluation results generally — should be required to survive corrected labels and whole-trajectory compute accounting before they are believed. The splits, acquisition methodology, and protocol record released with this paper are intended to make that standard cheap to meet.

## References (verified sources only)

- Li, J. et al. (2023). Can LLM Already Serve as a Database Interface? A Big Bench for Large-Scale Database Grounded Text-to-SQLs (BIRD). NeurIPS 2023 Datasets & Benchmarks. arXiv:2305.03111.
- Lei, F. et al. (2025). Spider 2.0: Evaluating Language Models on Real-World Enterprise Text-to-SQL Workflows. ICLR 2025. arXiv:2411.07763.
- Pourreza, M. et al. (2025). CHASE-SQL: Multi-Path Reasoning and Preference Optimized Candidate Selection in Text-to-SQL. ICLR 2025. arXiv:2410.01943.
- Jiang, Z. et al. (2026). SQLAgent: Learning to Explore Before Generating as a Data Engineer. Findings of ACL 2026, pp. 39311–39331. arXiv:2602.01952.
- Jin, Y. et al. (2026). [Audit of Text-to-SQL benchmark annotations.] arXiv:2601.08778 (v3). Corrected-label repository (Arcwise-Plat), CC-BY-SA-4.0.
- Packer, C. et al. (2023). MemGPT: Towards LLMs as Operating Systems. arXiv:2310.08560.
- Yao, S. et al. (2024). τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains. arXiv:2406.12045.
- Cemri, M. et al. (2025). Why Do Multi-Agent LLM Systems Fail? (MAST). NeurIPS 2025 Datasets & Benchmarks. arXiv:2503.13657.
- Zhang, D., Lin, Y., Wu, Z., Sun, Y., Li, B., Li, D., & Peng, H. (2026). Useful Memories Become Faulty When Continuously Updated by LLMs. arXiv:2605.12978 (v2).
- Jing, X. et al. (2025). DSBench: How Far Are Data Science Agents from Becoming Data Science Experts? ICLR 2025.

---

### Draft notes (not part of the paper)
- Venue shortlist (checked 2026-11-23): VLDB first (PVLDB Vol. 20 rolling monthly cycles; next abstract Nov 25, 2026 for the Dec 1 paper deadline, then Jan 1 / Feb 1 / Mar 1, 2027 final; Experiment, Analysis & Benchmark papers up to 12 pp excl. references); NeurIPS Datasets & Benchmarks 2027 (call not yet out; 2026 cycle used May 4 abstract / May 6 paper, 2027 projected ~May 2027 — confirm on the official call); CIDR 2027 is CLOSED (submission deadline was Aug 4, 2026; conference Jan 24–27, 2027) — next CIDR cycle is 2028. Formatting to the venue template happens after the venue call.
- Open items: pass^k analysis if the full study is commissioned; optional further figure (acquisition flow). Done: Zhang et al. citation verified against the arXiv record (2026-11-22); Figure 1 (paired-difference forest plot) built from the analysis JSONs and extended 2026-11-23 with the replication contrasts (§5.5); Figure 2 (drift-split composition, counts read directly from drift_spec.csv) built 2026-11-23; open-model replication complete and incorporated (§5.5, §7).
- Companion artefacts: Pilot Protocol v2 + Amendments A1–A13; Stage 1 and Stage 2 pilot reports; Replication Report (OpenRouter Qwen2.5-Coder-32B); Substrate Specification (manifest freeze record incl. Amendment M1).
