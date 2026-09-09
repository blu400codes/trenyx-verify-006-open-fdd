# Maintainer response — bbartling/open-fdd#875

- **2026-09-08 18:47Z** — note posted as a public issue (`MAINTAINER-NOTE-SENT.txt`, hash anchored to Bitcoin:
  `MAINTAINER-NOTE-SENT.txt.ots`). No security content; a correctness note is public by design.
- **2026-09-08 18:52Z** — Ben Bartling replied within five minutes: "Thank you for the feedback. I will render a fix asap!
  I appreciate the testing! This is something I am trying to take on is rigorous stress testing between updates to flag
  things like this."
- **2026-09-08** — fix merged: `3bedbf9 fix(3.3.41): FC3 + VAV-2 cookbook parity (#875) (#877)`. Issue closed.
- **2026-09-09** — re-check against the merged fix (main `5aed663`, static): all four points closed.
  - FC3 tolerance is now the registry parameters `{{EPS_MAT}}` / `{{EPS_RAT}}` / `{{EPS_OAT}}`
    (`sql_rules/fc3_mat_high.sql:27`; `registry.yaml` defaults 1.15).
  - Fan-signal priority aligned: SQL `fan_status` then `fan_cmd` (`:12-13`); Pandas `_fan` `fan-status` then `fan-cmd`
    (`open_fdd/rules/cookbook_catalog.py:582-587`, comment cites #875).
  - VAV-2 SQL now treats a numeric `occ_mode <= 0.05` as unoccupied (`sql_rules/vav2_night_setback.sql:15-16`), matching
    `_unoccupied_mask` (`:928-938`).
  - `crates/fdd_rules/src/oracle_parity_test.rs` now covers `fc3_mat_high` and `vav2_night_setback` (+164 lines) — the fence.
  - README recount: 62 public rules / 68 SQL registry ids.
- **2026-09-09** — re-check comment posted on #875.

Status: **maintainer-fixed** in release 3.3.41.
