# Dry Run — Open Coding by Annotator 2 (independent, blind)

- Date: 2026-11-22
- Annotator: Annotator 2 (independent second annotator). All code labels below were invented by this annotator from the trace/schema/gold evidence only.
- Material: 11 dry-run tasks, Arm A0, model qwen-coder-3b. 5 passed (no failure to code). 6 failed: q72, q96, q306, q469, q1006, q1029.
- Failure surface split: **4 failures surfaced as execution errors** (q72, q306, q469, q1029); **2 were silent** — the SQL executed and returned a wrong result (q96, q1006).

## Per-task coding

### q72 — california_schools — execution error, 3 attempts
Question: how many students aged 5–17 are enrolled at the State Special School in Fremont, 2014–15.
- **Observable defect:** The SELECT names `T2.Enrollment_Ages_5_17` on `schools`. No column of that name exists in any table. The real column is `Enrollment (Ages 5-17)` and it lives on `frpm` (the T1 table), not `schools`. The generated name looks like the real column name with spaces/parentheses converted to underscores, then attached to the wrong table. The school-identifying filters were also placed on `frpm` (`T1.District`, `T1."Educational Option Type"`); `District` belongs to `schools`, and the gold identifies the school on `schools` via `EdOpsCode = 'SSS'` and `City = 'Fremont'`. The model also wrapped the value in SUM where the gold selects the single stored value.
- **Retry behaviour:** Attempts 2 and 3 are byte-identical to attempt 1, including the nonexistent column, despite the "no such column" error being returned each time. No element of the query changed.
- **My labels:** `renamed-column-on-wrong-table`; `filters-on-wrong-table`; `repeat-identical-after-error`.

### q96 — financial — SILENT (executed, wrong result), 1 attempt
Question: how many customers who choose weekly statement issuance are Owner?
- **Observable defect:** The model filtered `client.gender = 'Owner'`. Inspection of the database shows `client.gender` contains only 'M' and 'F'; the value "Owner" lives in a different column — `disp.type`, whose stored values are 'OWNER' and 'DISPONENT'. The predicate as written can never match, so the count returned is wrong (zero by construction). The gold joins `account` to `disp`, filters `disp.type = 'OWNER'`, and counts accounts. The model's weekly-issuance literal ('POPLATEK TYDNE' on `account.frequency`) was correct; the owner concept was attached to the wrong column, and it also counted clients rather than accounts via an extra join.
- **Retry behaviour:** Single attempt; no execution error occurred, so nothing triggered a retry.
- **My labels:** `value-placed-on-column-that-cannot-hold-it` (silent).

### q306 — toxicology — execution error, 3 attempts
Question: which molecules have triple bonds, and list all the elements they contain.
- **Observable defect:** The model routed the query through the `connected` table with the join condition `m.molecule_id = c.molecule_id`. Inspection shows `connected` has only three columns — `atom_id`, `atom_id2`, `bond_id` — and no `molecule_id`. The join key was invented. The route was also unnecessary: `bond` itself carries `molecule_id` and `atom` carries `molecule_id`, and the gold joins `bond` directly to `atom` on `molecule_id`, never touching `connected` or `molecule`.
- **Retry behaviour:** Attempts 2 and 3 are byte-identical to attempt 1 after the "no such column: c.molecule_id" error. No change.
- **My labels:** `invented-join-key`; `unneeded-intermediate-table`; `repeat-identical-after-error`.

### q469 — card_games — execution error, 3 attempts
Question: did the set of cards with "Angel of Mercy" appear on Magic: The Gathering Online?
- **Observable defect:** The model tested `mtgoCode` on the `cards` table. `cards` has no `mtgoCode` column (it has the similar-looking `mtgoId` and `mtgoFoilId`). `mtgoCode` exists — on the `sets` table. The gold joins `cards` to `sets` on `cards.setCode = sets.code` and tests `sets.mtgoCode`. The model never joined to `sets`, so the one table holding the needed column was never reached. (Secondary: the model answered with a boolean `COUNT(*) > 0` where the gold returns 'YES'/'NO' — an output-shape difference, not the cause of failure.)
- **Retry behaviour:** The SQL is identical in all 3 attempts; attempts 2–3 differ only in dropping the markdown code fence. The defective column reference never changed after the "no such column: mtgoCode" error.
- **My labels:** `column-lives-on-unjoined-table`; `repeat-identical-after-error`.

### q1006 — formula_1 — SILENT (executed, wrong result), 1 attempt
Question: among all lap records, what is the time of the fastest one?
- **Observable defect:** The model ran `SELECT MIN(time) FROM lapTimes`. Inspection shows `time` is stored as text in 'M:SS.mmm' format (samples: '1:49.088', '1:31.600'), and some values carry a larger minute/hour-scale component. Text ordering does not match duration ordering across these formats: running the model's own query returns '10:32.179' — a lexicographic minimum that is plainly not the fastest lap. The gold parses the string into numeric seconds with a long CASE expression before ordering. (The table also has a numeric `milliseconds` column, which the model did not use.)
- **Retry behaviour:** Single attempt; no execution error, so no retry was triggered.
- **My labels:** `aggregate-over-text-that-does-not-sort-like-the-quantity` (silent).

### q1029 — european_football_2 — execution error, 3 attempts
Question: the build-up play speed of the top 4 teams with the highest build Up Play Speed.
- **Observable defect:** The SELECT list takes `T2.buildUpPlaySpeed` and `T2.buildUpPlaySpeedClass` from `Team` (T2). Inspection shows `Team` has neither column (its columns are id, team_api_id, team_fifa_api_id, team_long_name, team_short_name). Both columns exist on `Team_Attributes` (T1) — the other table already in the query, and the model itself referenced `T1.buildUpPlaySpeed` correctly in the ORDER BY. So the right columns were selected from the wrong table alias.
- **Retry behaviour:** Attempts 2 and 3 are byte-identical to attempt 1 after the "no such column: T2.buildUpPlaySpeed" error. No change.
- **Annotator's side note (about the gold, not the model):** the gold orders ASC with LIMIT 4 while the question asks for the "highest" speeds and its own evidence mentions MAX; the model's DESC matches the question wording better. This did not cause the failure (the execution error did), but the gold's ordering direction looks questionable on its face.
- **My labels:** `right-column-wrong-alias`; `repeat-identical-after-error`.

## Emergent categories (Annotator 2's grouping)

1. **Referenced a column the named table does not have** — the generated SQL points a column name at a table that does not contain it. Three observed shapes: (a) the name exists nowhere — it is a respelled version of a real column that lives on another table (q72); (b) the name exists on a different table that the query never joins to (q469); (c) the name exists on another table already in the query, but the wrong alias is used (q1029). **Tasks: 3 (q72, q469, q1029). All execution errors.**
2. **Fabricated join path** — a join is built on a key column the joined table does not have, via an intermediate table the correct query does not need (q306). **Tasks: 1. Execution error.** (A sibling of category 1 — both reference schema elements that do not exist as referenced — but the fabricated element here is the join route itself, so I keep it separate.)
3. **Identical resubmission after an error** — a retry-behaviour category, cross-cutting categories 1–2: whenever the run got an execution error and retried, the next attempt repeated the defective SQL unchanged. Observed in **all 4** execution-error failures (q72, q306, q469, q1029): 8 retry opportunities, 0 changes to the defective element (q469's only change was dropping a markdown fence). This behaviour is what turned each recoverable-looking error into a final failure.
4. **Question concept attached to a column whose values exclude it** — a filter value is placed on a plausible-sounding but wrong column whose stored values cannot contain it, while the column that does hold that value exists elsewhere in the schema (q96: 'Owner' against `client.gender`, whose values are only 'M'/'F', instead of `disp.type`). **Tasks: 1. Silent.**
5. **Comparison/aggregate applied to a formatted string as if it were the quantity** — MIN/ordering is applied directly to a text column whose string sort order does not correspond to the underlying quantity's order (q1006: duration strings 'M:SS.mmm'). **Tasks: 1. Silent.**

### Counts at a glance
- Execution-error failures: 4 of 6 (q72, q306, q469, q1029) — all involve referencing schema elements that do not exist as referenced (categories 1–2), and all exhibit category 3 on retry.
- Silent failures: 2 of 6 (q96, q1006) — one each of categories 4 and 5. Neither produced any signal that could trigger a retry.
- In all 6 failures the needed columns/values existed somewhere in the database; no failure was caused by missing data.

## Method note
Files read for this coding: `hidden_files/dry_run_traces.jsonl`; gold SQL/questions from `hidden_files/datasets/bird/dev_20240627/dev.json`; and read-only sqlite3 inspection of the six databases (california_schools, financial, toxicology, card_games, formula_1, european_football_2) under `dev_databases/dev_databases/`. No other workspace documents were read.
