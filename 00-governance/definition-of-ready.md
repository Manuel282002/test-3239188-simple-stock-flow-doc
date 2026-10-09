# Definition of Ready (DoR)

> A User Story is **Ready** when the entire team can start it in the next sprint
> without needing to resolve fundamental questions mid-sprint.
> If a story doesn't meet this DoR, it goes back to refinement.

---

## DoR checklist

Before moving a User Story to "Ready for Sprint", verify:

### Clarity

- [ ] The story is written in the format: **As [role], I want [action], so that [benefit]**
- [ ] The role is specific (not "as a user" — "as an admin" or "as a seller")
- [ ] The expected benefit is clear and verifiable

### Acceptance Criteria

- [ ] There are at least 2 acceptance criteria written in **Given / When / Then** format
- [ ] The criteria cover the happy path AND the main error cases
- [ ] The criteria are testable (it is possible to write an automated test for each one)
- [ ] There are no ambiguous criteria ("the response should be fast" is not valid)

### Dependencies

- [ ] All external dependencies (other services, APIs, data) are identified
- [ ] Blocking dependencies are resolved OR a workaround is defined
- [ ] If it depends on another story, that story is already Done or In Progress

### Estimation

- [ ] The team has estimated the story (story points or t-shirt size)
- [ ] There is agreement that the story fits in one sprint
- [ ] If it's too large, it has been broken down into smaller stories

### Technical readiness

- [ ] The necessary accesses and environments are available (PostgreSQL 16.14 `simple-stock-flow-db-1`)
- [ ] The API contracts (OpenAPI) are defined if the story involves new endpoints
- [ ] There is a definition of the data model if there are DB changes (aligned with `sales` schema)
- [ ] The impact on other services is identified

### Non-functional requirements

- [ ] Performance requirements are specified (if applicable)
- [ ] Security requirements are considered (authentication, authorization, role validations)
- [ ] Observability requirements are included (logs, metrics, traces)

---
