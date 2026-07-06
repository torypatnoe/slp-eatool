# Shape: slp-eatool — Stripe → QuickBooks Data Movement

## Changelog
- Cycle 1 (2026-07-06): Initial shape — automate step 2 (massaging the Stripe Excel export into the QuickBooks Online import format). Manual export (step 1) and manual import (step 3) remain in place as the tool's input and output for this cycle.

---

**Purpose:** Define the solution shape for automating SLP Bookkeeping's Stripe → QuickBooks Online workflow and record the reasoning behind each decision. Living document — grows as later cycles automate the export and import steps. Per-cycle commitments (appetite, scope boundary, DRI) live in the cycle record, not here.

---

## The Slice This Shape Currently Bets On (Cycle 1)

The monthly Stripe → QuickBooks Online process is three manual steps: **export** (Stripe → Excel), **massage** (reshape into QuickBooks format), **import** (into QuickBooks Online). Cycle 1 automates only the **massage** step — the most repetitive and error-prone of the three at ~1,000 transactions/month.

### Rough solution narrative

The bookkeeper still exports the Stripe transactions to Excel by hand and still imports the result into QuickBooks Online by hand — those ends are unchanged this cycle. In between, instead of reshaping ~1,000 rows manually, they hand the raw Stripe export to `slp-eatool`, which applies the massaging rules and produces a file in exactly the format QuickBooks Online's import expects. The manual reshaping step disappears; what remains at the seams is a plain export in and a ready-to-import file out.

Success for this slice: given a real Stripe Excel export, the tool produces a QuickBooks-Online-importable file that a bookkeeper would otherwise have produced by hand — correctly and repeatably.

### Identified unknowns
- **The exact massaging rules** — the precise column mapping and transformations from the Stripe export format to the QuickBooks Online import format. To be captured in Cycle 1 from a real export + a known-good hand-massaged result.
- **The QuickBooks Online import format** — CSV layout and field requirements the import expects (which fields are mandatory, date/amount formatting, account/category columns).
- **Edge cases in the data** — refunds, fees/payouts, multi-currency, disputes — whether they appear in the ~1,000 and how they must be represented.

### Alternatives considered
- **Automate the whole pipeline now (Stripe API → transform → QuickBooks API).** Rejected for Cycle 1: it forces the Stripe-export and QuickBooks-import API research (narrative open question #3) up front and couples three unknowns together. Automating the massaging step alone removes the biggest manual burden immediately and leaves clean seams to automate the export (M2) and import (M3) later.
- **A one-off spreadsheet macro / template.** Rejected: it hard-codes the current format, resists reuse as rules change, and doesn't establish a base the later API-driven milestones can build on.
