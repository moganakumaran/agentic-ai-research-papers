# Dry-run report (DRAFT — for Mohan's review, not yet recorded in the frozen protocol)

Date: 2026-11-22. Arm A0 (no-memory), model `qwen-coder-3b` (Qwen2.5-Coder 3B, local via Ollama).
Sources: `hidden_files/dry_run_log.txt` (finished 22:37 UTC) and `hidden_files/dry_run_traces.jsonl` (11 traces).
All figures below were recomputed from those two files, not copied from chat summaries.

## Headline result

**5 / 11 passed (45%) — paired pass@1, execution accuracy vs gold.**
Wilson 95% CI for the base rate: 21–72% (n=11, so the interval is wide by design; the pilot's
own A0 arm on ~100 tasks will pin it down). Total cost: 21,500 tokens, 617 s (~10 min).

This is a base rate for the power analysis, not a leaderboard number — Mohan was told in
advance the 3B local model would score well below frontier models.

## Per-task results

| q | database | result | attempts | label | difficulty (BIRD-given) |
|---|---|---|---|---|---|
| 72 | california_schools | FAIL — exec error, same SQL x3 | 3 | L1 | moderate |
| 96 | financial | FAIL — executed clean, wrong result | 1 | L2 | simple |
| 306 | toxicology | FAIL — exec error, same SQL x3 | 3 | L2 | challenging |
| 469 | card_games | FAIL — exec error, same SQL x3 | 3 | L1 | moderate |
| 591 | codebase_community | PASS | 1 | L2 | simple |
| 721 | superhero | PASS | 1 | L2 | simple |
| 1006 | formula_1 | FAIL — executed clean, wrong result | 1 | L2 | challenging |
| 1029 | european_football_2 | FAIL — exec error, same SQL x3 | 3 | L1 | moderate |
| 1192 | thrombosis_prediction | PASS | 1 | L1 | challenging |
| 1411 | student_club | PASS | 1 | L1 | simple |
| 1519 | debit_card_specializing | PASS | 1 | L2 | simple |

By difficulty: simple 4/5, moderate 0/3, challenging 1/3. By label: L1 2/5, L2 3/6
(no label effect visible at n=11; do not over-read either split).

## Finding 1 — the self-debug loop never changed anything

All 5 passes were first-attempt passes. All 4 execution-error failures repeated the
**identical SQL on all 3 attempts** (temperature 0 + the same prompt with the error appended
re-generates the same query). No task was ever rescued by a retry.

Implication for the pilot, and a decision for Mohan: the retry loop as built burns tokens
without changing outcomes. Either (a) keep it frozen as-is and let failure memory be the
mechanism that breaks the loop — the cleanest reading of the protocol, since the loop is
part of the A0 baseline — or (b) amend the harness before the 100-task run (e.g. force a
schema re-check / a different approach on retry). This should be a dated amendment if (b),
not a silent change. My judgement: keep (a) for the pilot — the inert loop is itself a
finding, and changing the baseline mid-stream muddies the memory comparison.

## Finding 2 — draft open coding of the 6 failures (for blind-labelling reconciliation)

Open coding from the observable traces only (execution errors + result mismatch as scored
by the harness). This is a draft for reconciliation, not the final taxonomy — the protocol
requires blind labelling with seed classes only at reconciliation.

- **Schema-grounding failure (4/6):** the model used a column that does not exist on the
  table it queried — q72 `T2.Enrollment_Ages_5_17`, q306 `c.molecule_id`, q469 `mtgoCode`,
  q1029 `T2.buildUpPlaySpeed` (attribute lives on Team_Attributes, model put it on Team).
  All 4 then failed to self-correct (see Finding 1).
- **Silent wrong-result / semantic misreading (2/6):** SQL executed cleanly but answered a
  different question — q96 (treated `gender` as the owner flag instead of the disp type),
  q1006 (`MIN(time)` with no race/context filter).
- **Retry non-adaptation (4/6, overlapping):** the error-feedback loop produced zero change
  in every exec-error case. Code it as a loop property, not a per-task failure cause.

Note: the dominant class (schema grounding, 4/6) is exactly the failure class SQLAgent-style
exploration is designed to prevent (Jiang et al., arXiv:2602.01952, verified 2026-11-22) —
worth a line in the related work, and a reason the memory arm's episodes should record the
schema correction, not just the fixed SQL.

## Power analysis (simulation) — proposed GO margin

Method: McNemar paired test on ~100 pilot tasks, two-sided alpha 0.05, 20,000 simulations.
Discordant-pair scenarios (p10 = A0-fail/memory-pass, p01 = A0-pass/memory-fail):

| Scenario | Implied delta | Power at n=100 |
|---|---|---|
| Fix 25% of A0 failures, break 5% of passes (p10 .25, p01 .05) | +20 pp | ~0.96 |
| Fix 20% of A0 failures, break 5% of passes (p10 .20, p01 .05) | +15 pp | ~0.84 |
| Fix 15% of A0 failures, break 5% of passes (p10 .15, p01 .05) | +10 pp | ~0.54 |
| Fix 10% of A0 failures, break 5% of passes (p10 .10, p01 .05) | +5 pp | ~0.18 |

**Proposed GO margin (draft, needs Mohan's approval): +15 percentage points paired pass@1.**
At n=100 that is the smallest round margin with ~80% power under the stated discordant
assumptions. The old "≥8 points" guess is already superseded by the protocol and is not
defensible here: +10 pp has only ~54% power.

Caveats, stated plainly: the dry run cannot estimate p10/p01 — there is no memory arm yet —
so the discordant assumptions are judgement, not measurement. If the memory arm breaks
more A0 passes than assumed (stale-memory harm is a real risk the protocol names), power
falls. The honest use of this margin: record it as a dated amendment to the frozen
protocol now, before the pilot runs, and judge the pilot against it.

## Next steps, in the frozen order

1. Mohan reconciles the draft open coding above (blind-labelling step) → failure taxonomy.
2. Mohan approves (or revises) the +15 pp GO margin → recorded as a dated amendment in
   the frozen Pilot Protocol v2 (Section 1 claim itself is not edited).
3. Split construction (repeated-failure + drift) and freezing the manifest — only after 1–2.
4. Daily briefing starts tomorrow (Mon 2026-11-23) ~08:23 PT, already scheduled.
