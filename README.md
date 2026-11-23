# Research Papers on Agentic AI for Data & AI

Working repository for Mohan's (Mogana Kumaran Sivaraman) research programme on
agentic AI for data work.

## Paper #1 — Failure Memory for Text-to-SQL Agents

A pre-registered, compute-fair, corrected-label evaluation of episodic failure
memory for Text-to-SQL agents on BIRD, including a purpose-built drift split, a
verifier ablation, and an open-model replication. Current manuscript:
`files/Paper 1 Draft v2.md` (pivot framing: negative-result paper, adopted in
Protocol Amendment A9).

Layout:

- `files/` — documents for the author: protocol, substrate specification,
  drafts, stage reports, replication report, critique and fix memo, figures.
- `hidden_files/` — working artefacts: the pilot harness (`pilot.py`,
  `pilot_lib.py`), split builders, frozen manifest and splits, all run traces
  and episode journals, analysis scripts and results JSONs, spend ledgers.
- `hidden_files/datasets/` — **not in git** (see `.gitignore`): BIRD Dev, the
  derived drift databases, and the Jin et al. corrected-label repo. Rebuild
  instructions are in the Substrate Specification and the builder scripts.

Protocol changes are made only as dated amendments to the frozen Pilot
Protocol v2 (`files/Paper 1 Pilot Protocol - Failure Memory for Data Agents.md`).
