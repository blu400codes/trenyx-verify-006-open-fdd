# trenyx-verify-006 — open-fdd (pre-registered blind read; maintainer-fixed in 3.3.41)

Independent verification of `bbartling/open-fdd` @ `c20ab483fb9c0d7835b10f09e80fd40389194130` — a fault-detection
platform for building HVAC (Rust central service, DataFusion SQL rule registry, a Pandas cookbook of the same rules).
Factory audit 004 of the 100-read blind program; the sixth published engagement.

**Kind: blind read, static.** Nothing was built, run or planted. The attack plan in `00-preregistration/ATTACK-PLAN.md`
was written from the public README + metadata only, hashed, and anchored to Bitcoin BEFORE any implementation file was
read (`00-preregistration/ATTACK-PLAN.md.ots`, block 965548). Verify: `shasum -a 256 00-preregistration/ATTACK-PLAN.md`
must equal `plan_sha256` in `ANCHORS` (`5ee8a8db0d24d260e950216151f2f27d0933dd523ad1f3b19e6cccbd6ff38480`); `ots verify 00-preregistration/ATTACK-PLAN.md.ots`.

**Result.** Two correctness findings against the README's central "parity-matched flavors" claim (FC3 tolerance and
fan-signal priority; VAV-2 occupancy handling), both outside the project's own parity harness; no security finding —
auth, MQTT ingest and the MCP write gate held. Reported as a public issue (#875); the maintainer replied in five minutes,
shipped the fix the same day (3.3.41), and added both rules to the parity harness. Re-checked 2026-09-09: closed.

- `01-recon/` — the frozen README, target metadata and AI-attribution counts the plan was written from
- `04-findings/FINDINGS.md` — every pre-registered claim with its verdict and citation; `RECORD.json` is the full record
- `04-findings/MAINTAINER-NOTE-SENT.txt` (+ `.ots`) — the note as posted, hash-anchored
- `04-findings/MAINTAINER-RESPONSE.md` — the reply, the fix commit, the re-check

**What this read did not do.** It read the code at one commit. It never touched the project's test suite. The sealed
engagement plants catalogue defects one at a time and measures what the suite catches; verify-005 (analitiq-engine) shows
what that looks like.

---
*Method, other audits, and the live record: [trenyx.io/audits](https://trenyx.io/audits.html)*
