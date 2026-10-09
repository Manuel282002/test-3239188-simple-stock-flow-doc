
# Problem Framing — Problem Definition

> **Why this document exists:** Before designing solutions, the team must be
> aligned on the problem it solves. This document captures that alignment.
> A well-defined problem is already halfway to a solution.

---

## 1. The problem in one sentence

**Internal retail store operators (admins and sellers)** who **register transactions and manage product catalogs under a single-currency platform** struggle with **historical data corruption and negative inventory syncs** because **previous systems did not freeze sales data at the time of the transaction nor enforce engine-level stock guards**, resulting in **unreliable business reports and frequent manual inventory reconciliations due to retroactively modified catalogs**.

---

## 2. Affected users

| Segment | Description | Estimated size | Priority |
|---------|-------------|---------------|---------|
| admin | Operador interno encargado de la gestión del catálogo vivos, precios e invariantes de negocio. | 2 operators | High |
| seller | Operador comercial de primera línea encargado de la facturación y rebaja de stock inmediata. | 5 operators | High |

### Jobs-to-be-done (JTBD)

**When** a sale is processed by a seller while an admin updates catalog attributes,
**I want** the system to process the stock deduction atomically and copy all product states in that exact instant,
**so that** historical sales reports remain completely stable and independent of future catalog modifications.

---

## 3. Evidence of the problem

| Evidence type | Source | Date | Key finding |
|--------------|--------|------|------------|
| User interviews | Interviews with internal store operators | 2026-09-19 | Operators report that changing a product name or category breaks old sales charts. |
| Support data | Database verification logs | 2026-09-19 | PostgreSQL audit queries found that `sale_item` records lacked foreign keys and proper null constraints. |
| Benchmarking | Domain-Driven Design standard practices | 2026-09-20 | Confirmed that freezing snapshots of properties at the time of purchase is required for immutable accounting. |
| Direct observation | Schema catalog check (§10) | 2026-09-19 | Discovered that 5 invariants were handled purely in C#, allowing dirty data via direct `psql` manual insertions. |

---

## 4. Current user solution (and its problems)

| Current solution | Limitations | Cost/Friction |
|-----------------|------------|--------------|
| Pure C# application memory validations | Any manual migration, `psql` command, or external process bypasses the rules without noise. | Permanent threat of corrupted tables |
| Live catalog table joints for report calculations | If an item is renamed or recategorized, old closed reports shift dynamically. | Inaccurate sales reports |

---

## 5. Solution hypothesis

We believe that **enforcing database engine-level constraints (`stock >= 0`), utilizing PostgreSQL `xmin` system columns for optimistic concurrency, and physically freezing product names and category identifiers inside the transactional `sale_item` lines** for **internal retail store operators**, will achieve **complete operational consistency and unalterable financial reporting**. We will know we succeeded when **the database schema returns 0 data violations under direct SQL stress tests**.

---

## 6. Success metrics (North Star)

| Metric | Current baseline | 6-month target | How to measure it |
|--------|----------------|---------------|-------------------|
| Data Consistency Rate | 5 rules living only in application | 100% of invariants enforced by engine | Schema checks against `pg_constraint` (§10) |
| Stable Reporting Reliability | Report items modify dynamically | 0 historical report variations | On-demand execution of Q9 aggregated report |

**North Star Metric:** **0 inventory integrity failures** (0 instances of negative stock or corrupted historical rows across any closed sales period).

---

## 7. Hypothesis risks

| Risk | Probability | Impact | Experiment to validate |
|------|------------|--------|----------------------|
| Performance drop on hot transactional paths | Medium | High | Load testing the `Sale.AddItem` execution concurrently to monitor engine lock levels. |
| Database migration issues with historical rows | Low | High | Executing pre-filled migration tests on staging (`simple-stock-flow-db-1`) before prod release. |

---

## 8. Out of scope (we do not solve)

- **Extended product specifications management:** DP-03 strictly prevents adding attributes beyond name, price, stock, category, and image.
- **Multi-currency conversions:** D-05 states the project is strictly monomoneda by construction.
- **Operator-specific analytic reports:** DP-02 completely excludes generating breakdowns by seller to protect internal personal data.

---

## Correlations

- Product vision → `plan.md`
- HUs that implement this solution → `tasks.md` (specifically T-11, T-12, and T-20)
- Core data specifications → `data_model.md`
