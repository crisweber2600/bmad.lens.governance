---
doc_type: architecture_brief
phase: businessplan
initiative: finish-northstaret-migration-3a10d3
status: draft
author: "Winston (Architect) — BusinessPlan Phase"
timestamp: "2026-02-26T13:00:00Z"
related_prd: docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/prd.md
related_ux: docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/ux-brief.md
---

# Architecture Brief
## Finish NorthStarET Migration — Angular.js → React 19

**Author:** Cris Weber / Winston (Architect)
**Date:** 2026-02-26
**Phase:** BusinessPlan
**Initiative:** finish-northstaret-migration-3a10d3

---

## 1. Technology Stack (Current State)

### 1.1 Frontend

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| UI Framework | React | ^19.2.3 | Concurrent mode enabled |
| Language | TypeScript | ~5.9.3 | Strict mode |
| Build | Vite | ^7.2.4 | `type: module`, ESM output |
| Routing | React Router DOM | ^7.10.1 | Hash router (`createHashRouter`) |
| Server State | TanStack Query | ^5.90.12 | Primary data-fetching layer |
| Forms | React Hook Form | ^7.68.0 | All form pages |
| HTTP | Axios | ^1.13.2 | Singleton client in `lib/` |
| Charts | Recharts | ^3.5.1 | Replacing D3 v3 |
| Selects | React Select | ^5.10.2 | Async selects |
| Sanitization | DOMPurify | ^3.3.1 | HTML content sanitization |
| Case conversion | camelcase-keys, snakecase-keys | ^10, ^9 | API response normalization |
| Styling | CSS Modules / clsx | — | No CSS-in-JS |
| Test - Unit | Vitest + @testing-library/react | ^3.2.4 / ^16.3.0 | |
| Test - E2E / Parity | Playwright | ^1.49.0 | |
| Test - A11y | vitest-axe, @axe-core/playwright | — | |

### 1.2 Backend (Complete — No Migration Scope)

| Layer | Technology | Notes |
|-------|-----------|-------|
| API | .NET 10 Web API | `NS4.WebAPI` project |
| ORM | Entity Framework 6 | EF Core migration is separate initiative |
| Auth | Cookie + JWT hybrid | `RequireAuth` HOC on client |
| DB | SQL Server | No changes in scope |
| Host | Aspire AppHost | `NorthStarET.AppHost` |

### 1.3 Legacy Stack (To Be Removed)

| Technology | Used By | Removal Phase |
|-----------|---------|--------------|
| Angular.js 1.4.5 | 35 remaining controllers | Phase 5 |
| angular-route.js | Angular SPA routing | Phase 5 |
| angular-bootstrap.js | Angular UI components | Phase 5 |
| D3 v3 | `LineGraphs.js`, `StackedBarGraphs.js` | Phase 3 |
| jQuery 1.x | Angular controller helpers | Phase 5 |

---

## 2. Source Layout

```
NS4.React/
  src/
    auth/                       # Auth provider, RequireAuth HOC, authApi.ts
    components/
      charts/                   # Reusable Recharts primitives (see §4)
      layout/                   # MainLayout, navigation shell
      shared/                   # Buttons, dialogs, form controls
    config/                     # API base URL, feature flags
    features/                   # Domain feature slices (see §3)
      sectionDataEntry/
        sectionDataEntryApi.ts
        sectionDataEntryTypes.ts
        [hooks, utils]
      sectionReports/
      stackedBarGraphs/
      lineGraphs/
      observationSummary/
      interventionToolkit/
      teamMeetings/
      ...
    hooks/                      # Shared custom hooks
    lib/                        # axios instance, query client
    pages/                      # Page-level components (thin shells)
      dataEntry/
      exports/
      settings/
      rollover/
    parity/                     # Playwright parity tests
    test/                       # Vitest setup, shared mocks
    theme/                      # CSS variables, color palette
    utils/                      # Pure utilities
    App.tsx                     # Query / Auth providers
    routes.tsx                  # createHashRouter definition
```

### 2.1 Feature Slice Pattern (Established Pattern)

Each feature slice follows this structure and **must not be broken**:

```
features/{featureName}/
  {featureName}Api.ts       # axios calls → raw API responses
  {featureName}Types.ts     # TypeScript interfaces for domain + API shapes
  use{FeatureName}.ts       # TanStack Query hooks (useQuery / useMutation)
```

Pages consume feature hooks; pages do not call axios directly.

---

## 3. Complete Feature Inventory

### 3.1 Existing Feature Slices

| Feature Slice | API File | Status |
|--------------|---------|--------|
| `assessmentAvailability` | assessmentAvailabilityApi.ts | ✅ |
| `assessments` | assessmentApi.ts, assessmentSyndicateApi.ts | ✅ |
| `batchPrint` | batchPrintApi.ts | ✅ |
| `benchmarks` | benchmarkApi.ts | ✅ |
| `calendar` | calendarApi.ts | ✅ |
| `consolidation` | consolidationApi.ts | ✅ |
| `dataExport` | dataExportApi.ts | ✅ |
| `dataImport` | dataImportApi.ts | ✅ |
| `districtDashboard` | districtDashboardApi.ts | ✅ |
| `districtSettings` | districtSettingsApi.ts | ✅ |
| `filterOptions` | filterOptionsApi.ts | ✅ |
| `help` | helpApi.ts | ✅ |
| `hfwReports` | hfwReportsApi.ts | ✅ |
| `interventionGroupDataEntry` | interventionGroupDataEntryApi.ts | ✅ |
| `interventionGroupLineGraph` | interventionGroupLineGraphApi.ts | ✅ |
| `interventionGroups` | interventionGroupApi.ts, interventionGroupAttendanceApi.ts | ✅ |
| `interventionToolkit` | interventionToolkitApi.ts | ✅ |
| `lineGraphs` | lineGraphApi.ts | ✅ — connect to page stubs |
| `mcaDashboard` | mcaDashboardApi.ts | ✅ |
| `navigation` | navigationApi.ts | ✅ |
| `observationSummary` | observationSummaryApi.ts | ✅ — verify Angular route removal |
| `passwordReset` | passwordResetApi.ts | ✅ |
| `personalSettings` | personalSettingsApi.ts | ✅ |
| `plcInterventionPlanning` | plcInterventionPlanningApi.ts | ✅ |
| `rollover` | rolloverApi.ts | ✅ |
| `schoolDashboard` | schoolDashboardApi.ts | ✅ |
| `sectionDataEntry` | sectionDataEntryApi.ts | ✅ — add autosave mutation |
| `sectionReports` | (inferred) | ✅ — verify all sub-types |
| `sections` | (inferred) | ✅ |
| `stackedBarGraphs` | (inferred) | ✅ — connect to chart primitives |
| `staff` | (inferred) | ✅ |
| `studentDashboard` | (inferred) | ✅ |
| `students` | (inferred) | ✅ |
| `teamMeetings` | (inferred) | ✅ — verify Angular route removal |
| `videos` | (inferred) | ✅ |

**No new feature slices should be needed** — all domain areas have existing slices. Work is in pages and chart primitive wiring.

---

## 4. Chart Primitive Architecture

### 4.1 Existing Primitives (`components/charts/`)

| File | Purpose | Tests |
|------|---------|-------|
| `ChartContainer.tsx` | Wrapper: padding, title, ARIA `role="img"` | — |
| `BarChart.tsx` | Single-series bar chart | — |
| `StackedBarChart.tsx` | Multi-series stacked bar | ✅ StackedBarChart.test.tsx |
| `LineChart.tsx` | Single-series line | ✅ LineChart.test.tsx |
| `LineChartWithBenchmarks.tsx` | Line + dashed benchmark overlays | — |
| `MultiLineChart.tsx` | Multi-series line (per student / per group) | ✅ MultiLineChart.test.tsx |
| `ChartExamples.tsx` | Developer reference (not production) | — |

### 4.2 Required Additions for Phase 3

| New Component | Wraps | Used By |
|--------------|-------|---------|
| `<ResponsiveLineChart>` | `LineChart` + `LineChartWithBenchmarks` | `StudentLineGraphPage`, `TrendLineGraphPage` |
| `<ResponsiveStackedBarChart>` | `StackedBarChart` | `StackedBarGraphGroupsPage`, `StackedBarGraphComparisonPage` |
| Chart color palette (reuse existing palette in `components/charts/chartUtils.ts`; no new `chartColors.ts` file) | — | All chart primitives |

**Critical print rule:** Recharts `ResponsiveContainer` collapses to zero height under `@media print`. All printable chart pages must conditionally render with fixed `width`/`height` props when `isPrintMode()` returns `true` (URL parameter `?printmode=1`, detected via `chartUtils.ts`). **Do not use `window.matchMedia('print')` — this approach is incompatible with the BatchPrint worker service.** See TD-002 for the accepted implementation.

```tsx
// Pattern for print-safe charts — use isPrintMode() from chartUtils.ts
import { isPrintMode } from '../charts/chartUtils';
const isPrint = isPrintMode();
<LineChart
  width={isPrint ? 600 : undefined}
  height={isPrint ? 280 : undefined}
/>
// OR wrap with ResponsiveContainer only when !isPrint
```

### 4.3 D3 → Recharts Mapping

| D3 Pattern | Recharts Equivalent |
|-----------|-------------------|
| `d3.scale.linear()` | recharts scales (internal) |
| `d3.svg.line()` | `<Line>` in `<LineChart>` |
| `d3.layout.stack()` | `<Bar>` with `stackId` in `<BarChart>` |
| `d3.svg.axis()` | `<XAxis>` / `<YAxis>` |
| `d3.scale.ordinal()` colors | Existing palette in `components/charts/chartUtils.ts` |
| Benchmark overlay lines | `<ReferenceLine y={value} stroke="dashed">` |
| Benchmark band fills | `<ReferenceArea y1={} y2={}>` with low opacity |
| Custom tooltips | `<Tooltip content={<CustomTooltip />}>` |
| High-DPI handling | Automatic via recharts SVG |

---

## 5. API Response Casing Strategy

### 5.1 Current State

The .NET API returns **PascalCase** (`StudentName`, `BenchmarkDateId`). React code currently uses `camelcase-keys` for conversion at the axios layer, but this is inconsistent — some pages use raw PascalCase directly.

### 5.2 Target State

**Option A (Recommended):** Fix at API layer — set `JsonNamingPolicy.CamelCase` in `Program.cs`. Update all React components from PascalCase to camelCase. Run full parity test suite.

**Option B:** Centralize conversion at axios interceptor — all responses auto-converted via `camelcase-keys` in axios response interceptor, all request bodies auto-converted via `snakecase-keys`. Zero API change.

**Recommendation:** Option A. It eliminates a runtime transformation on every response, simplifies TypeScript types, and aligns with REST convention. Cost: one PR that touches types + components. Risk: mitigated by parity tests.

### 5.3 Autosave API Contract

SectionDataEntry autosave uses the existing `BatchSave` POST endpoint (see TD-004 and TD-014 in `tech-decisions.md`):

```
POST /api/section-data-entries/batch-save
Content-Type: application/json

{
  "sectionId": 42,
  "benchmarkDateId": 7,
  "assessmentType": "reading",
  "entries": [
    { "studentId": 123, "score": 14, "notes": "..." },
    ...
  ]
}

Response: 200 OK (with saved timestamp)
```

The autosave hook must be idempotent (duplicate saves are safe).

> **Historical context:** An earlier draft of this brief proposed a new PATCH endpoint. That approach was superseded by TD-004 / TD-014, which require reuse of the existing `BatchSave` POST endpoint and prohibit new endpoints.

---

## 6. Routing Architecture

### 6.1 Router Type

`createHashRouter` (hash-based SPA). This is intentional — the .NET backend serves the SPA on `/`, and hash routing eliminates server-side route handling for React routes.

**Do not change the router type.** Angular routes are also hash-based in OldNorthStar; the transition is transparent to users.

### 6.2 Angular → React Route Mapping

Confirmed active Angular routes and their React equivalents:

| Angular Route | Angular Controller | React Page | In routes.tsx? |
|--------------|-------------------|-----------|---------------|
| `#/observation-summary-class/:benchmarkDateId` | `ObservationSummary.js` | `ObservationSummaryPage` | ✅ Verify |
| `#/observation-summary-filtered` | `ObservationSummary.js` | `ObservationSummaryPage` | ✅ Verify |
| `#/stackedbargraph-groups/:autoload` | `StackedBarGraphs.js` | `StackedBarGraphGroupsPage` | ✅ Verify |
| `#/stackedbargraph-comparison/:autoload` | `StackedBarGraphs.js` | `StackedBarGraphComparisonPage` | ✅ Verify |
| `#/stacked-bar-graph-summary/:scoreGrouping/:testDueDate` | `StackedBarGraphs.js` | `StackedBarGraphGroupsPage` (with params) | ⚠️ Add route |
| `#/ig-dashboard` | `InterventionDashboard.js` | `InterventionDashboardPage` | ⚠️ Verify |
| `#/ig-dashboard/:schoolyear/*` | `InterventionDashboard.js` | `InterventionDashboardPage` | ⚠️ Verify all variants |
| `#/ig-dashboard-printall/*` | `InterventionDashboard.js` | `InterventionDashboardPage` | ⚠️ Add print-all branch |
| `#/student-dashboard-printall/:id/:tab` | `StudentDashboard.js` | `StudentDashboardPage` | ⚠️ Add route variant |
| `#/district-calendar` | `DistrictSettings.js` | `settings/DistrictCalendarPage` | ✅ Verify |
| `#/school-calendar` | `DistrictSettings.js` | `CalendarManagementPage` | ✅ Verify |
| `#/tm-attend/:teamMeeting` | `TeamMeetings.js` | `TeamMeetingAttendPage` | ✅ Verify |
| `#/tm-attend-notes/:teamMeetingId/:studentId` | `TeamMeetings.js` | `TeamMeetingNotesPage` | ✅ Verify |
| `#/tm-email-invite/:teamMeetingId/:tddid/:staffId` | `TeamMeetings.js` | `TeamMeetingInvitationsPage` | ✅ Verify |
| `#/ig-assessment-resultlist/:assessmentId` | `IgAssessmentResultList` | `IgAssessmentResultListPage` | ✅ Verify |
| `#/section-assessment-resultlist/:assessmentId` | — | `SectionAssessmentResultListPage` | ✅ Verify |
| `#/section-assessment-resultlist-kntc/:assessmentId` | — | `SectionAssessmentResultListKntcPage` | ✅ Verify |
| `#/utility-batch-print` | — | `BatchPrintPage` | ✅ Verify |
| `#/system-benchmarks` | `DistrictSettings.js` | `SystemBenchmarksPage` | ✅ Verify |
| `#/district-benchmarks` | `DistrictSettings.js` | `settings/DistrictBenchmarksPage` | ✅ Verify |
| `#/district-yearlyassessment-benchmarks` | `DistrictSettings.js` | `settings/DistrictYearlyBenchmarksPage` | ✅ Verify |
| `#/section-report-avmr-structured-detail/*` | `SectionReports.js` | `SectionReportAvmrStructuredDetailPage` | ✅ Verify |

**Route verification process** (per route):
1. Check `routes.tsx` — route definition present?
2. Load page in browser — no Angular HTML in response?
3. Angular `app.js` — route removed?

### 6.3 `RequireAuth` Usage

All migrated pages must be wrapped in `<RequireAuth>`. The pattern is already established in `routes.tsx`:

```tsx
{
  element: <RequireAuth><MainLayout /></RequireAuth>,
  children: [
    { path: '/section-data-entry/:sectionId/:benchmarkDateId/:assessmentType',
      element: <SectionDataEntryGenericPage /> },
    // ...
  ]
}
```

---

## 7. Autosave Architecture (SectionDataEntry)

### 7.1 Hook Design

```ts
// features/sectionDataEntry/useAutosave.ts

export function useAutosave(
  sectionId: number,
  benchmarkDateId: number,
  assessmentType: string,
  entries: SectionDataEntry[]
) {
  const mutation = useMutation({ mutationFn: patchSectionDataEntries });
  
  // Debounced save: fires 1.5s after last change
  const debouncedSave = useDebouncedCallback(
    () => mutation.mutate({ sectionId, benchmarkDateId, assessmentType, entries }),
    1500
  );
  
  // Interval save: fires every 30s regardless of debounce
  useInterval(() => {
    if (isDirty) mutation.mutate(...)
  }, 30_000);
  
  // Return: saveStatus ('idle' | 'saving' | 'saved' | 'error'), lastSavedAt
}
```

### 7.2 Unsaved Changes Guard

```ts
// hooks/useUnsavedChangesGuard.ts
export function useUnsavedChangesGuard(isDirty: boolean) {
  const blocker = useBlocker(isDirty);
  
  useEffect(() => {
    const handler = (e: BeforeUnloadEvent) => {
      if (isDirty) e.preventDefault();
    };
    window.addEventListener('beforeunload', handler);
    return () => window.removeEventListener('beforeunload', handler);
  }, [isDirty]);
  
  return { blocker }; // consumed by <UnsavedChangesDialog>
}
```

---

## 8. Testing Architecture

### 8.1 Test Layers

| Layer | Tool | Location | Coverage Target |
|-------|------|---------|----------------|
| Unit (components) | Vitest + @testing-library/react | `src/**/__tests__/` or `*.test.tsx` inline | All new components |
| Unit (logic / calculations) | Vitest | `src/features/**/*.test.ts` | Section report scoring, autosave logic |
| A11y | vitest-axe | Colocated with component tests | All new page shells |
| Parity (visual regression) | Playwright | `src/parity/` | Mandatory for all migrated pages |
| E2E | Playwright | `e2e/` | Critical journeys (SectionDataEntry save, SectionReports print) |

### 8.2 Parity Test Pattern

```ts
// src/parity/SectionReport.parity.ts
test('WvSectionReportPage matches OldNorthStar screenshot', async ({ page }) => {
  await page.goto('#/section-report-wv/123/456');
  await expect(page).toHaveScreenshot('wv-section-report.png', {
    maxDiffPixelRatio: 0.01
  });
});
```

Every migrated page that replaces an Angular view **must** have a parity test before its Angular route is removed from `app.js`.

### 8.3 Section Report Calculation Tests

```ts
// features/sectionReports/wvScoring.test.ts
describe('WV Band Scoring', () => {
  it('returns band C for score 14-17 in grade 2 fall', () => {
    expect(getWvBand({ score: 15, grade: 2, benchmark: 'Fall' })).toBe('C');
  });
  // 20 known-good inputs from production data
});
```

These tests **must pass before** the Angular SectionReports.js routes are removed. Source of known-good inputs: run old Angular app against test data, capture output, encode as fixtures.

---

## 9. Migration Sequencing (Phase-by-Phase)

### Phase 1 — Foundation (Blocks All)

```
1. API camelCase fix
   └─ Update JsonNamingPolicy in NS4.WebAPI/Program.cs
   └─ Update all TS types to camelCase
   └─ Run full parity suite → all green

2. Chart primitive audit
   └─ Evaluate existing components/charts/ files
   └─ Create chartColors.ts (OldNorthStar color palette)
   └─ Add ResponsiveLineChart, ResponsiveStackedBarChart wrappers
   └─ Add print-safe rendering (fixed dimensions under @media print)
   └─ Add ARIA labels to all primitives
   └─ Write tests for new primitives
```

**Gate:** All `NS4.Parity.Tests` green after camelCase fix.

### Phase 2 — Data Entry (3–4 weeks)

```
1. SectionDataEntry autosave
   └─ Add useAutosave hook to features/sectionDataEntry/
   └─ Add useUnsavedChangesGuard to hooks/
   └─ Add <AutosaveIndicator>, <UnsavedChangesDialog> to components/shared/
   └─ Add <SectionBenchmarkSelector> to components/shared/
   └─ Update all SectionDataEntry*Page variants
   └─ Add/update Playwright E2E test for save behavior
   └─ Remove Angular SectionDataEntry routes from app.js

2. Student / Staff / Assessment close-out
   └─ Identify remaining flows in each Angular controller
   └─ Port outstanding flows to existing Page files
   └─ Add parity tests
   └─ Remove Angular routes
```

**Gate:** Zero Angular routes for data entry in app.js.

### Phase 3 — Charts (2–3 weeks)

```
1. Wire LineGraph page stubs to shared primitives
   └─ StudentLineGraphPage, TrendLineGraphPage, IGLineGraphPage, InterventionGroupLineGraphPage
   └─ Verify benchmark overlays render correctly
   └─ Parity tests
   └─ Remove Angular LineGraph routes

2. StackedBarGraph recharts rewrite
   └─ StackedBarGraphGroupsPage full recharts implementation
   └─ StackedBarGraphComparisonPage full recharts implementation
   └─ Add stacked-bar-graph-summary route handler
   └─ Parity tests (color band match critical)
   └─ Remove Angular StackedBarGraph routes

3. InterventionDashboard + StudentDashboard
   └─ ig-dashboard/* routes → InterventionDashboardPage
   └─ student-dashboard-printall → StudentDashboardPage (print branch)
   └─ Parity tests
   └─ Remove Angular routes
```

**Gate:** Zero Angular routes for charts in app.js.

### Phase 4 — Reports & Observations (3–5 weeks)

```
1. Read LEGACY_DOCUMENTATION.md (OldNorthStar repo)
   └─ Document WV band scoring rules
   └─ Document BAS level determination rules
   └─ Document AVMR scoring edge cases
   └─ Write unit tests with 20 known-good inputs per sub-type

2. SectionReports port / verification
   └─ For each sub-type: verify React page returns same HTML structure
   └─ Fix any discrepancies
   └─ Parity tests for all sub-types
   └─ section-report-avmr-structured-detail route → SectionReportAvmrStructuredDetailPage
   └─ Remove Angular SectionReports routes

3. ObservationSummary verification
   └─ observation-summary-class and observation-summary-filtered routes
   └─ Verify multi-step form state preserved across nav
   └─ Parity tests
   └─ Remove Angular routes

4. InterventionToolkit
   └─ Verify URL-state-aware step navigation
   └─ Parity tests
   └─ Remove Angular routes
```

**Gate:** Zero Angular routes for reports in app.js. All scoring unit tests pass.

### Phase 5 — Admin Cleanup & Decommission (1–2 weeks)

```
1. TeamMeetings route verification (attend, notes, invite)
2. Remaining admin routes (calendar, benchmarks, import, export, rollover)
3. Route audit: zero ng-app / ng-controller in any live HTML
4. Angular bundle removal:
   └─ Remove from Vite config / package.json
   └─ Remove D3 v3, jQuery, angular-* packages
   └─ npm run build → confirm no Angular chunk
   └─ Final npm ls angular → nothing
5. wwwroot cleanup: remove app/scripts/controllers/ from static path
```

**Gate:** `npm run build` produces no Angular chunk. Full route audit passes.

---

## 10. Risk Architecture

| Risk | Architectural Mitigation |
|------|--------------------------|
| camelCase fix breaks React pages | Wrap all API types in `camelcase-keys`-aware types first; migrate types file-by-file; parity tests as changeset gate |
| SectionReports scoring mismatch | Write scoring unit tests with known-good data BEFORE removing Angular routes; gate on 100% unit test pass |
| StackedBarGraph D3 visual parity | Capture baseline screenshots from Angular app first; lock them as parity test fixtures |
| Print layout regression | Add `@media print` screenshot tests to Playwright suite; run in CI |
| Angular cached routes after cutover | `Cache-Control: no-cache, no-store` on Angular `app.js` in .NET middleware; service worker invalidation on deploy |
| Scope creep in SectionDataEntry | autosave feature is the ONLY UX upgrade allowed; all other changes maintain parity |

---

## 11. Technical Decisions Log

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Router type | Hash router (keep) | Backend serves SPA at `/`; hash routing avoids server-side 404 for React routes |
| State mgmt | TanStack Query (already in use) | Server state only; no Redux needed |
| Form library | React Hook Form (already in use) | Performance, uncontrolled inputs |
| Chart library | Recharts (already in use) | D3 replacement; recharts already required |
| API casing | Fix at API layer (Option A) | Cleaner types; removes runtime camelcase-keys transform overhead |
| Angular removal strategy | Route-by-route per phase | Lower risk than big-bang; parity test suite gates each phase |
| Autosave implementation | Debounce + interval hybrid | PATCH on 1.5s debounce + 30s interval = covers both active typing and idle sessions |

---

## 12. Related Documents

- [PRD](./prd.md)
- [UX Brief](./ux-brief.md)
- [NorthStarET Architecture](../../../../lens-sync/NorthStarET/architecture.md)
- [NorthStarET Migration Map](../../../../lens-sync/NorthStarET/migration-map.md)
- [OldNorthStar Architecture](../../../../lens-sync/OldNorthStar/architecture.md)
- [OldNorthStar Migration Map](../../../../lens-sync/OldNorthStar/migration-map.md)
