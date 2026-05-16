---
permitted_tracks: [full, express, quickdev, hotfix-express, spike]
required_artifacts:
  planning:
    - business-plan
    - tech-plan
  dev:
    - stories
gate_mode: hard
sensing_gate_mode: informational
additional_review_participants: []
enforce_stories: true
enforce_review: true
---

# NorthStarET Org Constitution

## Preamble

This constitution defines universal, non-negotiable engineering rules that apply to every codebase, repository, and initiative within the NorthStarET organization. Child constitutions at domain, service, and repo layers may add rules; they may never remove or weaken the rules established here.

## Articles

### Article 1: Phase Gating

**Rule:** No phase may advance without all required gates passing for its track and size. Gate failures are blocking regardless of deadline pressure. A phase is not complete until its gate checklist is satisfied and recorded.

**Evidence required:** Gate checklist signed off in the phase artifact. CI must be green. No open blocking issues against the phase.

### Article 2: Additive Governance

**Rule:** Child constitutions at domain, service, or repo layers may only add governance rules. They may never remove, weaken, narrow, or create exceptions to any article ratified at a higher layer.

**Evidence required:** Any constitution amendment or new child constitution must include an inheritance validation record confirming no parent rule was weakened or removed.

### Article 3: TDD Red-Green Discipline

**Rule:** Test-Driven Development red-green discipline is mandatory for all production code. Every unit of new or modified behavior must begin with a failing test, followed by the minimum production code to make it pass, followed by refactoring with tests remaining green. No production code may be written without a prior failing test for that behavior.

**Evidence required:** Commit history must demonstrate the red-green cycle for each story or task. CI gates must include a test coverage check showing all new paths are covered.

### Article 4: BDD Acceptance Criteria Tests

**Rule:** Every story acceptance criterion must have a corresponding, fully implemented BDD test. Stub tests, skipped tests, pending tests, placeholder tests, or tests with empty bodies are prohibited. A story is not done until all acceptance criteria are verified by passing, non-stub BDD tests that exercise real production behavior.

**Evidence required:** BDD test suite must have one or more fully implemented, passing scenarios per acceptance criterion. No pending, skipped, or empty test constructs may appear in the accepted test suite.

## Amendment Log

| Date | Amendment | Rationale | Author |
|------|-----------|-----------|--------|
| 2026-02-27 | Initial ratification, root resolver copy | Restore canonical org constitution path required by Lens resolver | Cris Weber |