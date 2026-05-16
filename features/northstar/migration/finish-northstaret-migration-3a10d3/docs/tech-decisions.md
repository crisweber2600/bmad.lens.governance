---
doc_type: tech-decisions
phase: techplan
initiative: finish-northstaret-migration-3a10d3
status: draft
author: "Winston (Architect) — TechPlan Phase"
timestamp: "2026-02-26T14:00:00Z"
deciders: ["Cris Weber (Tech Lead)", "Winston (Architect)"]
related: architecture.md
---

# Tech Decisions — Finish NorthStarET Migration

**Author:** Cris Weber / Winston (Architect)
**Date:** 2026-02-26
**Phase:** TechPlan
**Initiative:** finish-northstaret-migration-3a10d3

---

## Decision Index

| ID | Decision | Status |
|----|---------|--------|
| TD-001 | Do not replace recharts — verify existing primitives | Accepted |
| TD-002 | URL-parameter-based print mode (match existing `isPrintMode()`) | Accepted |
| TD-003 | POST-for-reads on DataEntryController (no REST refactor) | Accepted |
| TD-004 | `batchSave` endpoint for autosave (no new PATCH endpoint) | Accepted |
| TD-005 | API camelCase via `JsonNamingPolicy.CamelCase` (server-side) | Accepted |
| TD-006 | No Redux / Zustand — TanStack Query + React context only | Accepted |
| TD-007 | Hash routing — no migration to BrowserRouter | Accepted |
| TD-008 | Angular Strangler Fig — removal is last step, route-by-route | Accepted |
| TD-009 | Parity test gate — required before each Angular route removal | Accepted |
| TD-010 | Scoring logic unit-tested against production data fixtures | Accepted |
| TD-011 | URL state as ground truth for filter/view state on complex pages | Accepted |
| TD-012 | Feature slice directory pattern mandatory | Accepted |
| TD-013 | react-hook-form for all forms | Accepted |
| TD-014 | No new API endpoints during Phase 2–5 | Accepted |
| TD-015 | TanStack Query v5 cache TTL conventions | Accepted |

---

## TD-001 — Do Not Replace recharts

**Status:** Accepted  
**Date:** 2026-02-26

### Context

The architecture brief (businessplan phase) suggested evaluating alternatives to recharts. Chart components `BarChart`, `LineChart`, `LineChartWithBenchmarks`, `StackedBarChart`, and `MultiLineChart` already exist in `components/charts/` and use recharts. The shared utility layer (`chartUtils.ts`) is established.

### Decision

Keep recharts (current version). Do not evaluate or migrate to Victory Charts, Chart.js, or any other charting library.

### Rationale

- Five chart components already implemented and tested (`StackedBarChart.test.tsx`, `LineChart.test.tsx`, `MultiLineChart.test.tsx` confirmed)
- `chartUtils.ts` is a stable shared layer with complete color palette, theme, print detection, and date formatting
- Switching libraries mid-migration would invalidate all existing component tests and require full re-implementation of print mode, benchmark overlays, and multi-series support
- recharts is well-documented, actively maintained (v2+), and meets all current visual requirements

### Trade-offs

| Pro | Con |
|-----|-----|
| Zero migration cost | Bundle size (~400KB min+gz) — acceptable for internal teacher tool |
| All tests preserved | Limited 3D/polar chart support — not needed |
| Print mode pattern established | — |

### Consequences

- Do NOT import from any charting library other than `recharts`
- All future chart components must follow the `isPrintMode()` / `ResponsiveContainer` pattern
- Import directly from `recharts`: `BarChart`, `LineChart`, `ResponsiveContainer`, `XAxis`, `YAxis`, `Tooltip`, `ReferenceLine`, `CartesianGrid`, `Legend`

---

## TD-002 — URL-Parameter Print Mode (Match `isPrintMode()`)

**Status:** Accepted  
**Date:** 2026-02-26

### Context

`chartUtils.ts` exports `isPrintMode()` which checks `window.location.search.includes('printmode=')`. This is the existing approach used in `LineChartWithBenchmarks`. An alternative would be `window.matchMedia('print')` or CSS `@media print`.

### Decision

All chart print mode detection must use `isPrintMode()` from `chartUtils.ts`, not CSS media queries or `window.matchMedia`.

### Rationale

- `isPrintMode()` is already implemented, tested, and used in production code
- URL-based print mode allows the `BatchPrint` worker service to navigate directly to `?printmode=1` and capture charts at a fixed pixel size (required for server-side PDF generation)
- CSS `@media print` cannot be queried programmatically at layout time — chart dimensions must be set before render, not reactively
- Parity with existing Angular print pages: Angular print pages also use a URL flag (`?printmode=1`)

### Trade-offs

| Pro | Con |
|-----|-----|
| BatchPrint worker compatibility | Pages must handle print=1 URL param in routing |
| Consistent with existing 5 chart components | Not "pure" CSS print solution |
| Enables fixed-size chart rendering (PDF safe) | Requires `?printmode=1` to be in URL — not browser Ctrl+P |

### Consequences

- When a teacher clicks "Print" in the UI: navigate to same route with `?printmode=1` appended
- `<ChartContainer onPrint>` handler: use the `URL` API to set or replace `printmode=1` in the query string (for example: `const url = new URL(window.location.href); url.searchParams.set('printmode', '1'); window.location.href = url.toString();`) and then call `window.print()`
- BatchPrint service: already navigates to print URL and calls `window.print()` — no change needed

---

## TD-003 — POST-for-Reads on DataEntryController

**Status:** Accepted  
**Date:** 2026-02-26

### Context

`DataEntryController` uses HTTP POST for read operations (e.g., `GetAssessmentResults`, `GetLineGraphData`). Standard REST practice would use GET for reads. Refactoring to GET would require changing route attributes, method bodies, and query parameter handling across multiple controllers.

### Decision

Do not refactor POST-for-reads endpoints. All `features/*/...Api.ts` files must use `axios.post()` for these endpoints matching the existing contract.

### Rationale

- Angular `$http.post()` calls are being replaced with identical axios post calls — zero server-side risk
- Any endpoint refactor introduces deployment risk (routing changes, IIS config, CORS)
- Request bodies provide richer filter/query capability than URL query params
- Refactoring is out of scope for migration phase: it is a post-migration optimization

### Trade-offs

| Pro | Con |
|-----|-----|
| Zero API regression risk | Non-standard HTTP semantics |
| Matches Angular→React transition 1:1 | Cannot be cached by HTTP intermediate proxies |
| No server deployment required | Developer surprise — document in API module headers |

### Consequences

- All `features/*/...Api.ts` files: use `axios.post` for DataEntryController reads
- Document with `// POST intentional — DataEntryController pattern` comment
- Post-migration: evaluate converting reads to GET in a dedicated refactor initiative

---

## TD-004 — `batchSave` for Autosave (No New PATCH Endpoint)

**Status:** Accepted  
**Date:** 2026-02-26

### Context

The PRD requires autosave for `SectionDataEntry`. Options:
1. Add a new PATCH endpoint to DataEntryController for partial save
2. Add a new POST endpoint for single-student save
3. Use existing `BatchSave` endpoint with a batch of one (or the changed students)
4. Use existing `SaveAssessmentResult` per student with a sequence of calls

### Decision

Use existing `POST /api/dataentry/BatchSave` (`BatchSaveRequest` / `BatchSaveResponse` already typed in `sectionDataEntryApi.ts` types).

### Rationale

- `BatchSaveRequest` and `BatchSaveResponse` are already present in the TypeScript types but were not fully wired to the Angular layer — verifying and wiring this is low risk
- Batching dirty students reduces API call frequency vs. per-student approach (typical autosave batch: 1–5 students per 1.5s debounce window)
- No server-side changes required — `BatchSave` already handles multi-student save
- Adding a PATCH endpoint requires server changes, redeployment, and controller tests — out of scope

### Trade-offs

| Pro | Con |
|-----|-----|
| No server changes | Can't retry individual student — must retry full batch |
| Existing typed interface | BatchSave may have different validation behavior than single Save |
| Fewer API calls (batching) | Response must be inspected per-student for partial failures |

### Consequences

- `useAutosave` hook signature: `useMutation({ mutationFn: ({sectionId, benchmarkDateId, changes}) => sectionDataEntryApi.batchSave({...}) })`
- On partial failure (some students fail): surface per-student error in grid cell UI
- Debounce: 1.5 seconds from last keystroke
- Fallback interval: 30 seconds if isDirty (safety net for rapid typers)
- Max batch size: 50 students (matches BatchSave server limit — verify with API)

---

## TD-005 — API camelCase via `JsonNamingPolicy.CamelCase`

**Status:** Accepted  
**Date:** 2026-02-26

### Context

The API currently returns PascalCase property names (e.g., `StudentId`, `BenchmarkDate`). The React codebase should use camelCase TypeScript interfaces. Options:
1. Transform on the client (axios interceptor using `camelcase-keys`)
2. Configure the server once with `JsonNamingPolicy.CamelCase`
3. Keep PascalCase everywhere

### Decision

Implement `JsonNamingPolicy.CamelCase` on the server in Phase 1. This is a single config change to `Program.cs`.

### Rationale

- One line of server config eliminates a client-side transform running on every response
- React code is more idiomatic with camelCase TypeScript interfaces
- `camelcase-keys` library can be removed from the axios interceptor after types are updated
- Angular frontend (still running during parallel migration) must be tested against camelCase output — however, Angular `$http` with `$resource` already normalizes case in the controller binding layer for most cases. Parity tests will catch regressions.

### Trade-offs

| Pro | Con |
|-----|-----|
| Idiomatic TypeScript | Breaks Angular frontend during transition (mitigated by parity tests) |
| Remove client transform code | Requires updating all TS type interfaces |
| Smaller axios interceptor | Must be done as a full Phase 1 migration block |

### Implementation Plan

1. Create branch `chore/api-camelcase`
2. Add `JsonNamingPolicy.CamelCase` to `NS4.WebAPI/Program.cs`
3. Run `NS4.Parity.Tests` — capture list of failing tests
4. For each failing test: update TypeScript types in `features/*/types.ts`
5. Update any `features/*/...Api.ts` request body types (request bodies must also be camelCase)
6. Remove `camelcase-keys` transform from axios interceptor
7. Merge when parity tests 100% green

### Consequences

- **Phase 1 gate:** This PR must merge before Phase 2 work begins
- TypeScript types will use camelCase going forward: `studentId` not `StudentId`
- API documentation (OpenAPI/Swagger) will show camelCase

---

## TD-006 — No Redux / No Additional State Libraries

**Status:** Accepted  
**Date:** 2026-02-26

### Context

The React codebase has no Redux, Zustand, Jotai, or Recoil. State is managed via TanStack Query (server state) and React state/context (UI state). The question: should any complex page introduce a client-side state store?

### Decision

No new state management libraries. TanStack Query + React `useState`/`useReducer` + React Context is the complete state management approach.

### Rationale

- 171+ existing React components operate under this model — introducing Redux mid-migration creates inconsistency
- TanStack Query covers the majority of "state" needs (loading, caching, invalidation, optimistic updates)
- URL state handles the remainder of view-configuration state (`useSearchParams`)
- `SectionDataEntry` autosave is the most complex state requirement — it is covered by `useAutosave` (local state + mutation pattern, §4.3)

### Trade-offs

| Pro | Con |
|-----|-----|
| Consistent with existing 171+ components | Complex inter-component communication requires lifting state up |
| Less bundle size | No time-travel debugging |
| Simpler dev onboarding | — |

### Consequences

- Reject PRs that import `redux`, `@reduxjs/toolkit`, `zustand`, `jotai`, or `recoil`
- For shared read-only data (e.g., current section/benchmark context): use React Context provider at page level, not global store

---

## TD-007 — Hash Routing (No Migration to BrowserRouter)

**Status:** Accepted  
**Date:** 2026-02-26

### Context

NorthStarET uses `createHashRouter` (hash-based routing: `/#/path`). The alternative `createBrowserRouter` (HTML5 history API) requires server-side handling of 404s (`catch-all → index.html`).

### Decision

Keep `createHashRouter`. Do not migrate to `createBrowserRouter`.

### Rationale

- The `.NET` backend serves the SPA from `/`. Hash routing means deep links are never sent to the server — no Razor `catch-all` route needed
- 171+ existing React pages already use hash links and hash-based navigation
- Angular.js also uses hash routing (`$routeProvider`) — hash routing is required for the coexistence period (Phase 1–4) to avoid confusion between Angular and React routes
- BrowserRouter migration can only happen after 100% Angular removal — it is a post-migration task

### Trade-offs

| Pro | Con |
|-----|-----|
| Works with existing .NET routing config | URL aesthetics (hash in URL) |
| Required for Angular coexistence | Cannot use `<Link to="..">` from `@reach/router` |
| Zero server changes | — |

### Consequences

- `routes.tsx` uses `createHashRouter` — do not change
- All `<Link>` and `navigate()` calls use relative paths (React Router DOM resolves within the hash)
- Server-side auth redirects should redirect to `/#/login`, not `/login`

---

## TD-008 — Strangler Fig: Angular Removal is Last Step

**Status:** Accepted  
**Date:** 2026-02-26

### Context

Two approaches to migration:
1. **Feature toggle**: disable Angular routes, enable React routes via config
2. **Strangler Fig with route removal**: Angular routes removed route-by-route as React parity confirmed

### Decision

Strangler Fig — Angular routes removed route-by-route from `wwwroot/app/app.js` as each React page achieves parity. Angular bundle (angular.js, angular-route.js, d3.min.js) is removed only in Phase 5 after **all** routes confirmed migrated.

### Rationale

- Route-by-route removal creates clean, reviewable PRs
- No feature flag infrastructure needed — simplifies the codebase
- Each removed Angular route is immediately tested (parity test gate, TD-009)
- Angular bundle removal in one final PR allows exhaustive verification before and after

### Trade-offs

| Pro | Con |
|-----|-----|
| Clean rollback per route (git revert) | Angular bundle in production until Phase 5 (~3+ months) |
| No feature flag tech debt | Larger bundle during transition (~400KB Angular overhead) |
| Each PR is self-contained | — |

### Consequences

- Angular bundle ships until Phase 5 — document this as known/accepted
- Each "route removal" PR must include: parity test, Angular route removal, route verification ticket closed
- The Angular bundle adds ~400KB to the initial load — acceptable given this is an internal tool on school-network bandwidth

---

## TD-009 — Parity Test Gate

**Status:** Accepted  
**Date:** 2026-02-26

### Context

How do we ensure React pages visually and functionally match OldNorthStar before decommissioning Angular routes?

### Decision

A Playwright visual regression parity test must be created and passing in CI before any Angular route is removed. Gate is enforced via PR review checklist.

### Rationale

- Visual regressions are the most common failure mode in brownfield React migrations
- OldNorthStar is running in parallel — it is the baseline
- Playwright screenshots with `maxDiffPixelRatio: 0.01` catch layout regressions, missing data, and styling issues
- Teachers' workflow correctness is the primary product risk

### Consequences

- `src/parity/` directory contains one test file per major feature area
- `npm run test:parity` runs parity suite against `http://localhost:5x0x` (NorthStarET) vs. `http://localhost:5x0y` (OldNorthStar)
- PRs removing Angular routes must add parity test in the same commit (not a follow-up)
- CI has a dedicated `parity` job that runs on every PR targeting `main`

---

## TD-010 — Scoring Logic Unit-Tested Against Production Fixtures

**Status:** Accepted  
**Date:** 2026-02-26

### Context

SectionReports contains complex scoring logic (WV band scoring, BAS level determination, AVMR structured scoring). This logic must produce the same output as OldNorthStar. Options:
1. Trust the visual parity tests alone
2. Unit-test scoring logic against known-good fixture data
3. Re-read Angular controllers and re-implement

### Decision

Before Phase 4 (SectionReports migration), scoring logic unit tests against 20+ known-good input/output pairs must be green. Fixture data is derived from OldNorthStar production output on a test dataset seed.

### Rationale

- Parity tests are screenshot-based — they cannot verify that the *correct* student gets the *correct* score (only that the page looks correct for the seeded demo dataset)
- Scoring bugs are the highest teacher-trust risk — a score that changes from WV to IG changes interventions
- Unit tests are fast and give precise failure messages
- 20 fixtures per scoring algorithm is a practical minimum for confidence

### Consequences

- Sprint task: `feat/section-reports-scoring-tests` (Phase 4 gate)
- Fixture JSON files stored at `src/features/sectionReports/scoring/__fixtures__/`
- Each fixture: `{ input: { score, grade, benchmarkId, ... }, expected: { band, level, label } }`
- All scoring logic in pure functions: `getWvBand(input)`, `getBasLevel(input)`, `getAvmrScore(input)`

---

## TD-011 — URL State as Ground Truth for Complex Pages

**Status:** Accepted  
**Date:** 2026-02-26

### Context

SectionDataEntry, SectionReports, and Chart pages have complex filter state (which section, which benchmark date, which report type). State could be managed in React state (lost on navigation) or in URL params (bookmarkable, shareable).

### Decision

URL `searchParams` are the ground truth for filter/view state on all complex pages. `useSearchParams` is used for mutable filter state; `useParams` for route segment identifiers.

### Rationale

- Teachers navigate away and return (parent/deep-link from email notifications) — state must survive navigation
- The BatchPrint worker navigates to specific URLs to capture pages — URL state is required for correct print output
- OldNorthStar Angular controllers use `$routeParams` extensively — React URL state maps naturally to this pattern
- Debugging: URL state is inspectable in browser address bar without dev tools

### Consequences

- Filter dropdowns: `setSearchParams(prev => ...)` not `setState()`
- On filter change: URL updates, `useQuery` reruns with new params (from URL), page renders new data
- Do NOT store section/benchmark selection in React state — it will be lost on back/forward navigation

---

## TD-012 — Feature Slice Directory Pattern (Mandatory)

**Status:** Accepted  
**Date:** 2026-02-26

### Context

The existing codebase has an established `features/{domain}/` pattern with `...Api.ts`, `...Types.ts`, and `use{Domain}.ts` files. Some migrated pages have inconsistent patterns.

### Decision

All new feature work must follow the feature slice pattern: `features/{domain}/{domain}Api.ts`, `features/{domain}/{domain}Types.ts`, `features/{domain}/use{Domain}.ts`.

### Consequences

- Code review gate: PRs that put API calls in page components are rejected
- Shared types across features go in `types/` not co-located in a feature
- The `lib/api.ts` (axios singleton) is the only shared API primitive

---

## TD-013 — react-hook-form for All Forms

**Status:** Accepted  
**Date:** 2026-02-26

### Context

The existing codebase uses `react-hook-form` for forms. The `SectionDataEntry` autosave page is a grid of inputs — not a traditional form submit. Some implementations have used `useState` arrays per-student.

### Decision

`react-hook-form` for form-submit-based pages. For data-entry grids (SectionDataEntry), use `useState` with a `useAutosave` hook — this is not a form in the submit-button sense.

### Rationale

- Data-entry grids have per-cell save semantics, not form-level submit
- `react-hook-form` field arrays with 30–40 students would require complex unregister/register on scroll
- `useAutosave` + `useState` is simpler, more debuggable, and directly parallels the Angular controller pattern (`$scope.results[i].score = value`)

### Consequences

- Traditional forms (create benchmark, configure settings, etc.): `react-hook-form` + `Controller` pattern
- Data-entry grids: `useState<StudentResult[]>` + `useAutosave` hook

---

## TD-014 — No New API Endpoints During Phase 2–5

**Status:** Accepted  
**Date:** 2026-02-26

### Context

Could new API endpoints improve the React UX (e.g., aggregated endpoints, PATCH, REST-ful reads)?

### Decision

No new API endpoints during Phase 2–5. The .NET API stays unchanged (except the Phase 1 camelCase config change). All React pages use existing controller actions.

### Rationale

- API changes require server deployment, regression testing of Android/iOS clients (if any), and controller unit tests
- All Angular pages bind to existing endpoints — parity requires identical data shapes
- New endpoint ergonomics (aggregation, filtering) can be introduced post-migration as a separate initiative
- Risk reduction: the migration is already complex — adding server changes compounds risk

### Consequences

- No new `[Route("...")]` added to `.NET` controllers
- If an existing endpoint returns insufficient data: fix it in the existing action, not a new endpoint
- Document endpoint shape mismatches in `docs/northstar/migration/api-gaps.md` for post-migration cleanup

---

## TD-015 — TanStack Query Cache TTL Conventions

**Status:** Accepted  
**Date:** 2026-02-26

### Context

TanStack Query defaults `staleTime: 0` (refetch on every component mount). For a teacher SPA, this causes unnecessary API calls.

### Decision

Standardize cache TTLs by data volatility (see architecture.md §4.1):

| Data Type | staleTime | gcTime |
|-----------|-----------|--------|
| Reference data (benchmarks) | 3600000 (1h) | 7200000 (2h) |
| Section/student lists | 300000 (5min) | 600000 (10min) |
| DataEntry results | 30000 (30s) | 60000 (1min) |
| Section reports | 120000 (2min) | 300000 (5min) |
| Auth/session check | 0 (always fresh) | 60000 |

### Implementation

Define in `lib/queryClient.ts`:

```ts
export const queryClientConfig = {
  defaultOptions: {
    queries: {
      staleTime: 30_000, // default conservative 30s
      retry: 1,
    }
  }
};
// Feature hooks override per data type:
// useQuery({ ..., staleTime: 3_600_000 })  // for reference data
```

### Consequences

- All `use{Domain}.ts` hooks must explicitly set `staleTime` matching the table above
- Do not rely on global default for reference data — always explicit

---

## Appendix: Decisions Considered and Rejected

| Decision | Rejected Approach | Reason Rejected |
|----------|------------------|----------------|
| TD-001 | Migrate to Victory Charts | Existing 5 tested recharts components — full rewrite cost not justified |
| TD-005 | Client-side `camelcase-keys` transform | Already in codebase but should not be permanent — server is cleaner |
| TD-007 | Migrate to BrowserRouter | Requires server routing config change and is blocked until Angular fully removed |
| TD-006 | Add Zustand for SectionDataEntry autosave state | `useAutosave` custom hook is sufficient — no store needed |
| TD-014 | Add `/api/dataentry/AutoSave` PATCH endpoint | batchSave covers the need; no server changes |
