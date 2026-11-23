# Dataset Suitability & Credibility Assessment — Paper #1 Pilot
Date of assessment: 2026-11-22 (America/Los_Angeles)
Standard applied: user's standing verification rule — official sources only, every claim grounded in a page/file actually read, unverified items labelled, discrepancies reported, no guessing.
Downloads location: `~/workspace/goals/research-papers-on-agentic-ai-for-data-ai/hidden_files/datasets/`

## ⚠️ Processing blocker (read first)
The shell step that cloned the Spider2 and Jin repositories and started the BIRD download triggered an execution-risk check on the delivered content. Under that check I must **not** extract/unpack archives, run any code from the cloned repos, run git inside them, or execute SQL against downloaded databases until the user explicitly approves processing that content. Consequences for this assessment:
- BIRD `dev.zip` was downloaded but **not extracted**; its contents were **not** inventoried file-by-file.
- **No SHA256 hashes were computed** and **no SQLite database was opened / no gold query was executed** in this pass. Local executability findings below therefore rest on documented format (official README/HF records + file listings read as plain text), not on a live execution test. That test remains outstanding.
- File/task counts below come from official statements and from plain-text reading of the task files (line counts visible in the reader), not from scripted counting.

## 1. BIRD Dev (classic dev set)
- **Source verified? YES.** Official site https://bird-bench.github.io/ (read 2026-11-22) links the dev package at `https://bird-bench.oss-cn-beijing.aliyuncs.com/dev.zip` (link found in the official site's own HTML). Paper: Li et al., NeurIPS 2023 Datasets & Benchmarks, arXiv:2305.03111. Maintainer: BIRD team (HKU), per official GitHub org `bird-bench`.
- **Downloaded? PARTIALLY.** `dev.zip` downloaded from the official OSS URL above into `hidden_files/datasets/bird/dev.zip` (HTTP 200 from AliyunOSS; server Last-Modified 2024-06-29, consistent with the `dev_20240627` version referenced by official tooling). Not extracted (blocker above). Exact byte size and SHA256: **could not verify** in this pass.
- **Version warning (discrepancy that matters):** There are now (at least) two different "BIRD Dev" artefacts:
  1. The classic dev (`dev_20240627`, the zip above): 1,534 tasks — count from the Jin et al. paper ("100 of the 1,534 BIRD Dev examples") and the Hugging Face viewer for the newer dataset also showing ~1.53k rows / 11 databases.
  2. A newer Hugging Face dataset `birdsql/bird_sql_dev_20251106` (update dated 2025-11-06, licence cc-by-sa-4.0 per its HF card data) with revised annotations, plus a "New Dev" leaderboard column on the official site, and a separately corrected BIRD Mini-Dev (official `bird-bench/mini_dev` README: "2026 update… corrected annotations", 2026-10-28).
  Jin et al.'s corrections were made against the **classic** dev/mini-dev, so classic dev is the version that joins to the corrected labels. Mixing versions would silently break the L1 label policy.
- **Format:** Documented (official tooling and dataset structure) as `dev.json` (question, evidence, gold SQL, difficulty, db_id) + `dev_databases/<db>/<db>.sqlite` — i.e. **local SQLite files with gold SQL included**. Not re-verified by extraction in this pass (blocker).
- **Counts (official/paper statements):** 1,534 dev tasks, 11 databases. Tasks per database ≈ 139 on average — ample for failure-class clusters and session ordering.
- **Licence:** cc-by-sa-4.0 on the official HF dev dataset card (verified via HF API card data for `birdsql/bird_sql_dev_20251106`). A licence file inside the classic zip was **not inspected** (not extracted) — licence for the classic zip specifically is therefore labelled **not directly verified**, though distribution is public and research use is the dataset's stated purpose.
- **Access friction:** None for the zip (direct public link). The newer HF datasets are also public; some related BIRD artefacts (LiveSQLBench / BIRD-Interact ground truth) are email-gated, but those are not needed here.
- **Suitability vs pilot criteria:**
  - Local executability: SQLite by design — fits 4 runs × 5 arms cheaply, *pending the outstanding live execution test*.
  - Clusters: 11 databases × ~139 tasks — strongest of the three candidates.
  - Drift applicability: SQLite files can be copied and modified per drift operator (silent and overt) — fits, *pending execution test*.
  - Label credibility: **weakest point.** Jin et al. (VLDB 2026, arXiv:2601.08778v3) measured a 52.8% annotation-error rate in BIRD Mini-Dev; only a 100-task corrected Dev subset exists from Jin (see §3). The rest of Dev is unaudited (L2), not proven clean.
  - Realism: real-world databases with external-knowledge evidence fields — good.
- **Verdict: CONDITIONAL GO** — the pilot's primary substrate, conditional on (a) the user approving content processing so the zip can be extracted and a live SQLite/gold-query test run, and (b) pinning the classic `dev_20240627` version for join compatibility with Jin corrections, with the 2025-11-06 HF revision explicitly *not* used.

## 2. Spider 2.0 (Snow, Lite, and the local/SQLite subset of Lite)
- **Source verified? YES.** Official repo https://github.com/xlang-ai/Spider2 (XLANG Lab; read 2026-11-22; current commit `cafb867313aab4e674652054198f383cf4018943` as shown on the repo page), ICLR 2025 Oral, arXiv:2411.07763, site spider2-sql.github.io linked from the repo. Repo licence: **MIT** (repo page).
- **Downloaded? PARTIALLY.** The full repo was cloned (shallow) into `hidden_files/datasets/spider2/Spider2` before the processing blocker was raised; after the blocker, the repo was only read as plain text (no code executed, no git commands inside it). The Spider2-Lite **local SQLite database package** (a Google Drive download linked from the repo's own `spider2-lite/README.md`) was **not** downloaded in this pass.
- **What the clone actually contains (plain-text inventory):**
  - `spider2-snow/spider2-snow.jsonl` — task file; reader shows 548 lines total (5 read + "543 more"). Fields seen: `instance_id` (e.g. `sf_bq011`), `instruction`, `db_id`, `external_knowledge`. **Discrepancy:** the repo README's official table says **547** Snow examples; the file as cloned shows 548 lines. One line may be blank/trailing — exact task count **could not be verified** without scripted counting (blocked).
  - `spider2-lite/spider2-lite.jsonl` — same pattern: 548 lines shown vs **547** in the official table. Fields: `instance_id`, `db`, `question`, `external_knowledge`.
  - Gold: `evaluation_suite/gold/sql/*.sql` present for a **subset only** (repo README: "only a small amount of gold SQL is available" / gold answers released for "about partial examples"), plus `gold/exec_result/*.csv` gold result tables and `spider2lite_eval.jsonl` / `spider2snow_eval.jsonl`. So gold SQL is **partial by design**.
  - Database resources in-repo are **schema metadata folders**, not data: `spider2-lite/resource/databases/{bigquery,snowflake,sqlite}/…` — e.g. 31 SQLite schema folders (AdventureWorks, chinook, Pagila, northwind, …) containing metadata, while the actual `.sqlite` data files are the separate Drive download above ("put all the `.sqlite` files into `spider2-lite/resource/databases/spider2-localdb`").
- **Access friction (major, from the repo README itself):**
  - Snow: requires a **Snowflake account via a Google Form access request**; databases are cloud-hosted, not local files. Repo news (2026-08-12) even reports the Snowflake evaluation account was suspended/disrupted.
  - Lite full set: requires the user's **own BigQuery and Snowflake accounts/credentials** (214 BigQuery + 198 Snowflake of the 547); "some cost incurred" per the official table. Only **135 of 547 Lite tasks are SQLite/local**.
  - Snow gold SQL is Snowflake-dialect and cannot be executed against a local SQLite copy without translation.
- **Counts (official repo table):** Snow 547, Lite 547 (BigQuery 214 / Snowflake 198 / SQLite 135), DBT 68. Multiple tasks share the same underlying databases (e.g. many `sf_bq*` tasks on GA360/GA4), so clustering is possible *within* the local subset, but the local subset is only ~135 tasks before any exclusions.
- **Label credibility:** Jin et al. audited Spider 2.0-Snow and reported a **62.8%** annotation-error rate — but released **no corrected Spider labels** (their corrections are BIRD-only, §3). Snow labels are therefore disputed with no corrected replacement: under the substrate spec's own policy, audited-and-flagged Snow items would be L3 (exclude), and the unflagged remainder cannot be distinguished without the audit data.
- **Suitability:** Local executability **fails** for Snow (cloud-only) and for most of Lite; drift operators cannot be applied to a hosted Snowflake/BigQuery database the pilot doesn't own; repeated runs (4 × 5 arms) would incur cloud cost, queueing, and credential risk. The Lite **local/SQLite subset (~135 tasks)** is the only part that fits the sandbox model, and its databases are a separate download not yet obtained.
- **Verdict: NO-GO for Spider 2.0-Snow as a pilot substrate. CONDITIONAL (secondary only) for the Spider2-Lite local/SQLite subset** — usable as a small supplementary pool *if* the local DB package is downloaded and passes the execution test; not usable as a primary substrate, and its labels carry the unaudited-remainder problem from the Jin Snow audit.

## 3. Jin et al. corrected labels (uiuc-kang-lab/text_to_sql_benchmarks)
- **Source verified? YES.** Repo https://github.com/uiuc-kang-lab/text_to_sql_benchmarks (read 2026-11-22; repo page shows current commit `fe766045c55b6875a43b30e9ac7683df5582f8cf`, licence **CC-BY-SA-4.0**). Paper: Jin, Choi, Zhu & Kang, "Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards", arXiv:2601.08778v3 (submitted 13 Jan 2026, rev. 19 Jan 2026); repo states the paper was **accepted to VLDB 2026** (news item 2025-12-15) and a companion analysis to CIDR 2026. Authors: University of Illinois (UIUC).
- **Downloaded? YES (repo clone, plain-text read only after the blocker).** Location: `hidden_files/datasets/jin_corrected/text_to_sql_benchmarks`.
- **What the corrections actually are (this corrects an assumption in the substrate spec):**
  1. **The paper's own correction set is 100 BIRD Dev examples** — "We randomly sampled 100 of the 1,534 BIRD Dev examples and… manually corrected all identified errors" (paper, via arXiv HTML) and repo README: "We corrected 100 examples sampled from the BIRD Dev set". The corrected/original Dev subsets for re-running agents are distributed via a **Google Drive** link in the repo README (not inside the git repo itself); that Drive file was **not** downloaded in this pass.
  2. **In-repo `data/` folder** contains two later files (repo news, 2026-02-21): `arcwise_plat_sql_only_with_diff.json` and `arcwise_plat_full_with_diff.json` — Arcwise-based corrections to **BIRD Mini-Dev** (SQL-only variant, and a variant also resolving question/evidence ambiguities and schema issues), plus `data/schemas/` (database-description CSVs for the 11 BIRD databases: california_schools, card_games, codebase_community, debit_card_specializing, european_football_2, financial, formula_1, student_club, superhero, thrombosis_prediction, toxicology — matching BIRD Dev's 11 databases).
  3. **No corrected Spider 2.0-Snow labels exist in this repo.** Spider was audited (error rate reported) but not corrected; the repo's Spider setup only points back to requesting Snowflake access via Spider2.
- **Join key:** BIRD `question_id` + `db_id`. Seen directly in the Arcwise file's records, e.g. `{"question_id": "1471", "db_id": "debit_card_specializing", "question": …, "evidence": …, "SQL": …}` — the same fields as BIRD Dev/Mini-Dev, so the join to classic BIRD tasks is by `question_id` (with `db_id` as a check). Note `question_id` is a **string** in this file.
- **How many corrected items / coverage of the pilot pool:**
  - Jin 100-task corrected Dev subset: 100 / 1,534 ≈ **6.5% of BIRD Dev**. If the pilot drew tasks only from this subset, the corrected pool (100) barely exceeds the pilot's total need (~110 tasks incl. dry run) *before* clustering, distance, and drift constraints — in practice too small to be the sole substrate.
  - Arcwise-Plat Mini-Dev files: exact entry count **could not be verified** in this pass (scripted counting blocked; the SQL-only file runs to ~3,572 lines of JSON at ~7 lines/entry as read, which is *consistent with* the 500-item Mini-Dev but is **not** a verified count — do not quote a number from this). Coverage, whatever the exact count, is over **Mini-Dev**, which overlaps BIRD Dev.
  - Spider coverage: **0 corrected items.**
  - Net L1 (corrected-available) coverage of the combined BIRD+Spider candidate pool is therefore **small and BIRD-only**; most pilot tasks will necessarily be L2 (original, unaudited) unless the pilot shrinks to the corrected subsets.
- **Credibility of the corrections themselves:** Expert-adjudicated (three-stage audit: SAR-Agent flagging → human SQL experts adjudicating → pattern-based manual extension; final decisions by human experts, per the paper). VLDB 2026 acceptance adds peer-review weight. Caveat: the Arcwise-Plat files build on a third party's (Arcwise's) corrections with Jin et al.'s additional fixes — provenance is two-layer and should be cited as such.
- **Verdict: GO as the pilot's label-quality layer for BIRD (with the coverage caveat above); NO-GO / does-not-exist for Spider.** The substrate spec's L1/L2/L3 policy survives, but its implicit assumption that corrected labels would cover a large share of the pool does **not** — plan on L2 being the majority label status, or restrict the primary metric to the corrected subsets and accept a smaller pool.

## 4. Bottom line for the pilot
| Dataset | Source verified | Downloaded | Format | Verdict |
|---|---|---|---|---|
| BIRD Dev (classic, dev_20240627) | Yes — official site + OSS link | Zip only; not extracted (blocker) | dev.json + per-DB SQLite (documented) | **CONDITIONAL GO** — primary substrate |
| BIRD Dev (HF 2025-11-06 revision) | Yes — official HF org | No | HF JSON | Not used — version incompatible with Jin joins |
| Spider 2.0-Snow | Yes — official repo (MIT) | Repo cloned (text-read only) | Cloud Snowflake; partial gold SQL | **NO-GO** — cloud-only, no corrected labels |
| Spider 2.0-Lite, local/SQLite subset (~135) | Yes — official repo | No — local DB package not downloaded | SQLite (separate Drive package) | **CONDITIONAL** — small supplementary pool only |
| Spider 2.0-Lite, BigQuery/Snowflake parts (412) | Yes | No (requires user cloud accounts) | Cloud | **NO-GO** for this pilot |
| Jin et al. corrections | Yes — official repo (CC-BY-SA-4.0), VLDB 2026 | Repo cloned; Drive subset not downloaded | JSON keyed by question_id | **GO for BIRD only**; none exist for Spider |

**Recommended substrate (grounded in the above):** classic BIRD Dev as the sole primary substrate; Jin 100-task corrected subset + Arcwise-Plat Mini-Dev corrections as the L1 layer; everything else in BIRD Dev as L2 with dual reporting as frozen; Spider dropped from the pilot's primary pool (Snow NO-GO; Lite-local only if a supplementary pool is later needed). This deviates from Substrate Spec §1/§4, which assumed a BIRD + Spider-Snow pool — that assumption should be amended before the manifest is built.

## 5. Outstanding / could not verify
1. **Live executability test** (open a BIRD SQLite DB, run a gold query) — blocked pending user approval to process downloaded content. Also blocked by the same check: extracting `dev.zip`, computing SHA256 hashes and exact byte sizes, and scripted counts.
2. Exact entry count of the two Arcwise-Plat files, and whether their question_ids are a subset of classic BIRD Dev's 1,534.
3. Exact task count of `spider2-snow.jsonl` / `spider2-lite.jsonl` (548 lines read vs 547 officially stated).
4. Licence text inside the classic BIRD `dev.zip` (HF card licence cc-by-sa-4.0 verified only for the 2025-11-06 HF revision).
5. Contents/counts of the two Google Drive packages (Jin corrected Dev subsets; Spider2-Lite local SQLite DBs) — neither downloaded in this pass.
6. Whether the 2026-corrected official BIRD Mini-Dev (HF `birdsql/bird_mini_dev`) agrees with the Jin/Arcwise corrections — two independent correction efforts for overlapping items were identified; their agreement was not checked.

## Sources read for this assessment (2026-11-22)
- https://bird-bench.github.io/ (official site; dev.zip link in its HTML)
- https://huggingface.co/datasets/birdsql/bird_sql_dev_20251106 (official HF dataset page + HF API card data: licence cc-by-sa-4.0, data file `data/dev_20251106-00000-of-00001.json`, ~1.53k rows, 11 db_id values)
- https://github.com/bird-bench/mini_dev (official Mini-Dev README: 2026 corrected update, Mini-Dev V2 780 instances / 500 SELECT-only)
- https://github.com/xlang-ai/Spider2 (repo page + README + `spider2-lite/README.md` read from the clone as plain text; local-DB Drive link and cloud-account requirements stated there)
- https://github.com/uiuc-kang-lab/text_to_sql_benchmarks (repo page + README + `data/` listing + first records of `arcwise_plat_sql_only_with_diff.json` read as plain text)
- https://arxiv.org/abs/2601.08778 and https://arxiv.org/html/2601.08778v3 (Jin et al.: 52.8% / 62.8% error rates; 100 of 1,534 BIRD Dev examples corrected; repo URL)
- Spider 2.0 paper, arXiv:2411.07763 (Lite hosted on BigQuery/Snowflake/SQLite; Snow entirely on Snowflake)
