# Split construction readiness — prepared 2026-11-22 (after taxonomy freeze, while power analysis runs)

Prepared by proactivity research, NOT yet approved or applied. Nothing in the
frozen protocol, substrate spec, or manifest_v1.csv was edited by this note.

## 1. Status at preparation time

- Taxonomy FROZEN 2026-11-22 (dry_run_taxonomy_reconciliation.md): F1–F9 with
  observation scope per model. Frontier dry run: GPT-5 gpt-5-2025-08-07, 5/11
  original labels, 6/11 corrected labels, $0.09 of the $5 budget (Amendment A3).
- Power analysis RUNNING in the main chat (power_analysis.py, started
  2026-11-22 23:49 UTC; output file power_analysis_out.txt still empty at
  check time). Its GO margin / MDE must be recorded as a dated protocol
  amendment before split sizes are final — the grid tests n = 60 / 100 / 160,
  so the repeated-failure split size is not yet fixed.
- Protocol order (§12): step 3 (dry run → taxonomy freeze → power analysis)
  is nearly complete; step 4 is "Build splits 3a/3b (with similarity distances
  and silent/overt drift labels)". This note prepares step 4.

## 2. Finding: q1006 is a verified gold error but is not yet logged as L3

Evidence (from dry_run_taxonomy_reconciliation.md, frontier addendum):
- Task bird-1006, formula_1: "Among all the lap records set on various
  circuits, what is the time for the fastest one?"
- Original gold's string-parse ordering returns '2:00.000' as the fastest lap.
- GPT-5 ordered by the milliseconds column and returned '1:07.411', which is
  in fact the fastest lap in the data.
- q1006 is NOT in Jin's audited 498 (L2 in the manifest), so Jin's corrected
  labels do not cover it. This is our own verified label error, found by the
  frontier dry run.

Current records contradict that finding:
- Substrate spec §7 execution log still says "L3 = 0 identified from the
  available files" (that line predates the frontier run; it was true of the
  Jin files alone).
- Substrate spec §6 exclusion log has NO rows at all.
- manifest_v1.csv still lists bird-1006 as label_status L2, split dry-run,
  exclusion_reason empty (verified by direct read 2026-11-22: 1,534 rows,
  L1 498 / L2 1,036, split dry-run 11 / unassigned 1,523).

Why it matters: q1006 is a dry-run task and is NOT reused in pilot splits, so
no pilot result is affected today. But if the L3 status is never logged, a
later split-construction pass working from the manifest/spec alone would treat
bird-1006 as an ordinary L2 task. The substrate policy (§2/§6) requires
disputed tasks with no corrected gold to be EXCLUDED from primary metrics and
listed, never silently dropped.

### Draft §6 exclusion-log row (for author approval — not yet applied)

| source_task_id | label_status | reason | decided_by | date |
|---|---|---|---|---|
| 1006 (bird-1006, formula_1) | L3 (proposed; currently L2 in manifest v1) | Verified gold error found by frontier dry run 2026-11-22: original gold returns '2:00.000' as fastest lap; milliseconds column shows '1:07.411' is fastest. No Jin corrected label exists (task outside Jin's 498). Exclude from all primary metrics; appendix set only. | Mohan (approval pending) | 2026-11-22 |

On approval, the matching edits are: manifest bird-1006 label_status L2 → L3
(+ exclusion_reason "verified gold error, frontier dry run 2026-11-22"), and
substrate spec §7 L3 line updated from 0 to 1 with this evidence. q1029 needs
NO such row: its original gold was an artefact, but Jin corrected it, so it
remains L1 under corrected labels.

## 3. Manifest readiness audit for split construction (checked 2026-11-22)

Verified by direct read of manifest_v1.csv (1,534 rows):
- split: dry-run 11, unassigned 1,523. The 11 dry-run tasks are exactly
  72, 96, 306, 469, 591, 721, 1006, 1029, 1192, 1411, 1519 — none may be
  reused in pilot splits (substrate §4).
- label_status: L1 498 / L2 1,036 (before the proposed q1006 L3 change above).
- Columns present: task_id, source, source_task_id, database, question,
  gold_sql_original, gold_sql_corrected, label_status, difficulty_given,
  sql_len_chars, join_count, table_count, slow_gold_query, split, cluster_id,
  session, drift_operator, drift_type, exclusion_reason.
- Columns REQUIRED by substrate spec §5 but MISSING from manifest v1:
  similarity distances to session-1 partner(s); schema_version_pre;
  schema_version_post. These must be added during split construction, not
  assumed present.
- cluster_id / session / drift_operator are empty by design until construction;
  drift_type is 'none' throughout.

## 4. Split-construction plan (protocol §3 + substrate §4, steps 2–8)

- Repeated-failure split (~60 tasks, final n set by the power analysis):
  cluster candidate tasks by failure opportunity (shared schema elements,
  join patterns, aggregation/metric patterns) — by task/database structure
  ONLY, never by any agent run, so selection cannot favour the method.
  Order clusters into Session 1 (populates memory) / Session 2+ (different
  tasks sharing a failure class). Leakage rule: memory for a task may only
  contain episodes from strictly earlier sessions.
- Similarity distance (§3a v2): for every Session 2+ task compute question-
  embedding similarity, schema-element overlap, and gold-SQL structural
  overlap vs its Session 1 partner(s); stratify results by distance tertile.
- Near-duplicate threshold: STILL UNSET (substrate §7 checkbox open — value
  and basis to be set from the distance distribution at inventory, and
  recorded before any runs). Pairs above it go to the recall-control set, not
  the primary split.
- Drift split (~40 tasks, derived from the repeated-failure tasks): exactly
  one drift operator per task, rotation across operator types, at least half
  SILENT drift (value-code remap / metric-definition change / unit-scale
  change / row-population change under unchanged names); remainder OVERT
  (column rename / table rename-split / value-format change). Record operator,
  target, and pre/post schema version per task.
- Balance check before freezing the manifest: counts by source, database,
  label status, cluster, session, drift type; no stratum empty or >50% of a
  split without a logged reason. Manifest frozen (hash + date) before the
  first pilot run.

## 5. Decisions still needed — before pilot RUNS, not before split construction

- Pilot budget: Amendment A3 caps API spend at $5, which the protocol itself
  says does NOT cover the full pilot (5 arms × ~100 tasks × 4 runs, estimated
  in the tens of dollars). A separate budget decision is required before any
  pilot run starts; split construction itself is local computation and needs
  no API spend.
- Open-model slot (Amendment A2): Qwen2.5-Coder-32B-Instruct or DeepSeek
  current V-series, API-served — not yet chosen; not needed for split
  construction.
- GO margin: pending the running power analysis; record as a dated amendment
  once it lands, then finalise split sizes.

## 6. What was NOT done here

- No edit was made to the protocol, substrate spec, or manifest. The q1006 L3
  row above is a draft for Mohan's approval.
- No power-analysis result is claimed: the simulation was still running at
  preparation time and its output file was empty. The main chat will report
  the power result when it lands.
