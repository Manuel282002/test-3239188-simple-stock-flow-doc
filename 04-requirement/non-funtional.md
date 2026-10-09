# Non-Functional Requirements (NFR)

> NFRs define the **qualities of the system** — not what it does but how well it does it.
> The golden rule: every NFR must have a metric. "The system must be fast" is not an NFR.
> "The P99 latency of the /orders endpoint must be < 200ms under 500 RPS load" is.

---

## How to write a measurable NFR?

| Bad | Good |
|-----|------|
| "The system must be fast" | "P95 latency must be < 300ms under 1000 concurrent RPS" |
| "The system must be secure" | "All endpoints require a valid JWT; tokens expire in 1 hour" |
| "The system must scale" | "The system must support up to 5000 concurrent users without degradation" |
| "The system must be available" | "Availability SLO: 99.9% monthly (maximum 44 min downtime/month)" |

---

## NFR-001: Performance

| Attribute | Metric | Test condition |
|-----------|--------|---------------|
| P95 latency — critical endpoints | < 200ms | Under 150 concurrent RPS load |
| P99 latency — critical endpoints | < 350ms | Under 150 concurrent RPS load |
| P95 latency — non-critical endpoints | < 500ms | Normal operational load |
| Minimum throughput | 200 RPS | Without inventory state degradation |
| Service startup time | < 10 seconds | Monolith cold start inside Docker container |

**Defined critical endpoints:**
- `POST /sales` — Punto de contención transaccional crítico (Q3/Q6). Descuenta stock atómicamente y congela atributos en una sola transacción local.
- `GET /sales/report` — Consulta analítica más costosa del sistema (Q9). Realiza agregaciones en tiempo real directamente en el motor sin persistencia.

**Load testing tools:**
- k6, Apache JMeter (targeting the `simple-stock-flow-api` port)

**Where is it validated?** CI/CD in the staging pipeline utilizing the `simple-stock-flow-db-1` test instance wrapper.

---

## NFR-002: Availability

| Environment | SLO | Maintenance window | Max downtime/month |
|------------|-----|-------------------|-------------------|
| Production | 99.9% | Sundays 2am-4am UTC | 44 minutes |
| Staging | 95.0% | No restriction | 36 hours |

**Monthly error budget in production:** 44 minutes
**Error Budget policy:** If > 50% of the error budget is consumed in the first half of the month, feature deploys are frozen until the next month and stability is prioritized.

**Health checks:**
- `GET /health` — Liveness: responds 200 if the C# API process is alive.
- `GET /health/ready` — Readiness: responds 200 only if the `sales` schema connection is active and seed rows are fully responsive.

---

## NFR-003: Scalability

| Scenario | Expected behavior |
|---------|------------------|
| Gradual load growth | Vertical container resource scaling managed when CPU usage > 70% |
| Sudden spike (High sales traffic) | Transaction isolation levels maintain queue order in < 1 minute |
| Load reduction | Memory allocation releases unused transactional blocks immediately |
| Horizontal scaling limit | 1 active monolith instance (State is unified within the database engine context) |

**Strategy:** Core database transaction isolation. State concurrency and double-write prevention are strictly handled by the native PostgreSQL `xmin` token inside the local repository adapter (ADR-002), avoiding memory cache dependencies.

---

## NFR-004: Security

### Authentication and Authorization
- All private endpoints require a valid JWT in the `Authorization: Bearer <token>` header (Defecto A-1 corregido: No-token -> 401 Unauthorized).
- JWT tokens expire in **1 hour**.
- Refresh tokens valid for **7 days**.
- RBAC (Role-Based Access Control): Set to the closed pair of internal operator roles defined in the domain: `admin` and `seller`. (High privilege constraints enforced by C# policy).

### Data transmission
- HTTPS mandatory in production (TLS 1.2+).
- HTTP only in local development setups.

### Sensitive data
- Passwords: Pure application layer hashing with Argon2id. The domain layer never stores or views the password in clear text (D-09).
- PII (personal data): Restricted access and masking for sensitive operator values (`username` and `sold_by`).
- Secrets/keys: Injected solely via environment variables or vault deployment tokens, **never in code repository or SQL scripts**.

### OWASP Top 10
Code must be reviewed against the OWASP Top 10 on each release.
Tools: SAST (SonarQube/Snyk), dependency scanning, and active direct engine vulnerability testing in staging.

### Regulatory compliance
- Habeas Data compliance for internal operator audit trails.

---

## NFR-005: Observability

| Pillar | Requirement | Tool |
|--------|------------|------|
| Logs | Structured JSON format masking personal data (username / sold_by) | Serilog / C# Built-in Logger |
| Metrics | RED (Rate, Errors, Duration) per qualified route | Prometheus + Grafana |
| Traces | In-memory execution span tracing across hexagon layers | OpenTelemetry |
| Alerts | Immediate notification if transactional errors or check constraint failures peak | Alertmanager |

**Correlation ID:** Each request generates a unique correlation UUID propagated through application logs and tied directly to the execution transaction context.

---

## NFR-006: Maintainability

| Metric | Target |
|--------|--------|
| Test coverage | ≥ 80% of lines (≥ 95% within pure C# domain invariants: Money, Quantity, Product) |
| Cyclomatic complexity | ≤ 10 per core domain handler or modifier method |
| Technical debt | Resolution time < 1 sprint from registration inside the explicit tracking backlog (`tasks.md`) |
| Onboarding time | A new dev can deploy the infrastructure locally in < 1 hour following docker runbooks |
| Average build time | < 5 minutes in CI pipelines |

---

## NFR-007: Portability

- All core backend layers are compiled and deployed as standardized Docker container images.
- Database configurations run natively on Docker Compose wrappers matching PostgreSQL 16.14.
- No service layer relies on host native file paths or specific machine operating systems.
- Environment variables are the single source of truth for runtime configurations (such as DB connections or seeds).

---

## NFR-008: Disaster Recovery (DR / Recovery)

| Scenario | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|---------|------------------------------|-------------------------------|
| Single service failure | < 2 minutes (Docker container restart) | 0 (Stateless API layer) |
| Primary database failure | < 5 minutes (Engine connection reset and verification) | 0 (Guaranteed by synchronous transactional boundaries) |
| Availability zone loss | < 15 minutes | < 1 second |
| Full region disaster | < 4 hours (Cold disaster recovery execution) | < 1 hour (From secure offsite UTC backup blocks) |

---

## NFR priority matrix

| NFR | Priority (P1/P2/P3) | Validated in CI? | Owner |
|-----|---------------------|-----------------|-------|
| Performance | P1 | Yes (k6 scripts in staging) | Core Tech Lead |
| Availability | P1 | Yes (readiness health checks) | DevOps Admin |
| Security | P1 | Yes (SAST + Role validation checks) | Core Tech Lead |
| Scalability | P3 | Manual (checked on schema migration milestones) | DevOps Admin |
| Observability | P1 | Yes (structured JSON log assertion tests) | Core Tech Lead |
| Maintainability | P2 | Yes (coverage metrics in CI gate) | Development Team |

---

## Correlations

- Core Database Constraints & Verifications → `data_model.md`
- Active Technical Debt and Roadmap Tracking → `tasks.md`
- Core Domain Glossary and Vocabulary Limits → `project-glossary.md`
- Definition of Done Rules → `00-governance/definition-of-done.md`

