# Paper #1 Draft v1 — Reviewer-Style Critique

**Date:** 2026-11-23. **Manuscript:** "Failure Memory Does Not Help Data Agents: A Compute-Fair, Corrected-Label Evaluation" (Draft v1, 2026-11-22).
**Method:** two independent passes — (1) a context-blind review written from the manuscript alone, in the voice of a PVLDB Experiment, Analysis & Benchmark (EAB) reviewer; (2) a context-aware cross-check of every number in the draft against the pilot/replication analysis files and harness code. Issues are merged and severity-ranked below; each carries a concrete fix and its cost.

**Overall verdict: major revision.** The core of the paper — a pre-registered, dual-label memory null — survives hostile review. But one of its three headline conclusions (the drift effect) currently rests on an invalid comparison, two internal contradictions sit in the results the paper is proudest of, and the title claims more generality than two non-frontier models on one benchmark earn. All fatal/major items are fixable; most fixes are writing-only or cost under $3 of compute.

---

## Fatal

### F1. "Drift halves accuracy" compares two different task sets (§5.4, Abstract, §6)
The paper states A0 "falls to 0.308 (from 0.539 undrifted)" — but 0.539 is A0's score on the *repeated-failure split*, a different sample. The 60 drift tasks were never run undrifted under A0, so the headline drift effect confounds task difficulty with drift. The Abstract's "(53.9% → 30.8%)" and §6's "drift itself is the dominant effect" both rest on this.
**Fix:** run the paired control — A0 (and A1) on the original databases for exactly these 60 tasks, 4 runs. ~240–480 trajectories ≈ **$1.40–$2.80** at measured mini rates. Then report the drift effect as a paired contrast on identical tasks, or delete the claim everywhere. *This is the single highest-value fix in this list.*

## Major

### M1. The corrected-label drift column contradicts the stated drift scoring (§3.2/§7 vs §5.4)
§3.2 and §7 say drifted corrected gold "was not constructed," yet §5.4 reports a full corrected-label column for the drift contrasts. Cross-check against the harness: the numbers are real — scoring executed Jin et al.'s corrected SQL against each drifted database (`gold_sql_corrected` run on the drifted DB path). The method is defensible; the text contradicts it.
**Fix (writing only):** state exactly that computation in §3.2, and rewrite the §7 limitation to what is actually limited (the corrected SQL was not itself drift-adjusted by hand; it was executed on the drifted DB).

### M2. The replication's acquisition arithmetic does not check out (§5.5)
§5.5 reports "0.132 (27 verified episodes from 237 failures)" — but 27/237 = 0.114. Cross-check: Qwen had 237 failed Session-1 *trajectories*, of which 205 were agent-observed failures (execution error or verifier rejection); 27/205 = 0.132. The 32 remaining failures were silent — the agent never detected them, so they were never acquisition candidates. As written, the sentence contradicts the paper's own headline statistic (§3.4's definition).
**Fix (writing only):** "27 verified episodes from 205 agent-observed failures (acquisition rate 0.132); a further 32 Session-1 failures were silent and never entered acquisition." The silent-failure count is itself worth the sentence — it explains the thin store mechanistically.

### M3. The Abstract overclaims the A3 label-flip relative to §5.2
The Abstract presents the flip (+9.8 → exactly 0.0) as a demonstrated artefact ("manufacture an apparent method win"). §5.2 is more honest: the corrected subset is 49 tasks at ±8 points, and "the data as they stand cannot distinguish" artefact from underpowering. By the paper's own power analysis, n=49 cannot detect even the 12-point GO margin — so the paper's headline methodological claim rests on its least-powered comparison.
**Fix (writing only):** align the Abstract and §8 with §5.2's two-readings framing. The replication adds a supporting leg (A3 null on original labels under an enforced cap, §5.5) — cite it in the Abstract instead of asserting the artefact reading outright.

### M4. Title and claim scope exceed the model coverage
All pilot results use gpt-5-mini; the replication's model had a thin store (0.132) and a truncated A3 (89% cap fallback). The unqualified title — "Failure Memory Does Not Help Data Agents" — is not earned by this coverage, and §7's caveats do not rescue a title.
**Fix (choose one):** (a) narrow the title/claims to the tested regime (e.g., "…Does Not Help Text-to-SQL Agents at Matched Compute: A Pre-Registered Evaluation"); or (b) run one frontier snapshot (e.g., the pinned GPT-5) on the A0/A1 repeated-split contrast — 4 runs ≈ $45+, 1 run ≈ $12 at GPT-5 prices. (a) is free and honest; (b) is the strongest single addition if budget allows.

### M5. The A1-vs-A2 "isolation" is uninterpretable as reported (§3.3, §5.3)
A2 exists to isolate the fingerprint design, but the paper never reports how often A1's exact fingerprint match fired versus falling back to the same embedding retrieval A2 uses. If exact matches were rare, A1 ≈ A2 by construction and "the null spans both designs" is one null measured twice; the A1−A2 contrast (−2.8, CI −7.7…+1.6) is itself non-significant.
**Fix (free, offline):** the traces record retrieved episodes' fingerprints — a retrieval whose episodes all share one fingerprint is an exact-match hit; mixed fingerprints indicate embedding fallback. Compute the exact-match rate from the existing traces and report it; soften the isolation claim to match.

### M6. Reproducibility gaps (several, all fixable)
- **The retrieval embedding model is never named** (§3.3), and the run artefacts do not record which of the harness's two embedding modes (local Ollama model vs hashed TF vectors) actually served the pilot. Name the embedder; if the mode cannot be established from the record, say so and instrument the released harness.
- **Amendment A13 is not disclosed in the paper.** The replication's first A3 run was discarded pre-analysis and re-run under a fixed cap estimator (36% of trajectories had made zero attempts). The protocol record shows it; the manuscript — whose entire pitch is transparency — does not. Add two sentences to §5.5.
- **"Pre-registered" is currently unverifiable by a reader.** The protocol and Amendments A1–A13 are cited but not linked; §4 calls the amendments "public" — they are not yet. Release the protocol record with the artefacts (repo link/DOI in the paper) or soften the wording.
- Prompts (including the self-verifier's) are not given; sampling is specified only as "temperature at the provider default" (§4). Publish the prompts in an appendix; state the actual sampling settings.
- Store-update semantics during Session 2 and the drift arms' acquisition counts (A4's pre-drift store size) are never reported. A stale-memory null with an unstated store size invites the "empty store" attack the paper elsewhere forecloses. Report both (the counts exist in the artefacts).

### M7. Missing related work a reviewer will name immediately
§2's memory section cites MemGPT and Zhang et al. and stops. Absent: **Reflexion** and **ExpeL** — the canonical failure-feedback / experience-memory methods this paper is implicitly a verdict on — and Generative Agents. On the data side, **MAC-SQL** and **DIN-SQL** are absent, which undermines the unqualified "only verified data-specific multi-agent win" claim for CHASE-SQL.
**Fix (writing only):** engage Reflexion/ExpeL-style reflection memory explicitly and justify why A1/A2 are fair representatives of the memory family; survey multi-agent Text-to-SQL beyond CHASE-SQL or qualify the claim.

### M8. The base agent is weak and the self-verifier is an unexamined confound (§3.3)
A0 reaches 0.539 on this split while published systems reach far higher on BIRD Dev (the paper itself cites CHASE-SQL at 73.01%). The split is a subset, so the numbers are not directly comparable — but the burden is on the paper to show the null is not specific to a single-pass agent whose built-in verifier may already absorb exactly the corrections memory would supply. The verifier's contribution is never ablated.
**Fix (choose one):** (a) run a verifier-off ablation for A0/A1 on the repeated split (needs a harness flag; A0-only ≈ 400 trajectories ≈ $2.30, both arms ≈ $4.60); or (b) argue transfer with evidence. (a) is cheap and closes the attack.

### M9. The frozen claim's own metric — pass^k — never appears (context-aware addition)
The frozen protocol claim is stated in terms of pass^k; the pilot gate used run-mean pass@1 (a dated, legitimate amendment), but the paper reports no pass^k number anywhere, while repeatedly invoking the frozen claim. A reviewer who reads the released protocol will ask.
**Fix (free, offline):** with 4 runs per task, pass^4 (and pass^2) per arm is computable from the existing traces today. Add a small pass^k table for the primary contrasts. If memory fails on pass@1 *and* pass^4, the null is stronger, not weaker.

## Minor

- **m1.** §5.1: "no stratum is positive (near −3.8, mid **+0.8**, far −2.3)" — the mid tertile *is* positive as a point estimate. Say "no stratum is significantly positive."
- **m2.** §5.3: A2's corrected-label contrasts are given without CIs, unlike every other contrast.
- **m3.** §5.5: the with/without-retrieval comparison (0.560 vs 0.588) is a post-hoc, self-selected subgroup with no intervals. Add CIs and a selection caveat, or cut it.
- **m4.** Distance interpretation is muddled: pair distances span only 0.086–0.307 (all pairs structurally similar), yet §6 floats "similarity too low for transfer" as an explanation, while the near-duplicate regime where memory is most plausible is excluded by design (§7 half-concedes this). Pick one coherent reading.
- **m5.** Reference list gaps: DABstep and "DataAgentBench" are cited in §2 but absent from References; the Jin et al. entry still has a bracketed placeholder title. Also §2 quotes Jin's error rate for BIRD *Mini-Dev* (52.8%) while §3.1 uses corrections for 498 BIRD *Dev* tasks — one sentence clarifying the relationship between the audited set and the correction set is needed.
- **m6.** Figure ordering: Figure 2 is introduced in §3.2, Figure 1 in §5. Renumber.
- **m7.** The manuscript keeps calling the study a "pilot" ("pilot gate," "total pilot cost," pass^k "reserved for the full study") while asking to be judged as a complete EAB paper. Scrub the pilot framing or defend the scale as the design.
- **m8.** Contribution 5 (the single q1006 gold error) is an anecdote for §3.1, not a contribution. Demote it.
- **m9.** Multiplicity: ~15 contrasts in Figure 1; the one significant positive is the narrative centrepiece. One sentence on the pre-registered contrast hierarchy pre-empts the question.
- **m10.** Draft header and §4 still say "Amendments A1–A11"; A12–A13 exist and A13 materially changed the replication. Update.

## Recommended fix order

1. **F1 control run** (~$1.40–$2.80) — without it, a headline conclusion is unsupported.
2. **Free offline analyses:** pass^k table (M9), exact-match rate (M5), drift-arm acquisition counts (M6).
3. **Writing fixes:** M1, M2, M3, m1–m10 — a single editing pass.
4. **Decisions for the author:** title scope (M4a vs the ~$12–45 frontier run, M4b); verifier ablation (M8a, ~$2.30–$4.60) vs argument (M8b); artefact/protocol release mechanics (M6).
5. **Related-work pass** (M7).

Total cost of every compute fix combined: under $10 except the optional frontier run. None of this threatens the PVLDB Feb 1, 2027 cycle (abstract Jan 25).

## What survives hostile review (for the record)

- The pre-registration discipline, including honouring the §9 pivot instead of shopping for a positive framing.
- The acquisition policy (§3.4) as a genuine methodological contribution — it forecloses the standard "empty store" rebuttal for the primary model.
- Dual-label reporting throughout; the A3 label-flip is only visible because both views were pre-committed.
- The paper's honesty about its own harness failures (A3 overshoot; replication cap truncation) — which is exactly why the A13 omission (M6) should be fixed rather than left for a reviewer to find.
