# Traceability Matrix

> Traceability connects every line of code to its business justification.
> It allows answering: "Why does this function exist?" and "Which HU covers this part of the system?"
> It also identifies: unimplemented requirements and code without a requirement (possible technical debt).

---

## How to use this matrix

Requirement → HU → Test Case → Implementation → Service
If a requirement has no HU: it is not planned
If a HU has no test case: it has no completeness criterion
If a test case has no implementation: there is test technical debt
If there is code without an HU: possible gold-plating or bug introduced without a story

---

## FR → HU → Test → Service matrix

| FR ID | FR Description | HU(s) | Tests that verify it | Service | Status |
|-------|---------------|-------|---------------------|---------|--------|
| FR-001 | Búsqueda y filtrado de productos activos del catálogo (Q1) | HU-SALES-009 | `ProductSearchTests.cs` | simple-stock-flow-api | ✅ Done |
| FR-002 | Retiro y actualización atómica de stock de productos (Q3) | HU-SALES-010 | `ProductStockConstraintTests.cs` | simple-stock-flow-api | ✅ Done |
| FR-003 | Registro de ventas inmutables con congelación de precio (Q6) | HU-SALES-006 | `SaleRegistrationTests.cs` | simple-stock-flow-api | 🟡 In progress |
| FR-004 | Congelación de nombres de categorías en líneas de venta (D-06) | HU-SALES-011 | `CategoryNameFreezeTests.cs` | simple-stock-flow-api | 🟡 In progress |
| FR-005 | Asignación de autoría de venta obligatoria por operador (T-12) | HU-SALES-012 | `SaleOperatorAuditTests.cs` | simple-stock-flow-api | 🔴 Pending |
| FR-006 | Generación de reporte agregado en tiempo real por el motor (Q9) | HU-SALES-013 | `AggregatedReportQueryTests.cs` | simple-stock-flow-api | 🔴 Pending |
| FR-007 | Autenticación interna de usuarios y validación de roles fijos (Q10)| HU-SALES-003 | `OperatorAuthTests.cs` | simple-stock-flow-api | ✅ Done |

---

## NFR → Validation matrix

| NFR ID | Description | How it is validated | Tool | Status |
|--------|-------------|-------------------|------|--------|
| NFR-001 | Latencia P95 < 200ms en el endpoint crítico de ventas | Prueba de carga síncrona en staging | k6 | ✅ Validated |
| NFR-002 | Disponibilidad mensual del sistema del 99.9% en producción | Monitoreo de respuestas del endpoint `/health/ready` | Prometheus + Grafana | 🟡 Monitoring |
| NFR-004 | Autenticación obligatoria mediante JWT para rutas privadas | Prueba automatizada de denegación sin token (Defecto A-1) | xUnit + Integration Tests | ✅ Validated |
| NFR-006 | Cobertura de pruebas ≥ 95% dentro de invariantes puras | Análisis de cobertura en la compuerta del pipeline de CI | Coverlet / SonarQube | 🟡 In progress |

---

## Inverse traceability: HU → FR

| HU | Title | FR(s) it implements | Sprint |
|----|-------|---------------------|--------|
| HU-SALES-009 | Aplicar soft delete y filtro global sobre activos (T-09) | FR-001 | Sprint 3 (Saldada) |
| HU-SALES-010 | Implementación del CHECK de stock no negativo (T-10) | FR-002 | Sprint 3 (Saldada) |
| HU-SALES-011 | Congelación de Category Name en líneas de venta (T-11) | FR-004 | Sprint 4 (Actual) |
| HU-SALES-012 | Renombrado y clave foránea restrictiva para sold_by (T-12) | FR-005 | Sprint 5 |
| HU-SALES-013 | Agregación por producto optimizada por índices compuestos | FR-006 | Sprint 5 |

---

## Status legend

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented, tested, and in production (Enforced by motor) |
| 🟡 In progress | Under development in the current sprint (October 2026 milestones) |
| 🔴 Pending | In the backlog, not started |
| ⏸ Blocked | Has an external blocker |
| ❌ Cancelled | Removed from scope |

---

## Identified gaps (requirements without coverage)

| Gap type | Description | Required action | Owner | Date |
|----------|-------------|----------------|-------|------|
| HU without test | HU-SALES-012 (Clave foránea restrictiva para sold_by_user_id) carece de aserciones de verificación física. | Escribir consultas directas contra `pg_constraint` antes del Sprint 5. | Core Tech Lead | 2026-10-08 |
| HU without test | HU-SALES-011 (Category Name congelado) requiere rellenar datos en la migración de la base activa. | Añadir script de relleno en la migración de Entity Framework Core. | Database Dev | 2026-10-08 |

---

## How to maintain this matrix

1. When an HU is created: add the row in the FR → HU → Test → Service section
2. When a test is written: note the file in the "Tests that verify it" column
3. When an HU is completed: change the status to ✅
4. At each Sprint Planning: review gaps and assign actions

---

## Correlations

- User Stories Backlog → `tasks.md`
- Non-Functional Requirements Matrix → `non-functional.md`
- Core Database Invariants and Physical Models → `data_model.md`
- Definition of Done General Rules → `00-governance/definition-of-done.md`
