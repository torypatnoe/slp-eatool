# slp-eatool

> A project managed under the [Forma](../forma-program-management) program-management methodology.

## Status

**Cycle 1 opening.** `slp-eatool` is an automation tool for **SLP Bookkeeping** that automates moving data in and out of **QuickBooks Online** for the **East Angles Association**'s monthly close — starting with the **Stripe → QuickBooks** workflow. Cycle 1 automates the "massaging" step (reshaping the Stripe Excel export into the QuickBooks import format, ~1,000 txns/month); the export and import steps stay manual until later cycles. Next: capture the exact massaging rules, then write the Spec.

**DRI:** Tory Patnoe

## How This Project Is Run

`slp-eatool` follows Forma's opinionated, gated workflow — from a raw idea all the way to shipped software:

```
Idea → Research → Customer Narrative → Project → Shape → Spec → Tickets → Build → Ship → Measure
```

You don't skip gates. Every stage produces a named artifact with a single DRI. Shape and Spec are living documents that span cycles; each cycle bets a slice, recorded in one flat `cycles/cycle-N.md` file (the Bet at start, the Measure at close).

The full rules, design decisions, and conventions live in [`../forma-program-management`](../forma-program-management) and are summarized for this repo in [`CLAUDE.md`](CLAUDE.md).

## Repo Structure

| Path | Purpose |
|------|---------|
| `project/` | Project-level artifacts: idea, customer narrative, project map, shape, spec |
| `research/` | Citation-grounded research — inputs to the Customer Narrative |
| `cycles/` | One record per cycle: the Bet (at start) + the Measure (at close) |
