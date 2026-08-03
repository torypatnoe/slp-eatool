# Cycle 1

**Status:** Restarted 2026-08-03. Bet re-cut around direct QuickBooks API import (core), superseding the file-output-first framing below (kept for history). Spec being updated to match; some derivation rules now captured, exact chart-of-accounts mapping and a real Stripe sample still needed before Build.

## The Bet (restarted)

*Rewritten 2026-08-03, superseding the 2026-07-06 Bet below.*

- **Milestone:** M3 — Automate step 3, writing transactions directly into QuickBooks Online via the Accounting API. Pulled forward as Cycle 1's committed core (was an optional stretch under the original Bet). The transformation/derivation logic originally scoped as M1 is still built — it now runs in-memory as the step that prepares each transaction before the API write, rather than shipping as a standalone `import.xlsx` deliverable.
- **Goal:** Given a real monthly Stripe Excel export (~1,000 transactions), `slp-eatool` transforms and writes the transactions directly into QuickBooks Online via the Accounting API — no manual massaging, no manual import, no SaasAnt. Sandbox company first; live-company writes are a separate, deliberate release gated behind an explicit flag.
- **Appetite:** **Large** (up to the 6-week ceiling). OAuth setup, rotating-refresh-token persistence, entity-mapping logic, and partial-failure handling are all now committed core, not stretch.
- **In scope:**
  - Capturing the exact massaging/derivation rules — including the **derivations SaasAnt's 1:1 mapping can't do** — and encoding them as the in-memory transform step
  - OAuth 2.0 setup against a QuickBooks Online **sandbox** company; secure, rotating-refresh-token persistence (rotates ~daily; see [research/qb-research.md](../research/qb-research.md))
  - Writing transactions to QuickBooks Online via the Accounting API using the entity combination below
  - Handling the ~1,000-transactions/month volume (batch endpoint, rate limits, partial-failure reporting)
  - **Secondary/optional:** a `--file` mode that writes `import.xlsx` instead of calling the API — useful for dry-run/audit before trusting the live write, not the primary deliverable this cycle
- **NOT in scope:**
  - Automating the Stripe export (step 1) — remains manual this cycle (later milestone; needs Stripe API research)
  - Live-company import as a default — sandbox-first and flag-gated; writing to SLP's real books is a separate, deliberate release
  - Report generation out of QuickBooks
- **DRI:** Tory Patnoe

---

### Superseded Bet (2026-07-06, kept for history)

- **Milestone:** M1 — Automate the Stripe-export → QuickBooks-Online-format massaging step
- **Goal:** Given a real monthly Stripe Excel export (~1,000 transactions), `slp-eatool` produces a QuickBooks-Online-importable file matching what an SLP bookkeeper would produce by hand — correctly and repeatably — eliminating the manual reshaping step (**and removing SaasAnt from the loop**, per Option B). **Stretch:** an *optional* flag that imports the result directly into QuickBooks Online via the Accounting API (sandbox-first).
- **Appetite:** *To confirm* once the massaging rules are detailed. The core massage-to-file path is Small (1–2 weeks); the optional API import (OAuth, token persistence, entity mapping) pushes the cycle toward **Large** — likely gated so the file path ships even if the import isn't finished.
- **In scope:**
  - Capturing the exact massaging rules — including the **derivations SaasAnt's 1:1 mapping can't do** — from a real Stripe export + a known-good hand-massaged result
  - Transforming the raw Stripe Excel export into a QuickBooks-Online-importable file (SaasAnt removed)
  - Handling the ~1,000-transactions/month volume
  - **Optional (stretch):** direct import into QuickBooks Online via the Accounting API, behind a flag, tested against a sandbox company first — see [research/qb-research.md](../research/qb-research.md)
  - **If the optional import is built:** OAuth 2.0 setup and secure, rotating-refresh-token handling
- **NOT in scope:**
  - Automating the Stripe export (step 1) — remains manual this cycle (later milestone; needs Stripe API research)
  - Live-company import as a default — the optional import is sandbox-first and flag-gated; writing to SLP's real books is a separate, deliberate release
  - Report generation out of QuickBooks
- **DRI:** Tory Patnoe

## Workflow Details

**Entity representation (QuickBooks side) — captured 2026-08-03:** a combination of entities per Stripe transaction, not a single type:
- **SalesReceipt** for the gross sale (income + payment).
- **JournalEntry** for the Stripe fee (debit a Stripe Fees expense account, credit the clearing/undeposited-funds account the SalesReceipt landed in).
- **Deposit** for the net payout landing in the bank account.

*This is the working synthesis from the DRI's answer ("fees go through a JournalEntry") plus the pattern research/qb-research.md flagged as common. Not yet validated against a real Stripe export — first Build task.*

**Derivation rules captured 2026-08-03:**
- **Fees:** recorded via a **JournalEntry** (Stripe fee expense debit, clearing-account credit) — separate from the SalesReceipt for the gross sale.
- **Refunds:** represented as **sign-flipped** entries (amount negated) relative to the original sale. *Open detail: which entity carries the flipped sign — a reversing JournalEntry, a CreditMemo, or a negative SalesReceipt line — to confirm against a real refund example.*
- **Categories/accounts:** Stripe categories are **mapped into QuickBooks categories/accounts** via a lookup. *Open detail: the actual Stripe-category → QuickBooks-account mapping table has not been enumerated yet — needs SLP's chart of accounts.*

**Still needed before Build can finish the transform + write logic (first tasks of this cycle):**
1. A real Stripe Excel export + a known-good hand-massaged/hand-entered result, to validate the entity combination and derivation rules above line-by-line.
2. The full Stripe-category → QuickBooks-account mapping table (from SLP's chart of accounts).
3. Confirmation of the refund entity type.
4. Edge cases present in a real month: multi-currency, disputes/chargebacks — whether they occur and how to represent them.

## Validation Log

*Raw feedback accumulated during the cycle, one entry per review. Distilled into the Measure's learning note at cycle close.*

### Review 1 — SLP Bookkeeping interview, 2026-07-06
- **Source:** SLP Bookkeeping team
- **Feedback:**
  - Current workflow: manually export Stripe → Excel, then use **SaasAnt Transactions (Online)** to import into QuickBooks Online. It's "new every time" — the column mapping / massaging is redone by hand each month.
  - SaasAnt actually *does* save/reuse mappings, but its mapping is 1:1 (column → field) and can't compute/derive values — so the recurring manual work is real *derivation* logic SaasAnt can't express. That derivation is the gap `slp-eatool` fills. See [research/saasant-incumbent-tool.md](../research/saasant-incumbent-tool.md).
  - SaasAnt credits are per-line (1 credit/transaction imported); ~1,000/month sits inside the Launch plan — cost is small today but scales linearly.
- **Disposition:** **Decision — Option B (build a custom Go tool, remove SaasAnt from the loop).** Rationale and trade-offs recorded in [research/saasant-incumbent-tool.md](../research/saasant-incumbent-tool.md); shape sharpened to "derivations SaasAnt can't do"; top open risk = whether QBO native file-import covers the needed transaction type or forces the QBO API (M3) sooner.

### Review 2 — Cycle restart, 2026-08-03
- **Source:** Tory Patnoe (DRI)
- **Feedback:** Restarting Cycle 1 with the QuickBooks API import promoted from optional stretch to the committed core (see restarted Bet above). Provided the entity representation ("a combination") and initial derivation rules: fees via JournalEntry, refunds as sign-flipped entries, categories mapped from Stripe into QuickBooks.
- **Disposition:** Bet rewritten; original Bet kept below it as superseded history. Shape and Spec being updated to match. Full derivation detail (chart-of-accounts mapping, refund entity type, edge cases) still needs a real Stripe export — logged as the first Build tasks.

## The Measure

*To be written when the cycle closes.*

- **Result against the goal:** —
- **Learning note:** —
- **Next cycle input:** —
