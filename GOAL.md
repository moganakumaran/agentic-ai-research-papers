# Research papers on agentic AI for Data & AI

Goal ID: goal_6b173cbcb5be
Goal slug: research-papers-on-agentic-ai-for-data-ai

## Description
 track recent trends in AI agentic engineering daily across arXiv and major AI, NLP, data and software engineering venues, identify research gaps, and turn the strongest gaps into papers. The first paper focus is agent evaluation and reliability for data work, including Text-to-SQL and data analytics agents.

## Setup (2026-11-22)
- Daily briefing cron: daily-agentic-ai-research-briefing, around 8am America/Los_Angeles
- Sources: arXiv (cs.AI/CL/LG/DB/SE), OpenReview, alphaXiv; NeurIPS, ICML, ICLR, COLM, AAAI, ACL, EMNLP, NAACL, SIGMOD, VLDB, ICDE, CIDR, KDD, ICSE, FSE, ASE

## Paper #1 — Agent evaluation & reliability for data: initial gap outline
Working angle: Current data-agent evaluation is mostly outcome-only (execution accuracy / exact match on Spider etc.), compute-unfair, and run-once. Gaps to review:
1. Compute-fair comparison: multi-agent vs single-agent at equal token/cost budgets for Text-to-SQL / analytics
2. Procedure-aware scoring: schema exploration, tool use, safe SQL, error recovery — not just final answer
3. Reliability across repeated runs: non-determinism, variance, pass^k rather than pass@1
4. Benchmark validity: label noise, contamination, semantic correctness vs execution match
Candidate contribution shapes: a reliability benchmark + protocol, a meta-evaluation of existing data-agent benchmarks, or a compute-fair re-evaluation study with a new trajectory-level metric suite.
Next step: daily briefings feed a scored gap matrix; then pick one contribution shape and draft lit review outline.
