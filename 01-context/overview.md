
# System Overview

> These rules determine how documentation is written, organized, and maintained in this project.
> Documentation that does not follow these rules may be rejected in code review.

---

## What is Simple Stock Flow?

Simple Stock Flow is a sales and catalog management system designed to control product stock, organize items by categories, and record immutable retail transactions. The system operates as a single-currency platform under a strict hexagonal architecture to ensure high operational consistency.

## Problem it solves

**Before the system:** Inventory and sales tracking were decoupled or relied on manual verification, leading to negative stock numbers, unstable pricing histories during product updates, and unvalidated manual database entries.

**With the system:** Invariants are strictly controlled by the domain and physical database checks, ensuring that stock can never drop below zero (`stock >= 0`). It provides stable billing records by freezing product prices and category names at the exact instance of the sale, preventing historical reporting corruption.

## Main users

| Role | Description | What they do in the system |
|------|-------------|--------------------------|
| admin | Internal operator with full administrative privileges. | Manages the product catalog, updates prices, and reviews sales aggregation reports. |
| seller | Internal front-line commercial operator. | Performs stock lookups and executes the registration of new, immutable sales transactions. |

## Technology stack

| Layer | Technology | Justification |
|-------|-----------|---------------|
| Backend | C# (.NET) | Centralizes domain logic invariants and isolates business rules from infrastructure using a hexagonal design. |
| Database | PostgreSQL 16.14 | Hosts the `sales` schema natively under UTC, enforcing optmistic concurrency tracking via the system column `xmin`. |
| Infrastructure | Docker Compose | Orchestrates the `simple-stock-flow-db-1` database engine wrapper and runtime environment configuration natively. |

## Current status

- **Phase:** In development (Resolving database migration debt)
- **Current version:** v0.4.0
- **Last release:** 2026-09-20 (Saldada de deuda técnica D-1, D-2 y D-3)
- **Next milestone:** Implementation of pending tasks T-11 (category_name freeze), T-12 (sold_by_user_id FK), and T-20 (CHECK constraints alignment)

## Project contacts

| Role | Name | Contact |
|------|------|---------|
| Tech Lead | Team Architecture Lead | tech-lead@simplestockflow.internal |
| Product Owner | Functional Domain Owner | product-owner@simplestockflow.internal |
| DevOps | Infrastructure Administrator | devops@simplestockflow.internal |
