
# HU-SALES-011: Congelación de Category Name en Líneas de Venta

---

## Story

**As** an admin or seller
**I want** the system to copy and freeze the category name of a product inside the sale item at the exact moment of the purchase
**So that** historical sales reports remain completely stable and unaffected if the product is later recategorized or the category is renamed.

---

## Acceptance criteria

- [ ] **AC1:** Given a valid product belonging to a category, when `Sale.AddItem` is executed, then the system must synchronously copy the current `category.name` into the `sale_item.category_name` column.
- [ ] **AC2:** Given a recorded sale with frozen data, when the category is later renamed in the catalog, then the existing `sale_item.category_name` must remain unaltered.
- [ ] **AC3:** Given a sale creation attempt, when a product lacks an active or valid category relation, then the system must throw a `DomainException` and abort the transaction.

---

## Technical notes

**Responsible service(s):** `simple-stock-flow-api` (Monolito)
**Endpoint(s) implemented:** `POST /sales`
**Events generated:** `SaleConfirmed`
**Required permissions:** `seller` or `admin` roles (Internal operators verified via `UserContext`)

---

## Definition of Done (DoD)

This HU can only be closed when it meets the team's full DoD.
See: `00-governance/definition-of-done.md`

**Additional checks specific to this HU (if applicable):**
- [ ] Mapear la nueva columna `category_name` como `varchar(120)` y `NOT NULL` en el modelo físico de `sale_item`.
- [ ] Verificar que la agregación del reporte (Q9) agrupe por el valor congelado `product_id, product_name, category_name` sin tocar la tabla `category` viva.

---

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | High |
| Target sprint | Sprint 4 |
| Dependencies | None (Table is currently empty, migration is immediate) |
