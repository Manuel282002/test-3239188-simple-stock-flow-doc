# Domain Events

> **What to fill in here:** A domain event is a fact that occurred in the business.
> They are the backbone of asynchronous communication between bounded contexts.
> The name is ALWAYS in past tense and in the ubiquitous language of the domain.

---

## What is a domain event?

A **Domain Event** communicates that something important occurred in the business.
It is an immutable message that describes the fact in past tense.

✓ SaleConfirmed
✓ ProductStockWithdrawn
✓ UserRegistered
✗ CreateSale (this is a command, not an event)
✗ ProductUpdated (too generic — what changed?)
✗ SaleEvent (does not indicate what occurred)

### Difference between Command and Event

| Concept | Intent | Tense | Can fail? |
|---------|--------|-------|-----------|
| **Command** | Instruction to do something | Present | Yes |
| **Event** | Notification of something that occurred | Past | No (it already happened) |

User → [CreateSale] → System → [SaleConfirmed] → Internal Aggregates
(Command)                  (Event)

---

## Event catalog

### Event: SaleConfirmed

| Field | Value |
|-------|-------|
| **Name** | `SaleConfirmed` |
| **Bounded Context** | `sales` (Monolith Schema) |
| **Aggregate** | `Sale` (Raíz de agregado de ventas) |
| **Trigger** | Confirmación e inserción de una venta con al menos una línea en el sistema. |
| **Consumers** | `Product` (Agregado del catálogo para la barrera de ADR-002) |
| **Channel (topic)** | `N/A` (In-memory synchronous execution via C# Hexagonal Ports) |
| **Schema version** | `v1` |
| **Delivery guarantee** | Exactly-once (Guaranteed by PostgreSQL local transaction block) |

**Payload (JSON schema):**

```json
{
  "eventId": "11111111-1111-4111-8111-111111111111",
  "eventType": "SaleConfirmed",
  "aggregateId": "uuid-sale-id",
  "aggregateType": "Sale",
  "occurredAt": "2026-09-19T17:55:13Z",
  "version": 1,
  "payload": {
    "sold_by": "string (username normalized to lowercase)",
    "sold_at": "timestamptz (UTC timestamp)",
    "items": [
      {
        "product_id": "uuid",
        "product_name": "string (frozen catalog name)",
        "quantity": "integer (> 0)",
        "unit_price": "numeric(18,2)"
      }
    ]
  },
  "metadata": {
    "correlationId": "uuid-xmin-system-token",
    "causationId": "uuid-command-id",
    "userId": "uuid-operator-id"
  }
}
```

**Real payload example:**

```json
{
  "eventId": "22222222-2222-4222-8222-222222222222",
  "eventType": "SaleConfirmed",
  "aggregateId": "33333333-3333-4333-8333-333333333333",
  "aggregateType": "Sale",
  "occurredAt": "2026-09-19T21:53:44Z",
  "version": 1,
  "payload": {
    "sold_by": "ana",
    "sold_at": "2026-09-19T21:53:44Z",
    "items": [
      {
        "product_id": "44444444-4444-4444-8444-444444444444",
        "product_name": "Martillo de Bola 3lbs",
        "quantity": 2,
        "unit_price": 450.00
      }
    ]
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| Internal Product Aggregate | Invokes `Product.Withdraw` to decrement stock atomically. | Yes — Managed by local Postgres transaction check. |
| Read Model Port (D-06) | Engine calculates active aggregations on demand. | Yes — Direct non-persisted SQL calculations. |

---

## Standard fields for all events

All events must include these fields in the envelope:

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | UUID | Unique event ID (for tracing) |
| `eventType` | string | Event name in PascalCase |
| `aggregateId` | UUID | ID of the aggregate that generated the event |
| `aggregateType` | string | Aggregate type |
| `occurredAt` | ISO 8601 | When the business fact occurred (UTC timezone) |
| `version` | integer | Schema version (always 1) |
| `payload` | object | Event data (specific per type) |
| `metadata.correlationId` | UUID | System concurrency token mapping (`xmin` reference) |
| `metadata.causationId` | UUID | ID of the command that caused this event |
| `metadata.userId` | UUID | User internal operator who initiated the chain |

---

## Event flow: Sale Registration Flow

> Document here the event flows for the main business processes.
> Use the Event Storming format: orange=event, blue=command, green=view/policy, yellow=aggregate.

[Operator]
│
│  CreateSale (command)
▼
[Aggregate: Sale] ──(At-least-one-line check)
│
│  SaleConfirmed (event)
▼
[Aggregate: Product] ──(Synchronous transaction check)
│
│  ProductStockWithdrawn (event)
▼
[PostgreSQL Engine] ──(Enforces ck_product_stock_non_negative)

### Example: Order creation flow

Operator (admin/seller)
│
│  CreateSale (command)
▼
[Aggregate: Sale]
│
│  SaleConfirmed (event)
├──────────────────────────────────┐
│                                   ▼
│                          [Aggregate: Product]
│                          Synchronously decrements stock
│                          ProductStockWithdrawn (event)
│
│  SaleConfirmed (event)
└──────────────────────────────────┐
▼
[Engine Catalog View]
Aggregates non-persisted sales report

---

## Schema evolution strategy

Events are contracts. Changing them in an incompatible way breaks consumers.

### What is a compatible change (does not break)?

✓ Add a new optional field to the payload
✓ Add a new event type
✓ Change a required field → optional

### What is an incompatible change (breaks)?

✗ Remove a field from the payload
✗ Change the type of a field (string → number)
✗ Change an optional field → required
✗ Change the event name

### How to evolve a schema without breaking consumers

**Strategy: Version the event**

Step 1: Publish EventV2 (new type with incompatible changes)
Step 2: Publish both EventV1 and EventV2 during the migration period
Step 3: Migrate internal C# domain maps to V2 one by one
Step 4: Deprecate EventV1 (announce 1 sprint in advance in tasks.md)
Step 5: Stop publishing EventV1

---

## Event summary table

| Event | Origin context | Topic | Consumers | Version |
|-------|---------------|-------|-----------|---------|
| `SaleConfirmed` | `sales` (Monolith) | `N/A` | Internal Domain | v1 |
| `ProductStockWithdrawn` | `sales` (Monolith) | `N/A` | Database Engine | v1 |

---

## Policies — Reactions to events

A **Policy** (or Saga step) describes what happens automatically when an event arrives.
It is the logic of "whenever X occurs, do Y".

Event:  SaleConfirmed
Policy: Whenever a SaleConfirmed arrives with items,
emit the domain rule Product.Withdraw synchronously.

| Trigger event | Policy | Emitted command | Service |
|--------------|--------|----------------|---------|
| `SaleConfirmed` | Whenever a sale occurs, then check stock restrictions | `Product.Withdraw` | simple-stock-flow-api |

---

## Resilience patterns for events

### At-least-once delivery + Idempotency

The message broker guarantees the event is delivered **at least once** but it may be
delivered more than once (in case of retries). Consumers must be **idempotent**.

```typescript
// Idempotent consumer — synchronous execution within PostgreSQL context
async function processSaleConfirmedEvent(event: SaleConfirmed): Promise<void> {
  // 1. Transactional isolation ensures database engine validation passes
  // 2. Concurrency token check uses xmin system column to verify state
  await updateModel(event.payload);
}
```

### Dead Letter Queue (DLQ)

When an event fails after N retries, it goes to the DLQ.

| Configuration | Recommended value |
|--------------|------------------|
| Retries before DLQ | `N/A` (Synchronous memory execution) |
| Backoff | `N/A` (Fails immediately to the API response layer) |
| DLQ retention | `N/A` |
| Alert | `N/A` |
> Asynchronous messaging structures and DLQs are omitted by core architecture configuration. All transaction verification resides directly on the PostgreSQL 16.14 engine constraints.


