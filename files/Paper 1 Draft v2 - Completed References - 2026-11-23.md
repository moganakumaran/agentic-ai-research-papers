# Paper 1 Draft v2 — Completed References (verified 2026-11-23)

Purpose: close the open item in Draft v2's draft notes — "full titles and author names
for the remaining short reference entries (MAG-SQL, CHESS, StaleBench, Experience Rot,
WikiTrace, Benchmarks' Last Exam) to be completed from the primary records at formatting
time". Each entry below was verified by reading the arXiv abstract page (primary source)
on 2026-11-23, 17:55–17:58 UTC. Draft v2 itself was NOT edited; these are ready to
paste at formatting time, or now if Mohan prefers.

## Completed entries (draft style)

- Xie, W., Wu, G., & Zhou, B. (2024). MAG-SQL: Multi-Agent Generative Approach with Soft Schema Linking and Iterative Sub-SQL Refinement for Text-to-SQL. arXiv:2408.07930.
  Replaces: "MAG-SQL. arXiv:2408.07930."
  Source: https://arxiv.org/abs/2408.07930 (submitted 15 Aug 2024, v4 7 Nov 2024).
  Abstract confirms BIRD 61.08% with GPT-4 vs MAC-SQL baseline 57.56%.

- Talaei, S., Pourreza, M., Chang, Y.-C., Mirhoseini, A., & Saberi, A. (2024). CHESS: Contextual Harnessing for Efficient SQL Synthesis. arXiv:2405.16755.
  Replaces: "CHESS. arXiv:2405.16755."
  Source: https://arxiv.org/abs/2405.16755 (submitted 27 May 2024, v3 25 Nov 2024).
  Abstract confirms 71.10% on the BIRD test set under high compute budgets.

- Murshed, S. M. M. (2026). StaleBench: Isolating Stale Retrieval in Agent Memory, and Pricing the Fix. arXiv:2611.19752.
  Replaces: "StaleBench: Isolating Stale Retrieval in Agent Memory, and Pricing the Fix. arXiv:2611.19752."
  Source: https://arxiv.org/abs/2611.19752 (submitted 15 Nov 2026; single author).
  Abstract confirms the headline the draft cites: retrieval metrics can rank memory
  policies backwards — identical statement at rank 1 for every query, yet 29.5 and
  65.6 points apart in reader accuracy; Contamination@5 ranks them correctly.

- Xu, Z., Yin, H., Zhang, Z., Zhao, K., & Liu, X. (2026). Experience Rot: Experienced Agents Fail Where Novices Succeed. arXiv:2611.20922.
  Replaces: "Experience Rot: Experienced Agents Fail Where Novices Succeed. arXiv:2611.20922."
  Source: https://arxiv.org/abs/2611.20922 (submitted 16 Nov 2026).
  Abstract confirms: silent MCP tool-interface change, experienced agents fail where
  novices succeed, failure in every memory form tested, announcing the update barely helps.

- Lee, Y., Choi, Y., Baek, J., & Hwang, S. J. (2026). Don't Just Trust Your Memory: Evidence-Linked Memory Reduces Memory-Induced Degradation in LLM Agents. arXiv:2611.14320.
  Replaces: "Don't Just Trust Your Memory (WikiTrace). arXiv:2611.14320."
  Source: https://arxiv.org/abs/2611.14320 (submitted 10 Nov 2026).
  Abstract confirms the "memory-induced degradation" term and WikiTrace as the
  evidence-linked system. The system name WikiTrace is in the abstract.

- Zhu, J., & Cat, Z. (2026). Benchmarks' Last Exam. arXiv:2611.27622.
  Replaces: "Benchmarks' Last Exam. arXiv:2611.27622."
  Source: https://arxiv.org/abs/2611.27622 (submitted 20 Nov 2026).
  Abstract confirms 749 tasks across eight benchmarks, 92.5% verified validity for
  closed-form vs 29.3% for multi-step suites, 71.7% of confirmed defects withholding
  credit from compliant solutions.

## Proposed addition (needs Mohan's yes)

- Jayaram, K. R., Isahagian, V., Aravindan, A. V., Muthusamy, V., Oum, P., Fang, G., Ulewicz, M., Thomas, G., & Duesterwald, E. (2026). How Much Should an Agent Remember? Deployment Lessons from Self-Distilled Memory for LLM Agents. arXiv:2611.26361.
  Source: https://arxiv.org/abs/2611.26361 (submitted 19 Nov 2026).
  Why: Draft v2 §7 says "the capability-dependence reported for guideline memory in
  general domains makes the frontier case a genuinely open question" — but no reference
  entry backs that sentence. The abstract confirms the finding: DeepSeek-V3.2 gains
  +9.5 task-completion points with its full 191-guideline set, while gpt-oss-120b does
  better with a ~29-guideline subset (55.4%) than the full 199 (51.2%). If added, it
  needs an in-text author cite at formatting time. In draft style it would read:
  "Jayaram et al. (2026). How Much Should an Agent Remember? Deployment Lessons from
  Self-Distilled Memory for LLM Agents. arXiv:2611.26361."

## Cautions / what was deliberately not changed

- MAG-SQL "dev" qualifier: the arXiv abstract reports 61.08% on BIRD with GPT-4
  without labelling dev vs test. The draft's "61.08% dev" rests on the 2026-11-23
  related-work verification, not on this abstract check. No figure changed.
- CHESS figures: the abstract's 71.10% is a BIRD test-set number under high compute;
  the draft's "dev ≈65–68%, test ≈67–71%" range is consistent with it. No figure changed.
- StaleBench is a single-author paper; the author is rendered "Murshed, S. M. M."
  from the arXiv display form "S M Mahbub Murshed".
- WikiTrace: the "52% of failures trace to a followed item" figure circulating in
  secondary digests is NOT in the abstract and remains unverified — it is not included
  here, and the draft correctly cites only the named concept.
- All seven are arXiv preprints with no venue; cited as arXiv only, no venue claimed.

## Status of the morning briefing's Paper #1 items (for the deferred brief discussion)

- StaleBench citation: DONE — cited in Draft v2 §2 and abstract framing.
- Frontier A1 run as a capability-gating test (briefing §4): DECIDED, not open — M4 was
  resolved 2026-11-23 by narrowing the title; the frontier A0/A1 contrast remains an
  optional follow-up (~$12 for 1 run / ~$45+ for 4 runs), not a next step. Do not re-open
  without new evidence.
- Dec 1 PVLDB cycle: RULED OUT in the Draft v2 revision plan (abstract Nov 25, paper
  Dec 1 — not feasible; do not chase). Working target remains the Feb 1, 2027 cycle
  (abstract Jan 25, 2027; Jan 1 cycle, abstract Dec 25, 2026, as stretch).

## What still remains before submission (unchanged from Draft v2 notes)

- Artefact/protocol public release (protocol + Amendments A1–A14, splits, manifest,
  traces, harness, prompts) with a repo link/DOI in §4. Files verified present locally
  2026-11-23: pilot.py, pilot_lib.py, manifest_v2.csv, 112 trace/episode .jsonl files,
  datasets/bird, bird_drift, jin_corrected; protocol file carries Amendments A1–A14.
  The remaining step is publishing, not assembling.
- PVLDB template formatting (after the venue call); optional Figure 2 extension with
  the A14 contrasts.
