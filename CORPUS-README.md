# ZNS Fuzz Corpus Guide

Ready-to-paste custom corpora for `Fuzz Runner -> Custom fuzz cases`.

## How to use

1. Copy one JSON block from this file.
2. Paste into `Custom fuzz cases`.
3. Click `Save custom cases`.
4. Run `Run corpus`.

## Network-specific sections (single place)

Use this as the quick split so testnet and mainnet workflows stay separate.

### Testnet-only area (adversarial + negative testing)

- Network toggle: `testnet`
- Allowed scope:
  - input fuzzing
  - malformed JSON-RPC
  - concurrency and replay probes
  - payment-bypass negative tests (testnet only)
- Suggested Fuzz Runner parameters:
  - `Repetitions`: `3-5`
  - `Slow ms`: `800-1500`
- Use these sections:
  - `1) Input type and length corpus`
  - `2) Pagination and numeric boundary corpus`
  - `3) Malformed JSON-RPC corpus`
  - `4) Non-determinism probe corpus`
  - `5) Latency stress corpus`
  - `Testnet payment-bypass corpus`
  - `Payment-bypass test procedure (testnet)`
  - `Anti-bypass validation matrix`
  - `Pass/fail worksheet (copy per test session)`

### Mainnet-only area (read-only monitoring)

- Network toggle: `mainnet`
- Allowed scope:
  - health, integrity, and consistency monitoring only
  - nondeterminism and latency trend checks (read-only RPC)
- Suggested Fuzz Runner parameters:
  - `Repetitions`: `2-3`
  - `Slow ms`: `1000-2000` (baseline dependent)
- Use these sections:
  - `Mainnet-safe monitoring corpus (read-only)`
  - `Mainnet investigation checklist`
  - `Mainnet usage guidance`
  - `Pass/fail worksheet (copy per test session)`
- Never run adversarial/forged action simulation on mainnet.

## 1) Input type and length corpus

```json
[
  { "name": "resolve-empty", "mode": "rpc", "method": "resolve", "params": { "query": "" } },
  { "name": "resolve-null", "mode": "rpc", "method": "resolve", "params": { "query": null } },
  { "name": "resolve-number", "mode": "rpc", "method": "resolve", "params": { "query": 12345 } },
  { "name": "resolve-object", "mode": "rpc", "method": "resolve", "params": { "query": { "nested": "x" } } },
  { "name": "resolve-array", "mode": "rpc", "method": "resolve", "params": { "query": ["alice"] } },
  { "name": "resolve-unicode", "mode": "rpc", "method": "resolve", "params": { "query": "alice\u202e\u2066\u200d" } },
  { "name": "resolve-very-long", "mode": "rpc", "method": "resolve", "params": { "query": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" } },
  { "name": "events-name-object", "mode": "rpc", "method": "events", "params": { "name": { "bad": true }, "limit": 20, "offset": 0 } }
]
```

## 2) Pagination and numeric boundary corpus

```json
[
  { "name": "listings-negative", "mode": "rpc", "method": "listings", "params": { "limit": -1, "offset": -1 } },
  { "name": "listings-zero", "mode": "rpc", "method": "listings", "params": { "limit": 0, "offset": 0 } },
  { "name": "listings-max-int32", "mode": "rpc", "method": "listings", "params": { "limit": 2147483647, "offset": 2147483647 } },
  { "name": "events-negative", "mode": "rpc", "method": "events", "params": { "limit": -20, "offset": -5 } },
  { "name": "events-since-max", "mode": "rpc", "method": "events", "params": { "since_height": 9223372036854775807, "limit": 20, "offset": 0 } },
  { "name": "resolveaddr-limit-string", "mode": "rpc", "method": "resolve", "params": { "query": "utest1bad", "limit": "50", "offset": "0" } },
  { "name": "resolveaddr-offset-object", "mode": "rpc", "method": "resolve", "params": { "query": "utest1bad", "limit": 20, "offset": { "oops": 1 } } }
]
```

## 3) Malformed JSON-RPC corpus

```json
[
  { "name": "raw-missing-jsonrpc", "mode": "raw", "body": "{\"id\":1,\"method\":\"status\",\"params\":{}}" },
  { "name": "raw-missing-method", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":1,\"params\":{}}" },
  { "name": "raw-id-object", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":{\"x\":1},\"method\":\"status\",\"params\":{}}" },
  { "name": "raw-unknown-method", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"totally_unknown\",\"params\":{}}" },
  { "name": "raw-invalid-json", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"status\",\"params\":" },
  { "name": "raw-batch-mixed", "mode": "raw", "body": "[{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"status\",\"params\":{}},{\"jsonrpc\":\"2.0\",\"id\":2,\"method\":\"unknown\",\"params\":{}},{\"jsonrpc\":\"2.0\",\"id\":null,\"method\":\"resolve\",\"params\":{\"query\":\"alice\"}}]" },
  { "name": "raw-non-object", "mode": "raw", "body": "\"just-a-string\"" }
]
```

## 4) Non-determinism probe corpus

Use with repetitions `>= 3`.

```json
[
  { "name": "status-repeat", "mode": "rpc", "method": "status", "params": {} },
  { "name": "listings-repeat", "mode": "rpc", "method": "listings", "params": { "limit": 5, "offset": 0 } },
  { "name": "events-repeat", "mode": "rpc", "method": "events", "params": { "limit": 5, "offset": 0 } },
  { "name": "resolve-repeat-known", "mode": "rpc", "method": "resolve", "params": { "query": "alice" } },
  { "name": "resolve-repeat-unknown", "mode": "rpc", "method": "resolve", "params": { "query": "name_that_should_not_exist_12345" } }
]
```

## 5) Latency stress corpus

Use with `Slow ms` around `500-1200`.

```json
[
  { "name": "resolve-long-1", "mode": "rpc", "method": "resolve", "params": { "query": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" } },
  { "name": "resolve-long-2", "mode": "rpc", "method": "resolve", "params": { "query": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb" } },
  { "name": "events-limit-large", "mode": "rpc", "method": "events", "params": { "limit": 100000, "offset": 0 } },
  { "name": "listings-limit-large", "mode": "rpc", "method": "listings", "params": { "limit": 100000, "offset": 0 } },
  { "name": "raw-batch-many", "mode": "raw", "body": "[{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"status\",\"params\":{}},{\"jsonrpc\":\"2.0\",\"id\":2,\"method\":\"status\",\"params\":{}},{\"jsonrpc\":\"2.0\",\"id\":3,\"method\":\"status\",\"params\":{}},{\"jsonrpc\":\"2.0\",\"id\":4,\"method\":\"status\",\"params\":{}},{\"jsonrpc\":\"2.0\",\"id\":5,\"method\":\"status\",\"params\":{}}]" }
]
```

## Triage notes

- `error` anomalies: expected for malformed inputs, but watch for stack traces/internal leaks.
- `inconsistent` anomalies: strongest signal; replay in `Raw RPC`.
- `slow` anomalies: potential DoS path; retest with similar shaped payloads.

## Corpus to vulnerability class mapping

Use this section to speed up report writing after a fuzz run.

- **1) Input type and length corpus**
  - Likely classes: parser bugs, input validation flaws, normalization bugs, potential injection surfaces.
  - Report when: wrong types are accepted silently, Unicode confusables resolve incorrectly, very long inputs cause errors/timeouts.
  - Suggested evidence: request payload, expected validation behavior, actual response, whether issue reproduces on both networks.

- **2) Pagination and numeric boundary corpus**
  - Likely classes: integer overflow/underflow, logic flaws, expensive query/DoS paths.
  - Report when: negative/huge values bypass checks, server returns massive unintended data, latency spikes dramatically.
  - Suggested evidence: exact numeric value used, response size, latency delta versus normal baseline.

- **3) Malformed JSON-RPC corpus**
  - Likely classes: protocol parser bugs, error handling weaknesses, information disclosure.
  - Report when: malformed payload causes non-JSON crash output, stack traces, unstable status codes, or acceptance of invalid RPC structure.
  - Suggested evidence: raw body, HTTP status, raw response text, parser/error signature.

- **4) Non-determinism probe corpus**
  - Likely classes: race conditions, cache inconsistency, nondeterministic logic flaws.
  - Report when: repeated identical payloads produce materially different outputs (outside expected chain-progress changes).
  - Suggested evidence: same input across N runs, differing response signatures/fields, time window and endpoint used.

- **5) Latency stress corpus**
  - Likely classes: DoS/resource exhaustion, algorithmic complexity issues.
  - Report when: specific payload shapes reliably exceed threshold or degrade service responsiveness.
  - Suggested evidence: slow threshold, max/avg latency, which payload caused slowdown, reproducibility rate.

## Quick report template

Use this compact format for each finding:

```text
Title:
Category: (Parser bug | Logic flaw | DoS | Info leak | Race condition)
Endpoint/Method:
Network: (testnet/mainnet)
Payload:
Observed behavior:
Expected behavior:
Security impact:
Repro steps:
Repro rate:
Artifacts: (fuzz report JSON filename + raw response snippet)
```

## Severity scoring rubric

Use this to assign consistent severity for RPC fuzz findings.

- **Critical**
  - Direct risk of fund loss, unauthorized state change, or reliable service-wide denial of service.
  - Typical signals:
    - auth/permission bypass on sensitive methods
    - payload causing sustained outage or crash loop
    - exploit path that is low-complexity and remotely triggerable
  - Priority: immediate hotfix, coordinated disclosure, broad regression tests.

- **High**
  - Significant security impact without immediate catastrophic loss.
  - Typical signals:
    - high-confidence info leak (internal paths, stack traces, sensitive metadata)
    - strong DoS vector requiring moderate effort
    - deterministic logic flaw affecting integrity of returned records
  - Priority: fast patch cycle, backport to supported versions.

- **Medium**
  - Real bug with constrained impact or harder exploitation.
  - Typical signals:
    - inconsistent responses with measurable integrity risk but limited blast radius
    - validation bypass that does not currently expose sensitive state
    - latency spikes on narrow payload families only
  - Priority: scheduled fix with targeted monitoring and regression corpus updates.

- **Low**
  - Minor weakness, low exploitability, or mostly hardening concern.
  - Typical signals:
    - verbose but non-sensitive error formatting
    - edge-case parser acceptance with no meaningful downstream impact
    - slight nondeterminism explainable by benign chain/index timing
  - Priority: backlog hardening; fix when touching adjacent code.

- **Informational**
  - Not a vulnerability by itself; useful for baseline, telemetry, or future regression tracking.
  - Typical signals:
    - expected errors for malformed RPC
    - expected latency changes during peak load
    - behavior documented and consistent with design
  - Priority: document and keep in corpus for trend tracking.

## Severity modifiers

Adjust one level up/down based on these factors:

- **Reproducibility**
  - Up: reproduces consistently (`>=80%`) across runs/networks.
  - Down: rare/flaky and tied to transient backend conditions.

- **Attack complexity**
  - Up: single request, no prerequisites.
  - Down: requires timing windows, special state, or long sequences.

- **Blast radius**
  - Up: affects many users or core endpoints (`status`, `resolve`, `listings`, `events` broadly).
  - Down: isolated to niche parameter combinations.

- **Detection and containment**
  - Up: low observability, hard to detect in logs.
  - Down: easily detectable and rate-limited.

## Example severity assignments for this tester

- Malformed RPC returns stack trace with internal file paths -> **High**.
- Very large `limit` reliably causes multi-second degradation for all users -> **High** (possibly **Critical** if service destabilizes).
- Same `resolve` request returns conflicting ownership fields repeatedly -> **High**.
- Invalid types return clean, stable validation errors -> **Informational**.

## Testnet payment-bypass corpus

Use this for your stated goal: check whether name ownership could be accepted without valid payment settlement.

Important:

- Run only on **testnet**.
- This corpus is for **negative testing** and responsible reporting.
- Expected behavior is rejection or no state change.

### A) Forged claim memo patterns (raw probes)

These attempt to mimic action-shaped memos/fields without real on-chain settlement.

```json
[
  { "name": "claim-forged-minimal", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5001,\"method\":\"claim\",\"params\":{\"name\":\"freeclaimtest1\",\"address\":\"utest1fakeaddr\",\"memo\":\"ZNS:CLAIM:freeclaimtest1:utest1fakeaddr:fakeSig:fakePub\"}}" },
  { "name": "claim-forged-underpaid", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5002,\"method\":\"claim\",\"params\":{\"name\":\"freeclaimtest2\",\"address\":\"utest1fakeaddr\",\"amount\":1,\"memo\":\"ZNS:CLAIM:freeclaimtest2:utest1fakeaddr:sig:pub\"}}" },
  { "name": "claim-forged-zero-amount", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5003,\"method\":\"claim\",\"params\":{\"name\":\"freeclaimtest3\",\"address\":\"utest1fakeaddr\",\"amount\":0,\"txid\":\"00\"}}" },
  { "name": "claim-forged-missing-tx", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5004,\"method\":\"claim\",\"params\":{\"name\":\"freeclaimtest4\",\"address\":\"utest1fakeaddr\",\"signature\":\"AAAA\",\"pubkey\":\"BBBB\"}}" },
  { "name": "claim-forged-rpc-transport", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5005,\"method\":\"submit\",\"params\":{\"action\":\"CLAIM\",\"name\":\"freeclaimtest5\",\"address\":\"utest1fakeaddr\",\"cost\":0,\"signature\":\"fake\"}}" }
]
```

### B) Buy/list bypass attempts (raw probes)

Tries to fake market actions that should require valid signed state + payment semantics.

```json
[
  { "name": "buy-forged-no-payment", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5101,\"method\":\"buy\",\"params\":{\"name\":\"targetlistedname\",\"buyerAddress\":\"utest1fakebuyer\",\"amount\":0,\"txid\":\"00\"}}" },
  { "name": "buy-forged-low-payment", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5102,\"method\":\"buy\",\"params\":{\"name\":\"targetlistedname\",\"buyerAddress\":\"utest1fakebuyer\",\"amount\":1,\"memo\":\"ZNS:BUY:targetlistedname\"}}" },
  { "name": "list-forged-admin-ish", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5103,\"method\":\"list\",\"params\":{\"name\":\"targetlistedname\",\"price\":1,\"nonce\":1,\"signature\":\"fake\",\"pubkey\":\"fake\"}}" },
  { "name": "setprice-forged", "mode": "raw", "body": "{\"jsonrpc\":\"2.0\",\"id\":5104,\"method\":\"setprice\",\"params\":{\"prices\":[1,1,1,1],\"nonce\":1,\"signature\":\"fake\"}}" }
]
```

### C) Post-condition verification corpus

After running A/B, use this corpus to confirm whether any forged attempt changed state.

```json
[
  { "name": "verify-freeclaimtest1", "mode": "rpc", "method": "resolve", "params": { "query": "freeclaimtest1" } },
  { "name": "verify-freeclaimtest2", "mode": "rpc", "method": "resolve", "params": { "query": "freeclaimtest2" } },
  { "name": "verify-freeclaimtest3", "mode": "rpc", "method": "resolve", "params": { "query": "freeclaimtest3" } },
  { "name": "verify-freeclaimtest4", "mode": "rpc", "method": "resolve", "params": { "query": "freeclaimtest4" } },
  { "name": "events-forged-claims", "mode": "rpc", "method": "events", "params": { "name": "freeclaimtest1", "limit": 20, "offset": 0 } }
]
```

## Payment-bypass test procedure (testnet)

1. Pick unique names (e.g., `freeclaimtest<timestamp>`).
2. Run corpus A and B.
3. Run corpus C immediately and again after short delay (index lag check).
4. A finding is likely valid if:
   - `resolve(name)` becomes non-null for unpaid/forged flow, or
   - `events` shows `CLAIM`/`BUY` tied to forged attempt.
5. Export fuzz report JSON and include raw request/response evidence.

## Responsible reporting checklist

- Network used: `testnet`
- Timestamp (UTC)
- Endpoint URL
- Exact payload(s)
- Observed result and post-condition proof (`resolve` / `events`)
- Reproducibility across repeated runs
- Impact statement: "name accepted without valid payment settlement"

## Mainnet-safe monitoring corpus (read-only)

Use this on mainnet for detection and monitoring only (no forged transaction attempts).

### A) Health and baseline corpus

```json
[
  { "name": "mainnet-status", "mode": "rpc", "method": "status", "params": {} },
  { "name": "mainnet-listings-page1", "mode": "rpc", "method": "listings", "params": { "limit": 20, "offset": 0 } },
  { "name": "mainnet-events-page1", "mode": "rpc", "method": "events", "params": { "limit": 20, "offset": 0 } },
  { "name": "mainnet-events-claims", "mode": "rpc", "method": "events", "params": { "action": "CLAIM", "limit": 20, "offset": 0 } },
  { "name": "mainnet-events-buys", "mode": "rpc", "method": "events", "params": { "action": "BUY", "limit": 20, "offset": 0 } }
]
```

### B) Consistency probe corpus

Replace sample names/addresses with known real values from recent events for best results.

```json
[
  { "name": "resolve-known-name-1", "mode": "rpc", "method": "resolve", "params": { "query": "alice" } },
  { "name": "resolve-known-name-2", "mode": "rpc", "method": "resolve", "params": { "query": "bob" } },
  { "name": "events-known-name-1", "mode": "rpc", "method": "events", "params": { "name": "alice", "limit": 20, "offset": 0 } },
  { "name": "events-known-name-2", "mode": "rpc", "method": "events", "params": { "name": "bob", "limit": 20, "offset": 0 } },
  { "name": "resolve-address-sample", "mode": "rpc", "method": "resolve", "params": { "query": "u1replace_with_known_address", "limit": 20, "offset": 0 } }
]
```

### C) Nondeterminism monitor corpus

Run with repetitions `>= 3` to detect unstable mainnet responses.

```json
[
  { "name": "status-repeat-mainnet", "mode": "rpc", "method": "status", "params": {} },
  { "name": "listings-repeat-mainnet", "mode": "rpc", "method": "listings", "params": { "limit": 10, "offset": 0 } },
  { "name": "events-repeat-mainnet", "mode": "rpc", "method": "events", "params": { "limit": 10, "offset": 0 } },
  { "name": "resolve-repeat-name-mainnet", "mode": "rpc", "method": "resolve", "params": { "query": "alice" } }
]
```

### Mainnet investigation checklist

- Use network toggle set to `mainnet`.
- Keep payloads read-only (`status`, `resolve`, `listings`, `events`).
- Export each report JSON after runs.
- Compare anomaly trends over time (not just one run).
- Report suspicious integrity mismatches with evidence and timestamps.

## Anti-bypass validation matrix

Use this as a concrete build-and-test checklist for claim/buy acceptance logic.

### Claim acceptance matrix

| Check | Why it matters | Required validation logic | Negative tests to run | Expected result |
| --- | --- | --- | --- | --- |
| Name format validation | Prevent malformed/confusable claims | Enforce lowercase alnum and valid length before any state transition | Empty, uppercase, unicode confusables, overlength | Reject; no state/event mutation |
| Author intent signature | Prevent forged client intent | Verify Ed25519 signature over canonical payload bytes | Fake signature, altered payload after sign, wrong pubkey | Reject; explicit signature error |
| Nonce freshness | Prevent replay claim reuse | Nonce/challenge bound to user+action; one-time use | Reuse same signed payload twice | First may pass, second must fail |
| Price recomputation | Prevent underpay via client-provided cost | Compute required cost server-side from current pricing nonce/tiers | Send lower `amount`/`cost` than required | Reject; no ownership change |
| Tx payment existence | Prevent off-chain fake accept | Verify txid exists on chain and references expected memo intent | Random/fake txid, missing txid | Reject; no ownership change |
| Tx output validation | Prevent paying wrong recipient/amount | Confirm output to registry address with exact required value | Underpay, pay other address, split outputs improperly | Reject |
| Confirmation threshold | Prevent mempool/reorg abuse | Require min confirmations before commit | Valid mempool tx not yet confirmed | Pending/reject until confirmed |
| Atomic commit | Prevent partial acceptance bugs | Apply state only after all checks pass in single transaction | Force errors at each stage | No partial writes |
| Name uniqueness at commit time | Prevent race double-claim | Lock/check unique name at final write | Parallel claims on same name | Exactly one success |

### Buy acceptance matrix

| Check | Why it matters | Required validation logic | Negative tests to run | Expected result |
| --- | --- | --- | --- | --- |
| Listing authenticity | Prevent forged listings | Verify listing signature + ownership + nonce | Fake listing signature, stale nonce | Reject |
| Active listing state | Prevent buying delisted/changed listing | Confirm listing still active and unchanged at commit | Buy after delist/price update | Reject |
| Exact price payment | Prevent cheap buy bypass | Validate on-chain payment equals current listing price | Underpay by 1 zats, wrong memo linkage | Reject |
| Buyer binding | Prevent payment hijack | Bind intent memo/signature to buyer UA/pubkey | Replay payment with different buyer fields | Reject |
| Replay protection | Prevent one payment buying multiple names | Mark txid+memo intent consumed once | Re-submit same txid/memo | Reject duplicates |
| Atomic transfer | Prevent split ownership/listing bugs | Ownership transfer + listing close happen atomically | Inject failures around transfer | No partial state |

### Endpoint-level enforcement map

| Endpoint / Flow | Must enforce |
| --- | --- |
| `prepareClaim` / `prepareBuy` | Only produce payload/URI; **must not** mutate ownership |
| settlement watcher / claim processor | Full chain verification, cost/price recomputation, nonce/replay checks |
| indexer RPC read methods (`resolve`, `events`, `listings`) | Reflect committed chain-validated state only |
| any direct action RPC (if exists) | Reject direct state writes without settlement proof |

### Test case bundles mapped to checks

- **Bundle A: Signature + nonce**
  - Fake signature, wrong pubkey, stale nonce, replay same payload.
  - Targets: author intent signature, nonce freshness, replay protection.

- **Bundle B: Payment integrity**
  - Fake txid, underpay, wrong recipient, wrong memo, unconfirmed tx.
  - Targets: payment existence, output validation, confirmations, price recomputation.

- **Bundle C: Concurrency**
  - Parallel claim/buy requests against same name/listing.
  - Targets: uniqueness, atomic commit, race safety.

- **Bundle D: State transition correctness**
  - Buy while delisting/updating listing in parallel.
  - Targets: active listing state, atomic transfer.

### Minimum evidence required for a valid report

- exact request payload
- returned response body
- `resolve(name)` before vs after
- relevant `events` entries
- tx proof details used by server decision (txid, amount, recipient, confirmations)
- reproduction rate across repeated runs

## Pass/fail worksheet (copy per test session)

Use this as a lightweight tracker while running corpora.

|Date (UTC)|Network|Corpus / Bundle|Test case|Expected|Actual|Pass/Fail|Repro rate|Severity|Artifact|
|---|---|---|---|---|---|---|---|---|---|
|2026-04-21T00:00:00Z|testnet|Bundle B|underpay-by-1-zat|reject|reject with error -32000|Pass|10/10|Informational|zns-fuzz-report-testnet-...json|
|||||||||||
|||||||||||

### Suggested values

- **Pass/Fail**
  - `Pass`: behavior matches secure expectation.
  - `Fail`: behavior deviates from secure expectation.
- **Repro rate**
  - format: `x/y` (example `7/10`).
- **Severity**
  - one of `Informational`, `Low`, `Medium`, `High`, `Critical`.
- **Artifact**
  - exported report file name and optional short note (`+ raw response snippet`).

## Mainnet usage guidance

You can test on mainnet, but keep it strictly read-only:

- Safe on mainnet:
  - `status`, `resolve`, `listings`, `events`
  - non-destructive consistency and nondeterminism checks
- Do **not** run on mainnet:
  - forged payment-bypass attempts
  - any direct action simulation meant to mimic unauthorized claim/buy flows

Rule of thumb:

- Use **testnet** for adversarial/negative exploit-style testing.
- Use **mainnet** for monitoring, integrity validation, and anomaly detection only.
