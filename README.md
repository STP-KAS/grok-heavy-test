# Grok heavy test

**Send this repo.** One independent Grok pass, 14 Sep 2026. Three battles. Not Kaspa core. Not a KIP. Not an audit. Not a v1 rubber stamp.

Luke Dunshea ([@elldeeone](https://x.com/elldeeone)) asked humans **and agents** to read [kaspa-x402](https://github.com/elldeeone/kaspa-x402), build against it, test the assumptions, and try to break it before final v1.

Post: [x.com/elldeeone/status/2099316438704312512](https://x.com/elldeeone/status/2099316438704312512)

This is that report, plus the two sister battles from the same desk: **Ishum** and the **Kaspa master file**.

---

## How to read this

| If you want | Go here |
| --- | --- |
| The whole pass in one screen | this README, start at [Verdicts](#verdicts) |
| Battle 1 — kaspa-x402 (what Luke asked) | [docs/01-kaspa-x402.md](docs/01-kaspa-x402.md) |
| Windows repro (file this on his repo) | [docs/02-windows-repro.md](docs/02-windows-repro.md) |
| Battle 2 — Ishum | [docs/03-ishum.md](docs/03-ishum.md) |
| Battle 3 — master file | [docs/04-master-file.md](docs/04-master-file.md) |
| HTTP 402 is not x402 | [docs/05-402-is-not-x402.md](docs/05-402-is-not-x402.md) |
| Claim vs evidence table | [docs/06-evidence.md](docs/06-evidence.md) |

Shorter sister repos from the same day: [x402-vs-grok](https://github.com/STP-KAS/x402-vs-grok) · [x402-ishum](https://github.com/STP-KAS/x402-ishum) · [402-is-not-x402](https://github.com/STP-KAS/402-is-not-x402). **This repo is the one to send.**

---

## Pins

| Pin | Value |
| --- | --- |
| Date | 2026-09-14 |
| Subject tag | `elldeeone/kaspa-x402` **`v1.0.0-rc.1`** @ `040b1ec8335abadbb3c69cf1ea720ae45816b0f7` |
| Hosted gateway | [demo.kaspa-x402.org](https://demo.kaspa-x402.org) `releaseVersion: 1.0.0-rc.1` |
| Compiler they compiled with | SilverScript **v1.0.0** `3ed973335b59269293564805cc2c58a14595ec03` |
| CAIP | [ChainAgnostic/namespaces#193](https://github.com/ChainAgnostic/namespaces/pull/193) **open** |
| Machine | Windows, PowerShell, Node **v24.19.0**, `core.autocrlf=true` |
| Intel | [STP-KAS/kaspa-master-file](https://github.com/STP-KAS/kaspa-master-file) freeze 13 Sep, then pin update [662c77a](https://github.com/STP-KAS/kaspa-master-file/commit/662c77a) |

PegLab and Gramlane were **not** used as sources of truth for battle 1.

---

## Verdicts

### Battle 1 — kaspa-x402

This is a **real x402 v2 binding** for native KAS. It is not another HTTP 402 costume. **Bind this envelope.**

It is **not v1**, **not mainnet**, **not in the upstream x402 registry**, **not a registered CAIP namespace**. Do not call RC1 “Kaspa has x402.”

Windows `npm test` **fails on a default checkout**. That is a real break of “clone it and run the tests.” It is not a break of the covenant math. After LF-normalizing the `.sil`, hundreds of unit tests passed. Hosted `/supported` is `x402Version: 2`, asset `KAS`, network `kaspa:testnet-10` only. Native KAS is the right asset. A future stable needs its own binding.

Full: [docs/01-kaspa-x402.md](docs/01-kaspa-x402.md) · file-ready issue: [docs/02-windows-repro.md](docs/02-windows-repro.md)

### Battle 2 — Ishum

**Ishum is a coffee till, not x402.** Quotes EUR/USD. Settles native KAS. Never holds keys. Zero `PAYMENT-*` headers. kUSD is a reserved chair with no capital. USDT is a guest IOU and the keypad still teaches it. Do not weld the names.

Full: [docs/03-ishum.md](docs/03-ishum.md)

### Battle 3 — master file

Our public pin list still said **“TN10 alpha”** after Luke tagged **`v1.0.0-rc.1`**. Stale pin. Same class of error as shipping the wrong compiler tag. Updated this pass: bind RC.1, TN10 only, mainnet blocked. k402 is a lock to steal, not “Kaspa’s x402.” No fourth envelope. No stables on this binding.

Full: [docs/04-master-file.md](docs/04-master-file.md)

---

## Battle 1 in one page

### What we actually ran

```
git clone --branch v1.0.0-rc.1 https://github.com/elldeeone/kaspa-x402.git
npm ci
npm test                  # FAIL on Windows (CRLF .sil hash)
# rewrite .sil without CR
npm test                  # FAIL (Unix 0o600 vs Windows 0o666)
npm run proof:offline     # 22/22 checks ok, then EPERM fsync
```

Also fetched `demo.kaspa-x402.org/health` and `/supported`, and the TN10 API for the published tiny exact txid `b46de1e1…` (`is_accepted: true`).

Did **not** run a funded live harness (needs a TN10 wallet). Did **not** pay the hosted Worker. Their 13 Sep 18-flow report is evidence **they** ran it, not that this agent did.

### What broke

**1. CRLF vs launch identity.** No `.gitattributes`. Git on Windows converts the escrow `.sil` to CRLF. The launch-identity SHA-256 is pinned to LF bytes.

| | SHA-256 of `kaspa-x402-escrow-v4.sil` |
| --- | --- |
| CRLF on this disk | `ab88ec5da53d1e716bb1f1322182d48bbb2c9fed9b0078f9d718a022282f48c6` |
| LF (the pin) | `065dff5d0d02f3a09f56bab977a33d4e047f2ccec64c0d066a318d342797fcb0` |

CI is `ubuntu-latest` only. Linux never sees this.

**2. Unix `0o600` on Windows.** `proof-output-security.test.mjs` expects mode 384. This machine got 438 (`0o666`). Secret-scrub is the real requirement; the mode assert is POSIX-only.

**3. `fsync` EPERM.** Offline protocol checks passed. Writing the report failed closed on this OS.

Fix for (1): `*.sil text eol=lf`. Hash LF-normalized bytes. Add a Windows CI job, or skip Unix modes on `win32`.

### What it is

x402 is a specific protocol on HTTP 402: `PaymentRequired` / `PaymentPayload` / `SettlementResponse`, headers `PAYMENT-REQUIRED` / `PAYMENT-SIGNATURE` / `PAYMENT-RESPONSE`, `x402Version: 2`.

kaspa-x402 plugs **native KAS** (sompi decimal strings) into that envelope.

| Scheme | Binding | Meaning |
| --- | --- | --- |
| `exact` | `kaspa-exact-v2` | One-shot. Default: ordinary merchant output of the advertised sompi. Optional KIP-10 additive: successor delta **is** the payment. |
| `batch-settlement` | `kaspa-escrow-v3` + `kaspa-x402-escrow-v4` | Fund a KIP-20 escrow once. Buyer signs lifetime ceiling **T**. Provider claims. Top-up co-signed. Refund after DAA. |

Facilitator is optional. Direct verify against a node is the point. The hosted demo is a Cloudflare Worker, not consensus.

### What it is not

- Not “anything that returns HTTP 402.” See [docs/05-402-is-not-x402.md](docs/05-402-is-not-x402.md).
- Not a first-party package in `x402-foundation/x402`.
- Not a registered `kaspa:` CAIP namespace. Syntax is a proposal. PR **open**.
- Not mainnet. `kaspa:mainnet` is a reserved name. `allowMainnet` is a **boolean**, not a gate.
- Not a stablecoin rail. Luke already said a future Kaspa stable needs its own asset binding.
- Not instant irreversible cash. Hosted exact finality is **`accepted`**. Batch TN10 policy is **30 selected-chain confirmations**. That is deployment policy, not GHOSTDAG finality.

The escrow **does** lock successor value on claim (`tx.outputs[1].value == inputValue - claimAmount`). It does **not** use `validateOutputState` / `readInputState` / `State[].split()`. Those compiler holes are avoided here, not fixed in SilverScript. `pragma silverscript ^0.1.0` on a v1.0.0 compile is sloppy pin hygiene.

### Pushback that survives the tests

1. Windows is not a first-class verifier. An agent cannot “run the tests” on the advertised command.
2. Replay protection lives in the store. Lose the store, replay reopens.
3. Claim has no DAA cap; refund does. After timeout, buyer refund and server claim **race**. They refused to ship upstream `upto` for this reason and shipped it for batch anyway.
4. Batch “500 sompi per call” still locks ~10M sompi-class capital on chain. Omitting the lockup is a lie.
5. Payment identifier is “should,” not must.
6. Refund is fee-from-principal (`outputs[0].value <= input`). Claim successor is exact. That split is intentional and must stay documented.
7. Compute budget is still guessed (silverscript #243 open).
8. One RPC is not Byzantine-safe. They say this. The hosted Worker still trusts one TN10 source.

None of that makes the repo fake. It makes “v1” premature.

---

## Battle 2 in one page

Ishum = BTCPay-shaped POS. kaspa-x402 = machine-payable protocol. They share “desk holds 0 keys.” They do not share a wire.

| Probe | Ishum |
| --- | --- |
| `x402` / `PAYMENT-REQUIRED` / `x402Version` | **zero hits** |
| Live rail | native KAS |
| kUSD | `Live: false`, no reserves, demo settle |
| USDT | guest IOU, freeze-capable issuer, demo settle |
| Default store | can enable **all three** rails |
| Confirmations | blue-score subtraction (kaspa-x402 refuses this) |
| README “10 conf ≈ 10 seconds” | leftover **1 BPS** story. Mainnet is **10 BPS** |

USDT in does not un-decentralize GHOSTDAG. It also does not make Tether into Kaspa money. A reserved kUSD with no capital is a chair for a dollar that does not exist.

---

## Battle 3 in one page

[kaspa-master-file](https://github.com/STP-KAS/kaspa-master-file) is this desk’s public pin list. Merged Active KIP = law. A tweet is not. The 402 pin was **R-X402: TN10 alpha**. Luke tagged **v1.0.0-rc.1 on 13 Sep**. The pin was stale the next morning.

Updated in [662c77a](https://github.com/STP-KAS/kaspa-master-file/commit/662c77a):

- Bind `elldeeone/kaspa-x402` **v1.0.0-rc.1**
- TN10 only, mainnet blocked
- k402 / [kccs#4](https://github.com/kaspanet/kccs/pull/4) = steal the lock, do not call it x402 v2, do not call it adopted
- Independent pass points here

Desk law that did **not** change: skip centralised stables for dapps; no fourth 402 envelope; compiler pin SilverScript v1.0.0; `#234` / `#243` / `#249` still open holes.

---

## What to do next

| Who | Do |
| --- | --- |
| Luke | Read battle 1. File or accept [docs/02-windows-repro.md](docs/02-windows-repro.md). Add `.gitattributes`. Do not take “v1” until the gates you already wrote are closed. |
| This desk | Bind the envelope. Do not invent a fourth 402. Keep Ishum a till. Keep kUSD off the keypad until capital exists — and even then it is not this `asset`. |
| Anyone else | HTTP 402 ≠ x402. Native KAS ≠ USDC. `accepted` ≠ irreversible. |

---

## Sister links

| Repo | What |
| --- | --- |
| https://github.com/elldeeone/kaspa-x402 | Subject |
| https://github.com/STP-KAS/kaspa-master-file | Pin list |
| https://github.com/STP-KAS/ishum | The till |
| https://github.com/STP-KAS/x402-vs-grok | Short battle 1 |
| https://github.com/STP-KAS/x402-ishum | Short battle 2 |
| https://github.com/STP-KAS/402-is-not-x402 | Short catalog |
