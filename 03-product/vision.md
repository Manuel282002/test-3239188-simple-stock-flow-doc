# Product Vision

> The vision is the team's north star. All sprints, design decisions,
> and trade-offs are evaluated against this vision.
> It must be ambitious yet achievable, inspiring but specific.

---

## Vision statement

**For** internal store operators (admins and sellers)
**who** manage catalogs and register fast-paced retail transactions under a single-currency platform
**the** Simple Stock Flow system
**is a** retail core software and catalog validator
**that** guarantees absolute operational consistency by preventing negative inventory drops through database constraints and isolating closed reporting windows from catalog mutations.
**Unlike** typical legacy inventory applications or unvalidated memory systems,
**our product** enforces a strict separation of domain rules via a monolito hexagonal architecture backed by direct engine invariants and concurrency controls.

---

## Team mission

Our team exists to build a bulletproof, zero-leak sales and inventory persistence backend that preserves transactional reality perfectly, ensuring that no technical migration, manual query, or race condition can ever corrupt the business's historical accounting data.

---

## Strategic pillars

Pillars are the focus areas that take us from mission to vision.
They should be few (3-5) and consistent over time.

| Pillar | Description | Success metrics |
|--------|-------------|----------------|
| Enforced Data Integrity | All critical business constraints are pushed directly to the database engine layer to block invalid inputs. | 100% of core invariants validated inside `pg_constraint`. |
| Unalterable Reporting | Sales histories and aggregated reports must remain stable and unaffected by future catalog mutations or updates. | 0 changes to closed period reports when product names or categories shift. |
| Robust Concurrency | Safe multi-user operations on shared catalog stocks without table locks or race conditions. | 0 data overwrites achieved through native PostgreSQL `xmin` column tokens. |

---

## High-level roadmap

The roadmap shows how the product evolves over time.
Horizon 1 (0-3 months): high certainty, detail in HUs
Horizon 2 (3-6 months): medium certainty, epics
Horizon 3 (6-12 months): low certainty, focus areas


October 2026 ──── November 2026 ──── December 2026 ──── Q1 2027
│                 │                 │                │
[H1 - Sync]      [H1 - DB Debt]    [H2 - Security]   [H3 - Operations]
Singularized     Deploy T-11, T-12  Audit auth flaws  Asynchronous cleanup
schema active    and T-20 rules     and token logs    routines for images

| Horizon | Period | Objective | Epics / Features | Uncertainty |
|---------|--------|----------|----------------|-------------|
| H1 (Now) | October 2026 | Complete core schema constraints and fix physical model tracking debts. | Epic: Schema Hardening (Deploy T-11 frozen fields, T-12 user foreign keys, and T-20 checks). | Low |
| H2 (Next) | Nov-Dec 2026 | Secure internal operator workflows and authorization roles. | Epic: Identity Alignment (Fix anonymous registration flaw A-1 and optimize access indices T-13). | Medium |
| H3 (Later) | Q1 2027 | Operational automation and background external asset synchronization. | Area: Asset Resilience (Design routine H-2 for orphaned image keys cleanup). | High |

---

## Product principles

These principles guide design and prioritization decisions when there are trade-offs.

1. **The Engine is the Last Line of Defense:** While the C# domain layer guides the application, no data modification rule is considered complete until it is backed by an explicit PostgreSQL constraint or unique index.

2. **Core Domain Minimalist Focus (DP-03):** We fiercely protect the catalog model from scope creep. A product only has a name, price, stock, category, and an opacity image key. No secondary descriptive attributes are permitted.

3. **Accounting Inmutability:** A sale is a finalized commercial fact. Once committed, a transaction record cannot be edited, deleted, or altered by any API workflow.

---

## Product Definition of Done

The product is "done" when it achieves these OKRs:

**Objective:** Build a robust, fully verified and zero-debt database architecture wrapper for Simple Stock Flow.

| Key Result | Baseline | Target | Date |
|------------|---------|--------|------|
| KR1: Engine-enforced business invariants | 1 constraint (`stock >= 0`) | 100% of checkable invariants moved to motor | 2026-10-31 |
| KR2: Resolution of opened database tracking debts | 3 active migration debts | 0 active migration issues inside `tasks.md` | 2026-10-31 |
| KR3: Historical report calculation stability | Vulnerable table joins | 0 variations using frozen names and categories | 2026-10-31 |

---

## Correlations

- Problem framing (the why) → `problem-framing.md`
- Backlog that implements the vision → `tasks.md`
- Term glossary → `project-glossary.md`
- Core data specifications → `data_model.md`
