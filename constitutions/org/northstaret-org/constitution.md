---
layer: org
name: northstaret-org
ratified_at: "2026-02-27T00:00:00Z"
ratified_by: "Cris Weber"
---

# NorthStarET Org Constitution

## Preamble

This constitution defines universal, non-negotiable engineering rules that apply to every codebase, repository, and initiative within the NorthStarET organization, regardless of technology stack, project size, or team composition. These rules exist to ensure correctness, traceability, and sustainable quality are built in — not bolted on. No initiative may proceed in any phase without honoring these articles. Child constitutions at domain, service, and repo layers may add rules; they may never remove or weaken the rules established here.

---

```yaml
permitted_tracks: null
required_gates:
  - tdd-compliance
  - bdd-acceptance-coverage
  - no-stub-tests
additional_review_participants: {}
```

---

## Articles

---

### Article 1: Phase Gating

**Rule:** No phase may advance without all required gates passing for its track and size. Gate failures are blocking regardless of deadline pressure. A phase is not complete until its gate checklist is 100% satisfied and recorded.

**Rationale:** Phase gates exist to prevent defects and regressions from compounding across phases. Bypassing a gate trades short-term speed for long-term cost. The organization has determined that gate integrity is non-negotiable.

**Evidence required:** Gate checklist signed off in the phase artifact. CI must be green. No open blocking issues against the phase.

---

### Article 2: Additive Governance

**Rule:** Child constitutions — at domain, service, or repo layers — may only add governance rules. They may never remove, weaken, narrow, or create exceptions to any article ratified at a higher layer. An article title is immutable after ratification. Amendments that contradict a parent article are void.

**Rationale:** The inheritance chain derives its authority from permanence. If child layers could undermine org rules, the org constitution would offer no real protection. Governance must be reliable across all governed work.

**Evidence required:** Any constitution amendment or new child constitution must include an inheritance validation record confirming no parent rule was weakened or removed.

---

### Article 3: TDD Red-Green Discipline

**Rule:** Test-Driven Development (TDD) red-green discipline is mandatory for all production code. Every unit of new or modified behavior must begin with a failing test (Red), followed by the minimum production code to make it pass (Green), followed by refactoring with tests remaining green. No production code may be written without a prior failing test for that behavior. Skipping this cycle — including writing tests after the fact — is a violation of this article.

**Rationale:** Red-green TDD is not a quality preference; it is a design discipline. It forces clarity of intent before implementation, catches regressions at the moment they are introduced, and ensures that the test suite is causally linked to the code it governs. Post-hoc tests cannot provide these guarantees.

**Evidence required:** Commit history must demonstrate the red-green cycle (failing test commit precedes passing implementation commit) for each story or task. CI gates must include a test coverage check showing all new paths are covered.

---

### Article 4: BDD Acceptance Criteria — Fully Implemented Tests

**Rule:** Every story's acceptance criteria must have a corresponding, fully implemented BDD test. Tests must be written before or in lockstep with implementation following red-green practices. Stub tests, skipped tests, pending tests, placeholder tests, or tests with empty bodies are prohibited. A story is not done until all its acceptance criteria are verified by passing, non-stub BDD tests that exercise real production behavior.

**Rationale:** Acceptance criteria represent the contract between product and engineering. If that contract is expressed as test stubs, it is not verified — it is deferred. Deferred verification is deferred risk. The organization has determined that "done" means the contract is provably satisfied.

**Evidence required:** BDD test suite must have one or more fully implemented, passing scenarios per acceptance criterion. No `pending`, `skip`, `xit`, `xdescribe`, empty `it()`, or equivalent stub constructs may appear in the accepted test suite. Code review must confirm no stubs bypass this rule.

---

## Amendment Log

| Date | Amendment | Rationale | Author |
|------|-----------|-----------|--------|
| 2026-02-27 | Initial ratification — 4 articles | Bootstrap org governance | Cris Weber |
