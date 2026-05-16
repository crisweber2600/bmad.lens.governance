---
doc_type: architecture
phase: techplan
initiative: finish-northstaret-migration-3a10d3
status: draft
author: "Winston (Architect) — TechPlan Phase"
timestamp: "2026-02-26T14:00:00Z"
baseline: docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/architecture-brief.md
related_prd: docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/prd.md
---

# Architecture — Finish NorthStarET Migration

**Author:** Cris Weber / Winston (Architect)
**Date:** 2026-02-26
**Phase:** TechPlan
**Initiative:** finish-northstaret-migration-3a10d3

---

## 1. Architectural Context

### 1.1 Migration Pattern

NorthStarET uses a **Strangler Fig** pattern. The Angular.js SPA and React SPA coexist behind the same Vite build and are both served from `wwwroot/`. Each Angular route is removed from `app.js` once its React equivalent is confirmed live and parity-tested.

```
┌──────────────────────────────────────────────────────────────┐
│  Browser: hash-based SPA routing                             │
│  createHashRouter (React Router DOM v7)                      │
└───────┬──────────────────────┬───────────────────────────────┘
        │ React routes         │ Angular routes (shrinking)
┌───────▼──────────┐  ┌───────▼──────────────────────────────┐
│  React 19 SPA    │  │  Angular.js 1.4.5 app                │
│  171+ TSX        │  │  35 controllers (26,092 lines)        │
│  components      │  │  wwwroot/app/scripts/controllers/     │
└───────┬──────────┘  └───────────────────────────────────────┘
        │ axios + TanStack Query
┌───────▼──────────────────────────────────────────────────────┐
│  ASP.NET Core 10 WebAPI (NS4.WebAPI)                        │
│  35 controllers | 60+ endpoints | JWT/Cookie auth           │
└───────┬──────────────────────────────────────────────────────┘
        │ EF 6.5.1
┌───────▼──────────────────────────────────────────────────────┐
│  SQL Server (Azure SQL)                                      │
│  45+ EF migrations                                          │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│  Worker Services (independent processes)                     │
│  BatchPrint | AutomatedRollover | BatchProcessor             │
└──────────────────────────────────────────────────────────────┘
```

### 1.2 Phase-End State (Post-Migration)

```
┌──────────────────────────────────────────────────────────────┐
│  React 19 SPA only — zero Angular bundle                     │
│  ~200+ TSX components | Vite 7 | Hash router                │
└───────┬──────────────────────────────────────────────────────┘
        │ axios + TanStack Query v5 (camelCase API)
┌───────▼──────────────────────────────────────────────────────┐
│  ASP.NET Core 10 WebAPI                                      │
│  JsonNamingPolicy.CamelCase | Same endpoints                │
└───────┬──────────────────────────────────────────────────────┘
        │ EF 6.5.1 (unchanged)
┌───────▼──────────────────────────────────────────────────────┐
│  SQL Server — unchanged                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Frontend Architecture

### 2.1 Source Directory Layout

```
NS4.React/src/
├── auth/
│   ├── authApi.ts              # Login, logout, session check
│   ├── AuthContext.tsx         # React context for auth state
│   └── RequireAuth.tsx         # Route guard HOC
├── components/
│   ├── charts/
│   │   ├── chartUtils.ts       # CHART_COLORS, CHART_THEME, isPrintMode,
│   │   │                       #   isBatchPrintMode, getBenchmarkColor,
│   │   │                       #   formatScore, formatDate*, getSeriesColor
│   │   ├── ChartContainer.tsx  # Wrapper: title, loading, error, onPrint, onExport
│   │   ├── BarChart.tsx        # Single-series bar (recharts)
│   │   ├── StackedBarChart.tsx # Multi-series stacked bar (recharts)
│   │   ├── LineChart.tsx       # Single-series line (recharts)
│   │   ├── LineChartWithBenchmarks.tsx  # Multi-series line + ReferenceLine
│   │   │                                #   overlays (uses isPrintMode())
│   │   ├── MultiLineChart.tsx  # Multi-series line, per-student/per-group
│   │   └── ChartExamples.tsx   # DEV ONLY — not exported
│   ├── layout/
│   │   ├── MainLayout.tsx      # App shell: nav + <Outlet>
│   │   └── [nav components]
│   └── shared/
│       ├── ConfirmDialog.tsx   # Reusable confirmation modal
│       └── [buttons, inputs, badges]
├── config/
│   └── api.ts                  # API base URL (from env)
├── features/                   # Domain feature slices (§2.3)
├── hooks/                      # Shared custom hooks
├── lib/
│   └── api.ts                  # axios singleton instance
├── pages/                      # Page shells (§2.4)
├── parity/                     # Playwright visual regression tests
├── test/                       # Vitest setup, MSW handlers, shared fixtures
├── theme/                      # CSS variables, global styles
├── utils/                      # Pure utility functions
├── App.tsx                     # QueryClientProvider + AuthProvider + Router
└── routes.tsx                  # createHashRouter definition
```

### 2.2 React Application Bootstrap

```tsx
// App.tsx — provider stack (order matters)
<QueryClientProvider client={queryClient}>
  <AuthProvider>
    <RouterProvider router={router} />
  </AuthProvider>
</QueryClientProvider>

// routes.tsx — all authenticated pages nested under RequireAuth + MainLayout
createHashRouter([
  { path: '/login', element: <LoginPage /> },
  {
    element: <RequireAuth><MainLayout /></RequireAuth>,
    children: [
      // all authenticated page routes
    ]
  },
  { path: '*', element: <NotFoundPage /> }
])
```

### 2.3 Feature Slice Pattern (Mandatory)

Every domain area follows this structure. **Pages must not call axios directly.**

```
features/{domain}/
├── {domain}Api.ts        # axios calls, returns typed responses
│                         #   API methods match controller action names
│                         #   POST bodies for reads (DataEntryController pattern)
├── {domain}Types.ts      # TypeScript interfaces for domain objects + API shapes
└── use{Domain}.ts        # TanStack Query hooks
                          #   useQuery for reads, useMutation for writes
```

**Example — sectionDataEntry:**
```ts
// sectionDataEntryApi.ts — all API calls use POST (DataEntryController pattern)
sectionDataEntryApi.getAssessmentResults(req)  // POST /api/dataentry/GetAssessmentResults
sectionDataEntryApi.saveAssessmentResult(req)  // POST /api/dataentry/SaveAssessmentResult
sectionDataEntryApi.batchSave(req)             // POST /api/dataentry/BatchSave
```

> ⚠️ **Critical:** The API uses POST for both reads and writes on DataEntryController (matches OldNorthStar Angular pattern). There is no PATCH endpoint. Autosave must use `batchSave` (BatchSaveRequest / BatchSaveResponse already typed in `types.ts`).

### 2.4 Page Component Pattern

Pages are thin shells. They:
1. Own URL param extraction (`useParams`, `useSearchParams`)
2. Call feature hooks
3. Render feature components
4. Handle page-level loading / error states

```tsx
// Canonical page pattern
export function SectionDataEntryGenericPage() {
  const { sectionId, benchmarkDateId, assessmentType } = useParams();
  const { data, isLoading, error } = useSectionDataEntry(
    Number(sectionId), Number(benchmarkDateId), assessmentType
  );
  // render: loading skeleton | error | content
}
```

---

## 3. Chart Primitive Architecture (Refined)

### 3.1 What Exists (Code-Verified)

`components/charts/chartUtils.ts` already provides the complete shared foundation:

| Export | Purpose | Status |
|--------|---------|--------|
| `CHART_COLORS` | Full palette: primary, benchmark band colors, series colors, status colors | ✅ Complete |
| `CHART_THEME` | Recharts config: font, margins, line width, dot radius | ✅ Complete |
| `isPrintMode()` | Detects `?printmode=` in URL — used by `LineChartWithBenchmarks` | ✅ Complete |
| `isBatchPrintMode()` | Detects `?batchprint=` in URL | ✅ Complete |
| `getBenchmarkColor()` | Returns cell background color from score + benchmark thresholds | ✅ Complete |
| `getSeriesColor(index)` | Cycles 8-color series palette | ✅ Complete |
| `formatScore()` | Round to N decimals | ✅ Complete |
| `formatDate()`, `formatDateLong()`, `formatDate2()`, `formatDateForChart()` | Date display variants | ✅ Complete |
| `calculateYAxisDomain()` | Y-axis with padding | ✅ Complete |

> **Architecture correction from businessplan brief:** Do NOT create a separate `chartColors.ts`. It already exists as `chartUtils.ts`. All new chart code must import from `chartUtils.ts`.

### 3.2 Print Mode Pattern (Code-Verified)

`isPrintMode()` checks `window.location.search.includes('printmode=')`, not `window.matchMedia`. This means:

- Pages that need to print navigate to `{currentRoute}?printmode=1` (or the batch print service sets this parameter).
- Chart components read `isPrintMode()` to switch from `<ResponsiveContainer>` to fixed `width`/`height` props.
- The `LineChartWithBenchmarks` component already implements this pattern.

**All new printable chart pages must follow this same URL-parameter-based print detection**, not CSS media queries for chart dimensions.

```tsx
// Correct pattern (matching existing code)
const printMode = isPrintMode();
return printMode
  ? <RechartsLineChart width={600} height={280} data={data}>...</RechartsLineChart>
  : <ResponsiveContainer width="100%" height={300}>
      <RechartsLineChart data={data}>...</RechartsLineChart>
    </ResponsiveContainer>;
```

### 3.3 Chart Component Props Summary (Code-Verified)

| Component | Key Props | print-safe? | Tests |
|-----------|-----------|------------|-------|
| `ChartContainer` | `title`, `subtitle`, `onExport?`, `onPrint?`, `isLoading?`, `error?` | Wrapper only | — |
| `BarChart` | `BarChartProps` (data, title, etc.) | — | — |
| `StackedBarChart` | `StackedBarChartProps` | — | ✅ |
| `LineChart` | `LineChartProps` | — | ✅ |
| `LineChartWithBenchmarks` | `BenchmarkData[]`, `InterventionMarker[]` | ✅ (`isPrintMode()`) | — |
| `MultiLineChart` | `MultiLineChartProps` | — | ✅ |

### 3.4 Gap Analysis: What Phase 3 Must Add

| Item | Priority | Notes |
|------|---------|-------|
| Verify `StackedBarChart` meets StackedBarGraph page requirements (groups + comparison modes) | P1 | May need `mode` prop |
| Add `isAnimationActive={false}` prop guard for ≥30 students in `MultiLineChart` | P1 | Performance |
| Add ARIA `role="img"` + `aria-label` to `ChartContainer` | P1 | A11y — currently title in header only |
| Verify `isPrintMode()` is called in `StackedBarChart` (not just `LineChartWithBenchmarks`) | P1 | Print parity |
| Add `Hrsiw4SectionDataEntryPage` if needed (file exists but verify route) | P2 | DataEntry close-out |

---

## 4. Data Flow Architecture

### 4.1 Standard Read Flow (TanStack Query)

```
Page
  └─ calls useFeature(params)
       └─ useQuery({ queryKey: ['feature', ...params], queryFn: featureApi.get })
            └─ axios.post('/api/Controller/Action', body)
                 └─ Response: camelCase (after Phase 1 API fix)
                      └─ Cached in QueryClient (staleTime varies by data type)
```

**Cache TTL conventions (to be codified in `lib/queryClient.ts`):**

| Data Type | Recommended staleTime | Rationale |
|-----------|----------------------|-----------|
| Reference data (benchmarks, assessments) | `Infinity` (or 1h) | Rarely changes, school-year scoped |
| Section/Student lists | 5 minutes | Changes at enrollment time |
| DataEntry results | 30 seconds | Active during entry session |
| Section reports | 2 minutes | Computed, not user-editable |
| Navigation/menu | 10 minutes | Role-based, session-scoped |

### 4.2 Mutation / Write Flow

```
Form (react-hook-form)
  └─ onSubmit → useMutation({ mutationFn: featureApi.save })
       └─ axios.post('/api/Controller/Save', body)
            └─ onSuccess: queryClient.invalidateQueries(['feature', ...keys])
```

### 4.3 Autosave Flow (SectionDataEntry — New)

The autosave implementation uses the existing `batchSave` endpoint (already typed as `BatchSaveRequest`/`BatchSaveResponse`), not a new PATCH endpoint.

```
User types score
  └─ Local state updated (optimistic)
       └─ isDirty = true
            ├─ Debounce timer (1.5s): fires useAutosave mutation
            └─ Interval (30s): fires useAutosave mutation if isDirty
                 └─ POST /api/dataentry/BatchSave
                      ├─ onSuccess: lastSavedAt = now, isDirty = false, saveStatus = 'saved'
                      └─ onError: saveStatus = 'error', retry UI presented
```

**Sequence diagram:**

```
Teacher          SectionDataEntryPage     useAutosave hook      API
   │ types score        │                       │                 │
   │────────────────────▶                       │                 │
   │                    │ setState(newScore)     │                 │
   │                    │ setIsDirty(true)       │                 │
   │                    │───────────────────────▶                 │
   │                    │                 debounce 1.5s           │
   │                    │                       │────────────────▶│
   │                    │                       │ BatchSave POST  │
   │                    │                       │◀────────────────│
   │                    │                 saveStatus='saved'      │
   │                    │◀──────────────────────│                 │
   │                    │ <AutosaveIndicator     │                 │
   │                    │   status='saved'>      │                 │
```

---

## 5. API Contract Changes

### 5.1 Phase 1: CamelCase Normalization

**Change:** Add `JsonNamingPolicy.CamelCase` to `NS4.WebAPI/Program.cs`:

```csharp
builder.Services.AddControllers().AddJsonOptions(opts => {
    opts.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    opts.JsonSerializerOptions.DictionaryKeyPolicy = JsonNamingPolicy.CamelCase;
});
```

**Impact surface:**
- All API responses change from `PascalCase` to `camelCase`
- All TypeScript types in `features/*/types.ts` must be updated to camelCase property names
- Remove `camelcase-keys` calls from axios interceptor (or keep as belt-and-suspenders during transition, then remove)
- `snakecase-keys` request body conversion can be removed once types are updated

**Migration approach (zero-downtime):**
1. Branch: `chore/api-camelcase`
2. Add `JsonNamingPolicy.CamelCase` to API
3. Run parity suite — identify failing tests (these are the affected types)
4. Update TS types file-by-file, guided by failing tests
5. Merge when 100% parity green

**Verification query:** `grep -r "PascalCase\|toLowerCase\|toUpperCase" src/features/` — all hits must be resolved before Phase 2 begins.

### 5.2 Existing API Endpoints (No New Endpoints Required)

All migration work uses existing API endpoints. Verified endpoints used by migrated pages:

| Controller | Key Actions | Angular → React |
|-----------|-------------|----------------|
| `DataEntryController` | GetAssessmentResults, SaveAssessmentResult, BatchSave | SectionDataEntry pages |
| `SectionReportController` | GetSectionReport (by type), GetAvmrStructuredDetail | SectionReport pages |
| `LineGraphController` | GetLineGraphData, GetIGLineGraphData | LineGraph pages |
| `StackedBarController` | GetStackedBarData, GetComparisonData | StackedBar pages |
| `ObservationController` | GetObservationSummary (class + filtered) | ObservationSummary |
| `InterventionToolkitController` | GetToolkit, SaveStep | InterventionToolkit |
| `InterventionDashboardController` | GetDashboard | InterventionDashboard |
| `TeamMeetingController` | GetAttendance, SaveAttendance, GetNotes, SaveNotes, GetInvite | TeamMeeting pages |
| `DataExportController` | ExportData | DataExport pages |
| `BenchmarkController` | GetDistrictBenchmarks, GetSystemBenchmarks, etc. | Settings pages |
| `CalendarController` | GetCalendar, SaveCalendar | Calendar pages |

---

## 6. Routing Architecture

### 6.1 Router Type and Rationale

`createHashRouter` (hash-based). The `.NET` backend serves the SPA shell at `/`. Hash routing means `/#/section-data-entry/123` never reaches the server — only `#/...` is parsed by React Router. **Do not change this.**

### 6.2 Route Definition Pattern

```tsx
// routes.tsx — all authenticated routes use nested structure
{
  element: <RequireAuth><MainLayout /></RequireAuth>,
  children: [
    {
      path: '/section-data-entry/:sectionId/:benchmarkDateId/:assessmentType',
      element: <SectionDataEntryGenericPage />
    },
    // ... all migrated pages
  ]
}
```

### 6.3 Angular → React Route Migration Table

Status key: ✅ React route confirmed in routes.tsx | ⚠️ Needs action | 🔜 Phase 2+

| Angular Hash Route | React Route | React Page | Status | Phase |
|--------------------|------------|-----------|--------|-------|
| `/observation-summary-class/:benchmarkDateId` | `/observation-summary/:benchmarkDateId` | `ObservationSummaryPage` | ⚠️ Verify param match | 2 |
| `/observation-summary-filtered` | `/observation-summary-filtered` | `ObservationSummaryPage` | ⚠️ Verify | 2 |
| `/stackedbargraph-groups/:autoload` | `/stackedbargraph-groups/:autoload` | `StackedBarGraphGroupsPage` | ⚠️ Verify | 3 |
| `/stackedbargraph-comparison/:autoload` | `/stackedbargraph-comparison/:autoload` | `StackedBarGraphComparisonPage` | ⚠️ Verify | 3 |
| `/stacked-bar-graph-summary/:scoreGrouping/:testDueDate` | `/stacked-bar-graph-summary/:scoreGrouping/:testDueDate` | `StackedBarGraphGroupsPage` (query mode) | 🔜 Add | 3 |
| `/ig-dashboard` | `/ig-dashboard` | `InterventionDashboardPage` | ⚠️ Verify | 3 |
| `/ig-dashboard/:schoolyear` | `/ig-dashboard/:schoolYear` | `InterventionDashboardPage` | ⚠️ Verify | 3 |
| `/ig-dashboard/:schoolyear/:igId` | `/ig-dashboard/:schoolYear/:igId` | `InterventionDashboardPage` | 🔜 Add variant | 3 |
| `/ig-dashboard-printall/:schoolyear` | `/ig-dashboard-printall/:schoolYear` | `InterventionDashboardPage` | 🔜 Print branch | 3 |
| `/student-dashboard-printall/:id/:tab` | `/student-dashboard-printall/:id/:tab` | `StudentDashboardPage` | 🔜 Add route | 3 |
| `/district-calendar` | `/settings/district-calendar` | `settings/DistrictCalendarPage` | ⚠️ Verify | 5 |
| `/school-calendar` | `/settings/calendar` | `CalendarManagementPage` | ⚠️ Verify | 5 |
| `/tm-attend/:teamMeeting` | `/team-meetings/:id/attend` | `TeamMeetingAttendPage` | ⚠️ Verify | 5 |
| `/tm-attend-notes/:teamMeetingId/:studentId` | `/team-meetings/:id/notes/:studentId` | `TeamMeetingNotesPage` | ⚠️ Verify | 5 |
| `/tm-email-invite/:teamMeetingId/:tddid/:staffId` | `/team-meetings/:id/invite/:tddid/:staffId` | `TeamMeetingInvitationsPage` | ⚠️ Verify | 5 |
| `/ig-assessment-resultlist/:assessmentId` | `/ig-assessment-results/:assessmentId` | `dataEntry/IgAssessmentResultListPage` | ⚠️ Verify | 5 |
| `/section-assessment-resultlist/:assessmentId` | `/section-assessment-results/:assessmentId` | `SectionAssessmentResultListPage` | ⚠️ Verify | 5 |
| `/section-assessment-resultlist-kntc/:assessmentId` | `/section-assessment-results-kntc/:assessmentId` | `SectionAssessmentResultListKntcPage` | ⚠️ Verify | 5 |
| `/utility-batch-print` | `/utility/batch-print` | `BatchPrintPage` | ⚠️ Verify | 5 |
| `/system-benchmarks` | `/settings/system-benchmarks` | `SystemBenchmarksPage` | ⚠️ Verify | 5 |
| `/district-benchmarks` | `/settings/benchmarks` | `settings/DistrictBenchmarksPage` | ⚠️ Verify | 5 |
| `/district-yearlyassessment-benchmarks` | `/settings/yearly-benchmarks` | `settings/DistrictYearlyBenchmarksPage` | ⚠️ Verify | 5 |
| `/section-report-avmr-structured-detail/:sectionId/:benchmarkDateId/:studentId` | matching React route | `SectionReportAvmrStructuredDetailPage` | ⚠️ Verify | 4 |

**Route verification checklist per route (run before Angular route removal):**
1. Route defined in `routes.tsx` with matching params
2. Page renders without error at that route
3. Parity test passes (visual match vs. OldNorthStar baseline)
4. Angular route removed from `wwwroot/app/app.js`

---

## 7. New Shared Components Architecture

These components do not yet exist and are required by the migration. Listed by implementing phase.

### 7.1 Phase 1 (Foundation — Required Before Phase 2)

#### `<AutosaveIndicator status saveTime? onRetry? />`

```tsx
// components/shared/AutosaveIndicator.tsx
type AutosaveStatus = 'idle' | 'saving' | 'saved' | 'error';

interface AutosaveIndicatorProps {
  status: AutosaveStatus;
  lastSavedAt?: Date;
  onRetry?: () => void;
}

// Renders: idle=nothing | saving="● Saving..." | saved="✓ Saved HH:MM" | error="⚠ Save failed [Retry]"
// Uses aria-live="polite" region
```

#### `<UnsavedChangesDialog open pendingPath onConfirm onCancel />`

```tsx
// components/shared/UnsavedChangesDialog.tsx
// Wraps existing ConfirmDialog with fixed copy:
//   title: "Leave without saving?"
//   message: "You have unsaved changes. Your changes will be lost."
//   confirmLabel: "Leave anyway", cancelLabel: "Stay"
```

### 7.2 Phase 2 (Data Entry)

#### `<SectionBenchmarkSelector>`

```tsx
// components/shared/SectionBenchmarkSelector.tsx
interface SectionBenchmarkSelectorProps {
  sectionId: number | null;
  benchmarkDateId: number | null;
  onSectionChange: (sectionId: number) => void;
  onBenchmarkDateChange: (benchmarkDateId: number) => void;
  syncToUrl?: boolean;  // default true — writes params to URL
}
// Internally: useQuery for sections list, useQuery for benchmark dates by section
// syncToUrl: uses useSearchParams to update URL on change
```

### 7.3 Phase 4 (Reports)

#### `<SectionReportFilterBar>`

```tsx
// components/shared/SectionReportFilterBar.tsx
interface SectionReportFilterBarProps {
  sectionId: number;
  benchmarkDateId: number;
  availableReportTypes: ReportType[];
  selectedReportType: ReportType;
  onSectionChange: (id: number) => void;
  onBenchmarkDateChange: (id: number) => void;
  onReportTypeChange: (type: ReportType) => void;
  onPrint?: () => void;
}
// Extracted from repeated pattern across all SectionReport*Page files
```

---

## 8. State Management Architecture

### 8.1 State Categories

| Category | Technology | Scope | Notes |
|----------|-----------|-------|-------|
| Server data | TanStack Query v5 | `QueryClient` (app-wide) | All API data |
| Form state | React Hook Form v7 | Component | All form pages |
| URL / navigation state | React Router `useParams` + `useSearchParams` | Page | Filter state, entity IDs |
| Local transient UI state | `useState`/`useReducer` | Component | Modals, tabs, sort order |
| Auth state | `AuthContext` | App | User identity, roles |
| No Redux | — | — | No Redux / Zustand / Jotai |

### 8.2 URL State as Ground Truth

For complex pages (data entry, reports, charts), URL params are the ground truth for:
- Which entity is being viewed (`sectionId`, `benchmarkDateId`, `studentId`)
- Which report type / tab is selected (`reportType`, `tab`)
- Whether print mode is active (`?printmode=1`)

**Pattern:** `useSearchParams` for mutable filter state; `useParams` for route segment identifiers.

```tsx
// Correct — encode filter state in URL
const [searchParams, setSearchParams] = useSearchParams();
const reportType = searchParams.get('reportType') ?? 'WV';
// on change:
setSearchParams(prev => { prev.set('reportType', newType); return prev; });
```

---

## 9. Authentication Architecture

### 9.1 Current Pattern (No Changes in Scope)

```
Browser      RequireAuth HOC       AuthContext         API
   │ navigates to /section-data-entry/...
   │──────────────────▶               │                  │
   │                  │ useAuth()     │                  │
   │                  │◀──────────────│                  │
   │               isAuthenticated?  │                  │
   │                  │ NO → navigate('/login')          │
   │                  │ YES → render children            │
```

- Session maintained via HTTP-only cookie (set by server at login)
- `RequireAuth` HOC calls `authApi.checkSession()` on mount
- All API requests include cookie automatically via `withCredentials: true` on axios instance

---

## 10. Testing Architecture

### 10.1 Test Layers

```
┌──────────────────────────────────────────────────────────────┐
│  Unit Tests (Vitest + @testing-library/react)                │
│  Location: src/features/**/*.test.ts, src/pages/**/*.test.tsx │
│  Coverage: hooks, API functions, scoring logic, components   │
└──────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────┐
│  A11y Tests (vitest-axe)                                     │
│  Location: colocated with component tests                    │
│  Coverage: all new page shells (axe scan on render)          │
└──────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────┐
│  Parity Tests (Playwright)                                   │
│  Location: src/parity/                                       │
│  Coverage: EVERY migrated page — screenshot vs. OldNorthStar │
│  Run: npm run test:parity                                    │
└──────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────┐
│  E2E Tests (Playwright)                                      │
│  Location: e2e/                                              │
│  Coverage: SectionDataEntry save, SectionReports print,      │
│            critical teacher journey                          │
│  Run: npm run test:e2e                                       │
└──────────────────────────────────────────────────────────────┘
```

### 10.2 Scoring Unit Tests (SectionReports — Critical Gate)

Before Phase 4 removes any Angular SectionReports route:

```ts
// features/sectionReports/scoring/wvScoring.test.ts
// 20 known-good inputs derived from OldNorthStar production output

describe('WV Band Scoring', () => {
  test.each([
    // [score, grade, benchmark, expectedBand]
    [14, 2, 'Fall', 'C'],
    [22, 2, 'Fall', 'D'],
    // ...18 more
  ])('score=%s grade=%s benchmark=%s → band %s', (score, grade, benchmark, expected) => {
    expect(getWvBand({ score, grade, benchmark })).toBe(expected);
  });
});
```

Same pattern required for BAS level determination and AVMR scoring.

**Source of known-good inputs:** Run the old Angular app with test data against a seeded DB, capture output HTML, extract score→band mappings as JSON fixtures.

### 10.3 Parity Test Gate (Per Route)

```ts
// src/parity/SectionDataEntry.parity.ts
test('SectionDataEntryGenericPage matches OldNorthStar', async ({ page }) => {
  await page.goto('/#/section-data-entry/42/100/Generic');
  // wait for data
  await page.waitForSelector('[data-testid="data-entry-grid"]');
  await expect(page).toHaveScreenshot('section-data-entry-generic.png', {
    maxDiffPixelRatio: 0.01
  });
});
```

**Gate rule:** Parity test must be committed and passing in CI before the corresponding Angular route is removed from `app.js`. PRs that remove Angular routes must include the parity test in the same commit.

---

## 11. Migration Phasing — Technical Sequencing

### Phase 1 — Foundation (unlocks all phases)

**Atomic work items (can be parallel PRs):**

| PR | Scope | Blocks |
|----|-------|--------|
| `chore/api-camelcase` | API JsonNamingPolicy + TS types update | Phase 2+ |
| `chore/chart-primitive-audit` | Add ARIA to ChartContainer, verify StackedBarChart print mode, add perf guard | Phase 3 |
| `feat/autosave-hooks` | `useAutosave`, `useUnsavedChangesGuard`, `<AutosaveIndicator>`, `<UnsavedChangesDialog>` | Phase 2 |
| `feat/section-benchmark-selector` | `<SectionBenchmarkSelector>` component | Phase 2 |

**Phase 1 gate:** `NS4.Parity.Tests` 100% green after camelCase PR merged.

### Phase 2 — Data Entry (3–4 weeks)

**PR per controller:**

| PR | Pages | Angular Routes Removed |
|----|-------|----------------------|
| `feat/section-data-entry-autosave` | All SectionDataEntry* + hooks | All SectionDataEntry routes |
| `feat/intervention-data-entry` | `InterventionGroupDataEntryPage` verification | Remaining IE routes |
| `chore/student-staff-close-out` | StudentConsolidation, StaffConsolidation close-outs | Remaining Student.js/Staff.js routes |
| `chore/assessment-close-out` | AssessmentSyndicate, close-outs | Remaining Assessment.js routes |

### Phase 3 — Charts (2–3 weeks)

| PR | Pages | Angular Routes Removed |
|----|-------|----------------------|
| `feat/line-graphs` | Wire lineGraph page stubs to MultiLineChart + LineChartWithBenchmarks | All LineGraph routes |
| `feat/stacked-bar-graphs` | Full StackedBarGraph recharts rewrite | All StackedBarGraph routes |
| `feat/ig-student-dashboard-complete` | InterventionDashboard print-all, StudentDashboard print-all | ig-dashboard*, student-dashboard-printall |

### Phase 4 — Reports & Observations (3–5 weeks)

| PR | Pages | Angular Routes Removed |
|----|-------|----------------------|
| `feat/section-reports-scoring-tests` | Scoring unit tests only (no page changes) | — |
| `feat/section-reports-complete` | Section report page verification + parity | All SectionReports routes |
| `feat/observation-summary` | ObservationSummaryPage route verification | observation-summary routes |
| `feat/intervention-toolkit` | URL-state audit + verification | InterventionToolkit routes |

### Phase 5 — Admin Cleanup + Decommission (1–2 weeks)

| PR | Scope |
|----|-------|
| `feat/team-meetings-complete` | TeamMeeting attend/notes/invite route wire-up |
| `feat/admin-routes-complete` | All remaining settings/calendar/benchmark/import routes |
| `chore/angular-bundle-removal` | Remove Angular from Vite config, package.json, wwwroot |
| `chore/final-route-audit` | Playwright route audit: zero ng-app/ng-controller in live HTML |

---

## 12. Angular Bundle Removal (Phase 5 Procedure)

### 12.1 Pre-removal Checklist

- [ ] All routes in §6.3 show status ✅ (React confirmed)
- [ ] All parity tests green in CI
- [ ] `grep -r "ng-controller\|ng-app\|ng-repeat\|angular\." src/pages/ src/features/` returns nothing
- [ ] Manual smoke test: open 5 representative pages, confirm no Angular console errors

### 12.2 Removal Steps

```bash
# 1. Remove Angular npm dependencies
npm uninstall angular angular-route angular-bootstrap d3 jquery

# 2. Remove from Vite config (vite.config.ts)
# Remove any angular-related optimizeDeps/vendor entries

# 3. Remove from wwwroot
# Remove: wwwroot/app/scripts/controllers/
# Remove: wwwroot/app/scripts/services/ (Angular services)
# Remove: wwwroot/app/app.js (Angular SPA root)

# 4. Remove script tags in wwwroot/index.html or Razor Views
# that load angular.js, angular-route.js, d3.min.js, jquery.min.js

# 5. Build verification
npm run build
# Check: dist/ contains no chunk matching /angular|d3|jquery/
```

### 12.3 Post-removal Verification

```bash
# Confirm zero Angular in bundle
ls dist/assets/ | grep -iE "angular|jquery|d3-v3"
# Expected: no output

# Confirm npm packages gone
npm ls angular 2>&1 | head -5
# Expected: (empty)
```

---

## 13. Related Documents

- [PRD](./prd.md)
- [UX Brief](./ux-brief.md)
- [Architecture Brief (businessplan baseline)](./architecture-brief.md)
- [Tech Decisions](./tech-decisions.md)
- [NorthStarET Architecture](../../../../lens-sync/NorthStarET/architecture.md)
- [NorthStarET Migration Map](../../../../lens-sync/NorthStarET/migration-map.md)
- [OldNorthStar Architecture](../../../../lens-sync/OldNorthStar/architecture.md)
