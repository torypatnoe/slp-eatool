# slp-eatool

> A project managed under the [Forma](../forma-program-management) program-management methodology.

## Status

**Cycle 1 — restarted 2026-08-03, Spec updated.** `slp-eatool` is an automation tool for **SLP Bookkeeping** that automates moving data in and out of **QuickBooks Online** for the **East Angles Foundation**'s monthly close — starting with the **Stripe → QuickBooks** workflow. Cycle 1's committed core is now a **direct import into QuickBooks Online via the Accounting API** (OAuth 2.0, sandbox-first): a dependency-free **Go** command-line binary reads the monthly Stripe Excel export (~1,000 txns), applies the massaging/derivation logic in memory, and writes the transactions straight into QuickBooks — **replacing the incumbent SaasAnt tool** (build-vs-buy resolved to build) *and* the manual import step. File output survives as a secondary `--file` dry-run/audit mode. Next: validate the entity mapping against a real Stripe export and capture the chart-of-accounts mapping.

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
