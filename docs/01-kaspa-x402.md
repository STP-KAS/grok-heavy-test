# Battle 1 — kaspa-x402

What Luke asked for. Full grok test / analyse / reasoning / advice.

Subject: [elldeeone/kaspa-x402](https://github.com/elldeeone/kaspa-x402) tag **`v1.0.0-rc.1`** (`040b1ec8335abadbb3c69cf1ea720ae45816b0f7`).  
Ask: [x.com/elldeeone/status/2099316438704312512](https://x.com/elldeeone/status/2099316438704312512)

**Status 14 Sep evening:** Windows clone-and-test merged as [`216ad77`](https://github.com/elldeeone/kaspa-x402/commit/216ad773d9dc522769dfab8e5deb01eb70f0e849). Public verdict: [https://sixpack.wtf](https://sixpack.wtf). The repro below is against the RC.1 tag, which still fails. Merged `main` retest: `.sil` LF, `npm test` green, `proof:offline` 24/24.

---

## grok test

Machine: Windows, PowerShell, Node v24.19.0, npm 11.17.0. Date: 2026-09-14.

### Commands

```
git clone --depth 1 --branch v1.0.0-rc.1 https://github.com/elldeeone/kaspa-x402.git
npm ci
git config --get core.autocrlf          # true
git ls-files --eol contracts/kaspa-x402-escrow-v4.sil   # i/lf  w/crlf
npm test                                # FAIL: covenant source SHA-256
# rewrite .sil without CR
npm --workspace @kaspa-x402/covenant test   # PASS 27
npm test                                # FAIL: proof-output-security 0o600 vs 0o666
npm run proof:offline                   # 22 checks ok; summary EPERM fsync
```

### Break 1 — CRLF vs launch identity

The covenant launch identity is a SHA-256 of the `.sil` **bytes**. Git on this machine converted LF → CRLF. There is **no** `.gitattributes`.

| Bytes | SHA-256 |
| --- | --- |
| CRLF checkout | `ab88ec5da53d1e716bb1f1322182d48bbb2c9fed9b0078f9d718a022282f48c6` |
| LF (pinned `ESCROW_V4_SOURCE_SHA256`) | `065dff5d0d02f3a09f56bab977a33d4e047f2ccec64c0d066a318d342797fcb0` |

Failing tests in `packages/covenant/test/covenant.test.ts`:

- `reproduces current silverc genesis and successor bytecode`
- `binds the launch identity to the checked source and compiled base`

After deleting `\r` from the working tree (not committed), both passed. Bytecode pins were fine. Only the **source-file hash** moved.

CI (`.github/workflows/ci.yml`) is **`ubuntu-latest` only**. Linux checkouts keep LF. Windows implementers are invisible to CI.

That is a break of the README claim “locally: `npm ci && npm test`.”

### Break 2 — Unix 0600 on Windows

`scripts/proof-output-security.test.mjs`:

```js
assert.equal(fs.statSync(reportFile).mode & 0o777, 0o600);
```

Actual `mode & 0o777` = **438** (`0o666`). Expected **384** (`0o600`). Secret-scrub assertions in the same test are the real requirement. The mode check is POSIX-only.

### Break 3 — proof:offline fsync

`npm run proof:offline` printed 22 checks, all `"ok": true`, then:

```json
"summary": { "ok": false, "checkCount": 22, "error": "EPERM: operation not permitted, fsync" }
```

Protocol path: pass. Report write-out: fail closed on this OS.

The 22 checks included: exact create/verify/settle, payment-id idempotency, exact replay `409 invalid_transaction_state`, batch genesis/voucher/claim/top-up/refund construction, fixture reproducibility against compiler `3ed9733`.

### What held after LF-normalize

| Workspace | Tests |
| --- | --- |
| `@kaspa-x402/core` | 163 pass |
| `@kaspa-x402/covenant` | 27 pass |
| `@kaspa-x402/server` | 214 pass |
| `@kaspa-x402/client` | 124 pass |
| `@kaspa-x402/facilitator` | 36 pass |
| `@kaspa-x402/cli` + mock examples | 6 pass |
| `@kaspa-x402/demo-gateway` | 107 pass |
| `scripts/*` (node:test) | 17 pass, 1 fail (0600) |

Mock examples (no wallet):

- `examples/paid-http-api` — exact 200 + batch two charges `"50000"`
- `examples/paid-mcp-tool` — `scheme: batch-settlement`, `chargedAmount: "250000"`
- `examples/self-hosted-facilitator` — verify/settle mock
- `examples/recovery` — exact replay `invalid_payload`

### Live network (no wallet)

| URL | Result |
| --- | --- |
| `https://demo.kaspa-x402.org/health` | `ok`, `releaseVersion: 1.0.0-rc.1`, `hostedExactSettlementEnabled: true`, `exactProfile: standard-native`, `chainBroadcastMode: pnn` |
| `https://demo.kaspa-x402.org/supported` | `x402Version: 2`, `exact` + `batch-settlement`, **`kaspa:testnet-10` only**, asset `KAS`, bindings `kaspa-exact-v2` / `kaspa-escrow-v3` template `kaspa-x402-escrow-v4` |
| TN10 API `b46de1e1ffc502c69003043e81f7f3c98e49b402a836eb0012300487b2b77b7c` | `is_accepted: true`, `version: 0` (tiny exact from their 13 Sep report) |

Did **not** send a paid `PAYMENT-SIGNATURE` to the Worker.

### npm audit

`npm audit --omit=dev`: **fast-uri** high (SSRF / host confusion), likely via **ajv**. Not demonstrated reachable as an exploit here. Full tree: 6 vulns (4 high). Dev-tool vulns are not a protocol hole. fast-uri on the validator path is a supply-chain fact.

### Not run

- `proof:live:check -- --live` (needs funded TN10)
- `validate:tx-v1-consensus` (needs rusty-kaspa at `c338d495bec29e4dc8b5149f99e8db6fa916ed4a`)
- silverc recompile of the `.sil`

Their 13 Sep funded 18-flow harness is evidence **they** ran it.

---

## grok analyse

### Object

A **proposed Kaspa network binding** for **x402 v2**. Not a new L1, not a KIP, not a stablecoin, not “HTTP 402 plus a Kaspa tx.”

x402 standardises:

1. how a server **requests** payment (`PaymentRequired`)
2. how a client **authorises** it (`PaymentPayload`)
3. how the server **verifies and settles** (`SettlementResponse`)

Transports: HTTP headers `PAYMENT-REQUIRED` / `PAYMENT-SIGNATURE` / `PAYMENT-RESPONSE` (base64 JSON), and MCP `_meta` keys. Historical `X-PAYMENT` is out.

Kaspa x402 plugs **native KAS** into that envelope.

### Schemes

| Scheme | Binding | Profile / template | Settlement |
| --- | --- | --- | --- |
| `exact` | `kaspa-exact-v2` | default `standard-native`; optional `additive` | One-shot. Ordinary merchant output, or KIP-10 successor delta. |
| `batch-settlement` | `kaspa-escrow-v3` | `kaspa-x402-escrow-v4` | Fund once. Sign lifetime ceiling T. Partial claims. Top-up. DAA refund. |

Upstream also defines `upto` and `auth-capture`. This repo **refuses** those names because Kaspa scripts cannot make a once-valid tx become invalid at an expiry instant. Batch refund is a **race** after DAA. They admit this in `docs/native-profile-boundary.md`.

### Asset

`asset` is `"KAS"`. Amounts are canonical decimal sompi strings. Display may say tKAS. Wire must not use floating KAS.

Luke, same thread: a future Kaspa stablecoin needs **its own asset binding**. That is the only non-delusional stables sentence.

### Networks

CAIP-2 **syntax**: `kaspa:testnet-10`, reserved `kaspa:mainnet`.  
CAIP **registry**: [namespaces#193](https://github.com/ChainAgnostic/namespaces/pull/193) open since 27 Jul 2026. Not merged.

`allowMainnet` is required for the reserved name. Docs: mainnet blocked. The type system does not check that audits exist.

### Covenant (batch)

`contracts/kaspa-x402-escrow-v4.sil`

- `pragma silverscript ^0.1.0` in source. Compiled with **v1.0.0** `3ed9733`. Pin hygiene is wrong. Bytecode pin in fixtures is the real pin.
- No `validateOutputState`. No `readInputState`. No `State[].split()`.
- Claim: singleton 1→1. Voucher = SHA-256(domain ‖ network ‖ covenantId ‖ T_le64). `outputs[1].value == inputValue - claimAmount`. Payout `<= claimAmount` (fee from **server** output).
- Top-up: both parties SIGHASH_ALL. Successor value **>** input.
- Refund: `tx.daa >= timeout`. 1→0 same-ID. Refund output `<=` input (fee from buyer remainder).

Application A/S/T/V/R (`0 <= S <= A <= T`, `(T-S)+R <= V`) is **not** all on-chain. The script sees claimed cumulative and voucher T. **A** is a server-store invariant. A buyer who pre-signs T can be claimed up to T even if remaining calls never ran.

### Facilitator

Optional. Direct mode is the point: a server with a node can verify. Hosted demo is a Cloudflare Worker (`chainBroadcastMode: pnn`). Capability intersection is supposed to stop a facilitator widening server kinds. Hosted operators still need auth, rate limits, tenant isolation.

### Versus Kaspa L1 (master-file pins)

| L1 fact | This binding |
| --- | --- |
| Toccata live (KIP-16/17/20/21) | Uses KIP-10 additive, KIP-20 covenant id, KIP-17 spend rules |
| SilverScript v1.0.0 | Fixture compiler `3ed9733` |
| #234 foreign state | Unused (good) |
| #243 no compute budget in artifact | Builder guesses 9999/10000 |
| #249/#250 split tuples | Unused (good) |
| Amount not locked by `validateOutputState` | Claim successor **does** `require(value == …)` |
| 10 BPS, not 100 | Not claimed |
| DAGKnight not shipped | Not used |
| Skip centralised stables | `asset: KAS` only |

---

## grok reasoning

### Not delusional

- Distinguishing HTTP 402 (1997 status code) from x402 (a specific envelope).
- Native KAS as the asset, not USDC-on-Base cosplay.
- Optional facilitator.
- Claim successor value lock. They did not copy the broken example-escrow pattern on the continuation.
- Mainnet gates written down. They did not declare victory.
- Hosted `/supported` advertises testnet-10 only.

### Delusional if you say it out loud

- “Kaspa is in x402.” Upstream lists EVM/SVM/…/Keeta. Not Kaspa.
- “CAIP `kaspa:` is registered.” PR 193 is open.
- “v1.” RC + draft spec + open gates.
- “Accepted means paid forever.”
- “500 sompi per call is an L1 micropayment UTXO.” It is a voucher against an escrow that keeps a 10M sompi-class reserve.
- “Stables will work the same.” Luke already said no. Code says no.
- “k402 is x402.” Custom `kaspa-channel`. kccs#4 open, dirty, not adopted.
- “Windows `npm test` matches CI.” It does not.

### Trust

Direct mode removes **facilitator-as-money**. It does not remove a lying RPC, a crashed in-memory store, a server that takes T and never delivers remaining calls, a timeout race, or a Worker operator.

L402/Lightning has years of production and preimage-tied delivery. This has a TN10 harness and mock examples. Stronger settlement *shape* (L1 UTXO) is not stronger *operations*.

Coinbase x402 is weaker on sovereignty (USDC, often CDP) and stronger on “it runs on mainnet today.” Do not copy their asset. Do not copy their “we shipped” as your status.

### Stables, firmly

Master file: skip centralised stablecoins for dapps. Fill is not a business.

x402-the-protocol is chain-agnostic. That does **not** mean Kaspa x402 should speak USDT. A till that quotes EUR and settles KAS is a till. A binding that settles USDC is Coinbase’s binding. A reserved kUSD with no capital is a seat for a dollar that does not exist.

If Kaspa ever has a native unit that is not KAS, it is a **new `asset` + new settlement path + new audit**. Not a flag on `kaspa-exact-v2`.

---

## grok advice

**Bind this envelope. Do not invent a fourth 402.** Steal lock/voucher *ideas* from k402 if you need a channel. Credit Kali. Do not call k402 “Kaspa’s x402 v2.” Do not call kccs#4 adopted.

### Kill-ifs

- `allowMainnet: true` before every gate in `docs/mainnet-readiness.md` is closed
- Foreign `readInputState` in the escrow
- `State[].split()` tuples on silverc v1.0.0
- Compiler pin floats off v1.0.0 `3ed9733`
- Claim/top-up successor without `tx.outputs[i].value == …`
- USDC / USDT / “kaspa dollar” / tPEG as `asset`
- Treating `accepted` as irreversible, or 10 BPS as 100 BPS
- Calling k402 / KCC-0402 / a till / a CDN “x402”
- In-memory stores or one RPC sold as Byzantine-safe
- Windows implementers told `npm test` without `.gitattributes`

### What v1 must prove

1. Independent covenant + replay audit (exact **and** batch), including the claim/refund race and store crash.
2. Independently corroborated chain evidence.
3. Durable transactional stores + distributed admission under concurrent load.
4. Fresh funded TN10 run **and** hosted-Worker paid canary.
5. CAIP #193 merged **or** every public sentence says “proposed identifier, unregistered.”
6. Upstream: a real Kaspa mechanism package, or a rejected-with-reasons issue.
7. Mainnet confirmation policy chosen. `accepted`-only content release documented as reorg risk.
8. `pragma silverscript` pinned to the compiler you actually used.
9. CI on Windows, or `.gitattributes` + skip Unix modes on `win32`.

### For agents building now

TN10 only. Native KAS. Prefer `standard-native` exact. Treat batch as **capital lock + signed ceiling**. Do not pre-sign **T** beyond delivered work. Do not put mainnet KAS in this.

File-ready Windows issue: [02-windows-repro.md](02-windows-repro.md).
