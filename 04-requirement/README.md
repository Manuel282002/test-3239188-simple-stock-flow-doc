# 04 — Requirements

> **What is this?** The formal specification of what the system must do.
> Functional: what it does. Non-functional: how well it does it.

## Why this section exists

Requirements are the contract between the team and the client/stakeholder.
Without them:
- There is no way to verify whether the system is complete
- Scope changes have no baseline for comparison
- Tests have no success criterion

---

## Types of requirements

### Functional (FR)
Describe **what the system does**: functions, behaviors, data transformations.
*Example: "The system must allow an authenticated seller to process a stock deduction atomically."*

### Non-functional (NFR)
Describe **how it does it**: quality, performance, availability, security.
*Example: "The system must respond in less than 200ms for 95% of database sales insertion requests."*

NFRs are usually harder to meet than FRs and are ignored more frequently. **They are equally important.**

---

## What is here and how to fill it in

### `functional.md` ⭐
List of all the system's functional requirements.
**Fill in:** numbered, with the module/service they belong to (simple-stock-flow-api), source (originating HU from tasks.md), priority.

**Format:**
```markdown

| ID | Module | Description | Source (HU) | Priority |
|----|--------|-------------|------------|---------|
| FR-001 | simple-stock-flow-api | The system must check stock limits natively | HU-SALES-010 | High |
```

### `non-functional.md` ⭐
Quality, performance, and technical constraint requirements.
**Fill in:** by category (performance, availability, security, scalability, etc.) using the project context.

**Format:**
```markdown
## Performance

| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-001 | Response time | p95 < 200ms | Load test with k6 against core container |

## Availability

| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-002 | Uptime | 99.9% monthly | Readiness endpoint monitoring |

## Security

| ID | Requirement | Description |
|----|------------|-------------|
| NFR-004 | Authentication | Strict JWT validation with active admin/seller roles |
```

### `user-stories.md`
Formalized user stories (coming from the database backlog `tasks.md`).
**Fill in:** with As/I want/So that format + verifiable acceptance criteria in Given/When/Then.

### `traceability-matrix.md` ⭐
Table that connects: HU → Requirement → Test case.
**Fill in:** when you have requirements and tests defined. Allows coverage verification.

**Format:**
```markdown

| HU | FR/NFR | Description | Test case | Status |
|----|--------|-------------|----------|--------|
| HU-SALES-011 | FR-004 | Category Name Freeze | `CategoryNameFreezeTests.cs` | 🟡 |
```

### `_template-hu.md`
Template for a complete User Story with acceptance criteria.

### `_template-nfr.md`
Template for specifying non-functional requirements with their verification metrics.

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `plan.md` | Performance/availability NFRs guide hexagonal layer and architecture decisions |
| `data_model.md` §4 | Each FR/NFR must be supported by an engine constraint or C# validator |
| `tasks.md` | Requirements are grouped by responsible implementation tasks |
| `data_model.md` §13 | Very demanding database structures usually generate tracking debt records |

---

## Common mistakes to avoid

❌ **"The system must be fast"** → Not measurable. Better: "p95 < 200ms"

❌ **"The system must be secure"** → Not verifiable. Better: "Authentication with JWT, roles set to admin/seller pair"

❌ Writing requirements that describe the solution instead of the problem.

✅ A good requirement is: **specific, measurable, achievable, relevant, and verifiable**.

---

## Questions this section must answer

- What must the system do for each type of internal operator?
- With what speed, availability, and security?
- Which requirement originates each test case?
- Are all requirements covered by tests inside the CI pipeline?

