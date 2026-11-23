# Review of Findings and Analysis So Far — Paper #1

Date: 2026-11-22. Scope: Step 1 briefing v2, Step 2 gap analysis v2, Pilot Protocol v2 (frozen), Substrate Specification v2 + dataset assessment, manifest v1, dry run (Arm A0, Qwen2.5-Coder 3B local), blind open coding + taxonomy reconciliation. Reviewed against the standing verification standard.

## Overall verdict

The chain of work is sound in structure and unusually honest in its labelling: every headline claim is either primary-verified or explicitly marked secondary/unverified, and the verification process demonstrably caught real errors (SQLAgent, Spider 2.0 and DAB version labels, two publication-status corrections). The Paper #1 decision — failure memory for data agents, compute-fair, under drift — remains supported **as a gap claim**. Nothing so far shows the method works, and nothing was claimed to.

But there are four substantive weaknesses and three document inconsistencies that should be fixed before the pilot runs. Ranked below.

## What holds up

1. **Sequence and provenance.** Step 2 was genuinely derived from the verified Step 1, per the author's instruction; gap claims G1–G7 are each traced to a Step 1 finding, and "open" is honestly scoped ("no [D] source in two scoped passes", not a systematic review).
2. **The protocol's acquisition policy.** Memory episodes only from agent-discovered, execution-verified remediations; gold never touches memory; acquisition rate as a headline statistic. This is the strongest single design decision in the project — it pre-empts the most damaging reviewer attack (oracle-built memory).
3. **Substrate work is real, not asserted.** BIRD Dev processed locally; 1,523/1,534 gold SQL execute cleanly (11 flagged slow, 0 errors); Jin corrected labels joined from the files: 498 audited tasks, 220 with corrected SQL. Manifest v1 hashed.
4. **Blind labelling was actually done blind.** Open coding was written and saved before the seed taxonomy was consulted, and the reconciliation logs every change (F5 broadening, new F9) instead of silently absorbing the open codes.
5. **The pivot rule protects the project.** If memory shows no gain, Paper #1 becomes a compute-fair negative-result paper (G2+G5). The project does not depend on a positive result to produce a paper.

## Substantive weaknesses (fix before the pilot)

### W1. The taxonomy is seeded from a weak model's failures — the biggest scientific risk
The dry run used Qwen2.5-Coder 3B on CPU. Its dominant behaviours — inventing column names with the full schema in the prompt, and resubmitting byte-identical SQL after errors (8/8 retries) — are plausibly small-model artefacts. Frontier models rarely loop byte-identically. The proposed new class F9, and the F1-heavy shape of the reconciled taxonomy, may describe *this model*, not data agents generally. The protocol says the taxonomy is "seeded from pilot dry runs, then frozen" — freezing it on 3B evidence alone would bake the artefact into the split construction (Section 3a clusters tasks by failure class).
**Fix:** treat the current taxonomy as provisional v0. Run one frontier-model dry run on the same 11 tasks (cheap: 11 tasks) before freezing; classes that survive both models freeze, model-specific classes are marked as such.

### W2. The power analysis would be built on the wrong base rate
Protocol Section 8 requires the GO margin to be set by simulation from "ONLY the A0 dry-run base rates". The only A0 base rate in hand is the 3B model's 45% (5/11, n=11 — a very wide interval). Section 6 of the same protocol plans "one frontier API model + one open model" for the pilot. A frontier model's A0 base rate on the repeated-failure split will differ materially, and a GO margin calibrated to 45% could be meaningless for it.
**Fix:** decide the pilot model(s) FIRST, get each model's A0 base rate (the frontier dry run in W1 doubles as this), then run the power analysis per model. This also resolves the open model question the dry run deferred.

### W3. Within-run repair failure ≠ cross-session memory benefit — do not conflate
The dry run's most striking finding (F9, the no-change repair loop) is a *within-run* phenomenon: one session, error feedback, no change. Paper #1's claim is about *cross-session* memory. A memory system could plausibly break an F9 loop on recurrence, but the dry run provides no evidence about that, and the paper must not cite F9 frequencies as motivation for expected memory gains. F9 motivates the *fingerprint* idea (identical error signatures recur), nothing more.

### W4. Corrected-label coverage is thinner than the substrate story suggests
Of 1,534 BIRD Dev tasks, 498 (~32%) are Jin-audited (220 corrected, 278 confirmed); 1,036 are original-label-only (L2). Dual reporting is pre-committed, which mitigates this, but any "corrected substrate" phrasing in the paper must say "one-third audited". Two loose ends remain open from the substrate work: Jin's separate 100-example corrected set (Google Drive, not obtained) and the unresolved discrepancy between Jin's 62.8% Spider 2.0-Snow error figure and the 66.1% figure on a UIUC CIDR listing (Spider is out of the primary pool now, so this matters only if Spider is ever reintroduced).

## Document inconsistencies (fix by dated amendment, claim untouched)

### D1. Protocol Section 2 contradicts the later substrate decision
The frozen protocol still names "BIRD Dev subset + Spider 2.0-Snow subset" as the substrate. The dataset assessment then found Spider 2.0-Snow NO-GO (cloud-only, partial gold, no drift possible, no Jin labels) and the Substrate Specification v2 amendment made the pool BIRD-only. The protocol was never amended to match. Anyone reading the protocol alone gets the wrong substrate. **Fix:** dated amendment to Sections 2 (and the substrate line wherever it recurs), pointing to Substrate Spec v2.

### D2. Step 2 report recommends what the author later forbade
Step 2 §4 step 4 recommends implementing the fingerprint "from the author's published representation (CJAST 2026 / JISEM 2022)" and the embedding from IJISAE 2023. The author's standing instruction of the same day — previous papers are reference only, NOT method, taxonomy, positioning, or citations — supersedes this, and Protocol v2's scope note already complies. But the Step 2 report itself still carries the recommendation uncorrected. **Fix:** add a superseded-by-instruction note at the head of that section in the report's goal-files copy, or annotate the report; do not leave it standing.

### D3. Step 1 report still contains the reversed Spider 2.0 version labels
Step 1's summary states "21.3% in v1 / 17.0% in the v2 arXiv text". Step 2 verified the reverse: arXiv v1 = 17.0%, v2 / ICLR 2025 = 21.3%. The correction exists in Step 2 and in MEMORY.md, but the Step 1 file itself was never errata'd. **Fix:** one-line erratum at the head of the Step 1 report pointing to Step 2's correction.

## Smaller observations

- **Even the 11-task sample surfaced gold noise:** q1029's gold orders ASC LIMIT 4 while its evidence says "highest". One suspect gold in 11 tasks is consistent with Jin's audit rates and reinforces the L3/exclusion policy. Logged during coding.
- **Step 1's single-vs-multi table leans on unreviewed 2026 preprints** (Tran & Kiela, BenchAgent, SOP study) for its general-agent claims. Correctly labelled [G]/preprint, but the daily briefing should re-check their status (any since accepted?) before the paper cites them.
- **Data-engineering agents remain a blind spot:** no peer-reviewed dbt/ELT agent benchmark was verified (ADE-bench is secondary-only). Given the author's data-engineering role, this is the briefing's largest coverage gap; the daily monitor should keep ADE-bench and CIDR/HILDA on watch.
- **No pilot budget estimate exists yet.** Step 2 noted DAB's authors spent $4,346 evaluating 3 agents. The pilot (5 arms × ~100 tasks × 4 runs) needs a cost estimate once W2's model decision is made.
- **Taxonomy freeze is blocked on the author checker pass** (protocol requires author + checker with an agreement figure). This is on the critical path: split construction clusters by failure class, so splits wait on the freeze, which waits on W1's frontier dry run and the checker pass.

## Decisions needed from the author

1. Pilot model(s): approve a frontier API model for the pilot (and which), or restrict the pilot to open/local models and accept the W1/W2 limitations in writing.
2. Checker pass on the Stage 1 open coding (agreements/disagreements) so agreement can be logged.
3. Approve the three document amendments (D1–D3) — none touches the frozen claim.

## What this review did not do
It did not re-verify the underlying papers (that was Steps 1–2's work, spot-checked here for internal consistency only), did not run the power analysis (step (2), pending W2), and did not evaluate venue strategy beyond what Step 2 scored.
