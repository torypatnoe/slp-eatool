# Customer Narrative: slp-eatool

**DRI:** Tory Patnoe
**Status:** Draft — core parameters confirmed with the DRI; full workflow, reports, and roadmap still to be confirmed with the SLP Bookkeeping team

---

## The Customer

The **SLP Bookkeeping team** — the bookkeepers responsible for keeping the books for the **East Angles Foundation**. They own the monthly close and the reporting that comes out of it, and they work primarily inside **QuickBooks**.

Their day is shaped by getting data *into* and *out of* **QuickBooks Online**: transactions from the systems the Foundation actually uses (starting with **Stripe**) have to land in QuickBooks accurately, and reports have to come back out. Today that movement is done by hand.

---

## The Problem

Closing the month and producing reports depends on data that lives outside QuickBooks. Getting it in — and getting reports back out — is manual: export from the source system, reshape it, import it into QuickBooks, check it, repeat for the next system. It is repetitive, time-consuming, and easy to get wrong, and it has to happen every single month.

**Stripe is the first and clearest case.** Today the Stripe → QuickBooks process is three manual steps, run every month against roughly **1,000 transactions**:

1. **Export** transactions from Stripe into an Excel document.
2. **Massage** the exported data into the format QuickBooks Online expects.
3. **Import** the formatted data into QuickBooks Online.

Steps 2 and 3 — reshaping ~1,000 rows by hand and then importing them — are the most repetitive and error-prone, and together they are the first automation target. Step 1 (the Stripe export) stays manual for now; automating it via the Stripe API is later work.

*(The step-by-step massaging rules and the reports the close must produce will be confirmed by working through the real workflow with the SLP Bookkeeping team — see [research/current-state.md](../research/current-state.md).)*

---

## What the Customer Can Do After slp-eatool Exists

- **Move Stripe data into QuickBooks without doing it by hand.** The export-reshape-import cycle for Stripe becomes an automated workflow, so the monthly Stripe reconciliation is a check, not a data-entry task. The idea is a command-line tool — `./slp-eatool export.xlsx` — that reshapes the Stripe export and writes the transactions into QuickBooks Online directly.
- **Close the month faster and with fewer errors.** The manual steps that today introduce mistakes are automated, so the close is more reliable and repeatable.
- **Extend the same pattern to the next system.** Stripe is the first workflow; the tool is built so the next source of import/export data follows the same automated path rather than another hand-built process.

---

## What We Are NOT Solving

- **We are not replacing QuickBooks.** QuickBooks stays the system of record. `slp-eatool` moves data in and out of it; it does not become the ledger.
- **We are not doing the bookkeeping judgement.** Categorisation rules, chart-of-accounts decisions, and sign-off remain the bookkeeper's job. The tool automates the mechanical data movement, not the accounting decisions.
- **We are not a general-purpose integration platform.** This is built for SLP Bookkeeping's workflow for the East Angles Foundation, starting with Stripe — not a configurable connector for arbitrary systems.
- **We are not (yet) automating every source system.** Stripe is the first workflow. Other sources come later, informed by the first.
- **We are not (yet) generating the reports the close produces with less manual effort.** Getting data out of QuickBooks for reporting is part of the same automated flow, not a separate manual export.


---

## Open Questions

1. ~~**QuickBooks Online or Desktop?**~~ **Resolved:** QuickBooks **Online**.
2. ~~**What does the current Stripe → QuickBooks process look like?**~~ **Resolved (high level):** three manual steps — export from Stripe to Excel, massage into QuickBooks format, import to QuickBooks Online. The detailed massaging rules are captured in Cycle 1.
3. **Open — Stripe export API and QuickBooks import API.** Automating steps 1 and 3 requires researching both. Deferred to a later cycle; Cycle 1 automates step 2 only and continues to use the manual Excel export as input and manual import as output.
4. **Open — what "done" for a monthly close looks like, and which reports must come out of it.** To be confirmed by interviewing the SLP Bookkeeping team.
5. **Open — the next source system to automate after Stripe.** To be confirmed by interviewing the SLP Bookkeeping team.
6. ~~**What volumes are involved?**~~ **Resolved:** ~1,000 transactions per month — the design assumption for Cycle 1.
