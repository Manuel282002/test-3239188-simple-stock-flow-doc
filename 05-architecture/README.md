# 05 — Architecture

> **What is this?** The system's design decisions: how it is organized, why,
> what alternatives were evaluated, and how it is deployed. ADRs are the treasure of this section.

## Why this section exists

A system's architecture is the set of decisions that are hard to change later.
Documenting them has three benefits:
1. **New team members** understand the system without having to ask everything from scratch
2. **The team** does not repeat already-resolved discussions
3. **Years later**, everyone remembers why each decision was made

---

## What is here and how to fill it in

### `system-architecture-overview.md` ⭐ (Start here)
High-level view of the complete system.
**Fill in:** C4 Level 1 (System) and Level 2 (Container) diagram using Mermaid, list of monolithic responsibilities, how the layers communicate synchronously in memory, and the core database engine technologies.

**Recommended format:**
```markdown
## Architecture diagram
[Mermaid graph mapping simple-stock-flow-api and simple-stock-flow-db-1 container wrapper]

## Core Component

| Component | Responsibility | Technology | DB |
|-----------|----------------|------------|-----|
| simple-stock-flow-api | Catalog and sales orchestration | C# (.NET) | PostgreSQL 16.14 |

## Communication patterns
- Sync: In-memory synchronous Layer Mappings (Hexagonal Ports)
- Persistence: Entity Framework Core mapping to qualified sales schema tables
- Gateway: N/A (Direct endpoint termination)
```

### `deployment.md` ⭐
How the system is deployed in each environment.
**Fill in:** infrastructure configuration for running the `simple-stock-flow-db-1` database engine wrapper and the C# core API under a localized Docker Compose environment.

### `cross-cutting.md`
Concerns that apply to all domain layers.
**Fill in:** standard logging masking operator data, in-engine calculated reports isolation (D-06), centralized configuration strings, and concurrency error handling via `xmin` system tokens.

### `pattern-guide.md`
Catalog of design patterns used in the project.
**Fill in:** for each pattern: name (Factory Method, Builder, Adapter, Away-from-zero rounding Strategy), when to use it, when NOT to use it, concrete example from the C# codebase.

### `security-threat-model.md`
Security threat analysis of the system.
**Fill in:** using the STRIDE methodology: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege. Mitigations include fixing the anonymous registration flaw A-1, enforcing the static admin/seller roles, and utilizing Argon2id for password hashes.

### `adr/` ⭐⭐ — Architecture Decision Records (ADRs)

#### What is an ADR?
A record of ONE important architectural decision: what was decided, why, what alternatives were evaluated, and what the consequences are. In this repository, ADRs are structural laws.

**When to create an ADR:**
- Any decision that, if changed, requires significant data refactoring or schema migrations.

adr-001-propiedad-del-esquema.md     → Why DDL is reserved strictly to EF migrations
adr-002-concurrencia-optimista.md    → Why stock >= 0 is the last barrier of defense
adr-003-baja-logica.md               → Why products use logic deletion instead of physical delete
adr-004-reporte-agregado-y-congelado.md → Why closed report stability prevents catalog rewriting

---

## Correlations with other sections

| This section is fed by... | And feeds... |
|--------------------------|-------------|
| `domain-map.md` → bounded contexts | `hexagonal-architecture.md` → domain layer project isolation |
| `non-functional.md` → NFRs | Decisions about vertical scaling and concurrency controls |
| ADRs chosen here | `data_model.md` implements the decided structural database rules |
| `deployment.md` | Core infrastructure compose environment files |

---

## The 5 most common architecture mistakes

1. **Distributed complexity too early** — Implementing message brokers or microservices when a single monolithic codebase handles the small data footprint easily.
2. **Bypassing the database engine** — Trusting memory code validations alone. A data validation rule is incomplete until backed by a physical constraint or unique index.
3. **Historical reporting tables** — Creating complex caching or persistent report logs instead of calculating on-demand aggregates via composite indices.
4. **Leaking infrastructure into domain** — Allowing Entity Framework ORM types or SQL DDL attributes to leak into the pure C# domain entities.
5. **No documented decisions** — Bypassing ADR logs, leading to teams repeating already-resolved architectural discussions.

---

## Questions this section must answer

- How is the monolith organized into logical hexagonal blocks?
- Why was PostgreSQL 16.14 chosen under a single-currency architecture?
- What alternatives were evaluated and why were they discarded?
- How is the system container deployed?
- What design patterns does the C# team apply and how?

**The four structural decisions of this project:**

