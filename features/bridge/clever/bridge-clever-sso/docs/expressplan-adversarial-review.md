# Adversarial Review: bridge-clever-sso / expressplan

**Reviewed:** 2026-05-16T18:03:45Z

**Source:** manual-rerun

**Overall Rating:** pass-with-warnings

## Summary

The previous critical scope drift is resolved. `business-plan.md`, `tech-plan.md`, and `sprint-plan.md` now consistently target `TargetProjects/bridge/clever/clever-demo-app` as the Clever-facing integration harness and `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend` as the authority for identity verification, district/user linking, migration state, and backend-approved session or link outcomes. The express slice no longer promises immediate replacement of the production Upgrade React login UI, and that correction removes the prior hard blocker.

The plan is coherent enough to proceed into FinalizePlan, but it still carries material decisions that must be converted into story-level acceptance criteria before dev. The largest remaining risks are security boundary hardening for the demo-to-backend server-to-server path, unresolved branch decisions around OIDC versus OAuth fallback and roster-sync ownership, and cross-feature overlap with adjacent migration/student work. Under the hard governance gates, these are acceptable as FinalizePlan warnings only if the generated stories preserve TDD red-green discipline and fully implemented BDD coverage for every acceptance criterion.

Verdict: pass-with-warnings. No critical blocker remains, but FinalizePlan should not soften the hard gates or leave the remaining choices as vague implementation discretion.

## Findings

### Critical

None.

The earlier critical finding about contradictory product and implementation scope should be closed. The refreshed planning artifacts now agree that the first slice proves a demo-app-to-Upgrade-backend integration boundary before any production Upgrade login UI replacement.

### High

| ID | Dimension | Finding | Evidence | Impact | Required FinalizePlan Treatment |
| --- | --- | --- | --- | --- | --- |
| H-1 | Complexity and Risk | The demo app is now a security-relevant browser-facing harness, but the security model is still distributed across several bullets rather than expressed as one enforceable boundary. | The artifacts require server-to-server credentials, backend identity revalidation, no SQLite token persistence, logout cleanup, direct browser-origin rejection, HTTPS outside local, safe logging, and removal of the demo super-admin override. | If FinalizePlan turns these into partial stories, the demo app can accidentally become a trusted auth relay without complete token, session, origin, CSRF/state, logging, and admin-authorization controls. | Create explicit stories or acceptance criteria for the demo-to-backend trust boundary, credential handling, callback/session cleanup, support-safe errors, and audit/log redaction. |
| H-2 | Coverage Gaps | Roster sync ownership remains a forked implementation path. | The business and tech plans allow either backend-owned Clever API v3 pull or demo-app normalized staging into backend tables; `sprint-plan.md` carries both options in SSO-6. | Both paths can be valid, but they imply different credentials, pagination handling, audit evidence, data validation, rollback, and BDD scenarios. | FinalizePlan should either choose one path for this feature slice or split the alternatives into clearly conditional stories with a decision gate before implementation starts. |
| H-3 | Assumptions and Blind Spots | The backend-issued session/link outcome contract is not yet concrete enough for contract tests. | The tech plan leaves JWT versus opaque backend session reference unresolved while the demo app must store the minimum backend-approved context and render linked, pendingLink, blocked, and error outcomes. | Without a concrete response schema and session lifetime/invalidation rule, Node/backend contract tests and logout behavior can pass against a placeholder shape that later breaks integration. | Define the response schema, token/session reference type, lifetime, invalidation semantics, and failure codes before story implementation. |
| H-4 | Cross-Feature Dependencies | Related migration/student features were detected but no explicit dependency contract exists. | Related features `finish-northstaret-migration-3a10d3` and `3261-students` were detected; no explicit `depends_on`, `blocks`, service refs, or missing service refs were found for this feature. | Adjacent work may already own identity keys, student account behavior, roster migration, or district-context decisions, creating hidden conflicts during dev. | FinalizePlan should record a checked dependency decision: either no dependency after review, or explicit dependency/blocker links for overlapping identity, roster, student, or migration ownership. |

### Medium / Low

| ID | Dimension | Finding | Evidence | Impact | Recommended Action |
| --- | --- | --- | --- | --- | --- |
| M-1 | Logic Flaws | OIDC is preferred, but OAuth plus `/userinfo` or versioned `/me` remains a fallback without a firm selection point. | All artifacts name OIDC enablement/scopes as unresolved and describe OAuth fallback behavior. | The protocol choice changes validation logic, nonce/state handling, claim shape, tests, and certification evidence. | Treat protocol confirmation as a pre-dev decision or write explicit alternative acceptance criteria that fail closed for the unavailable path. |
| M-2 | Coverage Gaps | Existing NorthStar claim consumers are referenced but not fully inventoried. | The plans cite `NSBaseController`, `NSBaseDataService`, IdentityServer refresh isolation, and claim compatibility requirements. | Hidden consumers may continue assuming IdentityServer-issued claims or email-like identifiers. | Add an auth-claim consumer inventory task or acceptance criterion before backend session issuance is finalized. |
| M-3 | Complexity and Risk | Admin impersonation and the demo super-admin override are called out but not resolved. | Business, tech, and sprint artifacts all say impersonation or the demo super-admin path must be retained, replaced, or retired. | Privileged behavior can leak through the integration if backend authorization policy is not the sole source of admin capability. | Make this a security decision with explicit BDD coverage for admin denial and any approved replacement path. |
| M-4 | Coverage Gaps | Certification and rollout evidence is named but not tied to durable artifact outputs. | The sprint plan asks for redirect allowlist, sandbox users, link-status screens, blocked-state screens, logout, roster evidence, and migration reporting. | Evidence may exist transiently but be hard to review or reproduce at the next gate. | FinalizePlan should name evidence artifacts, storage location under the feature docs path, and reviewer expectations. |

## Accepted Risks

- Express track is permitted for this feature, and the required planning artifacts are present: `business-plan.md`, `tech-plan.md`, and `sprint-plan.md`.
- Proceeding with a demo-app-to-backend integration harness is acceptable because the production Upgrade React login UI remains out of this first slice.
- Carrying OIDC enablement as an external dependency is acceptable if the OAuth `/userinfo` or versioned `/me` fallback is made explicit and test-covered.
- Preserving current NorthStar claim semantics is acceptable as a compatibility bridge, provided Clever identity keys, district maps, and account links remain the durable migration authority.
- Keeping roster sync ownership undecided is acceptable only until FinalizePlan converts the choice into a decision gate or separate conditional stories.
- The lack of explicit dependencies on `finish-northstaret-migration-3a10d3` and `3261-students` is acceptable only after the overlap check is recorded in the FinalizePlan handoff.

## Party-Mode Challenge

Raina (Product Reviewer): The scope is now coherent, but the business promise still depends on proving that a demo-facing harness can produce backend-approved outcomes that stakeholders trust. What evidence will convince support and district implementation teams that this is more than a local OAuth demo?

Malik (Security Architect): The backend is correctly named as the authority, but the demo app is still touching browser sessions, Clever tokens, backend credentials, and admin screens. Where is the single testable security contract that says exactly what the demo app may store, render, log, and forward?

Tessa (Delivery Lead): The stories are plausible, but several contain branch decisions that can expand mid-sprint. Which choices must be locked during FinalizePlan so dev is executing stories rather than discovering architecture?

## Gaps You May Not Have Considered

- Clever-initiated login behavior can interact badly with strict state/nonce expectations; the plan says to restart an app-controlled flow when state is required, but this needs a concrete callback behavior and BDD scenario.
- The demo app's existing Passport profile/session behavior may require refactoring before the backend call, not after it, because raw token retention and routing decisions happen in the callback path.
- Backend integration authentication should be testable independently from Clever identity verification; otherwise a valid Clever token could mask a weak demo-app credential boundary.
- Multi-role and multi-district users need negative tests that combine `user_id`, `multi_role_user_id` or `sub`, `district_id`, and `authorized_by`; testing each field in isolation may miss misrouting defects.
- Migration reporting needs a privacy posture: audit records should be useful for support without storing full profile payloads or raw identifiers where hashing or redaction is required.

## Open Questions Surfaced

- Which path will FinalizePlan choose for roster authority in this slice: backend-owned Clever API pull, demo-app normalized staging push, or a gated decision before SSO-6 starts?
- Will the demo-to-backend session result be a JWT, an opaque session reference, or a link-status-only response for the express prototype?
- What exact response schema and error codes will `clever-demo-app` and `NS4.WebAPI` use for `linked`, `pendingLink`, `blocked`, and `error` outcomes?
- Who will verify whether `finish-northstaret-migration-3a10d3` or `3261-students` owns overlapping identity, roster, student, or migration decisions?
- Where will certification and rollout evidence be written so the hard review gate can inspect it later?