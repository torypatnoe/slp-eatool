# slp-eatool

## What This Repo Is

`slp-eatool` is a project managed under the **Forma** program-management methodology. The rules, workflow, and file conventions come from [`../forma-program-management`](../forma-program-management) — that repo is the source of truth for *how* we work; this repo applies that method to *this* project.

> **Idea (gate passed):** `slp-eatool` is an automation tool for **SLP Bookkeeping** that automates the recurring bookkeeping workflows for the **East Angles Association**. See [`project/idea.md`](project/idea.md).

**DRI:** Tory Patnoe

## The Forma Workflow (gated — do not skip forward)

```
PROJECT LEVEL
Idea → Research → Customer Narrative → Project

CYCLE LEVEL (repeating within the Project)
Shape → Spec → Tickets → Build → Ship → Measure
         ↑
  refined by the previous cycle's Measure (learning note)
```

Each stage is a gate. You may move backward, never skip forward. Every stage produces a named artifact, and every artifact has exactly **one DRI**. A downstream artifact (Shape, Spec) stays a "pending" stub until the gate before it clears.

## Repo Structure

```
project/           — PROJECT LEVEL artifacts (single living documents)
  idea.md          — one-sentence problem + owner
  customer-narrative.md — who / problem / success / non-goals; grounded in research
  project.md       — milestone map, DRI, project hill chart, cycle history
  shape.md         — living shape doc with a Changelog at the top
  spec.md          — living spec doc with a Changelog at the top

research/          — citation-grounded inputs to the Customer Narrative

cycles/            — one flat file per cycle: cycle-N.md
  cycle-1.md       — opened with the Bet, closed with the Measure
```

## File Conventions — Read Before Creating Any Files

**Artifacts are single living documents, not per-cycle copies.**

- Do **NOT** create per-cycle subdirectories (`cycles/cycle-1/`) or per-cycle shape/spec files. Maintaining a delta file *and* a merged file creates two sources of truth that drift. `cycles/` holds exactly one flat `cycle-N.md` per cycle — nothing else.
- `project/shape.md` and `project/spec.md` are each **one** file, always the current full state, with a `## Changelog` section at the top recording what changed each cycle. Git history provides full version tracking.
- `cycles/cycle-N.md` is the **only** per-cycle file. Opened at cycle start with **the Bet** (milestone, goal, appetite, in/out of scope, DRI), carries the cycle hill-chart position and a validation log during Build, and is closed with **the Measure** (result against the goal, learning note). Once closed, immutable.

**Where things go:**
- New research → `research/`
- Shape updates → edit `project/shape.md`, add a Changelog entry at the top
- Spec updates → edit `project/spec.md`, add a Changelog entry at the top
- Cycle start → open `cycles/cycle-N.md` with the Bet; cycle close → complete it with the Measure

**Per-cycle facts live only in the Bet.** Appetite, scope boundary, and DRI are written in the cycle record and never duplicated into the Shape document — two homes for one fact is how drift starts.

## Key Principles

- **Single DRI on everything** — Idea, Narrative, Project, Shape, Spec, Ticket. One named person, not a team.
- **Think like a PM, not an engineer** — start with the user problem, not the solution.
- **6-week maximum appetite** per cycle. What doesn't fit gets split into a later cycle, not extended.
- **Cycles sharpen the Customer Narrative; they cannot change it.** Same customer, same problem, better understood → sharpen. Different customer or problem → new project.
- **Research is effort-scalable, not skippable.** Findings must be *recorded* before a Customer Narrative is written — even "we know this space well, and here's why."
- **Ship is not Done.** A cycle moves to Measure when code is deployed; a Project closes only when the Customer Narrative is validated.

## Current Status

### Done
- Repo scaffolded to the Forma structure (`project/`, `research/`, `cycles/`), `CLAUDE.md`, `README.md`
- **Idea gate passed** — `project/idea.md`
- **Research gate** — `research/current-state.md` records that research is pending direct collaboration with SLP Bookkeeping (knowingly deferred, not skipped)
- **Customer Narrative** — `project/customer-narrative.md`; core parameters confirmed (QuickBooks Online, three-step Stripe process, ~1,000 txns/month). Reports and roadmap still to confirm with SLP
- **Project** — `project/project.md`; milestone map with the massaging automation as M1
- **Shape** — `project/shape.md`; Cycle 1 slice = automate step 2 (massaging)
- **Cycle 1 opened** — `cycles/cycle-1.md` Bet written (appetite provisional; massaging rules pending)

### Next Steps
1. Capture the exact massaging rules — column mapping/transforms from a real Stripe export to the QuickBooks Online import format — into `cycles/cycle-1.md`, then confirm the Cycle 1 appetite
2. Write `project/spec.md` (acceptance criteria, success metric) from those rules
3. Interview SLP Bookkeeping on what a "done" monthly close and its reports require (narrative open questions #4, #5)
