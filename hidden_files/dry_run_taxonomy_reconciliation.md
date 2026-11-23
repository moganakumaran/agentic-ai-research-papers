# Dry Run — Taxonomy Reconciliation (Stage 2 of 2)

- Date: 2026-11-22
- Input: Stage 1 open coding (`dry_run_open_coding.md`, written blind) reconciled against Pilot Protocol v2, Section 5 seed classes F1–F8.
- Status: **TAXONOMY FROZEN 2026-11-22** (see freeze record addendum below): second-annotator agreement 6/6, frontier dry run coded, classes annotated with model-observation scope. Author override remains possible via dated amendment.

## Mapping: open codes → seed classes

| Open code | Seed class | Verdict |
|---|---|---|
| OC-1 Invented column name | F1 Wrong schema link | Covered by F1, but keep as named subcode **F1a** — a name that exists nowhere in the schema (incl. normalised renamings of a real column) is distinguishable from picking the wrong existing column, and a fingerprint can key on it differently. |
| OC-2 Right column, wrong table | F1 Wrong schema link | Covered by F1; keep as subcode **F1b**. |
| OC-3 Invented join key / wrong join path | F3 Wrong join / join fan-out | Covered by F3. (No fan-out instance observed; the observed form is an invented join key.) |
| OC-4 Exact-repeat repair loop | — none — (nearest: F6) | **NEW CLASS — add as F9.** F6 ("execution error not recovered") names the outcome; OC-4 names the mechanism actually observed in 4 of 4 multi-attempt failures and 8 of 8 retries: byte-identical resubmission after error feedback. It is separately measurable, and it is precisely the behaviour a failure-memory intervention would target, so it must be its own class. F6 is retained for non-recovery by other mechanisms (varied but wrong repairs, giving up). |
| OC-5 Phrase grounded to wrong attribute (value outside the column's domain) | F5 Value-format / filter literal mismatch (cross-ref F1) | Partially covered. F5 as seeded says "filter literal mismatch"; q96's literal was fine as a string but was aimed at a column (`gender`) whose domain cannot contain it, while the correct attribute (`disp.type`) existed. Recommend amending F5's definition to: "filter literal or value-domain mismatch, including a question phrase grounded to an attribute whose value domain excludes it." |
| OC-6 Operation over unparsed formatted value | F5 (cross-ref F4 Aggregation error) | Partially covered. MIN/ORDER BY over a stored duration string is a value-format failure inside an aggregation, not a filter literal. Covered if F5 is broadened per OC-5's amendment to "value-format mishandling (filters, comparisons, aggregations)". Otherwise add subcode under F4. Recommendation: broaden F5 as above and cross-reference F4. |

## Primary class per failed dry-run task (reconciled labels)

| Task | Primary | Secondary | Silent? |
|---|---|---|---|
| q72 california_schools | F1a (+F1b filters) | F9 | No (exec error) |
| q306 toxicology | F3 | F9 | No |
| q469 card_games | F1b | F9 | No |
| q1029 european_football_2 | F1b | F9 | No |
| q96 financial | F5 (amended def.) | F1 | **Yes** |
| q1006 formula_1 | F5 (amended def.) | F4 | **Yes** |

## Seed classes with no observed instance in the dry run
- **F2 Stale schema assumption** — unobserved (expected: the dry run has no drift). KEEP — it is a design class for the drift split, not a dry-run class.
- **F7 Correct SQL, wrong interpretation / answer format** — unobserved. Mark UNOBSERVED, keep for the pilot (frontier models may exhibit it even though the 3B model did not).
- **F8 Unsafe or destructive operation attempted** — unobserved (all dry-run tasks are read-only SELECTs). Mark UNOBSERVED, keep as log-only per protocol.
- F4 Aggregation/grouping error — no pure instance; appears only as cross-reference (q1006). Mark as observed-only-as-secondary.
- F6 Execution error not recovered — no instance distinct from F9 in this sample. Keep, marked "mechanism unspecified; all observed non-recoveries were F9."

## Proposed frozen taxonomy (pending author check)
F1 Wrong schema link — F1a invented column name; F1b right column, wrong table.
F2 Stale schema assumption (design class for drift split; unobserved in dry run).
F3 Wrong join / join fan-out (observed form: invented join key).
F4 Aggregation / grouping error incl. metric-definition error (secondary-only in dry run).
F5 Value-format / value-domain mishandling — filters, comparisons, aggregations (definition broadened at reconciliation, change logged here).
F6 Execution error not recovered, mechanism other than F9 (unobserved in dry run).
F7 Correct SQL, wrong interpretation / answer format (unobserved).
F8 Unsafe or destructive operation attempted, log only (unobserved).
F9 NEW — No-change repair loop: identical resubmission after error feedback (observed in 4/4 multi-attempt failures).

## Addendum 2026-11-22 — Inter-annotator agreement (Annotator 2, independent, document-blind)

Annotator 2 (fresh agent; barred from the Stage 1 coding, this reconciliation, the seed taxonomy, and all project documents; worked from traces + gold + read-only DB inspection only; file `dry_run_open_coding_annotator2.md`). Disclosed limit: Annotator 2 shares the project conversation context, so it is document-blind, not context-blind. The author's own checker pass remains the gold standard and overrides this figure.

Annotator 2's emergent categories vs Annotator 1's open codes:
- A2 "Referenced a column the named table does not have" (q72, q469, q1029; three shapes incl. respelled-name and wrong-alias) ↔ OC-1 + OC-2 (A2 merged what A1 split; A2's shape analysis independently reproduces the F1a/F1b distinction).
- A2 "Fabricated join path" (q306) ↔ OC-3.
- A2 "Identical resubmission after an error" (4 tasks, 8 retries, 0 changes) ↔ OC-4 / proposed F9.
- A2 "Question concept attached to a column whose values exclude it" (q96, silent) ↔ OC-5.
- A2 "Aggregate applied to a formatted string as if it were the quantity" (q1006, silent) ↔ OC-6.

Agreement:
- Primary defect per task: **6/6 tasks identical** (100%).
- Silent vs execution-error split: identical (4 execution, 2 silent).
- Retry-behaviour finding: identical (8/8 retries unchanged; A2 also noted q469's only change was a dropped code fence).
- Independent convergence: A2 also flagged the q1029 gold ASC/"highest" quirk without seeing A1's note.
- Only difference: category granularity (A2 merged OC-1/OC-2 into one category with sub-shapes). A grouping preference, not a labelling disagreement; the reconciled taxonomy already keeps them as subcodes F1a/F1b of one class, which matches A2's structure.

Conclusion: the Stage 1 open coding is corroborated by an independent document-blind pass. No label changes result. Freeze remains gated on the frontier dry run (Amendment A2) and is subject to author override.

## Addendum 2026-11-22 — Frontier dry run (GPT-5) coding and TAXONOMY FREEZE

Frontier run: Arm A0, model gpt-5-2025-08-07 (pinned), same 11 tasks, identical prompts/scoring as the 3B run. Result: **5/11 on original labels — the same headline rate as the 3B model, with a disjoint failure profile.** Total API cost $0.09. Traces: `dry_run_traces_frontier.jsonl`. All GPT-5 attempts executed without error (zero execution errors, zero retries); every failure is silent. Diagnosis verified by running gold, corrected gold, and predicted SQL and comparing result sets.

Frontier failures, coded:
- q72 — NOT a model error under corrected labels. GPT-5 answered 375.0 (SUM over the two matching schools). Original gold returns two rows (40, 335); Jin's CORRECTED gold is a SUM and returns exactly [(375.0,)] — **GPT-5 passes q72 under L1 corrected labels.** (On the 3B run this task was F1a+F9; the frontier model grounded the schema correctly.)
- q306 — genuine model error (L2, no corrected label). Molecule set diverges from gold (includes TR499, misses gold molecules); result shape also differs (grouped elements + extra column vs gold's molecule/element pairs). Primary: content/semantic divergence, mechanism not fully pinned from outputs alone; secondary F7 (shape).
- q469 — F7 answer-shape mismatch under both label sets. GPT-5 returned a single 'Yes' (question as asked); original gold returns 15 YES/NO rows; Jin corrected gold returns the 6 set names. Model answered the question form; neither gold matches that form.
- q1006 — **VERIFIED GOLD ERROR, discovered by this dry run; task is NOT in Jin's audited 498 (L2).** Original gold's string-parse ordering returns '2:00.000' as the fastest lap; GPT-5 ordered by the milliseconds column and returned '1:07.411', which is in fact the fastest lap in the data. Model correct, label wrong. Logged as a disputed-task (L3) candidate with this evidence; excluded-from-primary handling per substrate policy if ever reused.
- q1029 — original gold was an artefact (returned the four LOWEST speeds, 20/20/20/23, for a "highest" question — the quirk both annotators flagged). Jin corrected gold returns (80, 77, 73, 71) using latest-attributes-per-team. GPT-5 returned team names + MAX-over-history (80, 78, 78, 77): still a mismatch under corrected labels. Genuine frontier failure: **temporal semantics of attribute history (MAX over all snapshots vs latest snapshot)** + extra output column. Primary F4 (aggregation semantics), secondary F7.
- q1411 — F7 multiplicity mismatch. GPT-5's DISTINCT returns the same 7 expense kinds as gold's 12-row multiset with duplicates. Jin audited this task and left gold unchanged, so the scored failure stands under both label sets.

Dual-label frontier score: **5/11 original labels; 6/11 with Jin corrected labels applied where they exist (q72 flips); q1006 additionally verified as a gold error outside Jin's set.** The 3B model scores 5/11 on original labels (its q72/q1029 failures remain failures under any label — they were execution errors).

### FREEZE RECORD — 2026-11-22
Gate conditions met: (i) blind open coding by Annotator 1; (ii) independent document-blind Annotator 2, agreement 6/6 on primary defects; (iii) frontier-model dry run on the same tasks coded before freezing (Protocol Amendment A2). The taxonomy below is **FROZEN** for the pilot. Changes require a dated amendment. Author override remains possible and would be logged as an amendment.

Frozen classes, with observation scope:
- F1 Wrong schema link — F1a invented column name; F1b right column, wrong table. Observed: 3B run only. Not observed at frontier (n=11).
- F2 Stale schema assumption. Unobserved in both dry runs (no drift tasks). Design class for the drift split.
- F3 Wrong join / join fan-out (observed form: invented join key). Observed: 3B run only.
- F4 Aggregation / grouping error incl. metric-definition and temporal-snapshot semantics (MAX-over-history vs latest). Observed: frontier run (q1029).
- F5 Value-format / value-domain mishandling (broadened definition, reconciliation above). Observed: 3B run (q96, q1006-as-model-error).
- F6 Execution error not recovered, mechanism other than F9. Unobserved in both runs.
- F7 Correct content, wrong answer shape/format (extra/missing columns, grouping shape, case, duplicate multiplicity, question-form mismatch). Observed: frontier run (q469, q1411; secondary q306, q1029). Not observed in the 3B run.
- F8 Unsafe or destructive operation attempted (log only). Unobserved (all tasks read-only).
- F9 No-change repair loop (identical resubmission after error feedback). Observed: 3B run only (4/4 multi-attempt failures). Not observed at frontier (no execution errors occurred).

Explicitly NOT taxonomy classes: gold-label errors and scoring-shape artefacts (q1006 gold, q1029 original gold, q469 gold shapes). These are substrate/label-policy matters (L1/L2/L3), handled by dual reporting — the dry runs produced verified instances of them, which is evidence for the substrate design, not for agent failure classes.

## Addendum 2026-11-22 — Pilot-model spot-check (GPT-5 mini, per Amendment A6)

Pilot primary model gpt-5-mini-2025-08-07 ran Arm A0 on the same 11 tasks (traces: `dry_run_traces_mini.jsonl`; cost $0.01). Result: **4/11 original labels; 5/11 with Jin corrected labels (q72 flips — mini's SUM answer 375.0 matches corrected gold exactly, as GPT-5's did).** Zero execution errors; all failures silent, single-attempt. Failure-by-failure, every mini failure maps onto the FROZEN taxonomy — **no new failure class appeared**:
- q72: passes under corrected labels (not a model error).
- q306: same content divergence + shape difference as GPT-5 (near-identical SQL, incl. the TR499 molecule).
- q469: F7 (single 'Yes', question form) — same as GPT-5.
- q1006: verified gold error (same as frontier coding); mini's per-circuit-record reading still returns the genuinely fastest lap, 1:07.411.
- q1029: F4 (MAX over attribute history vs latest snapshot) + F7 (extra name column) — same as GPT-5.
- q1192: F7 — correct patient set, wrong answer shape (6 columns returned vs gold's ID-only). The one task GPT-5 and the 3B passed and mini failed; class is not new.
- q1411: F7 multiplicity (DISTINCT) — identical SQL to GPT-5's.
Base rates for the record: mini A0 = 0.36 original / 0.45 corrected (n=11) — inside the power analysis's simulated range [0.30, 0.55], so the Amendment A4 GO margin (12 points) stands without further amendment. Taxonomy freeze unaffected.

## Also logged during coding (not taxonomy)
- q1029 gold SQL orders ASC LIMIT 4 while its evidence says "highest" — a suspected gold-label quirk. If this task is ever reused, route it through the substrate label policy (disputed → exclude from primary, log). Not used in any dry-run split.
- All 5 passes were first-attempt; the harness's error-feedback loop was exercised only by failures, and never once produced a changed query. This is a property of this model + harness, recorded as observed, not generalised.
