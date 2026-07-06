# Project: slp-eatool

**DRI:** Tory Patnoe

## Customer Narrative

See [customer-narrative.md](customer-narrative.md) — the SLP Bookkeeping team automating the movement of data in and out of QuickBooks Online for the East Angles Association's monthly close, starting with Stripe.

## Milestone Map

| Milestone | Description | Status |
|-----------|-------------|--------|
| M1 | Automate step 2 — massaging the Stripe Excel export into the QuickBooks Online import format (~1,000 txns/month) | In progress (Cycle 1) |
| M2 | Automate step 1 — pull Stripe transactions directly via the Stripe API (remove the manual Excel export) | Rough |
| M3 | Automate step 3 — write to QuickBooks Online directly via its API (remove the manual import) → end-to-end Stripe close | Hypothesis |
| M4+ | Extend the pattern to the next source system, and to report generation out of QuickBooks | Hypothesis |

*Only M1 is specced. M2 is rough; M3+ are hypotheses — kept deliberately loose until the preceding milestone ships and is measured. The Stripe/QuickBooks API research (narrative open question #3) is the main unknown gating M2/M3.*

## Project Hill Chart

*Left side (figuring it out) / right side (making it happen).*

Current position: **left side.** The customer, the QuickBooks Online target, the three-step Stripe process, and the ~1,000/month scale are known. Still being discovered: the exact data-massaging rules (captured in Cycle 1), what a "done" monthly close and its reports require, and the Stripe/QuickBooks API mechanics for later milestones.

## Cycle History

| Cycle | Bet | Status |
|-------|-----|--------|
| [1](../cycles/cycle-1.md) | Automate the Stripe-export → QuickBooks-format massaging step | Opening |

*Each cycle's full record (Bet + Measure) lives in `cycles/cycle-N.md`.*
