# Business Plan - Clever Demo App to Upgrade Backend Integration

## Feature Context

- feature_id: bridge-clever-sso
- domain: bridge
- service: clever
- track: express
- phase: expressplan
- write scope: docs/bridge/clever/bridge-clever-sso/
- primary implementation surfaces: TargetProjects/bridge/clever/clever-demo-app and TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend

## Business Problem

NorthStar needs a safe path to adopt Clever SSO while preserving current users, district context, and authorization semantics. The newly cloned `clever-demo-app` already demonstrates Clever OAuth login, role dashboards, and Clever API v3 roster synchronization, but it is disconnected from the Upgrade backend that owns NorthStar identity, district mapping, data access, and migration state.

This feature slice should turn the demo app into a controlled Clever-facing integration harness for the Upgrade backend. A user signs in through Clever in the demo app, the demo app calls `NS4.WebAPI` through a server-to-server integration boundary, and the backend verifies the Clever identity, maps the Clever district and user to NorthStar records, and returns a backend-approved link/session status. This proves the Clever identity and migration boundary before changing the production Upgrade login UI.

## Users And Stakeholders

- District administrators, staff/admins, teachers, and students represented in Clever sandbox or pilot districts.
- Existing NorthStar users whose accounts must continue to resolve to the correct district, staff record, role, and authorization behavior after linking.
- Engineering teams responsible for `clever-demo-app`, `NS4.WebAPI`, EF/Login DB migration data, authentication boundaries, and test automation.
- District implementation and support teams responsible for Clever app setup, redirect URI configuration, sandbox testing, account-link review, and certification evidence.
- Operations and security stakeholders responsible for secrets, audit logs, rollout controls, and rollback behavior.

## Goals

- Integrate `clever-demo-app` with the Upgrade backend through explicit server-to-server API endpoints.
- Use `cleverdocs` OAuth/OIDC guidance to verify Clever identity through id_token validation, `/userinfo`, or versioned `/me` rather than trusting a local demo profile object alone.
- Preserve current NorthStar authorization expectations by mapping Clever identity to existing NorthStar district, staff, user, and claim semantics in the backend.
- Migrate or link current users without using email as the primary identity key, because Clever email may be absent or unverified.
- Keep the demo app's SQLite roster data as demo/staging data only; authoritative migration state belongs to the Upgrade backend.
- Keep the existing Upgrade login behavior untouched during this prototype/integration slice until the backend identity boundary is proven.
- Produce implementation slices that satisfy the hard governance gates for TDD red-green discipline and fully implemented BDD acceptance coverage.

## Scope

### In Scope

- `clever-demo-app` configuration for calling the Upgrade backend.
- Demo app callback integration that calls backend endpoints after Clever OAuth/OIDC login.
- Upgrade backend integration endpoints for Clever session/link status, identity verification, roster sync staging or triggering, and logout/session invalidation.
- Backend-owned account-linking strategy between Clever user identifiers and current NorthStar users.
- Claim compatibility planning for existing API and EF code that reads authenticated account, preferred username, and district claims.
- Current-user migration planning, including candidate matching, admin review, durable link records, and friendly blocked states.
- Testing and evidence for demo app login, backend verification, link outcomes, roster sync staging, and rollback.

### Out Of Scope

- Direct replacement of the production Upgrade React login UI in this express feature slice.
- Removing IdentityServer or every IdentityServer-dependent project file before the Clever/backend integration is proven.
- Making the demo app SQLite database authoritative for NorthStar account links, district maps, or migration state.
- Direct governance file edits or manual governance publication.
- Full Secure Sync implementation unless matching or provisioning requires roster data beyond SSO identity fields.
- Supporting both SAML and OIDC on the same Clever SSO application.

## Migration Considerations

- Clever User ID and `multi_role_user_id` / `sub` must be stored as external identifiers where available. Email can be used only as a matching hint or display/contact field.
- The Upgrade backend, not the demo app, must decide whether a Clever identity is linked, pending review, blocked, or invalid.
- Existing NorthStar behavior resolves staff through claims that ultimately map to `StaffDistricts` and district Staff records. The backend should initially preserve those internal claim values after a Clever account link is established.
- Existing users should be linked through a staged process: import candidate matches, review ambiguous matches, record durable Clever account links, and complete missing links during first login only with explicit policy approval.
- If roster-level matching is needed, the backend should own the Secure Sync or district-app roster synchronization path, or explicitly accept normalized demo-app roster data into backend staging tables.
- The demo app can visualize Clever roster and role data, but UI labels must distinguish Clever-sourced demo data from backend-approved NorthStar link state.
- If the current Clever app is SAML-based, a new OIDC app is required because a single Clever SSO app cannot support both SAML and OIDC.

## FinalizePlan Reconciliation Decisions

- Demo-to-backend trust boundary: the demo app may initiate Clever login and render backend-approved outcomes, but `NS4.WebAPI` remains the authority for integration authentication, Clever identity verification, district/user linking, admin capability, audit, and session invalidation.
- Session/link outcome contract: the backend returns a status of `linked`, `pendingLink`, `blocked`, or `error`, a support-safe reason code, sanitized display context, an optional opaque backend session reference for `linked`, and an expiry timestamp. The demo app must not mint or persist NorthStar JWTs.
- Protocol decision: Clever OIDC is the preferred configured path. If OIDC is unavailable, OAuth plus `/userinfo` or explicit-version `/me` is an approved fallback only when it produces the same normalized identity contract and fail-closed tests.
- Roster authority: this feature chooses backend-owned Clever API v3 roster pull for migration staging. Demo-app normalized staging is deferred unless a later reviewed story adds a backend-owned staging endpoint and validation contract.
- Cross-feature overlap: full-depth Lens context fetch on 2026-05-16 returned no related, depends_on, blocks, or service reference records for this feature. Dev must recheck overlap before changing shared identity, roster, student, or district-context models.
- Evidence location: certification, rollout, migration, and security evidence must be written under `docs/bridge/clever/bridge-clever-sso/evidence/` or an equivalent feature-doc subfolder referenced by the story.

## Success Criteria

- A supported sandbox user can sign in through Clever in `clever-demo-app`, and the demo app calls the Upgrade backend before rendering a linked, pending, blocked, or error outcome.
- The Upgrade backend revalidates Clever identity through OIDC id_token validation, `/userinfo`, or versioned `/me` before trusting user, district, or role data.
- A migrated current user resolves to the existing NorthStar district and account context without relying on Clever email as the primary key.
- Missing, disabled, or ambiguous account links produce a backend-approved friendly state rather than an incorrect login.
- The demo app does not call NorthStar databases directly and does not persist Clever access tokens in SQLite.
- Roster sync authority is explicit: either backend-owned pull from Clever API v3 or demo-app normalized staging into backend-controlled migration tables.
- Logout clears the Express session and any backend session reference created by the integration.
- Browser, sandbox user, supported user type, redirect URI, district mapping, link-status, roster-staging, and logout evidence is captured for readiness.
- Every story acceptance criterion has implemented BDD tests with no skipped, pending, or stub tests.

## Risks And Dependencies

- Clever OIDC requires Clever Support/application configuration and enabled scopes; OAuth plus `/userinfo` or `/me` must be available as a planned fallback.
- `passport-clever` may normalize fields differently from Clever OIDC `/userinfo`; backend verification is mandatory before linking.
- The demo app currently stores profile/token data in Passport session state, so token retention and session cleanup need explicit hardening before production-like validation.
- District SSO versus Library SSO behavior affects `authorized_by`, user types, available fields, and provisioning expectations.
- Existing API and EF code still relies on email-like internal claims for staff resolution; this creates a compatibility dependency until a deeper identity refactor is planned.
- Admin impersonation and the demo super-admin override need explicit retain, replace, or retire decisions before production rollout.
- Secure Sync may be required if SSO identity fields are insufficient for reliable matching and provisioning.
- Related NorthStar migration or student features may own overlapping identity, roster, or district-context decisions and should be checked before FinalizePlan.

## Hard Gates

- Express track is permitted for this feature.
- Required planning artifacts are business-plan and tech-plan; this QuickPlan also produces sprint-plan for FinalizePlan input.
- Gate mode is hard.
- enforce_stories is true.
- enforce_review is true.
- Phase gates must pass before phase advancement.
- Child governance cannot weaken parent rules.
- Production code must follow TDD red-green discipline.
- Every story acceptance criterion requires fully implemented BDD tests; skipped, pending, or stub tests do not satisfy the gate.

## Source References

- cleverdocs/oauth-implementation.md
- cleverdocs/oidc-implementation.md
- cleverdocs/the-userinfo-endpoint.md
- cleverdocs/migrating-to-oidc.md
- cleverdocs/security.md
- cleverdocs/users.md
- cleverdocs/district-sso-vs-library-sso.md
- cleverdocs/testing-your-sso-integration.md
- cleverdocs/district-sso-certification-guide.md
- cleverdocs/secure-sync-rostering.md
- cleverdocs/secure-sync-certification-guide.md
- TargetProjects/bridge/clever/clever-demo-app/server.js
- TargetProjects/bridge/clever/clever-demo-app/db.js
- TargetProjects/bridge/clever/clever-demo-app/package.json
- TargetProjects/bridge/clever/clever-demo-app/README.md
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Program.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Controllers/AuthController.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Infrastructure/NSBaseController.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/IdentityServer/Configuration/UserService.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NorthStar.EF6/NSBaseDataService.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NorthStar.EF6/LoginContext.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/Northstar.Core/Identity/SimpleEntities.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/EntityDto/LoginDB/Entity/StaffDistrict.cs
- TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/EntityDto/Entity/Staff.cs

## Unresolved Assumptions

- Clever OIDC will be enabled for the NorthStar Clever application. If not, the same account-linking model should be implemented with Clever OAuth authorization code plus `/userinfo` or `/me`.
- The authoritative mapping between Clever `district_id` and NorthStar DistrictId is not yet defined.
- The backend-issued session format for demo app integration, JWT versus opaque session reference, is not yet selected.
- Ownership of district-app roster sync must be decided: backend-owned pull is safer, while demo-staged push is faster for prototype validation.
- Admin impersonation and the demo super-admin override have not been approved, replaced, or retired for Clever SSO.