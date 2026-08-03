# Shape: slp-eatool — Stripe → QuickBooks Data Movement

## Changelog
- Cycle 1 (2026-08-03, restart — API import promoted to core): Cycle 1 re-cut. Direct QuickBooks Online API import (previously an optional stretch) is now the committed core; the massaging/derivation logic still gets built, but runs in-memory ahead of the API write rather than shipping as a standalone `import.xlsx`. File output demoted to a secondary `--file` (dry-run/audit) mode. Entity representation captured: a combination — SalesReceipt (sale) + JournalEntry (fee) + Deposit (net payout). See [cycles/cycle-1.md](../cycles/cycle-1.md) restarted Bet.
- Cycle 1 (2026-07-06, optional-import expansion): Cycle 1 stretch added — an *optional*, flag-gated direct import into QuickBooks Online via the Accounting API (sandbox-first). No native CSV import API exists, so the programmatic path is creating transaction entities via OAuth 2.0; this brings secret handling and rotating-refresh-token persistence into scope for the import path. Default remains file output. See [research/qb-research.md](../research/qb-research.md).
- Cycle 1 (2026-07-06, incumbent-tool decision): Build-vs-buy resolved to **build (Option B)** — remove SaasAnt from the loop; the tool produces a QuickBooks-Online-importable file directly. Slice sharpened: the tool's job is the **per-transaction derivation SaasAnt's 1:1 mapping can't do**, not just column renaming. SaasAnt (buy) and hybrid (Go-transform → SaasAnt import) recorded as rejected alternatives. See [research/saasant-incumbent-tool.md](../research/saasant-incumbent-tool.md).
- Cycle 1 (2026-07-06, post customer review): Delivery form confirmed — a standalone command-line binary (`slp-eatool <stripe-export.xlsx>` → `import.xlsx`) that runs on a Mac with no dependencies and can be driven unattended from a Linux automation host later. Language/runtime details live in [spec.md](spec.md).
- Cycle 1 (2026-07-06): Initial shape — automate step 2 (massaging the Stripe Excel export into the QuickBooks Online import format). Manual export (step 1) and manual import (step 3) remain in place as the tool's input and output for this cycle.

---

**Purpose:** Define the solution shape for automating SLP Bookkeeping's Stripe → QuickBooks Online workflow and record the reasoning behind each decision. Living document — grows as later cycles automate the export and import steps. Per-cycle commitments (appetite, scope boundary, DRI) live in the cycle record, not here.

---

## The Slice This Shape Currently Bets On (Cycle 1, restarted 2026-08-03)

The monthly Stripe → QuickBooks Online process is three manual steps: **export** (Stripe → Excel), **massage** (reshape into QuickBooks format), **import** (into QuickBooks Online). Cycle 1 now automates the **massage** step *and* the **import** step together: the tool transforms the raw Stripe export in-memory and writes the result directly into QuickBooks Online via the Accounting API. Only **export** (step 1) remains manual this cycle.

### Rough solution narrative

The bookkeeper still exports the Stripe transactions to Excel by hand — that end is unchanged this cycle. From there, `slp-eatool` applies the massaging **and the per-transaction derivations**, then writes the resulting transactions directly into QuickBooks Online via the Accounting API (OAuth 2.0, sandbox company first). **SaasAnt drops out of the loop, and so does the manual QuickBooks import.** What remains at the seams is a plain Stripe export in and reconciled books out — no intermediate file to hand-import.

Each Stripe transaction becomes a **combination** of QuickBooks entities: a **SalesReceipt** for the gross sale, a **JournalEntry** for the Stripe fee, and a **Deposit** for the net payout. Refunds are represented as sign-flipped entries. Categories are mapped from Stripe into the corresponding QuickBooks accounts. (Captured 2026-08-03 — see [cycles/cycle-1.md](../cycles/cycle-1.md) Workflow Details; the exact chart-of-accounts mapping and refund entity type still need validation against a real export.)

The tool's real value is the **derivation SaasAnt's 1:1 column mapping can't express** — the computed transforms (splitting Stripe gross into sales/fees/payouts, sign flips for refunds, deriving accounts/categories) that used to force SLP to redo the mapping by hand each month — now delivered straight into the books instead of into a file someone still has to import.

A secondary `--file` mode still writes `import.xlsx` for dry-run/audit purposes, but it is not the primary deliverable this cycle.

Success for this slice: given a real Stripe Excel export, the tool creates the correct transactions in a QuickBooks Online sandbox company — matching what a bookkeeper would otherwise have produced by hand through SaasAnt and manual import — correctly and repeatably.

### Identified unknowns
- **The exact chart-of-accounts mapping** — which QuickBooks account each Stripe category maps to. To be captured from SLP's chart of accounts against a real export + known-good result.
- **The refund entity type** — reversing JournalEntry, CreditMemo, or negative SalesReceipt line. Confirmed: sign-flipped; not yet confirmed which entity carries it.
- **Edge cases in the data** — multi-currency, disputes/chargebacks — whether they appear in the ~1,000 and how they must be represented.
- ~~**The QuickBooks Online import format — and whether native import even covers this transaction type.**~~ **Resolved by the restart:** moot — the tool no longer targets QBO's native file import at all; it writes accounting entities directly via the API (see [research/qb-research.md](../research/qb-research.md)).

### Alternatives considered
- **Buy — lean fully on SaasAnt (Option A).** Save a SaasAnt mapping, or adopt PayTraQer for API auto-sync. Rejected: SaasAnt's mapping is 1:1 and can't perform the required per-transaction derivations (which is exactly why SLP re-maps by hand monthly); PayTraQer's automatic mapping is unlikely to match the Foundation's chart-of-accounts. Cheapest short-term but doesn't close the gap.
- **Hybrid — Go transform → SaasAnt import (Option C).** Go tool produces a constant, saved-mappable file; SaasAnt still does the QBO import. Rejected for now: keeps a paid third party and its per-line credit cost in the loop, and adds a second tool to the chain for no long-term benefit.
- **Build — custom Go tool, SaasAnt removed (Option B).** **Chosen.** Full control of the derivation logic, no metered per-line cost as volume grows, and a clean foundation for M2 (Stripe API) and M3 (QuickBooks API). Trade-off: we own the QBO import path — see Identified unknowns. Full rationale in [research/saasant-incumbent-tool.md](../research/saasant-incumbent-tool.md).
- **Automate the whole pipeline now (Stripe API → transform → QuickBooks API).** Rejected for Cycle 1: it forces the Stripe-export and QuickBooks-import API research (narrative open question #3) up front and couples three unknowns together. Automating the massaging + derivation step alone removes the biggest manual burden immediately and leaves clean seams to automate the export (M2) and import (M3) later.
- **A one-off spreadsheet macro / template.** Rejected: it hard-codes the current format, resists reuse as rules change, and doesn't establish a base the later API-driven milestones can build on.
