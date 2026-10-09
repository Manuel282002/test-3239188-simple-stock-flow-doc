
# 03 — Product Definition

> **What is this?** The answer to "what are we going to build?". It is not "how" — that comes in
> architecture. Here the validated problem, product vision, and build plan are defined.

## Why this section exists

Without a clear product definition:
- The team builds features nobody asked for
- Scope grows out of control (scope creep)
- There is no way to know whether the project was successful

This section is the contract between the team and stakeholders about **what will be built and why**.

---

## What is here and how to fill it in

### `problem-framing.md` ⭐ (Start here)
Articulates the problem before proposing solutions.
**Fill in:** who has the problem (internal operators: admins and sellers), exactly what pain (historical data corruption and negative inventory syncs), evidence of the problem (5 domain invariants bypassing constraints via manual `psql` insertions), how they solve it today (purely application-level memory validations).

### `discovery-brief.md`
Findings from user research.
**Fill in:** active diagnostic interviews with domain experts on transactional stability and constraints checking.

### `vision.md` ⭐
The product's north star in 1-2 sentences.
**Fill in:** "For **internal store operators (admins and sellers)**, who **manage catalogs and register retail transactions under a single-currency platform**, the **Simple Stock Flow system** is a **retail core software** that **guarantees absolute operational consistency by preventing negative inventory drops through database constraints and isolating closed reporting windows from catalog mutations**. Unlike **typical legacy inventory applications or unvalidated memory systems**, our product **enforces a strict separation of domain rules via a monolito hexagonal architecture backed by direct engine invariants**."

### `roadmap.md`
Delivery plan over time.
**Fill in:** milestone tracking mapped to the critical milestones of October 2026.

**Format:**
```markdown
## Phase 1 — MVP (October 2026)
- Singularized schema active (`sales` schema)
- Engine-enforced stock verification (`stock >= 0`)

## Phase 2 — Iteration (October 2026)
- Deployment of T-11 frozen fields and categories.
- Deployment of T-12 user foreign keys and T-20 constraint checks.
```

### `product-backlog.md` ⭐
Prioritized list of everything that must be built.
**Fill in:** using the core technical tasks from the database backlog (`tasks.md`). Order by data protection value.

### `_template-prd.md`
Complete Product Requirements Document for academic and engineering formalizations.

### `_template-discovery-brief.md`
Template for documenting system behavior diagnostics.

### `_template-problem-framing.md`
Structured template for framing transactional database problems.

### `_template-backlog.md`
Template for data configuration issues and task alignment.

---

## User Story format

```markdown
## HU-SALES-[NNN]: [Title]
**As** internal operator (admin / seller)
**I want** to execute sales or catalog updates under engine constraints
**So that** the data store remains completely clean, consistent, and audit-ready

### Acceptance criteria
- [ ] AC1: Given an unconfirmed sale, when `Sale.AddItem` is invoked, then the system must freeze product prices and category names synchronously.
- [ ] AC2: Given a concurrent stock operation, when execution triggers, then the system must check the `xmin` token to prevent race conditions.

### Technical notes
Must map fields natively to the single-currency `sales` schema tables.
```

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `tasks.md` | Core roadmap milestones translate directly to active technical tasks |
| `data_model.md` | Solution definitions dictate table structure edits and unique indices |
| `data_model.md` §13 | Identified database tracking debts are registered as explicit liabilities |

---

## Questions this section must answer

- What problem exactly are we solving?
- What does product success look like?
- What do we build first and why?
- What do we NOT build in this cycle?
