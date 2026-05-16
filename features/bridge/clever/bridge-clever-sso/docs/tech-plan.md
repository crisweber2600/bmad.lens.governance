# Tech Plan - Clever Demo App to Upgrade Backend Integration

## Architecture Summary

Integrate `TargetProjects/bridge/clever/clever-demo-app` with `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend` using the local `cleverdocs` guidance as the protocol and data-access source of truth. The demo app remains the Clever-facing prototype for OAuth login, role dashboards, and roster synchronization. The Upgrade backend becomes the NorthStar authority for district mapping, current-user migration, account linking, and any application session or API token that grants access to NorthStar data.

This is no longer scoped as a first-step replacement of the existing Upgrade login UI. Instead, the first implementation should make the Node/Express demo app call the Upgrade backend through explicit integration endpoints. The backend must verify Clever identity, map the Clever district and user to existing NorthStar records, and return a NorthStar-compatible login context or session result. The demo app may keep its SQLite database for local Clever roster exploration, but authoritative NorthStar migration state belongs in the backend data stores.

Preferred identity protocol is Clever OIDC when enabled for the app because it provides issuer `https://clever.com`, discovery at `https://clever.com/.well-known/openid-configuration`, id_token validation, nonce, and identity claims. If OIDC is not enabled, use Clever OAuth 2.0 authorization code login plus `https://api.clever.com/userinfo` or `/me` as documented in `cleverdocs/oauth-implementation.md` and `cleverdocs/the-userinfo-endpoint.md`.

## Current Implementation Surface

- `TargetProjects/bridge/clever/clever-demo-app/server.js` is a Node/Express app using `passport-clever`, `express-session`, EJS views, SQLite, and Clever OAuth callback `/auth/clever/callback`.
- `server.js` stores the Clever SSO access token on the Passport profile, derives role and district data, creates local SQLite users, routes students/teachers/admins to dashboards, and runs admin roster sync from Clever API v3.
- `TargetProjects/bridge/clever/clever-demo-app/db.js` creates local SQLite tables for users, schools, sections, enrollments, user roles, user schools, and events. These are useful for demo and staging, not authoritative NorthStar identity records.
- `TargetProjects/bridge/clever/clever-demo-app/README.md` documents a local redirect URI of `http://localhost:3000/auth/clever/callback`, OAuth-enabled Clever Demo App setup, Data API permissions, and local `CLEVER_CLIENT_ID` / `CLEVER_CLIENT_SECRET` configuration.
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Program.cs` currently validates JWT bearer tokens using the configured `IdentityServer` authority and `ValidateAudience=false`.
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Controllers/AuthController.cs` refreshes IdentityServer refresh tokens at `IdentityServer/connect/token`.
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Infrastructure/NSBaseController.cs` and `NorthStar.EF6/NSBaseDataService.cs` depend on NorthStar claims such as authenticated account, preferred username, and district id to resolve current user and tenant context.

## Target Integration Topology

The integration boundary should be server-to-server between the demo app and the Upgrade backend. The browser should interact with the demo app, and the demo app should call backend endpoints over HTTPS with a configured integration credential.

1. User starts Clever login in `clever-demo-app`.
2. Clever redirects to `/auth/clever/callback` in the demo app.
3. The demo app receives the Clever SSO access token and profile from `passport-clever`.
4. The demo app calls the Upgrade backend integration endpoint with a short-lived Clever identity envelope. The backend must not trust the profile alone; it should revalidate with `/userinfo`, `/me`, or OIDC id_token validation where available.
5. The backend resolves `CleverDistrictMap` and `CleverUserLink` records against existing NorthStar users, `StaffDistricts`, district staff records, and any approved migration data.
6. The backend returns a NorthStar login context, link status, or application session/token result.
7. The demo app stores only the minimum backend session/context needed for the browser session and renders dashboard/linking status from backend-approved data.

## Integration Contract Decisions

- The backend response schema is anchored on `status`, `reasonCode`, `displayContext`, `correlationId`, `expiresAt`, and optional `sessionRef`. `status` must be one of `linked`, `pendingLink`, `blocked`, or `error`.
- `sessionRef` is an opaque backend-owned reference returned only for `linked` outcomes. The demo app stores it in the Express session only long enough to render the prototype flow and call logout; it must not persist it in SQLite.
- The demo app must not store raw Clever access tokens, authorization codes, backend credentials, or full Clever profiles in SQLite, views, logs, or long-lived session data.
- The backend must authenticate the demo app integration credential before any Clever identity verification work and must reject direct browser-origin calls to integration endpoints.
- Backend-owned Clever API v3 roster pull is the selected migration-staging path for this feature. Any demo-staged roster push is out of scope until a later reviewed plan defines validation, audit, and rollback behavior.
- Admin capability for migration actions must come from backend authorization policy. The demo super-admin override may not authorize backend migration or roster actions.
- Certification and rollout evidence belongs under the feature docs evidence folder so later gates can inspect redirect allowlists, sandbox users, link-status screens, blocked-state screens, logout, roster sync, and migration reports.

## Demo App Changes

### Backend API Client

Add a small backend client module to `clever-demo-app` that owns all calls to `NS4.WebAPI`.

Required configuration:

- `UPGRADE_API_BASE_URL`
- `UPGRADE_API_CLIENT_ID` or named integration principal
- `UPGRADE_API_CLIENT_SECRET` or HMAC/private-key credential
- `UPGRADE_API_TIMEOUT_MS`
- `UPGRADE_API_TLS_REQUIRED` for non-local environments

The demo app should not call NorthStar databases directly. It should call backend endpoints and display backend responses.

### Clever Callback Integration

After successful `passport.authenticate('clever')`, call the backend before routing to `/dashboard` or `/admin`.

The backend request should include:

- SSO access token, sent server-to-server only when the backend will revalidate with `/userinfo` or `/me`.
- Clever profile identifiers: `user_id`, `sub` or `multi_role_user_id`, `district_id`, `user_type`, `authorized_by`, email if present, and display names.
- Login source: Clever Portal, Instant Login, or Log in with Clever when detectable.
- Demo session correlation id for support tracing.

The backend response should include one of:

- `linked`: NorthStar user and district context resolved, with a backend-issued app token/session reference if needed.
- `pendingLink`: candidate match exists but requires approval.
- `blocked`: disabled district, unsupported user type, ambiguous match, missing district map, or conflict.
- `error`: retryable Clever/backend failure with a support-safe message.

### Dashboard And Admin Integration

- Show backend link status and NorthStar district/user context in the demo dashboard.
- Keep SQLite roster views as Clever demo data, but label them as Clever-sourced data unless confirmed by backend mapping.
- For admin sync, either call a backend endpoint to trigger backend-owned roster synchronization or push a normalized roster snapshot to backend staging endpoints. Do not make SQLite the migration authority.
- Add a demo app logout path that clears the Express session and any backend session reference returned by `NS4.WebAPI`.

## Upgrade Backend Endpoints

Proposed endpoint names are placeholders and should be aligned with existing `NS4.WebAPI` controller conventions.

- `POST /api/integrations/clever/sessions`
  - Accepts the demo app's server-to-server Clever login envelope.
  - Authenticates the demo app integration credential.
  - Revalidates identity with OIDC id_token validation, `/userinfo`, or versioned `/me`.
  - Normalizes Clever identity and resolves NorthStar district/user link state.
  - Returns linked, pending, blocked, or error result.

- `POST /api/integrations/clever/roster-sync`
  - Triggers backend-owned district-app roster sync for this feature slice.
  - Uses district-app tokens only on the server side.
  - Processes Clever API v3 pagination for `/users`, `/sections`, and `/schools` if the backend owns the pull.
  - Stages candidate account links and school/district mappings for review.

- `GET /api/integrations/clever/link-status`
  - Returns the current migration/link status for a Clever user and district.
  - Requires integration authentication and should not expose raw tokens or secrets.

- `POST /api/integrations/clever/logout`
  - Clears or invalidates backend-issued session references when the demo app logs out.

## Backend Services

- `CleverIntegrationAuth`: validates the demo app integration credential and rejects browser-origin direct calls.
- `CleverIdentityVerifier`: validates OIDC id_tokens or calls `/userinfo` or `/me` with the SSO bearer token. Always requests explicit API version when using `/me`.
- `CleverIdentityNormalizer`: normalizes `user_id`, `sub`, `multi_role_user_id`, `user_type`, `district_id`, `email`, names, and `authorized_by`.
- `CleverRosterSyncService`: ports the useful parts of the demo app's `fetchAllCleverRecords` logic into backend-owned services if the backend will pull roster data directly.
- `CleverAccountLinkingService`: maps Clever identities to existing NorthStar users, staff, district context, and migration state.
- `NorthStarAppSessionService`: issues the backend session, JWT, or login context accepted by existing `NS4.WebAPI` flows.
- `CleverMigrationAdminService`: imports candidates, records approved links, reports unresolved users, and supports rollback without deleting existing users.

## Token And Session Strategy

- Do not send Clever client secrets to the browser.
- Do not persist Clever SSO access tokens in SQLite.
- Do not serialize full Clever access tokens into long-lived Express session state. Store only short-lived server-side values needed to complete backend verification.
- The Upgrade backend should verify Clever identity itself instead of trusting a demo app profile object.
- Clever SSO access tokens are not full Data API tokens. Per `cleverdocs`, they can access only `/me`, `/districts/{id}`, and `/users/{id}` unless district-app or Secure Sync access is used.
- The demo app's district sync currently retrieves district-app tokens from `https://clever.com/oauth/tokens`; production use of that capability should move to backend-owned configuration and audit.
- Existing IdentityServer refresh-token behavior must remain isolated from Clever-authenticated sessions. The backend should never try to refresh Clever SSO tokens through `AuthController.GetRefreshToken`.
- If a backend-issued JWT is returned to the demo app, it must contain the same NorthStar-compatible claims current backend services expect and must be validated by local issuer/signing-key configuration rather than Clever directly.

## Claim And Context Mapping

The backend should preserve existing NorthStar claim expectations so current data services continue to resolve district and current staff context.

| NorthStar expectation | Clever or linking source | Notes |
| --- | --- | --- |
| AuthenticatedAccount claim | Existing linked NorthStar staff/account identifier | Preserve compatibility for `NSBaseController` and `NSBaseDataService`. |
| preferred_username | Existing linked NorthStar username or staff email | Compatibility value only; not the durable Clever identity key. |
| district claim | NorthStar DistrictId from `CleverDistrictMap` | Map from Clever `district_id`, not email domain. |
| display email | Clever email or existing staff email | Display/contact only. Clever email may be absent or unverified. |
| external user id | Clever `user_id` | Store as external identity key where applicable. |
| external subject | Clever `sub` / `multi_role_user_id` | Required for OIDC subject continuity and multi-role behavior. |
| user type | Clever `user_type` | Enforce supported roles and map to NorthStar authorization policy. |
| authorized_by | `/userinfo` `authorized_by` | Needed when district and other authorization modes can coexist. |

## Data Model And Account Linking

Authoritative migration records belong in the Upgrade backend data stores, not in the demo app SQLite database.

Proposed backend entities:

- `CleverDistrictMap`
  - `CleverDistrictId`
  - `NorthStarDistrictId`
  - `IntegrationType` or `AuthorizedByPolicy`
  - `IsEnabled`
  - `CreatedAt`, `UpdatedAt`

- `CleverUserLink`
  - `CleverUserId`
  - `CleverMultiRoleUserId`
  - `CleverDistrictId`
  - `NorthStarDistrictId`
  - `NorthStarUserId` if available
  - `InternalStaffId` or current staff/account identifier where available
  - `InternalStaffEmail` as compatibility metadata only
  - `UserType`
  - `AuthorizedBy`
  - `LinkStatus`: Pending, Active, Disabled, Conflict
  - `LastLoginAt`, `CreatedAt`, `UpdatedAt`

- `CleverDemoIntegrationAudit`
  - `CorrelationId`
  - `CleverDistrictId`
  - `CleverUserId` or hash where privacy requires
  - `ResultCode`
  - `CreatedAt`
  - No raw access tokens, authorization codes, client secrets, or full profile payloads.

Matching strategy:

- Use existing internal identifiers when already known.
- Use Clever IDs as durable external keys.
- Use email only as a candidate signal requiring deterministic policy approval or manual review.
- Use Secure Sync or district-app roster data if SSO fields are insufficient for reliable matching.
- Block login on ambiguous matches rather than choosing a user automatically.

## Migration Plan

1. Add demo app backend configuration and a dedicated backend API client.
2. Add `NS4.WebAPI` Clever integration endpoints and integration authentication.
3. Write failing tests for identity verification, demo-app envelope validation, district mapping, account-link states, and fail-closed responses.
4. Wire the demo app Clever callback to call the backend before dashboard/admin routing.
5. Add backend data records for Clever district maps and user links.
6. Generate candidate links from existing `StaffDistricts`, district staff records, and Clever roster/user data.
7. Update the demo dashboard/admin views to display backend-approved NorthStar link status.
8. Move roster sync authority to the backend or explicitly stage demo app sync results for backend review.
9. Validate current-user migration with sandbox users before enabling any production district.
10. Keep existing IdentityServer login untouched until the demo-to-backend integration proves the Clever identity and migration boundary.

## Security And Configuration

- Store Clever client secret, demo session secret, backend integration credential, and backend signing/session secrets in environment variables or secure deployment configuration.
- Require HTTPS for non-local demo-to-backend calls.
- Revalidate Clever identity in the backend and do not trust the demo app's profile object by itself.
- Validate issuer `https://clever.com`, audience, expiry, nonce, and state where OIDC is available.
- Follow Clever guidance for Clever-initiated logins that omit state: restart an app-controlled flow when state is required.
- Log failures with correlation ids and support-safe identifiers only. Do not log tokens, client secrets, authorization codes, or full Clever profiles.
- Reject unsupported user types, missing district map, disabled district, ambiguous link, and conflicting link states.
- Remove the hard-coded demo super-admin override before any production-like validation. Administrative access should come from backend authorization policy, not a demo email address.

## Testing Strategy

Hard gate: production code must follow TDD red-green discipline, and every story acceptance criterion must have fully implemented BDD tests with no skipped, pending, or stub tests.

- Node/demo unit tests for backend API client request signing, callback result handling, blocked-link rendering, and logout cleanup.
- Backend unit tests for integration authentication, Clever identity normalization, district mapping, user-link state transitions, and fail-closed decisions.
- Backend integration tests for `/userinfo` or `/me` verification, OIDC id_token validation when enabled, expired token handling, unsupported user type, missing district map, and ambiguous account link.
- Contract tests between `clever-demo-app` and `NS4.WebAPI` for linked, pendingLink, blocked, and error responses.
- Roster sync tests for Clever API v3 pagination, role/school mapping, candidate generation, and no email-primary-key enforcement.
- BDD/E2E tests for Log in with Clever through the demo app, backend link success, backend block states, admin sync staging, logout, and migrated-user district context.
- Certification evidence for redirect allowlist, supported user types, sandbox users, school picker/district behavior, browser/mobile login, and logout.

## Rollout And Fallback

- Run locally first with `clever-demo-app` on port 3000 and `NS4.WebAPI` on its configured local URL.
- Validate with a Clever sandbox district and demo app redirect URI before any production district.
- Keep the demo app integration behind explicit backend configuration so it can be disabled without touching current IdentityServer login.
- Roll out as a prototype/integration harness first, then decide whether the final user-facing flow should remain in the demo app, move into the Upgrade UI, or be implemented directly in the backend.
- Monitor login success, backend link outcomes, district mapping misses, roster sync failures, unsupported role failures, and support ticket volume.
- Rollback disables demo-to-backend Clever integration and leaves account-link records intact for audit.

## Risks

- `passport-clever` may hide or normalize fields differently from Clever OIDC `/userinfo`; the backend must verify against Clever before linking.
- The demo app currently stores profile/token data in Passport session state; production-like integration must reduce token lifetime and avoid persistence.
- District-app roster sync in the demo app is useful for discovery but should not become an unaudited path into NorthStar migration tables.
- Existing EF district context uses email-like internal claims; preserving compatibility may delay a cleaner internal identity model.
- Users with multiple roles or districts can be misrouted if `multi_role_user_id`, `user_id`, `district_id`, and `authorized_by` are not stored and tested correctly.
- Admin impersonation and demo super-admin behavior are not production policies and must be replaced with backend authorization decisions.

## Source References

- `cleverdocs/oauth-implementation.md`
- `cleverdocs/oidc-implementation.md`
- `cleverdocs/the-userinfo-endpoint.md`
- `cleverdocs/migrating-to-oidc.md`
- `cleverdocs/security.md`
- `cleverdocs/users.md`
- `cleverdocs/district-sso-vs-library-sso.md`
- `cleverdocs/testing-your-sso-integration.md`
- `cleverdocs/district-sso-certification-guide.md`
- `cleverdocs/secure-sync-rostering.md`
- `cleverdocs/secure-sync-certification-guide.md`
- `TargetProjects/bridge/clever/clever-demo-app/server.js`
- `TargetProjects/bridge/clever/clever-demo-app/db.js`
- `TargetProjects/bridge/clever/clever-demo-app/package.json`
- `TargetProjects/bridge/clever/clever-demo-app/README.md`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Program.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Controllers/AuthController.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NS4.WebAPI/Infrastructure/NSBaseController.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/IdentityServer/Configuration/UserService.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NorthStar.EF6/NSBaseDataService.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/NorthStar.EF6/LoginContext.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/Backend/Northstar.Core/Identity/SimpleEntities.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/EntityDto/LoginDB/Entity/StaffDistrict.cs`
- `TargetProjects/bridge/clever/NorthStarET/Src/Upgrade/EntityDto/Entity/Staff.cs`

## Unresolved Assumptions

- Final endpoint names and routing conventions should follow existing `NS4.WebAPI` style.
- The backend-issued session format for demo app integration, JWT versus opaque session reference, must be selected during FinalizePlan.
- The authoritative internal user key for non-staff users is not established in the gathered source facts.
- Ownership of district-app roster sync must be decided: backend-owned pull is safer, while demo-staged push is faster for prototype validation.
- Admin impersonation and demo super-admin behavior have not been approved, replaced, or retired for Clever SSO.