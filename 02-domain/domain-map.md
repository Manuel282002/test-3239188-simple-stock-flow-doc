# Domain Map — Bounded Contexts

> **What to fill in here:** The domain map is the central DDD (Domain-Driven Design) artifact.
> It defines the system's boundaries and how they relate to each other.
> Build it first with the team and domain experts in an Event Storming session.

## Before filling in this document: Event Storming

**Event Storming** is a collaborative workshop for modeling the domain before writing code.
It lasts 2–4 hours with the whole team (dev + PO + business expert).

**Materials:** Long wall, 4-color sticky notes, markers.

**Standard colors:**

| Color | Represents | Example |
|-------|-----------|---------|
| 🟠 Orange | **Domain events** (something that happened, past tense) | `SaleConfirmed`, `ProductStockWithdrawn` |
| 🔵 Blue | **Commands** (action that triggers the event) | `CreateSale`, `WithdrawStock` |
| 🟡 Yellow | **Actors** (who executes the command) | `admin`, `seller` |
| 🩷 Pink | **External systems** or integration points | `Almacenamiento Externo de Imágenes` |

**Session steps:**
1. (30 min) Post all events that occur in the business, in chronological order, on the wall
2. (30 min) Identify which command or actor triggers each event
3. (45 min) Group related events — each group is a candidate Bounded Context
4. (30 min) Draw relationships between Bounded Contexts (who depends on whom)
5. (30 min) Discuss the resulting map and agree on names

**Result:** The session output directly feeds the 3 documents in `02-domain/`:
- Identified events → `domain-events.md`
- Entities and their rules → `data_model.md` §2
- Bounded Contexts and their map → this document

---

---

## 1. Domain overview

> One paragraph of context about the business and what problem the system solves.
> Write it without technical terms — it must be readable by a business expert.

The system manages the complete transactional cycle of catalog stock depletion and retail sales registration under a monomoneda model. It ensures operational consistency by blocking transactions that drop stock below zero and freezing product names, prices, and categories at the exact instance of the sale to maintain immutable historical records.

---

## 2. Identified Bounded Contexts

A **Bounded Context** is the explicit boundary within which a particular domain model
has consistent meaning. Each bounded context has its own Ubiquitous Language.

### Bounded Context: Sales & Core Transaccional

| Field | Value |
|-------|-------|
| **Name** | `SalesContext` |
| **Responsibility** | Captura hechos comerciales consumados, valida stock transaccional y congela históricos. |
| **Owning team** | Core Development Team |
| **Microservice(s)** | `simple-stock-flow-api` (Monolito) |
| **Database** | PostgreSQL 16.14 (Base `simple_stock_flow`, esquema `sales`) |
| **Ubiquitous Language** | `Sale`, `SaleItem`, `Quantity`, `Frozen Price`, `Total` |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Venta (Sale) | Hecho comercial consumado e inmutable. | No |
| Línea de venta (SaleItem) | Copia congelada de datos de producto unida a su venta por cascada física. | No |

---

### Bounded Context: Catalog & Inventory Management

| Field | Value |
|-------|-------|
| **Name** | `CatalogContext` |
| **Responsibility** | Administra artículos del catálogo, control numérico de stock y referencias de imágenes opacas. |
| **Owning team** | Core Development Team |
| **Microservice(s)** | `simple-stock-flow-api` (Monolito) |
| **Database** | PostgreSQL 16.14 (Base `simple_stock_flow`, esquema `sales`) |
| **Ubiquitous Language** | `Product`, `Category`, `Stock`, `Money`, `Image Key` |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Producto (Product) | Atributo vivo del catálogo con stock mutable. | Yes — En `SalesContext` sus valores se congelan en la línea. |
| Categoría (Category) | Conjunto fijo de cinco filas sembradas de solo lectura. | No |

---

## 3. Context Map

The Context Map shows relationships between bounded contexts. Relationships define
how contexts communicate and who holds the "power" in the integration.

┌─────────────────────────┐            ┌──────────────────────────┐
│  CatalogContext         │            │  SalesContext            │
│                         │───────────▶│                          │
│  Domain:                │    C/S     │  Domain:                 │
│  Product & Stock        │ (In-memory)│  Immutable Sales Records │
└─────────────────────────┘            └──────────────────────────┘
▲                                       │
│                                       │
│              U → D                    │  U → D
│         (In-memory Auth)              ▼
└─────────────────────────────────┌──────────────┐
│ UserContext  │
│ (Identity)   │
└──────────────┘

### Context relationship types

| Type | Symbol | Description | Example |
|------|--------|-------------|---------|
| **Upstream → Downstream** | `U → D` | U provides, D consumes. D depends on U. | UserContext → SalesContext |
| **Shared Kernel** | `SK` | Two teams share part of the model | Shared Schema Context (`sales.*`) |
| **Customer/Supplier** | `C/S` | Supplier (U) negotiates with Customer (D) | CatalogContext → SalesContext |
| **Anti-Corruption Layer** | `ACL` | D translates U's model to protect itself | External Image Storage Adapter |
| **Open Host Service** | `OHS` | U publishes a published protocol | C# Domain Interface / Hexagonal Ports |
| **Published Language** | `PL` | Explicit shared language | Data Model Specification |

### Relationships table

| Context A | Relationship | Context B | Communication channel | Contract |
|-----------|-------------|-----------|----------------------|---------|
| `CatalogContext` | C/S | `SalesContext` | In-Memory Port Calls (C# Adapter) | Domain Entities Assembly |
| `UserContext` | U → D | `SalesContext` | In-Memory Synchronous Auth Mapping | User Entity Reference |
| `UserContext` | U → D | `CatalogContext` | In-Memory Privilege Verification | User Entity Reference |

---

## 4. Core Domain, Supporting, Generic

DDD classifies subdomains by their strategic value:

### Classification of this project's bounded contexts

| Bounded Context | Type | Justification |
|----------------|------|---------------|
| `SalesContext` | Core | Controla las transacciones comerciales, el cálculo dinámico del reporte sin persistencia y la inmutabilidad contable del sistema. |
| `CatalogContext` | Supporting | Provee soporte operativo al core inyectando la regla de negocio crítica de `stock >= 0` mediante restricciones físicas en el motor (`ck_product_stock_non_negative`). |
| `UserContext` | Generic | Resuelve la identidad interna simplificada restringida a dos roles estáticos (`admin` / `seller`) y hashes de contraseñas. |

---

## 5. Modeling decisions

### How were these decisions made?

- **Event Storming session:** 2026-09-19, Core Development Team.
- **Tool used:** Physical whiteboard verified against database schema queries.
- **Map iterations:** v1 (Initial plural database structure), v2 (Refactored to singular qualified `sales` layout under ADR-001).

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| Mantener el diseño en un Monolito Hexagonal | Separar en Microservicios distribuidos | El volumen de datos actual (5 categorías semilla, 1 usuario inicial) no justifica la complejidad de red, patrones Saga o DLQs asíncronas. |
| Reportes calculados en caliente en el motor (D-06) | Tabla de reportes persistida o base de datos analítica | Evita la desnormalización y la reescritura de reportes antiguos, apalancándose de los índices compuestos creados en T-13. |

---

## 6. How to update this map

1. Before adding any new entity, verify whether it belongs to an existing root aggregate (`Product`, `Sale`, `User`).
2. If a context's ubiquitous language is changing, review the schema singularization rules in `data_model.md` §0.
3. Run a baseline verification against PostgreSQL constraints every time a database migration is introduced.

> **Important correlation:** The bounded contexts in this document →
> C# Project Domain Layer Assemblies →
> Active Constraints and Physical Entities in `data_model.md` →
> Technical debt tracking issues inside `tasks.md`.
