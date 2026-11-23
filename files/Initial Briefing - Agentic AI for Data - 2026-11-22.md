# Initial Briefing — AI Agentic Engineering for Data & AI
Date: 2026-11-22 | Prepared for: Mohan | Focus for Paper #1: Agent evaluation & reliability for data

This is the completed initial task. Its output is the explicit input for the research gap analysis that follows.

## 1. Venues / sources consolidated
- Preprints & reviews: arXiv (cs.AI, cs.CL, cs.LG, cs.DB, cs.SE), OpenReview, alphaXiv
- AI / ML: NeurIPS, ICML, ICLR, COLM, AAAI, IJCAI
- NLP: ACL, EMNLP, NAACL
- Data: SIGMOD, VLDB, ICDE, CIDR, KDD, EDBT
- Software engineering (agentic SE): ICSE, FSE, ASE

## 2. Recent trends identified (initial scan, 2026)
1. Long-horizon, stateful agents — shift from single-turn completion to persistent memory, structured / self-distilled memory, and context as a scarce, compacted resource.
2. Evaluation & accountability over raw capability — verifiability, policy enforcement, auditable reasoning, unfaithful work reports; benchmarks moving to workflow-level tasks.
3. Agentic software engineering maturing — SWE-bench Verified reported rising from 1.96% (Oct 2023) to ~78.4% (Apr 2026) in a 2026 synthesis; open problems: evaluation, governance, technical debt, skill redistribution, economics of attention.
4. Multi-agent scepticism under compute-fair comparison — at equal token budgets, multi-agent often loses to a single agent; lazy-agent behaviour in multi-agent reasoning (ICLR 2026 work).
5. Agents became trainable / environments as data — synthetic executable environments, RL / GRPO post-training for small agent models, recursive language models.
6. Data-specific: agent-first data systems — agents as the dominant data-system workload, "agentic speculation" (many redundant attempts; only ~10-20% unique sub-plans in a Text-to-SQL case study), calling for new query interfaces, processing, and agentic memory stores. Exploration-before-generation for Text-to-SQL, e.g. SQLAgent (ACL 2026 Findings): explore an unseen database first, then generate with a dual-agent setup.

## 3. Strengths & weaknesses — for both

### A. Single-agent data agents
Strengths:
- Lower cost, latency, and token use; simpler cost accounting
- Easier to reproduce, debug, and evaluate — one trajectory to audit
- Often strongest on execution accuracy / latency in open-model Text-to-SQL evaluations (e.g. Qwen2.5-Coder single-agent: 72.44% execution accuracy on Spider in the 2026 CMC study, lowest latency for Qwen3 single-agent)

Weaknesses:
- No independent cross-check — a wrong schema link or SQL error can cascade unchecked
- Limited parallel exploration on large, messy, or multi-database schemas
- Planning, execution, and verification compete for the same context / reasoning budget

### B. Multi-agent data agents
Strengths:
- Role specialisation (explorer, planner, generator, verifier/critic) can help on complex, multi-step tasks
- Parallel exploration and debate / verification can catch errors a single pass misses
- Hierarchical orchestrator-worker patterns show real gains on parallelisable tasks in industry systems

Weaknesses:
- Gains are not uniform — in the 2026 CMC Text-to-SQL study, multi-agent decomposition helped some model families, hurt others, and consistently increased latency and token consumption
- Coordination failures, out-of-sync agents, and lazy-agent collapse (one agent dominates, others contribute little)
- Harder reliability: more variance across runs, harder procedure traceability, harder to attribute failure to an agent / step
- Most published comparisons do not control compute — so reported gains may be a token-budget effect, not an architecture effect

## 4. Candidate gaps carried forward (to be tested, not assumed)
1. Compute-fair evaluation for data agents
2. Procedure-aware / trajectory-level evaluation for data pipelines
3. Reliability across repeated runs (pass^k, variance, non-determinism)
4. Benchmark validity (label noise, contamination, execution-match vs semantic correctness)
Plus, to check in gap analysis: schema/data drift, long-horizon data workflows, destructive-SQL safety, agent memory for data.

## 5. How the gap analysis must use this briefing
The gap analysis should: (a) verify / challenge each trend and strength-weakness claim above against primary 2024-2026 papers, (b) assess whether each candidate gap is still open or already closed, (c) score gaps for novelty, feasibility, benchmark availability, and venue fit, and (d) recommend 1-3 paper opportunities with problem formulation and experiment plan.
