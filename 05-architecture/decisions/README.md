# ADRs — Architecture Decision Records

ADRs document important architectural decisions. Each file = one decision.

## How to create an ADR

1. Copy `_template-adr.md`
2. Name it `ADR-NNN-short-title.md` (e.g.: `ADR-001-message-broker.md`)
3. Fill it in completely — especially the evaluated alternatives
4. Once accepted, the status is **permanent** (it is not deleted, it is "Superseded" by another ADR)

## Possible statuses

- `Proposed` — under discussion
- `Accepted` — approved by the team
- `Rejected` — evaluated and discarded (document why)
- `Superseded` — superseded by ADR-NNN (indicate which one)

## ADR register

| # | Title | Status | Date |
|---|-------|--------|------|
| 001 | Propiedad del Esquema (DDL reservado a migraciones de EF Core) | Accepted | 2026-09-19 |
| 002 | Concurrencia Optimista mediante xmin y checks físicos | Accepted | 2026-09-19 |
| 003 | Baja Lógica (deleted_at) en lugar de borrado físico de productos | Accepted | 2026-09-19 |
| 004 | Reporte Agregado dinámico en motor con congelación de líneas | Accepted | 2026-09-19 |
| 005 | Política de Plataforma Monomoneda sin columnas de divisas | Accepted | 2026-10-08 |

---

## Correlations

- Architecture baseline records → `system-architecture-overview.md`
- Active Technical Debt Backlog → `tasks.md`
- Engine Constraints Mapping → `data_model.md` §4

