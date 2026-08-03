# Research: SaasAnt — Incumbent Tool

**Source:** SLP Bookkeeping interview (2026-07-06) + SaasAnt official docs (cited below).
**Why it matters:** SaasAnt is the tool SLP uses today for the Stripe → QuickBooks Online step. Understanding what it does — and what it can't — defines the gap `slp-eatool` fills and drove the build-vs-buy decision for Cycle 1.

---

## What SLP does today

They **manually export** Stripe transactions to Excel, then use **SaasAnt Transactions (Online)** to import that file into QuickBooks Online. Reported pain: it is effectively **"new every time"** — they have to set up the column mapping / massaging by hand on each monthly run.

## What SaasAnt actually is

SaasAnt (saasant.com) ships two relevant products:

- **SaasAnt Transactions (Online)** — file-based import/export/modify/delete for QuickBooks Online. Imports XLS/XLSX/CSV/IIF (and more) with a **field-mapping** step. This is what SLP uses.
- **PayTraQer** — API-based auto-sync that connects Stripe directly to QuickBooks Online and syncs sales, **fees, payouts, refunds, taxes, and customers** on a schedule. Essentially a productized end-to-end Stripe→QBO pipeline.

## Key finding 1 — SaasAnt *does* save and reuse mappings

Contrary to the "new every time" experience, SaasAnt Transactions Online **saves and reuses column mappings** (and auto-remembers the last one). A saved mapping only sticks if the **input file's shape is constant** month to month. Its mapping is **1:1** — "file column → QuickBooks field." It can rename/place columns; it **cannot compute or derive** values.

**Diagnosis:** SLP re-maps every month because the work they do is *derivation*, not renaming — the kind of transform SaasAnt's 1:1 mapping fundamentally can't express (e.g. splitting Stripe gross into sales/fees/payouts, sign flips for refunds, deriving accounts/categories). The manual Excel massage before import is the tell. **This derivation gap is precisely what `slp-eatool` exists to close.**

## Key finding 2 — Credit model is per-transaction, not per-job

A SaasAnt "credit" is a **per-line-item** unit, not a per-import-job unit:

| Action | Credit cost |
|---|---|
| Import / Modify | **1 credit per line item** |
| Export | 0.1 per transaction |
| Delete | 0.5 per transaction |
| Live Edit | 1 per line |
| Reports | 1 per report |

Monthly allowances: Trial 100 · **Launch 5,000** · Scale 50,000 · Automate 100,000. Credits reset monthly, no rollover.

**Cost implication at SLP's ~1,000 txns/month:** importing ~1,000 transactions ≈ ~1,000 credits/month — comfortably inside the Launch plan. Re-runs recharge (a delete + re-import of 1,000 rows ≈ 1,000 + 500 + 1,000 credits), so getting the file right the first time has direct dollar value. Cost is **small today** but scales **linearly** with volume and added source systems.

## Build-vs-buy options considered

| Option | Description | Verdict |
|---|---|---|
| **A — Buy** | Lean fully on SaasAnt (saved mapping, or PayTraQer auto-sync) | Rejected — SaasAnt's 1:1 mapping can't do the required derivations; PayTraQer's automatic mapping is unlikely to match the Foundation's chart-of-accounts |
| **C — Hybrid** | Go tool does the derivation into a constant file; SaasAnt still imports | Rejected for now — still depends on a paid third party and its per-line credit cost; keeps SaasAnt in the loop |
| **B — Build** | Custom Go tool produces a QuickBooks-Online-importable file directly; **SaasAnt removed from the loop** | **Chosen (2026-07-06)** |

## Decision (2026-07-06): Option B

Build a custom Go tool and take SaasAnt out of the workflow.

**Rationale:** full control over the derivation logic SaasAnt can't express; no per-line metered cost as volume grows; a foundation that extends cleanly to M2 (Stripe API export) and M3 (QuickBooks API import). The team chose long-term control over the lowest-short-term-effort path (SaasAnt at ~1,000/month is cheap and low-risk — this is a deliberate trade of effort now for control and headroom later).

**Trade-offs accepted:**
- More to build and maintain than configuring an incumbent tool.
- We now own the QuickBooks Online import path. **Open risk:** QBO's *native* file import is limited (mainly bank transactions via CSV and list imports). If the transaction type SLP needs isn't natively file-importable, Option B may force the **QuickBooks Online API (M3) earlier than planned**. This is the top technical unknown to resolve while writing the Spec.

---

## Sources
- [PayTraQer — Stripe / QuickBooks Online integration](https://www.saasant.com/integrations/paytraqer-stripe-quickbooks-online-integration/)
- [How to map files to QuickBooks fields in SaasAnt Transactions](https://www.saasant.com/blog/how-to-map-file-in-saasant-transactions-field-mapping/)
- [How to map a file in SaasAnt Transactions (Online)](https://support.saasant.com/support/solutions/articles/14000053042-how-to-map-a-file-in-saasant-transactions-online-/)
- [Understanding the Credit Structure in SaasAnt Transactions Online](https://support.saasant.com/support/solutions/articles/understanding-credit-structure-saasant-transactions-online/)
- [Pricing for SaasAnt Transactions (Online)](https://support.saasant.com/support/solutions/articles/14000067369-pricing-for-saasant-transactions-online-/)
