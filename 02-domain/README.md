# 02 — Problem Domain

> **What is this?** The mental model of the business. It is not technology — it is understanding
> the problem the system solves before writing code. This section comes from Domain-Driven Design (DDD).

## Why this section exists

The most costly mistakes in software are not bugs — they are domain misunderstandings.
When developers do not deeply understand the business:
- They create incorrect abstractions that have to be rewritten
- Names in the code do not match the business's names → permanent confusion
- Bounded context and transactional boundaries are drawn incorrectly

This section captures domain knowledge **before** designing the architecture.

---

## Key concepts you must know

**Entity:** Domain object with a unique identity (e.g.: a `Product` identified by its UUID).

**Value Object:** Object with no identity of its own, defined by its attributes (e.g.: `Money`, `Quantity`).

**Aggregate:** Group of entities treated as a unit. Only the aggregate root can be referenced from outside (e.g.: `Sale` managing its internal `SaleItem` rows).

**Domain Event:** Something that occurred in the business that other parts of the system must know (e.g.: `SaleConfirmed`, `ProductStockWithdrawn`). They are facts, stated in past tense and executed synchronously in memory.

**Bounded Context:** Area of the system where a particular model applies. Within this monolithic architecture, they define logical boundaries aligned to core business functions.

---

## What is here and how to fill it in

### `domain-map.md` ⭐
Map of all bounded contexts and how they relate.
**Fill in:** draw the contexts as rectangles and the relationships between them (upstream/downstream, customer/supplier) mapped in C# domain layer port interactions.

**Format:**
```markdown
## Bounded Contexts

### SalesContext
**Responsibility:** Captures immutable transactions and freezes sales history records.
**Main entities:** Sale, SaleItem
**Owning team:** Core Development Team

## Relationship map
[ASCII diagram mapping sifting between CatalogContext, SalesContext and UserContext]

| Context A | Relationship | Context B | Description |
|-----------|-------------|-----------|-------------|
| CatalogContext | customer-of | SalesContext | SalesContext consumes product catalog details to create frozen sale rows |
```

### `entities-and-rules.md` ⭐
Catalog of entities, value objects, and business rules.
**Fill in:** for each entity: name, attributes, invariants (rules that MUST ALWAYS hold), behaviors mapped to C# code blocks.

**Format:**
```markdown
## Entity: Product
**Belongs to:** CatalogContext
**Identifier:** Id (Guid/UUID)

### Attributes

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| stock | int | Available stock units | Yes | Enforces stock >= 0 via database check |

### Business rules (invariants)
- [ ] stock >= 0 (ck_product_stock_non_negative constraint protection)

### Behaviors (domain methods)
- `Withdraw(quantity)`: Atomically decrements the available product stock units
```

### `domain-events.md` ⭐
List of all events that occur in the domain.
**Fill in:** event name (past tense), what triggers it, what data it carries, who consumes it in memory.

**Format:**
```markdown

| Event | Triggered by | Data | Consumers | Bounded Context |
|-------|-------------|------|-----------|----------------|
| SaleConfirmed | Inserción de venta confirmada | UUIDs, quantities, frozen prices | Product Aggregate | SalesContext |
```

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `plan.md` & `adr/` | Logical bounded contexts determine the structure of the hexagonal core layers |
| `data_model.md` | Entities translate directly into singular tables (`product`, `sale`, `sale_item`, `user`, `category`) |
| `tasks.md` | Business rules debts are tracked as pending migrations (e.g., T-11, T-12, T-20) |
| `spec.md` | Business invariants match directly with functional requirements and acceptance criteria |

---

## Recommended tool: Event Storming

**Event Storming** is a domain discovery workshop with sticky notes:
1. 🟠 Orange: Domain events (past tense like `SaleConfirmed`)
2. 🔵 Blue: Commands (what triggers the event like `CreateSale`)
3. 🟡 Yellow: Actors (internal operators like `admin` or `seller`)
4. 🟣 Purple: Policies (automatic synchronous domain reactions)
5. 🟦 Light blue: External systems (like the external image binary storage)

Running an Event Storming session with the team before filling in this section saves weeks of redesign.

---

## Questions this section must answer

- What are the main business entities?
- What rules can NEVER be violated in the system?
- What important events occur in the domain?
- Where are the natural boundaries of the system (for defining database layers and tables)?

