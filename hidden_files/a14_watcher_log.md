# A14 fixes watcher log

## 2026-11-23 07:46 PST — mass stall, all 12 relaunched
- State at check: no pilot.py processes. fix_ctrl 23/25/23/25 of 60; abl_A0 76/78/83/74 of 200; abl_A1 77/79/78/6 of 200. Ledger $16.0373 / $24 cap.
- 11 workers died ~15:42 UTC: OpenAI helper rc=1 traceback at `call("POST", "/v1/chat/completions")` (stderr truncated in log). Helper healthy on immediate re-probe (`models` list OK) → transient shared failure.
- abl_A1_run4 died ~15:33 UTC: torn-read race on pilot_spend.json (ledger_load without flock vs concurrent non-atomic write_text) → JSONDecodeError.
- Fix: pilot_lib.py ledger_charge now writes pilot_spend.tmp + os.replace (atomic). Concurrency-tested in /tmp (4 writers/4 readers, 0 torn reads).
- Relaunched all 12 with exact original commands at 07:49 PST; verified alive + traces growing; ledger $16.0611. No message to Mohan.
