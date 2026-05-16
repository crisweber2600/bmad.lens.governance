# Sprint Plan - Clever Demo App to Upgrade Backend Integration

## Planning Context

- feature_id: bridge-clever-sso
- track: express
- phase: expressplan
- target implementation roots: TargetProjects/bridge/clever/clever-demo-app and TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend
- planning artifacts: business-plan.md, tech-plan.md, sprint-plan.md
- first-slice intent: prove the Clever demo app to Upgrade backend identity, linking, and migration boundary before changing the production Upgrade login UI

## Sprint Sequencing

| Order | Story | Purpose | Key Dependencies |
| --- | --- | --- | --- |
| 1 | SSO-1 Demo/backend configuration and integration auth boundary | Establish secure demo app, backend, feature flag, integration credential, and test harness. | Clever app details, backend URL, redirect URI policy. |
| 2 | SSO-2 Demo Clever callback to backend session handshake | Wire `clever-demo-app` callback flow to call `NS4.WebAPI` before dashboard/admin routing. | SSO-1. |
| 3 | SSO-3 Backend Clever identity verification and normalization | Revalidate Clever identity through OIDC id_token, `/userinfo`, or versioned `/me` and normalize identity data. | SSO-1, Clever OIDC or OAuth fallback decision. |
| 4 | SSO-4 District and account linking model | Map Clever identity to NorthStar district and current users without email as primary key. | SSO-3, migration source data. |
| 5 | SSO-5 Demo dashboard link-status and blocked-state UX | Display backend-approved linked, pending, blocked, and error outcomes in the demo app. | SSO-2, SSO-4. |
| 6 | SSO-6 Roster sync staging and migration workflow | Stage or trigger backend-owned roster sync and produce candidate links, approvals, and reports. | SSO-3, SSO-4, Clever roster access decision. |
| 7 | SSO-7 Certification evidence, rollout controls, and fallback | Validate supported demo-to-backend flows and prepare a reversible rollout decision. | SSO-5, SSO-6. |

## Quality Gates

- Gate mode is hard.
- enforce_stories is true.
- enforce_review is true.
- Production implementation must follow TDD red-green discipline.
- Every acceptance criterion below requires a fully implemented BDD test. Skipped, pending, or stub tests do not satisfy the gate.
- Phase advancement is blocked until review passes.

## Stories

### SSO-1 - Demo/Backend Configuration And Integration Auth Boundary

Create the secure configuration, feature flag, and integration authentication needed for `clever-demo-app` to call the Upgrade backend without changing default production Upgrade login behavior.

Dependencies:

- Clever client id, client secret, redirect URI, issuer/discovery URL, enabled protocol, and supported user types.
- Upgrade backend local and deployed API base URL.
- Decision on demo-to-backend integration credential type: client secret, HMAC, private key, or equivalent server-to-server credential.

Acceptance criteria:

- Given required Clever or backend integration config is missing, when the demo app starts or sends a backend request, then it fails closed with an operational error and does not expose secrets.
- Given the demo app calls `NS4.WebAPI`, when the request reaches the backend, then the backend authenticates the integration credential and rejects direct browser-origin calls to integration endpoints.
- Given non-local environment config is loaded, when demo-to-backend URL and Clever redirect URI are validated, then HTTPS is required.
- Given the repository is searched, when checking source and committed config, then Clever client secrets and backend integration secrets are not present outside environment-backed configuration.

Tests and evidence:

- Node unit tests for demo config validation and backend API client credential handling.
- Backend unit tests for integration authentication and fail-closed behavior.
- BDD test for missing configuration and rejected unauthenticated integration calls.
- Static/config evidence that secrets are environment-backed and not committed.

### SSO-2 - Demo Clever Callback To Backend Session Handshake

Wire the existing `clever-demo-app` Clever login callback so the demo app calls the Upgrade backend before routing users to dashboard or admin views.

Dependencies:

- SSO-1.
- Existing `passport-clever` callback behavior in `TargetProjects/bridge/clever/clever-demo-app/server.js`.

Acceptance criteria:

- Given Clever redirects to `/auth/clever/callback` with a valid login, when Passport completes authentication, then the demo app calls the backend session endpoint before rendering any role dashboard.
- Given the backend returns `linked`, when the demo app continues, then the session stores only the minimum backend-approved context needed for display and does not persist raw Clever access tokens in SQLite.
- Given the backend returns `pendingLink`, `blocked`, or `error`, when the callback completes, then the demo app renders a friendly support-safe state instead of routing to normal dashboards.
- Given the user logs out, when logout completes, then the Express session and any backend session reference are cleared.

Tests and evidence:

- Node unit tests for callback result handling, backend client failures, and logout cleanup.
- Contract tests for backend response types: `linked`, `pendingLink`, `blocked`, and `error`.
- BDD tests for linked callback routing, blocked callback rendering, and logout.

### SSO-3 - Backend Clever Identity Verification And Normalization

Implement backend verification of Clever identity so `NS4.WebAPI` does not trust the demo app profile object by itself.

Dependencies:

- SSO-1.
- Clever OIDC enablement or confirmed OAuth-only fallback through `/userinfo` or versioned `/me`.

FinalizePlan decision:

- The implementation must support a configured protocol path. OIDC is preferred; OAuth plus `/userinfo` or explicit-version `/me` is allowed only as a tested fallback that returns the same normalized identity contract.

Acceptance criteria:

- Given an OIDC id_token is supplied, when the backend verifies identity, then issuer `https://clever.com`, audience, expiry, and nonce are validated before any link state is returned.
- Given identity must be fetched through `/userinfo`, when the backend calls `https://api.clever.com/userinfo` with the SSO bearer token, then it normalizes `user_id`, `sub` or `multi_role_user_id`, `user_type`, `district_id`, email, names, and `authorized_by`.
- Given `/me` is used as fallback, when the backend calls Clever, then it requests an explicit API version and stores the correct user ID format for the integration.
- Given the token is expired, invalid, or missing required identity fields, when verification fails, then the backend returns a blocked/error result and no NorthStar link/session context.

Tests and evidence:

- Backend unit tests for issuer, audience, expiry, nonce, required fields, and identity normalization.
- Backend integration tests with mocked Clever discovery, `/userinfo`, `/me`, and token failure responses.
- BDD tests for successful verification, invalid token failure, and missing required identity fields.

### SSO-4 - District And Account Linking Model

Create backend-owned district and user-link mapping so Clever identities resolve to existing NorthStar district and user context without using email as the primary key.

Dependencies:

- SSO-3.
- Existing Login DB and district Staff data access patterns.
- Decision on where to store `CleverDistrictMap`, `CleverUserLink`, and integration audit records.

Acceptance criteria:

- Given a Clever `district_id` is known, when a verified user reaches the backend, then the system maps it to a configured NorthStar DistrictId before resolving staff context.
- Given a Clever user ID has an active link, when the backend evaluates the login, then it resolves the linked NorthStar account and does not match by Clever email as the primary key.
- Given only email matches an existing user, when no approved link exists, then the backend returns `pendingLink` or `blocked` rather than auto-linking silently.
- Given multiple candidate NorthStar users match a Clever identity, when backend evaluation runs, then the backend blocks access with a friendly support path.
- Given `authorized_by` or `user_type` violates policy, when login occurs, then access is blocked or routed according to configured policy.

Tests and evidence:

- Backend data/service tests for district mapping, active link resolution, no email-primary matching, conflict blocking, and policy handling.
- BDD tests for active link login, email-only candidate pending state, ambiguous match block, and unauthorized role behavior.
- Migration evidence showing unresolved and ambiguous records are reportable.

### SSO-5 - Demo Dashboard Link-Status And Blocked-State UX

Update the demo app dashboard/admin experience to display backend-approved NorthStar link status while keeping local SQLite roster data clearly separated as Clever demo data.

Dependencies:

- SSO-2.
- SSO-4.
- Existing EJS dashboard and admin views in `clever-demo-app`.

Acceptance criteria:

- Given the backend returns a linked NorthStar context, when the user opens the demo dashboard, then the page shows the mapped NorthStar district/user context separately from Clever roster demo data.
- Given the backend returns pending or blocked status, when the page renders, then the message is understandable and does not expose tokens, secrets, raw profile payloads, or raw exception details.
- Given local SQLite roster data is displayed, when the dashboard renders, then it is labeled as Clever-sourced demo data unless backend mapping confirms it.
- Given an admin user opens admin tools, when backend authorization does not grant admin capability, then the demo super-admin override is not sufficient for backend migration actions.

Tests and evidence:

- Node view/controller tests for linked, pending, blocked, and error rendering.
- BDD/E2E tests for demo login, dashboard link-status display, blocked state, and admin authorization denial.
- Evidence that raw Clever tokens and backend secrets are not rendered or logged.

### SSO-6 - Roster Sync Staging And Migration Workflow

Define and implement the migration workflow needed to link existing NorthStar users to Clever identities safely, including roster data when required.

Dependencies:

- SSO-3.
- SSO-4.
- Existing data in `StaffDistricts`, DistrictDbs, district Staff, and any available Clever/Secure Sync roster data.
- Backend-owned Clever API v3 roster pull selected for this feature slice.

Acceptance criteria:

- Given roster sync runs, when the backend pulls from Clever API v3, then it processes pagination for users, schools, sections, and enrollments without relying on demo SQLite as authority.
- Given a demo-staged roster push is attempted, when no reviewed backend staging endpoint exists, then the backend rejects the push and records a support-safe reason.
- Given existing StaffDistricts and district Staff records, when candidate generation runs, then it produces proposed links and clearly identifies unresolved or ambiguous users.
- Given a candidate uses email only, when approval policy requires confirmation, then the candidate is not activated automatically.
- Given an approved link is activated, when the user logs in through the demo app, then the backend returns the linked existing NorthStar account.
- Given an unshared or disabled Clever user appears, when migration or login evaluates the link, then access follows the archive/disable policy instead of creating a duplicate account.

Tests and evidence:

- Backend service tests for roster pagination, staging validation, candidate generation, ambiguity detection, approved activation, and disabled/unshared behavior.
- Contract tests for demo-staged roster submission if that path is selected.
- BDD tests for migrated user login, unresolved user support path, ambiguous candidate review, and email-only candidate non-activation.
- Report artifact or command output listing migration totals and unresolved records.

### SSO-7 - Certification Evidence, Rollout Controls, And Fallback

Complete certification-oriented validation and prepare a controlled rollout decision for the demo-to-backend integration.

Dependencies:

- SSO-5.
- SSO-6.
- Clever sandbox users and redirect URI allowlist.

Acceptance criteria:

- Given sandbox users for supported user types, when certification tests run through `clever-demo-app`, then district admins, staff/admins, teachers, and students receive backend-approved outcomes according to configured support policy.
- Given browser coverage is required, when E2E tests run, then supported desktop browser login, link-status, blocked-state, and logout flows pass.
- Given school picker or district_id behavior is part of setup, when users from mapped districts log in, then the backend selects the correct NorthStar district context.
- Given demo-to-backend integration causes elevated failures during rollout, when the feature flag is disabled, then integration calls stop without deleting account-link records.
- Given rollout readiness is reviewed, when evidence is assembled, then it includes redirect allowlist, supported user types, sandbox users, link-status screens, blocked-state screens, logout, roster sync or staging evidence, and migration reporting.
- Given evidence is captured, when it is committed for review, then it is stored under `docs/bridge/clever/bridge-clever-sso/evidence/` or a story-referenced feature-doc subfolder.

Tests and evidence:

- BDD/E2E certification suite for Clever Portal or Log in with Clever through the demo app, backend link outcomes, logout, friendly errors, and district mapping.
- Rollback test showing feature flag disablement preserves links and migration records.
- Review package with certification screenshots/logs, backend audit events, and migration report.

## Risks Carried To FinalizePlan

- OIDC enablement and scopes are not confirmed.
- District SSO versus Library SSO coexistence policy is not confirmed.
- Backend-issued session format is constrained to an opaque `sessionRef` for linked outcomes unless a later reviewed change selects another format.
- Backend-owned roster pull is selected for this feature slice; demo-staged roster push is deferred.
- Secure Sync availability is not confirmed.
- Admin impersonation replacement or retirement is not confirmed.
- Existing internal email-based staff resolution remains a compatibility dependency until a broader identity refactor is planned.
- Final production user-facing login placement remains a later decision: demo app, Upgrade UI, or backend-owned flow.

## Definition Of Done For This Feature Slice

- business-plan.md and tech-plan.md have passed required review.
- Story implementation follows TDD red-green discipline.
- Every acceptance criterion has implemented BDD coverage with no skipped, pending, or stub tests.
- Clever secrets and backend integration secrets are not present in source or committed configuration.
- `clever-demo-app` can authenticate a Clever sandbox user and receive backend-approved link/session status from `NS4.WebAPI`.
- Current users can be linked, reported as pending, or blocked with clear support paths.
- Roster data authority is explicit and test-covered.
- Rollout can be enabled and disabled without destructive data changes.