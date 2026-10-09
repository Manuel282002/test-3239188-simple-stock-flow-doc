# Definition of Done (DoD)

> A User Story is **DONE** when it meets ALL criteria on this checklist.
> If even one is missing, the story is NOT done — it goes back to In Progress.

## Mandatory checklist

### Code
- [ ] Code implements all acceptance criteria of the user story
- [ ] Code was reviewed and approved by at least 1 team member (PR review)
- [ ] Code follows project standards (linting and formatting pass in CI)
- [ ] No technical debt introduced without registering it in `tasks.md`

### Tests
- [ ] Unit tests written for new business logic (such as domain invariants in C#)
- [ ] Test coverage does not decrease from the project baseline
- [ ] All tests pass locally and in CI
- [ ] Acceptance criteria verified (manual or automated)

### Integration
- [ ] Changes do not break other services (integration tests pass)
- [ ] If API changes: OpenAPI contract updated in `07-api/contracts/`
- [ ] If data model changes: service `data_model.md` updated and verified against PostgreSQL 16.14
- [ ] If new/modified events: `event-catalog.md` updated

### Deployment
- [ ] Code is mergeable to `dev` (no conflicts)
- [ ] CI/CD green on the branch
- [ ] Deployed to staging environment (`simple-stock-flow-db-1`)
- [ ] Basic smoke test passing on staging

### Documentation
- [ ] Service `README.md` updated if the public interface changed
- [ ] If a significant technical decision was made: ADR created or updated in `adr/`

---
