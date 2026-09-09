# verify-006 findings — bbartling/open-fdd @ c20ab483 (read 2026-09-04; factory audit 004)

**Kind: blind read, static.** No code was built or run; nothing was planted into the project's suite. The plan
(`00-preregistration/ATTACK-PLAN.md`) was written from the frozen README alone, hashed and anchored to Bitcoin before
any implementation file was opened. Every verdict below cites the line it rests on at the pinned commit. The `credit` column records exposure before the plan was frozen; there was none, so every row is `blind`.

**Verdict: two correctness findings against the README's central claim; no security finding.** Auth, MQTT ingest and
the MCP write gate held better than the README's LAN-only scoping requires.

| claim | verdict | severity (merits) | credit | where |
|---|---|---|---|---|
| C1 "62 rules … two parity-matched flavors … the same rules" | violated | medium-high | blind | FC3 tolerance is a literal `1.15` in SQL (`sql_rules/fc3_mat_high.sql:25-26`) but tunable `eps_mat/eps_rat/eps_oat` in Pandas (`open_fdd/rules/cookbook_catalog.py:536`, fc3 body); fan-signal priority differs (SQL `fan_status` then `fan_cmd`; Pandas `fan-cmd` then `fan-status`) |
| C2 silent non-detection on a plausible reading | violated | medium-high | blind | VAV-2: a fractional `occ_mode` in (0, 0.05] is occupied under SQL's exact-string list (`sql_rules/vav2_night_setback.sql:12`) and unoccupied under Pandas' numeric mask (`cookbook_catalog.py:927-938`) — the fault can fire in one flavor and not the other |
| C3 placeholder JWT secret | holds | — | blind | fail-closed off loopback, no hidden fallback (`services/central/src/auth.rs:108-186`) |
| C4 MCP token scope | design_note | — | blind | admin-equivalent by documented design (`mcp/src/bridge.rs`); writes gated by `OPENFDD_MCP_ALLOW_WRITES=1` plus per-call confirm (`mcp/src/gate.rs`) |
| C5 MQTT ingest auth | holds | — | blind | mTLS required, `allow_anonymous false` (`services/mqtt/mosquitto.conf`); per-edge cert + ACL issuance (`crates/openfdd_mqtt/src/provision.rs`) |
| C6 "66 SQL registry ids" | violated (info) | info | blind | `sql_rules/registry.yaml` has 68 `rule_id` entries |
| C8 update rollback | latent | medium | blind | rollback restores images and env, never the historian (`scripts/openfdd_maint_update_resume.sh`); no historian-mutating migration found, so latent |
| I3/I4 MCP write gate · I5 · I7 | holds | — | blind | `mcp/src/gate.rs` (unit-tested), `auth.rs:133-138` |

Both violated rows sit outside the project's own oracle-parity harness (`crates/fdd_rules/src/oracle_parity_test.rs`
covered ~22 of the 62 rule pairs at the pinned commit; FC3 and VAV-2 were not among them). That is the finding: not a
bug in either flavor, but two hand-maintained implementations answering the same rule id differently, with no fence.

Flagship spot-check (2026-09-04): headline CONFIRMED with one line correction (the SQL literal is at `:25-26`, not
`:11-13`); substance exact. Coverage 9 of 9 pre-registered rows reached a cited verdict.
