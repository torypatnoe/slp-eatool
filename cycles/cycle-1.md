# Cycle 1

**Status:** Opening — Shape gate. Bet written against the initial Shape; appetite and detailed massaging rules to be confirmed with the SLP Bookkeeping team.

## The Bet

*Written 2026-07-06.*

- **Milestone:** M1 — Automate the Stripe-export → QuickBooks-Online-format massaging step
- **Goal:** Given a real monthly Stripe Excel export (~1,000 transactions), `slp-eatool` produces a QuickBooks-Online-importable file matching what an SLP bookkeeper would produce by hand — correctly and repeatably — eliminating the manual reshaping step. The manual Stripe export and manual QuickBooks import remain in place as the tool's input and output.
- **Appetite:** *To confirm* once the massaging rules are detailed (provisional: Small, 1–2 weeks).
- **In scope:**
  - Capturing the exact massaging rules from a real Stripe export + a known-good hand-massaged result
  - Transforming the raw Stripe Excel export into the QuickBooks Online import format
  - Handling the ~1,000-transactions/month volume
- **NOT in scope:**
  - Automating the Stripe export (step 1) — remains manual this cycle (later milestone; needs Stripe API research)
  - Automating the QuickBooks import (step 3) — remains manual this cycle (later milestone; needs QuickBooks API research)
  - Report generation out of QuickBooks
- **DRI:** Tory Patnoe

## Workflow Details

*To be provided — the step-by-step massaging rules (column mapping and transformations from the Stripe export to the QuickBooks Online import format), captured from a real export and a known-good result. These feed the Spec.*

## Validation Log

*Raw feedback accumulated during the cycle, one entry per review. Distilled into the Measure's learning note at cycle close.*

### Review 1 — SLP Bookkeeping (pending)
- **Source:** —
- **Feedback:** —
- **Disposition:** —

## The Measure

*To be written when the cycle closes.*

- **Result against the goal:** —
- **Learning note:** —
- **Next cycle input:** —
