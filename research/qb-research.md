# Research: QuickBooks Online — Import APIs & Credentials

**Question:** With SaasAnt removed (Option B), how does `slp-eatool` get transactions *into* QuickBooks Online programmatically, and what credentials does that require? Cycle 1 is expanding to include an **optional** direct import.

**Source:** Intuit developer documentation + 2026 integration guides (cited below).

---

## TL;DR

- QuickBooks Online has **no "import API"** in the CSV sense. The CSV/QBO/QFX bank-transaction import is **UI-only** (manual, in-app) — there is no endpoint to drive it.
- The programmatic path is the **QuickBooks Online Accounting API** (REST + JSON, OAuth 2.0): you *create transaction entities* (SalesReceipt, JournalEntry, Deposit, Purchase, etc.) directly. That is the real "import" for a custom tool.
- Auth is **OAuth 2.0**, not a username/password. We register **one** app (client id/secret); SLP grants consent **once**; we receive tokens tied to their company (`realmId`). **We never hold SLP's QuickBooks login.**
- The operationally important catch for unattended automation: the **refresh token rotates ~daily** and must be persisted after every run (details below).

---

## Two ways in

### 1. Native file import (UI-only) — *not automatable*
QuickBooks Online can import a list of transactions from **CSV / QBO / QFX** through the banking UI (3-column: Date, Description, Amount; or 4-column: Date, Description, Credit, Debit; up to ~1,000 lines/upload). This is exactly the manual path SLP has today. **There is no API to trigger it**, so it does not serve an automated import. It stays the fallback if the API path is deferred.

### 2. QuickBooks Online Accounting API — *the automatable path*
REST/JSON API where you **create accounting objects**. For Stripe activity the candidate entities are:

- **SalesReceipt** — an immediate sale with payment (income + the item sold).
- **Deposit** — money landing in an account, with line items (useful for Stripe payouts).
- **JournalEntry** — explicit debit/credit lines against Chart-of-Accounts accounts (debits must equal credits). Most flexible; good for representing gross sale + Stripe fee + net payout in one balanced entry.
- **Purchase / Expense** — for the Stripe processing fees as an expense.

**Which entity is correct depends on how SLP wants Stripe activity represented in the books — this is an open design decision, tied to the massaging rules.** (A common pattern: JournalEntry or SalesReceipt for the sale + a fee expense + a Deposit/transfer for the payout.) Note: this creates *real accounting transactions*, not "bank feed" items to review — the public API has no third-party bank-feed ingestion.

**Batch & limits:**
- **Batch endpoint:** up to **30 operations per request** (≈34 batches for 1,000 txns).
- **Rate limits:** **500 requests/min per company (realmId)**, ~10 concurrent, **120 req/min on the batch endpoint**; over-limit returns HTTP 429 → use exponential backoff with jitter. At ~1,000 txns/month, limits are a non-issue.
- **Writes are free**; only heavy *read* metering exists (not relevant to import).
- Every entity carries a **SyncToken** that must be current on update, else HTTP 400.

---

## Credentials & auth (the "app tokens" question)

You need **two different kinds of secret**, one per side of the pipeline:

### QuickBooks Online — OAuth 2.0 (Authorization Code flow)
1. Register a **QuickBooks app** on the Intuit Developer portal → get a **Client ID + Client Secret** (owned by the tool/us, not SLP).
2. SLP goes through the consent screen **once**, authorizing the app against their company. We receive an **access token + refresh token** bound to their **`realmId`**.
3. **Token lifetimes** (design-critical for a Linux cron host):
   - **Access token: 60 minutes.**
   - **Refresh token: 100 days — but rotates ~every 24h.** Each refresh call returns a *new* refresh token and expires the old one. **Store the latest refresh token after every run.** If the tool doesn't run for 100 days, the token dies and SLP must re-consent.
4. A **sandbox company** is available for development before touching SLP's real books.

> **Key clarification for SLP:** this is *not* their username/password. It's a one-time OAuth consent that issues revocable tokens scoped to their company. They can revoke access anytime from QuickBooks without changing their password.

### Stripe — API key (for the export side, M2)
Stripe uses a **secret API key**, not OAuth. Use a **restricted key with read-only** access to Charges/Balance Transactions/Payouts — that's all the export needs. Simpler than the QBO side.

---

## Implications for `slp-eatool` (Go)

- **No official Intuit Go SDK.** The REST API is straightforward from Go's `net/http`; community QBO libraries exist but a thin custom client keeps the dependency surface small (matches the "minimal deps, static binary" standard). We own the OAuth refresh + token persistence logic.
- **Secret handling becomes in-scope** the moment optional import lands. Needed: Stripe key, Intuit client id/secret, and the rotating QBO refresh token. These must be:
  - loaded from **environment variables or an OS keychain / a restricted-permission config file** — never hardcoded, never committed;
  - the **refresh token written back** to that store on every run (it rotates).
  This changes the Spec's security posture from "no secrets this cycle" to "secrets required for the optional import path."
- **Keep the import optional / flagged.** Default output stays the file (works today, no auth). The API import is the stretch: build and test against the **sandbox** first, gate the real-company import behind an explicit flag, and treat writing to SLP's live books as a separate, deliberate release step.

---

## Open decisions this raises
1. **How should Stripe activity be represented in QBO?** SalesReceipt vs JournalEntry vs Deposit(+fee expense) — decides which API entities the import writes. Confirm with SLP; ties directly to the massaging rules.
2. **Where do secrets live** on the Mac (dev) and the future Linux host (automation)? Env vars vs keychain vs restricted file.
3. **Is the optional import in Cycle 1 sandbox-only, or live-company?** Recommend sandbox-only for Cycle 1; live import as its own release once verified.

---

## Sources
- [Intuit — Set up OAuth 2.0](https://developer.intuit.com/app/developer/qbo/docs/develop/authentication-and-authorization/oauth-2.0)
- [Intuit — OAuth 2.0 & authorization FAQ](https://developer.intuit.com/app/developer/qbo/docs/develop/authentication-and-authorization/faq)
- [Intuit — Refresh Token Expiration and Validity Policy](https://help.developer.intuit.com/s/article/Validity-of-Refresh-Token)
- [Intuit — JournalEntry API reference](https://developer.intuit.com/app/developer/qbo/docs/api/accounting/all-entities/journalentry)
- [Intuit — Deposit API reference](https://developer.intuit.com/app/developer/qbo/docs/api/accounting/all-entities/deposit)
- [QuickBooks Online API Guide 2026: OAuth, Endpoints & Rate Limits (Satva)](https://satvasolutions.com/blog/quickbooks-online-api-guide)
- [Top 5 QuickBooks API Rate Limits 2026 (Satva)](https://satvasolutions.com/blog/quickbooks-online-api-limitations-guide)
- [Intuit — Format CSV files to import bank transactions](https://quickbooks.intuit.com/learn-support/en-us/help-article/bank-transactions/format-csv-files-excel-get-bank-transactions/L4BjLWckq_US_en_US)
