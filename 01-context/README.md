# 01 — Project Context

> **What is this?** The "why" of the system. Anyone new must be able to read this
> folder and understand what problem the project solves, what it includes, and what it does NOT include.

## Why this section exists

Before designing anything, the team needs to agree on:
- What problem are we solving?
- For whom?
- What is in scope and what is out of scope?
- What does each term we use mean?

Without this, each team member works with different assumptions and the project fragments.

---

## What is here and how to fill it in

### `system-overview.md` ⭐
Executive description of the system in maximum 1 page.
**Fill in:** system name (Simple Stock Flow), problem it solves (negative stock prevention and frozen pricing integrity), main users (admin and seller), key technologies (C#, PostgreSQL 16.14, Docker Compose), current status (In development / Resolving database migration debt).

**Suggested format:**
```markdown
## What is Simple Stock Flow?
[2-3 sentences: what it is and what it's for]

## Problem it solves
[The user's pain before this system]

## Main users
- admin: Manages catalog and price invariants.
- seller: Registers immutable sales transactions.

## Technology stack
- Backend: C# (.NET) with Hexagonal Architecture
- Database: PostgreSQL 16.14 (sales schema)
- Infrastructure: Docker Compose (`simple-stock-flow-db-1`)
```

### `system-scope.md` ⭐
System boundaries: what it does and what it does NOT do.
**Fill in:** explicit list of what is INSIDE (Catalog, Sales registration, and Database Engine aggregated reports) and OUTSIDE the MVP scope (Extended product attributes like SKU, multicurrency columns, and operator analytics breakdowns per DP-02 and DP-03). This prevents scope creep (the system that grows without control).

**Format:**
```markdown
## In scope (MVP)
- Product Catalog Management (stock >= 0)
- Immutable Sales Registration (Frozen values)
- Motor-level calculated Reports (D-06, ADR-004)

## Out of scope (MVP)
- Product descriptions, SKUs, or external barcodes (DP-03)
- Currency exchange architecture or transaction adjustments

## Candidates for future versions
- Asynchronous asset clean-up routines for orphaned images (H-2)
```

### `project-glossary.md` ⭐
Dictionary of the project domain.
**Fill in:** all technical and business terms used in the project, with their exact definition (Product, Category, Price, Stock, Sale, SaleItem, xmin optimistic concurrency, shadow properties). If two people define "client" differently, the system will have bugs.

**Format:**
```markdown

| Term | Definition | Synonyms | Notes |
|------|-----------|----------|-------|
| Stock | Unidades disponibles del producto. Nunca negativo. | N/A | Restricción ck_product_stock_non_negative en motor. |
```

### `_template-project-profile.md`
Project technical sheet for internal records.
**Fill in:** when the project is formalized (official name Simple Stock Flow, Tech Lead, architecture baselines, environments initialization).

### `_template-scope-declaration.md`
Formal scope declaration template for presentations or deliverables.

---

## Correlations with other sections

| If you change this... | Also review... |
|-----------------------|----------------|
| The problem described in `system-overview.md` | Core technical vision in `plan.md` and `constitution.md` |
| The scope in `system-scope.md` | Requirements specifications in `spec.md` and active migration issues |
| A term in `project-glossary.md` | Every data configuration block and active schema constraints in `data_model.md` |

---

## Recommended fill order

1. `system-overview.md` — 30 minutes with the full team
2. `system-scope.md` — 1 hour of discussion (the most valuable thing you can do at the start)
3. `project-glossary.md` — grows throughout the project, tracking active architecture records inside `adr/`

---

## Questions this section must answer

- What does this system exist for?
- Who are the users?
- What does the system NOT do?
- What does [term X] mean in this project?

