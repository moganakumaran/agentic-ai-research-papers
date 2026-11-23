# Dry Run — Blind Open Coding of Failure Traces (Stage 1 of 2)

- Date: 2026-11-22
- Coder: Muse (first annotator), coding from `dry_run_traces.jsonl` plus schema/gold inspection for diagnosis only.
- Blinding: coded WITHOUT consulting the seed taxonomy in the Pilot Protocol. Seed classes are introduced only in Stage 2 (reconciliation), in a separate file.
- Material: 11 dry-run tasks, Arm A0 (no-memory), Qwen2.5-Coder 3B local. 5 passed (all on attempt 1, no failure to code). 6 failed: q72, q96, q306, q469, q1006, q1029.
- Note: in every failure below, the full database schema (CREATE TABLE statements) was present in the model's prompt. None of these are missing-information failures.

## Per-trace coding

### q72 — california_schools — FAIL, 3 attempts, execution error
Question: students aged 5–17 enrolled at the State Special School in Fremont, 2014–15.
- Model selected `T2.Enrollment_Ages_5_17` from `schools`. No such column exists anywhere; the real column is `Enrollment (Ages 5-17)` on `frpm`. The model appears to have normalised the real column name (spaces/parentheses → underscores) and attached it to the wrong table.
- Filters also placed on the wrong table (`District`, `Educational Option Type` on `frpm`; the identifying attributes live on `schools`, and the gold routes "State Special School" through the code value `EdOpsCode = 'SSS'`, which the evidence spelled out).
- Attempts 2 and 3 are **byte-identical** to attempt 1 despite the "no such column" error being fed back each time.
Codes: OC-1, OC-2, OC-4.

### q306 — toxicology — FAIL, 3 attempts, execution error
Question: molecules with triple bonds; list their elements.
- Model routed the join through the `connected` table using `c.molecule_id`. `connected` has only `atom_id`, `atom_id2`, `bond_id` — no `molecule_id`. The gold does not use `connected` at all (`bond` itself carries `molecule_id`).
- Attempts 2 and 3 byte-identical after the error.
Codes: OC-3, OC-4.

### q469 — card_games — FAIL, 3 attempts, execution error
Question: did the set containing "Angel of Mercy" appear on Magic: The Gathering Online?
- Model used `mtgoCode` on `cards`. `mtgoCode` exists — on `sets`, not `cards` (`cards` has the near-miss names `mtgoId`, `mtgoFoilId`). The required join to `sets` was never attempted.
- Attempts 2 and 3 identical SQL (only the code fence dropped).
Codes: OC-2, OC-4.

### q1006 — formula_1 — FAIL, 1 attempt, executed, wrong result (silent)
Question: time of the fastest lap record.
- Model: `SELECT MIN(time) FROM lapTimes`. `time` is a stored duration string (e.g. "1:32.456", some with hour components); a text MIN is a lexicographic minimum, not the shortest duration. The gold parses the string into seconds with a long CASE expression before ordering.
- No execution error, so the retry loop never fired.
Codes: OC-6.

### q1029 — european_football_2 — FAIL, 3 attempts, execution error
Question: build-up play speed of the top 4 teams by buildUpPlaySpeed.
- Model selected `T2.buildUpPlaySpeed` from `Team`; the column exists only on `Team_Attributes` (which the model itself used correctly in the ORDER BY as T1). Existing column, wrong table attribution.
- Attempts 2 and 3 byte-identical after the error.
- Side note (gold quirk, not coded against the model): the gold orders ASC with LIMIT 4 while its own evidence says "highest" — flagged for the corrected-label log, L3-style, if this task is ever reused.
Codes: OC-2, OC-4.

### q96 — financial — FAIL, 1 attempt, executed, wrong result (silent)
Question: how many customers with weekly statement issuance are Owner?
- Model mapped "Owner" to `client.gender = 'Owner'`. `client.gender` holds only 'M'/'F'; "Owner" is a value of `disp.type` ('OWNER'/'DISPONENT'), which the gold uses. The model also joined through `client` unnecessarily and counted `client_id` where the gold counts `account_id`.
- The frequency value ('POPLATEK TYDNE' = weekly) was mapped correctly.
- No execution error; silently wrong count.
Codes: OC-5 (primary), OC-3-adjacent (unnecessary join path — noted, not double-counted as a primary code).

## Emergent open codes (Stage 1, pre-taxonomy)

- **OC-1 — Invented column name.** The SQL names a column that does not exist under that name in the schema (includes plausible renamings of a real column, e.g. normalising `Enrollment (Ages 5-17)` to `Enrollment_Ages_5_17`). (q72)
- **OC-2 — Right column, wrong table.** The column exists in the schema but is attributed to a table that does not have it; the join that would reach the owning table is missing or the alias is wrong. (q72 filters, q469, q1029)
- **OC-3 — Invented join key / wrong join path.** A join uses a key column the table does not have, or routes through a table the correct query does not need. (q306; secondary in q96)
- **OC-4 — Exact-repeat repair loop.** After an execution error is fed back, the next attempt is byte-identical (or cosmetically identical) SQL. Occurred in **all 4** multi-attempt failures (q72, q306, q469, q1029) — 8 of 8 retry opportunities produced no change. This code is orthogonal to the generation error: it converts a recoverable error into a certain failure.
- **OC-5 — Phrase grounded to the wrong attribute.** A question phrase is mapped to a column whose value domain does not contain it (e.g. "Owner" → `gender`), while the correct attribute exists elsewhere. Executes silently, wrong result. (q96)
- **OC-6 — Operation over an unparsed formatted value.** An aggregation/comparison (MIN/ORDER BY) is applied directly to a stored string whose format (durations, dates) does not order the way the question requires. Executes silently, wrong result. (q1006)

## Grouping observed at open-coding stage
- Generation-time schema-grounding failures: OC-1, OC-2, OC-3 (4 of 6 failures; all surfaced as execution errors).
- Repair-behaviour failure: OC-4 (co-occurs with every generation-time schema failure; never observed alone in this sample).
- Silent semantic failures: OC-5, OC-6 (2 of 6 failures; no execution signal, invisible to an error-feedback loop).

## Limits of this coding
- n = 6 failures from one small local model; code frequencies will not transfer to frontier models. The codes describe what happened in these traces, nothing more.
- Single coder so far. Protocol requires a second checker and an agreement figure at reconciliation.
