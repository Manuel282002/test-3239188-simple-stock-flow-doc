# User Stories — Backlog

> **What to fill in here:** The product's User Story backlog.
> Each HU uses the standard format with Acceptance Criteria in Given/When/Then.
> Refined (Ready) HUs go to the sprint. Unrefined ones are epics or ideas.

---

## Backlog status

| Cut | Sprint | Total HUs | Refined | In progress | Completed |
|-----|--------|-----------|---------|-------------|-----------|
| Cut 1 | Sprint 1-3 | 3 | 3 | 0 | 3 |
| Cut 2 | Sprint 4-5 | 4 | 2 | 2 | 0 |

---

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | Schema Hardening & Invariants | Migración de invariantes del dominio hacia restricciones físicas (`CHECK`, `NOT NULL` e índices únicos) en el motor PostgreSQL. |
| EP-002 | Historical Data Consistency | Garantía de inmutabilidad y estabilidad en reportes analíticos mediante congelación de instantáneas de catálogo. |

---

## User Stories

### HU-SALES-011 — Congelación de Category Name en líneas de venta {#HU-SALES-011}

**Epic:** EP-002

> **As** an admin or seller
> **I want** the system to copy and freeze the category name of a product inside the sale item at the exact moment of the purchase
> **so that** historical sales reports remain completely stable and unaffected if the product is later recategorized or the category is renamed.

**Acceptance Criteria:**

```gherkin
Scenario 1: Copiar y congelar etiqueta en la inserción (Happy Path)
  Given a valid product belonging to an active category
  When  the operator executes a sale via Sale.AddItem
  Then  the system must synchronously copy the current category.name into the sale_item.category_name column
  And   the field must be saved as NOT NULL in the sales.sale_item table

Scenario 2: Aislamiento del histórico ante renombrados del catálogo (Edge Case)
  Given a recorded sale item with a frozen category_name
  When  an administrator renames that category inside the catalog table later
  Then  the existing sale_item.category_name must remain unaltered
  And   the aggregated report Q9 must group rows using the frozen value
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written for C# domain behavior mapping
- [ ] Acceptance criteria verified against database migrations
- [ ] Data migration written to handle null values before production release
- [ ] Deployed to staging (`simple-stock-flow-db-1`)

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 4 |
| Assigned to | Core Development Team |
| Status | In Progress |
| Dependencies | None (Table is currently empty) |
| Affected service(s) | simple-stock-flow-api |

---

### HU-SALES-012 — Vinculación y clave foránea restrictiva para la auditoría de ventas {#HU-SALES-012}

**Epic:** EP-001

> **As** an administrator
> **I want** the table sales.sale to require a strict foreign key constraint referencing a valid user id
> **so that** the authorship of a closed sale can never be orphaned or deleted from the system database.

**Acceptance Criteria:**

```gherkin
Scenario 1: Bloquear eliminación de usuarios con ventas registradas (Error Case)
  Given an internal operator user that has registered at least one sale
  When  a manual query or process attempts to physically delete that user row from sales.user
  Then  the database engine must throw a foreign key restriction error via FK_sale_user_id
  And   the deletion transaction must be aborted loudly

Scenario 2: Renombrado de columna e integridad de datos (Happy Path)
  Given a new sale registration request
  When  the transaction commits to the database engine
  Then  the system must map the operator to the renamed sold_by_username column
  And   simultaneously populate the sold_by_user_id UUID reference field
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 5 |
| Status | Backlog |
| Dependencies | HU-SALES-011 |
| Affected service(s) | simple-stock-flow-api |

---

## Rules for writing HUs

### 1. The role matters
Do not write "As a user" — that says nothing. Use the specific role:

✓ As an administrator
✓ As a seller
✗ As a user
✗ As a person

### 2. The benefit justifies the work
The "so that" must describe a business benefit, not redescribe the action:
✓ so that historical sales reports remain completely stable and unaffected
✗ so that I can see the category column (this only describes the feature)

### 3. ACs are verifiable
Each AC must be verifiable manually or automatable as a test:
✓ Then the database engine must throw a foreign key restriction error
✓ Then the system must copy the current category.name into the column
✗ Then the system works well (not verifiable)
✗ Then the data looks nice (not verifiable)

### 4. One HU = one unit of value
If the HU has 15 ACs, it is probably 3 HUs.
The team must be able to complete it in one sprint (maximum 2 weeks).

---

## Ready-to-copy HU template

```markdown
### HU-SALES-00X — [Name] {#HU-SALES-00X}

**Epic:** EP-00X

> **As** [role: admin/seller]
> **I want** [action mapped to C# or schema rules]
> **so that** [benefit aligned with data integrity]

**Acceptance Criteria:**

\```gherkin
Scenario 1: [name]
  Given [context verified via directe engine state]
  When  [action executed by the hexagonal ports]
  Then  [expected result checked in pg_constraint]
\```

| Field | Value |
|-------|-------|
| Story Points | |
| Priority | |
| Target sprint | |
| Status | Backlog |
| Dependencies | |
```

---

## Correlations

- Full template with DoD checklist → `00-governance/definition-of-done.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Traceability matrix → `04-requirements/traceability-matrix.md`
- Core database physical mappings → `data_model.md`
