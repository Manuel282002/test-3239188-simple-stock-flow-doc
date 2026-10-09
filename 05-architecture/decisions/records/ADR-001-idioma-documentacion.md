# ADR-001 — Documentation Language

| Field | Value |
|-------|-------|
| **ID** | ADR-001 |
| **Date** | 2026-09-19 |
| **Status** | Accepted |
| **Authors** | Core Tech Lead |
| **Reviewers** | Core Development Team |

---

## Context

The software industry standard — library documentation, technical articles, frameworks, and open-source tooling — operates in English. However, the business domain language, functional requirements, and institutional governance for this project are naturally handled in Spanish. Mixing languages arbitrarily across artifacts creates confusion, so a clear and permanent architectural boundary must be established. 

Without an explicit rule, the project risks having a chaotic mix of Spanish comments in English code, English documentation that mismatches core business requirements, or mismatched naming conventions between the database schema and domain entities.

---

## Evaluated alternatives

### Alternative A — Everything in English
- **Pros:** Full industry standard; eliminates the translation boundary between docs and code.
- **Cons:** Bypasses the Spanish business definitions set by stakeholders; increases translation friction for localized core requirements.

### Alternative B — Everything in Spanish
- **Pros:** Natural communication with Spanish-speaking clients and stakeholders.
- **Cons:** Uncomfortable mix with C# language keywords (if, for, return, etc.); inconsistent with the Entity Framework Core library ecosystem.

### Alternative C — Split by layer (CHOSEN)
- **Pros:** Each artifact uses the most natural language for its context. Code, infrastructure, and mappings stay in English (Article XI) for technical consistency, while core business specifications, user task tracking, and high-level architectural documentation are kept in Spanish to preserve domain fidelity without a translation boundary.
- **Cons:** Requires strict discipline and an explicit glossary to map domain terms to code properties.

---

## Decision

**Alternative C:** Split the project language strategy strictly by layer.

| Artifact | Language | Reason |
|----------|----------|--------|
| Variables, functions, classes in code | English | Enforced by Article XI for framework consistency |
| Table and column names in DB | English | Coherence with the code that maps them (sales schema) |
| Commits (Conventional Commits) | English | Established standard, readable on GitHub |
| Git branch names | English | Consistent with commits |
| Markdown documentation | Spanish | Matches the source documents (data_model.md, tasks.md) |
| OpenAPI contracts (descriptions) | English | Enforced for API standard consistency |
| End-user error messages | English | Localization handled at runtime |
| Internal system logs | English | Facilitates search in library documentation and alerts |
| ADRs and technical documentation | Spanish/English | Hybrid (Metadata in English, technical notes in Spanish) |

---

## Consequences

**Positive:**
- Business rules and database constraints are captured accurately in Spanish without losing functional meaning.
- Technical code and database schema columns follow pure English standards (Article XI), ensuring full compatibility with EF Core.
- New team members have a clear, binding division of language boundaries from day one.

**Negative:**
- Team members must strictly maintain the domain glossary to ensure translations do not drift between Markdown docs and C# properties.

**Mitigation:**
- Maintain an exact domain dictionary with the canonical English translation for each business term inside `project-glossary.md`.
- Ensure that if a term contradicts the physical engine database layout, the motor wins (Artículo X).

---

## References

- Team documentation conventions → `00-governance/documentation-rules.md`
- Domain term glossary → `project-glossary.md`
- Physical model specifications → `data_model.md`

