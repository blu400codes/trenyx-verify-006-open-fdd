# Attack plan — bbartling/open-fdd (BLIND pre-registration) — audit 004/100

**Written from public claims ONLY.** Sources frozen beside this file: TARGET.txt, README-as-read.md,
ATTRIBUTION.txt. No implementation file has been opened. Hash and anchor before any read.

Language: Rust (central) + React (web) + Python (pandas cookbook) — Rust counts toward the new-language
stratum. Domain: building analytics / HVAC fault detection & diagnostics (FDD) — new to the corpus.
Attribution at freeze: 142 AI-trailer lines/100 commits, Cursor-dominant (Claude 0). Suite: 309 test paths.
The README scopes the product explicitly: LAN/VPN/OT networks only, "not intended for the public
internet," internet hardening "targets Fall 2026," Railway is "a lab/demo path, not a production-hardening
claim." Those scopings are binding on how security items are graded (see non-findings).

## 0. Contamination
blind-gate: PASS 2026-09-04 — absent from the ledger. Seen: README (frozen), metadata, trailer COUNTS.

## 1. The claims I am testing, and how I would falsify each

The heart of the project, by its own README, is a catalog of **62 fault-detection rules "published in two
parity-matched flavors"** — SQL and Pandas, "the same rules." Two hand-maintained copies of 62 rules is
the single most falsifiable thing here, and the failure mode that matters for FDD is the *silent* one: a
fault that should fire and doesn't.

| # | claim (quoted from README-as-read.md) | how I would falsify it |
|---|---|---|
| C1 | "62 fault-detection rules, published in two parity-matched flavors ... the same rules" (SQL and Pandas) | read the two catalogs side by side: same thresholds, same units, same time windows/rolling periods, same NULL/missing handling, same comparison direction. Any divergence = the same fault answered differently by flavor |
| C2 | "runs fault detection with DataFusion SQL over an Apache Arrow historian" on "live or historical OT/CSV data" | data-quality handling: on a missing point, a NaN/null sample, a unit mismatch (°F/°C, CFM), a stale timestamp, or a timezone edge — does a rule fire (false positive) or silently not fire (false negative)? |
| C3 | the documented run uses `OPENFDD_JWT_SECRET='change-me'` and `OPENFDD_ADMIN_PASSWORD='change-me'`; "SPA talks to central /api only" | does central refuse or loudly warn on the placeholder secret, or silently accept it? If the env var is UNSET, is there a hardcoded fallback secret shared by every install? |
| C4 | "Optional slim MCP stdio sidecar → central API" with `OPENFDD_MCP_TOKEN` | token scope: does the MCP token confer full admin? Can any MCP tool mutate rules/config or reach the shell? |
| C5 | "MQTTS ingest, Feather historian" | ingest authentication: can an unauthenticated client publish points into the historian (poisoned data → fabricated or suppressed faults)? Does central validate point identity? |
| C6 | "62 cookbook rules (66 SQL registry ids)" | claim-vs-code: registry/catalog consistency — INFO/LOW |
| C7 | "Version 3.3.4 on master · PyPI 4.4.2" | version skew between repo and package — INFO |
| C8 | "Update a running stack (pull, backup, rollback if health fails)" | does the rollback actually restore on a failed health check, or can a failed update leave the historian in a broken/lost state? |

**Pre-committed severity stance.** C1 divergence MEDIUM-HIGH (a correctness break of the central
promise). C2 silent non-detection MEDIUM-HIGH; false-positive-on-bad-data MEDIUM. C5 unauthenticated
ingest HIGH in principle, but graded MEDIUM/`design_note` where the README's LAN-only scoping covers it —
unless the code contradicts its own docs. C3: `design_note` for a documented change-me default; LOW-MEDIUM
only if code silently substitutes a hardcoded fallback secret when unset. C4 MEDIUM. C8 MEDIUM if data
loss. C6/C7 INFO.

**Pre-committed non-findings.** Internet-facing hardening (documented as future, Fall 2026); the Railway
lab path (documented as non-production); 'change-me' placeholders the docs tell you to change; the
"coming soon" OT edge (`n_a`, it does not exist); the PyPI library being a separate thing from the
runtime; anything not read.

**Predicted outcome, recorded before reading (falsifiable):** the most likely real findings are
correctness, not security — C1 (two hand-maintained rule sets drifting: a threshold, a unit, or a window
that differs between SQL and Pandas) and C2 (NaN/missing samples causing a rule to silently not fire).
Security items will mostly land as `design_note` because the README scopes deployment honestly. Expect
1–2 correctness findings and 0 security escalations. If the rule sets are truly parity-matched, that is a
notable result and the record says so.

## 2-4. Invariants and planted-defect set (catalogue skeleton follows)

# Attack plan — <engagement id> (pre-registered)

**Domain:** all  ·  **Catalogue:** faults v3

## 2. Invariants (from catalogue; add target-specific ones)

- **I1 no lookahead.** a decision for period t uses data ≤ t; execution no earlier than the next observable price; valuation surfaces use the current date
- **I2 accounting identity.** balance never negative without explicit credit; total = cash + Σ position × mark; costs charged on every event in the adverse direction; nothing double-counted at boundaries
- **I3 exits fire.** an exit/cancel/revoke signal closes the thing; ratcheting limits move one way only
- **I4 gates gate.** a computed halt/deny verdict is honored by the action path; limits use current state, not stale
- **I5 safe boundary.** safe mode is the default and unrecognized input fails closed; side-effecting requests are idempotent under retry; errors surface, never success-shaped
- **I6 determinism.** same inputs → identical outputs; seeds recorded
- **I7 config rejection.** invalid or contradictory configuration is rejected, not defaulted

## 4. Planted-defect set

### 4a. From the catalogue (ranked by historical escape rate — plant the ones that fit)

| id | defect | class | how to plant | escape rate so far |
|---|---|---|---|---|
| F-LOOKAHEAD-1BAR | signal for bar t sees bar t | lookahead | relax the strict '<' on the strategy's data window to '<=' | 100% (1) |
| F-TRAIL-ANCHOR | trailing stop anchored to entry, not running peak | exit-logic | replace peak with entry_price in the stop formula | 100% (1) |
| F-DEFAULT-FAILOPEN | safe/unsafe default flipped (paper→live, sandbox→prod, deny→allow) | config | flip the default on the boolean parser or constructor | 100% (2) |
| F-STALE-MARK | valuation uses previous period's price | accounting | date - 1 in the mark lookup | 100% (2) |
| F-BOUNDARY-DOUBLE | last element of the window processed twice | iteration | append the last session to the loop list | 100% (1) |
| F-FUTURE-MARK-ACCOUNT | account/equity surface marked at end-of-data prices (no date passed) | lookahead | drop the date argument on the valuation call | 100% (1) |
| F-FABRICATED-DATA | missing input silently replaced by synthetic data in a production path | silent-failure | n/a — usually native; test for it | 100% (1) |
| F-UNSEEDED-RNG | randomness unseeded by default; driver never seeds | determinism | n/a — usually native; test for it | 100% (1) |
| F-NO-CANONICAL-FORM | policy/rule matching on raw input; no canonical form (case, whitespace, quoting) | matching | n/a — usually native; test with a whitespace variant | 100% (1) |
| F-AUDIT-SILENT-LOSS | audit/ledger write failure swallowed with no log or counter | error-handling | drop the error log on the deferred write | 100% (1) |
| F-TRUNCATE-BEFORE-EVAL | input truncated before rule evaluation (suffix invisible) | matching | slice the command to N bytes before building the request | 100% (1) |
| F-OUTAGE-POSTURE-SPLIT | two implementations of the outage rule disagree (one fails open) | risk-gate | n/a — found by blind test; compare every outage path | 100% (1) |
| F-DENY-ENCODED-AS-NOOP | a deny for one client/agent type encoded as a no-op response | risk-gate | n/a — test each encoder with a deny on every blocking event | 100% (1) |
| F-LOCK-PARTIAL | serialization lock taken for only some of the claimed resources | concurrency | lock loop over resourceIds.slice(0,1) | 100% (1) |
| F-IDEMPOTENCY-KEY-IGNORED | idempotency key stored but a replay is not refused | idempotency | duplicate check threshold >0 → >1 | 100% (1) |
| F-TZ-DATEONLY-UTC-SHIFT | date-only string parsed as UTC midnight → previous calendar day west of UTC | time | n/a — usually native; test for it | 100% (1) |
| F-DEFAULT-ACCESS-OPEN | generated collections ship with framework-default access (any authenticated user reads/mutates all rows) | authz | n/a — native; assess with access on and a second user | 100% (1) |
| F-DELETE-BYPASSES-POLICY | a business rule enforced on the status path is skipped by document deletion | exit-logic | n/a — native; test delete against the policy | 100% (1) |
| F-NONINTEGER-UNITS | fractional quantity accepted where units must be integers | boundary | n/a — native; submit 1.5 | 100% (1) |
| F-WIRING-UNFENCED | fixed issue's COMPONENT is unit-tested but the DISPATCH wiring is not | test-gap | disable the dispatch branch that selects the fixed component | 100% (1) |
| F-GUARD-NONCONFORMING | guard against nonconforming input has no test feeding that input (compute the guard's ACTUAL failure mode; never inherit the code comment's claim) | test-gap | neuter a reject-on-missing-field guard | 100% (1) |
| F-LOG-BATCH-VS-SINGLE | batch-level loss log untested where single-record log is asserted | observability | neuter the batch-summary critical condition | 100% (1) |
| F-TIEBREAK-NONE | record missing the tie-breaker field — hard TypeError crash on any nullable tie-breaker column (maintainer-reproduced) | boundary | invert the None ordering in tie-breaker compare | 100% (1) |
| F-STRCMP-DATETIME | datetime strings compared lexically — SILENT persisted-watermark corruption, permanent record skips; realistic trigger '.000Z' vs 'Z' (ord('.')<ord('Z')). Maintainer-reproduced, top of its set | boundary | disable the ISO-parse branch | 100% (1) |
| F-NONE-ID-ACCEPTED | ref missing its id accepted; engine reads None-named resource | config | neuter the None-id guard | 100% (1) |
| F-FIXTURE-MONOCULTURE | every fixture supplies well-formed homogeneous input; no test constructs the input that separates a guard from its happy path (positive budget / populated tie-breaker / same-precision Z) | test-gap | plant a guard-neuter or fallback-path inversion; escapes because no fixture reaches the branch — fix is adversarial fixture data | 100% (1) |
| F-UNREACHABLE-GUARD | defensive branch that cannot fire through the validated entry path (upstream validator always populates the field) | test-gap | neuter the guard; escapes trivially — grade INFO, recommend a delete-protection test only | 100% (1) |
| F-DEFAULT-LIVE-ON-OMISSION | a safety/paper/dry-run flag defaults to LIVE when its config key is merely omitted (`?? false`), against the project's own docs or sibling defaults | risk-gate | plant: flip a `dryRun ?? true` to `?? false`, or delete the key from a fixture config and assert live is NOT reached | 100% (2) |
| F-LLM-PATH-AUTHZ-BYPASS | the chat/LLM/agent surface dispatches the same tool table as REST/MCP without the role gate those surfaces apply; enforcement is prompt text | authz | plant: remove the role check from the agent dispatch path only; assert a viewer-role call reaches a write handler | 100% (1) |
| F-SIBLING-ROUTE-AUTHZ | one route in a family lacks the ownership/role middleware its siblings carry (copy-paste asymmetry); the handler returns PII with no ownership check | authz | plant: drop requireStaff/ownership from one GET /:id route; assert cross-customer read is refused | 100% (1) |
| F-TOLERANT-ASSERTION | a money/cents assertion allows a percentage tolerance band, so a materially wrong split passes (vacuous assertion; cousin of fixture monoculture) | test-gap | plant: introduce a 1-cent truncation error; the tolerant test stays green | 100% (1) |
| F-SUITE-MOCKS-DB | the suite mocks the persistence layer entirely, so conflict/atomicity/authz defects at the DB boundary are structurally uncatchable | test-gap | plant: any transaction/conflict-check removal; the suite cannot see it | 100% (1) |
| F-DEDUP-FAILS-OPEN | a fail-closed guard's RESPONSE is asserted but its EFFECT is not: the error branch still writes 503 while processing continues | idempotency | change only the return value of a fail-closed branch, leaving the response write intact — a status-only assertion stays green | 100% (1) |
| F-EXPIRY-SKIPS-LEDGER | a balance is zeroed but its compensating ledger leg is never posted | accounting | invert the nil-guard so the ledger post is skipped | 100% (1) |
| F-CASH-UNBOUNDED | spend not bounded by available balance | accounting | remove/skip the balance check before debit | 50% (2) |
| F-KILLSWITCH-UNREAD | halt verdict computed but ignored by the action path | risk-gate | call the check, discard its result, return allowed | 50% (2) |
| F-RETRY-NOIDEMPOTENCY | retry resubmits a side-effecting request without an idempotency key | external-io | remove client_order_id / idempotency header on the retried call | 50% (2) |
| F-ATTRIBUTION-FROM-INPUT | identity in the record taken from caller-supplied fields | audit | n/a — assess where identity comes from | 50% (2) |
| F-COST-SIGN | slippage/fee sign flipped (costs improve the price) | accounting | swap +/- on the adverse-direction fill adjustment | 0% (1) |
| F-FEE-ONESIDED | fee charged on one side only | accounting | guard the fee line with side=='buy' | 0% (1) |
| F-EXIT-IGNORED | exit signal ignored (position never closes on its own signal) | exit-logic | invert the position-exists condition on the sell branch | 0% (2) |
| F-BREAKER-INVERTED | loss-limit comparison inverted (trips when safe) | risk-gate | >= to <= on the limit check | 0% (2) |
| F-EXCEPTION-SWALLOWED | failure swallowed into a success-shaped return | error-handling | replace raise with return None in the except block | 0% (3) |
| F-LIMIT-OFFBYONE | max-count limit off by one | risk-gate | >= to > on the count check | 0% (3) |
| F-ORPHAN-ON-TIMEOUT | timeout returns 'not sent' while the request thread keeps running | external-io | n/a — test for reconciliation after timeout | 0% (1) |
| F-CONFIG-PARTIAL-ACCEPT | invalid policy/config entry skipped with a warning; remaining applied | config | replace the validation error return with `continue` | 0% (2) |
| F-PRECEDENCE-FLIP | deny/forbid precedence broken (any permit wins) | auth | force allow when any determining permit exists | 0% (1) |
| F-OBSERVE-EVIDENCE-CORRUPT | dry-run/observe evidence records the wrong verdict | audit | write 'would allow' for a deny in the observe reason | 0% (1) |
| F-BUFFER-ONESIDED | buffer applied on one side only (before-buffer dropped) | overlap | computeBlockedWindow: effectiveStart = startTime (ignore bufferBefore) | 0% (1) |
| F-TZ-RANGE-EXCLUSIVE | inclusive date range treated exclusive at the end (last day bookable) | time | target <= end → target < end | 0% (1) |
| F-HOLD-CONVERT-DOUBLE | a consumed hold is not released after conversion | idempotency | skip the hold delete after a successful booking | 0% (1) |
| F-ACTIVE-ITEMS-UNCHECKED | inactive resource referenced only via items[] accepted | gate | validateActive iterates items.slice(0,1) | 0% (1) |
| F-TOKEN-LEAK | server-generated secret echoed in the API response | authz | return the raw reservation instead of the token-stripped copy | 0% (1) |
| F-CAPACITY-MODE-IGNORED | per-guest capacity counted as one unit per booking | capacity | units = 1 regardless of capacityMode | 0% (1) |
| F-CKPT-ON-FAILURE | failed batch advances the checkpoint | checkpoint | accept/emit a committed cursor on a failure disposition or ack | 0% (1) |
| F-NOACK-SUCCESS | send that reached no ack reported successful | checkpoint | flip success on the no-ack verdict builder | 0% (1) |
| F-DLQ-PHANTOM | lost failed-record reported/counted as stored | error-handling | return True / increment count on the failed capture path | 0% (1) |
| F-DIGEST-KEYORDER | row-identity digest depends on key order | idempotency | sort_keys=True -> False in the canonical dump | 0% (1) |
| F-UNSAFE-CAST | lossy numeric narrowing silently truncates | types | safe=True -> False on the boundary cast | 0% (1) |
| F-DECIMAL-FLATTEN | Decimal flattened to float in a durable record | types | drop the type tag / str() -> float() on encode | 0% (1) |
| F-TZ-DROP | aware datetime coerced naive across a round-trip | types | strip tzinfo in the serializer | 0% (1) |
| F-MALFORMED-TAG-PASSTHRU | malformed tagged value passes through instead of failing loud | error-handling | raise -> return raw on the tag guard | 0% (1) |
| F-MANIFEST-OPTIONAL | bundle without its required manifest accepted | config | empty the REQUIRED_FILES tuple | 0% (1) |
| F-NOTREADY-WRITE | not-ready sink no longer rejects; write attempted | risk-gate | bypass the readiness guard | 0% (1) |
| F-REVIEW-FILTER-INVERT | failed-record review filter inverted (your records hidden) | observability | != -> == on the filter | 0% (1) |
| F-TRIALBAL-UNCHECKED | trial balance reports Balanced regardless of the debit/credit totals | accounting | replace the totals comparison with a literal true | 0% (1) |
| F-ROUNDING-REMAINDER | proration/split truncates cents instead of reconciling the remainder | accounting | narrow a ratio slightly so int64() truncation loses a cent | 0% (1) |
| F-STUB-FACTOR-DROPPED | a prorated stub period is billed at the full rate (the factor is ignored) | accounting | neutralise the period factor while keeping it referenced | 0% (1) |
| F-STATE-ILLEGAL-TRANSITION | an operation illegal for the current status is accepted | state-machine | disable the status guard on a lifecycle transition | 0% (1) |
| F-CLOSE-GATE-BYPASS | period close no longer blocks on an unbalanced book or open discrepancies | accounting | neutralise the blocker condition in the close gate | 0% (1) |
| F-SIGN-FLIP | normal-balance sign inverted for one account class (credit-normal read as debit-normal) | accounting | swap the operands in the normal-balance computation | 0% (1) |
| F-ABNORMAL-UNFLAGGED | the abnormal-sign flag is never raised, hiding a posting bug from the report | observability | force the abnormal flag to false | 0% (1) |
| F-COMMITMENT-SHORTFALL-UNTAXED | a generated true-up/shortfall line is added to the subtotal but its tax is dropped | accounting | zero the tax contribution of the generated line | 0% (1) |
| F-COMMITMENT-OFFBYONE | a floor/commitment comparison uses <= so an exactly-at-floor amount generates a zero-value line | accounting | flip < to <= on the floor check | 0% (1) |
| F-DEDUP-DUPLICATE-PROCESSED | an event already recorded as processed is processed again | idempotency | disable the already-processed short-circuit | 0% (1) |
| F-RECOVERY-UNQUALIFIED | a metric counts unqualified events (a first-try payment recorded as a dunning recovery) | observability | widen the qualification threshold by one | 0% (1) |

### 4b. NOVEL — at least 3 faults NOT in the catalogue, chosen for this target (fill before hashing)

| N01 | <defect> | <class> | <how to plant> | new |
| N02 | <defect> | <class> | <how to plant> | new |
| N03 | <defect> | <class> | <how to plant> | new |

Detection rule, disclosure, environment, timebox: copy from the runbook template. Hash this file BEFORE reading the target's implementation; email the hash to the client.
