# Spec: slp-eatool

## Changelog
- Cycle 1 (2026-08-03, restart — API import promoted to core): Direct QuickBooks Online API import is now the tool's **default, committed behavior**, not an opt-in flag. Default target is the **sandbox** company; writing to the live company requires an explicit `--live` flag. File output moves behind a new `--file` flag for dry-run/audit use. Entity representation captured: SalesReceipt (sale) + JournalEntry (fee) + Deposit (payout); see [cycles/cycle-1.md](../cycles/cycle-1.md). Security posture updated accordingly: secrets are now required for the default path, not just an optional one.
- Cycle 1 (2026-07-06, Option B + optional import): SaasAnt removed from the workflow — the tool targets QuickBooks Online directly. Added an **optional, flag-gated direct import** via the QuickBooks Online Accounting API (OAuth 2.0, sandbox-first). Security posture updated: secrets are now in scope for the import path (Stripe key, Intuit client id/secret, rotating QBO refresh token). See [research/qb-research.md](../research/qb-research.md) and [research/saasant-incumbent-tool.md](../research/saasant-incumbent-tool.md).
- Cycle 1 (2026-07-06): Initial spec — CLI contract, Go/single-binary runtime, I/O formats, scale, scriptability, and security pinned down. The detailed massaging transformation rules are still pending from SLP Bookkeeping; the transformation acceptance criteria below are placeholders until those land.

---

**Milestone:** M3 — Automate step 3, writing Stripe transactions directly into QuickBooks Online via the Accounting API (from the restarted [Cycle 1 Bet](../cycles/cycle-1.md)). Pulled forward as the cycle's committed core; the M1 massaging/derivation logic is still built, but now runs in-memory ahead of the API write rather than shipping as a standalone file.

**DRI:** Tory Patnoe

**Scope reminder:** Cycle 1's committed core is now **direct QuickBooks Online API import**: transform the Stripe export and write the transactions straight into QuickBooks. **SaasAnt is removed from the workflow (Option B).** The Stripe export (step 1) remains manual. Default target is the **sandbox** company; the live company requires an explicit flag. A secondary `--file` mode writes a QuickBooks-Online-importable file instead, for dry-run/audit — it is not the primary deliverable. See [shape.md](shape.md), [research/qb-research.md](../research/qb-research.md).

---

## The Tool

A single-command CLI, distributed as a self-contained binary:

```
slp-eatool <stripe-export.xlsx>              →  transforms and writes transactions into QuickBooks Online sandbox (default)
slp-eatool <stripe-export.xlsx> --live       →  writes to the live QuickBooks Online company (explicit, deliberate)
slp-eatool <stripe-export.xlsx> --file       →  writes import.xlsx instead of calling the API (dry-run/audit; no network calls)
```

- **Language / runtime:** Go. Ships as one statically-linked binary with **no runtime dependencies** on the target machine (nothing to `install`). Built from one source tree for **macOS** (Apple Silicon + Intel) and **Linux** (amd64) via cross-compilation.
- **Input:** the Stripe transactions Excel export (`.xlsx`), as produced today by the manual Stripe export (step 1).
- **Output (default):** the transformed transactions are written directly into a QuickBooks Online **sandbox** company via the Accounting API (OAuth 2.0). Each Stripe transaction becomes a combination of entities — **SalesReceipt** (sale) + **JournalEntry** (fee) + **Deposit** (net payout); refunds sign-flipped; categories mapped from Stripe to QuickBooks accounts. See [cycles/cycle-1.md](../cycles/cycle-1.md) Workflow Details and [research/qb-research.md](../research/qb-research.md).
- **`--live`:** writes to SLP's real QuickBooks Online company instead of the sandbox. Off by default — a separate, deliberate switch from sandbox validation.
- **`--file` (secondary/optional):** writes `import.xlsx` in the QuickBooks Online import format instead of calling the API — no network calls, no credentials needed. Useful for dry-run/audit, not the primary path this cycle.
- **Invocation:** non-interactive. The default (API) path needs credentials supplied via environment/keychain (see Security); `--file` needs no config and runs unattended with nothing configured.

---

## Acceptance Criteria (observable outcomes)

**Confirmed now (runtime / contract):**
1. On a clean macOS machine with nothing installed, the downloaded binary runs and produces output — no interpreter, no package install, no `PATH` setup beyond the binary itself.
2. Runs non-interactively and returns **exit code 0 on success, non-zero on any failure** (unreadable/invalid input, unexpected columns, API error) with a clear one-line error to stderr — so it is safe to call from an automation script.
3. Comfortably processes a monthly batch of **~1,000 transactions** (design headroom to a few thousand) within QuickBooks' batch/rate limits (30 ops/batch, 500 req/min, 120 req/min on the batch endpoint).
4. Invalid or unexpected input is rejected with an actionable message rather than silently creating wrong transactions.

**Core — direct API import (`slp-eatool <stripe-export.xlsx>`, default):**
5. Given a real Stripe export, the tool creates the correct combination of entities (SalesReceipt + JournalEntry + Deposit per transaction) in a QuickBooks Online **sandbox** company, matching what an SLP bookkeeper would otherwise have produced by hand via SaasAnt + manual import.
6. Credentials are read from environment/keychain (never hardcoded/committed); the rotating QuickBooks refresh token is **persisted back** after each run so unattended runs keep working.
7. Partial-failure behaviour is safe: an API error reports which transactions succeeded/failed and returns non-zero, without silently double-importing on re-run.
8. Documented edge cases — *refunds (sign-flipped), Stripe fees/payouts, multi-currency, disputes/chargebacks* — are transformed and written correctly (or explicitly reported when they can't be).
9. `--live` writes to SLP's real QuickBooks Online company instead of the sandbox; off by default, requires an explicit switch.

*Criteria 5 and 8 are finalised once the chart-of-accounts mapping, refund entity type, and remaining edge cases are captured in [cycles/cycle-1.md](../cycles/cycle-1.md) → Workflow Details, against a real Stripe export + known-good result.*

**Secondary — file output (`--file`):**
10. With `--file`, behaviour is file-only — no credentials required, no network calls — and produces `import.xlsx` in the QuickBooks Online import format; the same command, same input, produces byte-identical output on macOS and on Linux.

---

## Success Metric

The manual reshaping *and* the manual import of ~1,000 Stripe rows each month are **both eliminated**: the bookkeeper runs one command and the transactions are already in QuickBooks, spending zero time hand-massaging or hand-importing data. Measured at the next monthly close by (a) the sandbox run producing correct, complete transactions with no manual correction and (b) the close reconciling as it did before. Secondary: reshaping + import time per close drops from *[current baseline — capture from SLP]* to effectively zero.

---

## Feature Flag Plan

This is a distributed CLI binary, not a deployed service, so there's no server flag — but two flags gate risk: **`--live`** is the meaningful gate on the API path — direct writes target the **sandbox** company by default, and only switch to SLP's real books with this explicit, deliberate flag. **`--file`** is the roll-back path — it drops back to the untouched file-output behavior (no network, no credentials) if the API path needs to be bypassed. This keeps deploy (build the binary) separate from release (let it write to SLP's real books), and keeps the tool safely abandonable at any stage.

---

## Scale Considerations

- **Volume:** ~1,000 transactions/month; design for a few thousand without concern. No pagination or streaming needed at this size — the whole file fits in memory.
- **Persistence:** none. The tool is a pure file-in → file-out transform; it stores nothing between runs.
- **Rate limits:** none this cycle (no network calls). Relevant only when the Stripe/QuickBooks APIs are added in M2/M3.

---

## Security Sign-off

- **Default path — secrets now required.** The default (API import) path holds: the **Intuit** client id/secret and the **rotating QuickBooks Online refresh token** (plus a Stripe API key once M2 pulls the export side in). Requirements:
  - Secrets loaded from **environment variables or an OS keychain / restricted-permission config file** — never hardcoded, never committed to git.
  - The QuickBooks **refresh token rotates ~daily and must be persisted** back to that store on every run (access token 60 min; refresh token 100-day window). If unused for 100 days, SLP re-consents.
  - Auth is **OAuth 2.0** — SLP grants one-time consent; the tool never holds SLP's QuickBooks login. Access is revocable by SLP without a password change.
  - **Sandbox company by default** for all development and normal runs; live-company writes gated behind the explicit `--live` switch.
- **`--file` path — unchanged fallback:** operates on **local files only — no network, no transmission, no telemetry.** No secrets, no credentials. Financial data never leaves the machine.
- **Dependencies:** minimise third-party Go modules (realistically `excelize` for xlsx when `--file` is used, plus a thin HTTP/OAuth client from the stdlib); a small, auditable dependency surface for a static binary.
- **`.gitignore`** must exclude any local secrets/token files from the outset.

---

## Organization Standards Check

No standing organization-standard documents exist for this project yet (security requirements, architecture standards, compliance rules). Nothing to check against; recorded here so the gate is answered, not skipped. The choices above (local-only, no persistence, static binary, minimal deps) are the de-facto standard this project sets.

---

## Open Items (resolve before criteria 5 and 8 are final)
- The exact **chart-of-accounts mapping**: which QuickBooks account each Stripe category maps to (from SLP's chart of accounts) → capture in [cycles/cycle-1.md](../cycles/cycle-1.md).
- Confirm the **refund entity type** — reversing JournalEntry, CreditMemo, or negative SalesReceipt line (sign-flip direction is confirmed; the entity carrying it is not).
- Enumerate the remaining **edge cases** present in a real month (multi-currency, disputes/chargebacks) and the required handling for each.
- A real Stripe export + known-good hand-produced result, to validate the SalesReceipt + JournalEntry + Deposit combination end-to-end.
- Confirm the **current baseline** time spent massaging + importing per close, for the success metric.
- Where **secrets live** on the Mac (dev) and the future Linux host.

~~How Stripe activity should be represented in QuickBooks~~ — **resolved 2026-08-03:** a combination of SalesReceipt (sale) + JournalEntry (fee) + Deposit (payout). See [cycles/cycle-1.md](../cycles/cycle-1.md).
