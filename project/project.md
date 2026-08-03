# Project: slp-eatool

**DRI:** Tory Patnoe

## Customer Narrative

See [customer-narrative.md](customer-narrative.md) — the SLP Bookkeeping team automating the movement of data in and out of QuickBooks Online for the East Angles Foundation's monthly close, starting with Stripe.

## Milestone Map

| Milestone | Description | Status |
|-----------|-------------|--------|
| M1 | Massaging/derivation logic for the Stripe Excel export (~1,000 txns/month) | Absorbed into Cycle 1 — built as an in-memory step feeding M3's API write, not a standalone file deliverable |
| M2 | Automate step 1 — pull Stripe transactions directly via the Stripe API (remove the manual Excel export) | Rough |
| M3 | Automate step 3 — write to QuickBooks Online directly via its API (remove the manual import) → end-to-end Stripe close | **In progress (Cycle 1) — pulled forward as the cycle's committed core** |
| M4+ | Extend the pattern to the next source system, and to report generation out of QuickBooks | Hypothesis |

*Cycle 1 restarted 2026-08-03: direct QuickBooks API import (M3) is now the committed core, not an optional stretch. M1's massaging/derivation logic is still built, but runs in-memory ahead of the API write rather than shipping as a standalone file — file output survives only as a secondary `--file` dry-run/audit mode. Build-vs-buy resolved to **build** (SaasAnt removed — see [research/saasant-incumbent-tool.md](../research/saasant-incumbent-tool.md)). M2 remains rough; M4+ remain hypotheses. See [research/qb-research.md](../research/qb-research.md).*

## Project Hill Chart

*Left side (figuring it out) / right side (making it happen).*

Current position: **left side.** The customer, the QuickBooks Online target, the three-step Stripe process, and the ~1,000/month scale are known. The entity representation (SalesReceipt + JournalEntry + Deposit) and initial derivation rules are now captured. Still being discovered: the exact chart-of-accounts mapping, the refund entity type, remaining edge cases, what a "done" monthly close and its reports require, and the Stripe API mechanics for M2.

## Cycle History

| Cycle | Bet | Status |
|-------|-----|--------|
| [1](../cycles/cycle-1.md) | Restarted 2026-08-03: direct QuickBooks API import (build, SaasAnt removed) as committed core; massaging/derivation logic built in-memory to feed it | Bet rewritten; Spec updated; awaiting real Stripe sample + chart-of-accounts mapping before Build |

*Each cycle's full record (Bet + Measure) lives in `cycles/cycle-N.md`.*
