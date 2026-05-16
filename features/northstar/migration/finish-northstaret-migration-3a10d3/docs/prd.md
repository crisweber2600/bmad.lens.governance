---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-02b-vision
  - step-02c-executive-summary
  - step-03-success
  - step-04-journeys
  - step-05-domain
  - step-06-innovation
  - step-07-project-type
  - step-08-scoping
  - step-09-functional
  - step-10-nonfunctional
  - step-11-polish
  - step-12-complete
inputDocuments:
  - docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/product-brief.md
  - docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/research-summary.md
  - docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/brainstorm-notes.md
  - docs/lens-sync/NorthStarET/architecture.md
  - docs/lens-sync/NorthStarET/migration-map.md
classification:
  projectType: "Brownfield SPA Migration (Angular.js → React 19)"
  domain: "K-12 EdTech — Literacy Assessment & Intervention Tracking"
  complexity: "High"
  projectContext: "Brownfield"
workflowType: prd
generated: true
phase: businessplan
initiative: finish-northstaret-migration-3a10d3
doc_type: prd
timestamp: "2026-02-26T12:00:00Z"
author: "John (PM) — BusinessPlan Phase"
status: draft
---

# Product Requirements Document
## Finish NorthStarET Migration — Angular.js → React 19

**Author:** Cris Weber
**Date:** 2026-02-26
**Phase:** BusinessPlan
**Initiative:** finish-northstaret-migration-3a10d3
**Status:** Draft

---

## 1. Executive Summary

NorthStarET requires completion of its Angular.js 1.4.5 → React 19 frontend migration. As of 2026-02-26, 171 React TSX components are shipped and serving ~45% of application routes. 35 Angular.js controllers remain in production at 26,092 total lines, blocking full decommission of the Angular.js bundle.

This PRD defines requirements for eliminating all Angular.js routes from production and achieving a single-SPA React 19 frontend. The work is a structured completion initiative, not a scope-expansion exercise. All backend (.NET 10) and infrastructure work is complete.

**Target outcome:** Zero Angular.js routes in production by end of Q3 2026, all visual parity tests passing, no regression on any currently-passing test suite.

---

## 2. Problem Statement

### 2.1 Business Problem

NorthStarET operates in dual-framework mode: some routes serve React 19 components, others still serve Angular.js 1.4.5 views from `wwwroot/app/scripts/controllers/`. This creates three compounding problems:

**Security & Maintainability**
- Angular.js reached End-of-Life in December 2021. No security patches are available upstream.
- The Angular.js bundle (`angular.js`, `angular-route.js`, jQuery, D3 v3) adds significant weight to every page load, including pages already migrated to React.
- New engineers cannot be onboarded to Angular.js without specialist knowledge.

**User Experience Debt**
The 35 remaining Angular controllers represent the highest-traffic, most user-visible workflows:
- `SectionDataEntry.js` (2,303 lines) — teachers' primary daily data-entry workflow has no autosave, no URL-shareable state, and no unsaved-changes guard.
- `SectionReports.js` (3,461 lines) — literacy coaches' most-used screen has four distinct sub-report types with complex print layouts that fail on modern browsers.
- `ObservationSummary.js` (1,903 lines) — classroom observation capture has multi-step form state that resets on back-navigation.
- `StackedBarGraphs.js` (2,813 lines) and `LineGraphs.js` (1,493 lines) — D3 v3 imperative rendering fails on high-DPI displays and produces non-accessible charts.

**Developer Velocity Tax**
Every new feature that touches a migrated page boundary requires reasoning across both frameworks. Each API response that serves an Angular page must maintain the legacy PascalCase casing convention, preventing a global `camelCase` serialization fix.

### 2.2 Technical Problem

The split-framework architecture cannot be collapsed until every Angular route is replaced:

```
Current routing split (active, 2026-02-26):
  Serving React:    ~45% of routes (171 TSX components)
  Serving Angular:  ~55% of routes (35 controllers, 26,092 lines)

Angular routes still active (from app.js):
  /observation-summary-class/:benchmarkDateId
  /observation-summary-filtered
  /stackedbargraph-groups/:autoload
  /stackedbargraph-comparison/:autoload
  /stacked-bar-graph-summary/:scoreGrouping/:testDueDate
  /section-report-avmr-structured-detail/...
  /ig-dashboard  (intervention group dashboard)
  /ig-dashboard/:schoolyear/...
  /ig-dashboard-printall/...
  /student-dashboard-printall/:id/:tab
  /district-calendar / /school-calendar
  /tm-attend/:teamMeeting  (team meeting attendance)
  /tm-attend-notes/:teamMeetingId/:studentId
  /tm-email-invite/...
  /ig-assessment-resultlist/:assessmentId
  /section-assessment-resultlist/:assessmentId
  /section-assessment-resultlist-kntc/:assessmentId
  /utility-batch-print
  /system-benchmarks / /district-benchmarks / /district-yearlyassessment-benchmarks
```

---

## 3. Goals & Vision

### 3.1 Initiative Vision

> **A fully React 19 NorthStarET frontend:** zero Angular.js routes in production, all existing users served without regression, and a codebase where every engineer can contribute to every page.

### 3.2 Non-Vision (What This Is Not)

This is a **completion initiative**. It is not:
- A design system overhaul
- A new feature build
- An EF6 → EF Core migration
- An SSR / Next.js adoption
- A mobile-native build

Any new-feature work not directly required to replace an Angular controller is deferred to post-migration.

### 3.3 Measurable Goals

| # | Goal | Metric |
|---|------|--------|
| G1 | Zero Angular.js in production | `npm ls angular` returns nothing; Angular bundle removed from Vite config |
| G2 | All routes served by React | Route audit: zero `ng-app` / `ng-controller` attributes in live HTML |
| G3 | Visual parity maintained | All Playwright parity tests pass in CI |
| G4 | No regression | All `NS4.WebAPI.Tests` and `NS4.Parity.Tests` pass |
| G5 | UX improvements delivered | SectionDataEntry: autosave confirmed via manual QA checklist |
| G6 | API casing consistent | Zero new `toLowerCase()` / `toUpperCase()` workarounds added after migration |

---

## 4. Target Users & User Journeys

### 4.1 Users

| User Type | Primary Workflows | Migration Impact |
|-----------|-------------------|-----------------|
| **Teachers** | SectionDataEntry (daily), DataEntryForms | **Critical** — daily workflow on Angular |
| **Literacy Coaches** | SectionReports, ObservationSummary | **Critical** — primary analysis tool on Angular |
| **Interventionists** | InterventionToolkit, IG Dashboard, IG Data Entry | **High** — multi-step workflow partially on Angular |
| **Administrators** | SchoolDashboards, DistrictSettings, DataExport, Import | Medium — periodic, not daily |
| **School Admins** | TeamMeetings, CalendarManagement, SystemBenchmarks | Medium |
| **Engineers** | All pages | High — maintenance velocity blocked |

### 4.2 Critical User Journeys

#### Journey 1: Daily Teacher Data Entry (SectionDataEntry)
```
Teacher opens section → selects benchmark date → enters scores per student
→ [CURRENT: no save guard — data lost on back-navigation]
→ [CURRENT: no autosave — connection drops lose full session]
→ [TARGET: autosave every 30s, URL state preserved, unsaved-changes guard on nav away]
```

#### Journey 2: Literacy Coach Reviews Section Report (SectionReports)
```
Coach views section → selects report type (WV / BAS / FP / HFW)
→ selects benchmark date → views scores + progress chart → prints
→ [CURRENT: print layout breaks on modern browsers; D3 charts not included in print]
→ [TARGET: React print CSS, recharts prints cleanly, all report types migrated]
```

#### Journey 3: Interventionist Monitors Student Progress (LineGraphs / IG Dashboard)
```
Interventionist opens student → selects metric → views trend line
→ [CURRENT: D3 v3 high-DPI rendering glitch; chart not keyboard-navigable]
→ [TARGET: recharts responsive, ARIA labels, keyboard nav]
```

#### Journey 4: Administrator Runs Data Export
```
Admin selects fields → applies filters → generates CSV/Excel
→ [CURRENT: Angular form, no progress indicator for large exports]
→ [TARGET: React form, same file output format, visual progress]
```

---

## 5. Domain Model

### 5.1 Core Domain Concepts

| Concept | Description | Key Relationships |
|---------|-------------|-------------------|
| **Section** | A classroom section (teacher + students + assessment) | Has many Students, one Teacher, many SectionDataEntries |
| **SectionDataEntry** | Scored assessment data for a section on a benchmark date | Belongs to Section, BenchmarkDate, Assessment |
| **BenchmarkDate** | A scored assessment event (Fall, Winter, Spring) | Has many SectionDataEntries |
| **Assessment** | An assessment instrument (WV, BAS, FP, HFW, CAP, LID, HRSIW, etc.) | Has type, scoring rules, report template |
| **SectionReport** | Aggregated report view for a section on a benchmark date | Computed from SectionDataEntries; has sub-types |
| **ObservationSummary** | Classroom observation record | Belongs to Section, BenchmarkDate |
| **InterventionGroup** | A group of struggling students receiving targeted intervention | Has many Students, an Interventionist, a Stint |
| **Student** | A student in the system | Has many Sections, InterventionGroups |
| **Staff** | A school staff member | Has roles and school assignments |
| **TeamMeeting** | A collaborative PLC meeting record | Has Agenda, Attendees, Notes |

### 5.2 Assessment Sub-Types (Critical for SectionReports Migration)

SectionReports.js handles multiple report sub-types, each with distinct scoring logic:

| Code | Name | Complexity | Key Challenge |
|------|------|-----------|---------------|
| WV | Word Vocabulary | High | Multi-column scoring, color bands |
| BAS | Benchmark Assessment System | High | Level grid, progress arrows |
| FP | Fountas & Pinnell | Medium | Reading level ladder |
| HFW | High-Frequency Words | Medium | Word list tracking |
| CAP | Concepts About Print | Low | Checklist |
| LID | Letter ID | Medium | Letter grid |
| HRSIW | Hearing & Recording Sounds | Medium | Phoneme scoring |
| AVMR | Adapted from VM/R | High | Structured detail sub-page |
| SPELL | Spelling | Medium | Stage scoring |
| KNTC | Kinesthetic/Tactile | Low | Simple list |

**⚠️ LEGACY_DOCUMENTATION.md GATE:** Before implementing WV, BAS, or AVMR scoring, `LEGACY_DOCUMENTATION.md` in the OldNorthStar repo must be reviewed. These contain undocumented calculation rules not derivable from the Angular code alone.

---

## 6. Scope

### 6.1 In Scope

#### Priority 1 — Foundation (unblocks all phases)

| Item | Rationale |
|------|-----------|
| API response casing fix (`JsonSerializerOptions.PropertyNamingPolicy = CamelCase`) | Eliminates casing bug class; must land before new form work |
| Shared React chart primitive library (`ChartContainer`, `ResponsiveLineChart`, `ResponsiveStackedBarChart`) | Required before Phase 3; partial primitives exist in `components/charts/` |

#### Priority 2 — High-Value Data Entry (blocks teachers daily)

| Controller | Lines | React Pages Needed | Notes |
|-----------|-------|-------------------|-------|
| `SectionDataEntry.js` | 2,303 | All SectionDataEntry* page variants | Must add: autosave, URL state, unsaved-changes guard |
| `Student.js` (close-out) | 703 | StudentListPage, StudentEditPage, StudentConsolidationPage | ~70% done |
| `Staff.js` (close-out) | 516 | StaffListPage, StaffEditPage, StaffConsolidationPage | ~65% done |
| `Assessment.js` (close-out) | 886 | AssessmentEditPage, AssessmentListPage, AssessmentSyndicatePage | ~50% done |
| `InterventionDataEntry.js` | 500 | InterventionGroupDataEntryPage | Verify completeness |

#### Priority 3 — Charts (require shared primitives)

| Controller | Lines | React Pages Needed | Notes |
|-----------|-------|-------------------|-------|
| `LineGraphs.js` | 1,493 | StudentLineGraphPage, TrendLineGraphPage, IGLineGraphPage, InterventionGroupLineGraphPage | Route stubs exist; connect to shared primitives |
| `StackedBarGraphs.js` | 2,813 | StackedBarGraphGroupsPage, StackedBarGraphComparisonPage | All 3 Angular routes must be covered |
| `InterventionGroups.js` (close-out) | 657 | InterventionGroupListPage, InterventionGroupEditPage | Finish remaining flows |
| `StudentDashboard.js` | 675 | StudentDashboardPage, student-dashboard-printall | Complete modal + print-all |

#### Priority 4 — Reports & Observations (highest complexity)

| Controller | Lines | React Pages Needed | Notes |
|-----------|-------|-------------------|-------|
| `SectionReports.js` | 3,461 | All section report page variants | Read LEGACY_DOCUMENTATION.md first; extract `SectionReportBase` |
| `ObservationSummary.js` | 1,903 | ObservationSummaryPage (class + filtered) | Multi-step form state |
| `InterventionToolkit.js` | 1,274 | InterventionToolkitPage | Multi-step wizard; URL state-aware |
| `InterventionDashboard.js` | 500 | InterventionDashboardPage (all ig-dashboard routes incl. printall) | ~30% done |

#### Priority 5 — Admin Cleanup & Decommission

| Controller | Lines | Notes |
|-----------|-------|-------|
| `TeamMeetings.js` | 1,038 | tm-attend, tm-attend-notes, tm-email-invite routes |
| `SchoolDashboards.js` | 712 | SchoolDashboardPage, DistrictDashboardPage |
| `DataExport.js` | 742 | DataExportPage in settings |
| `DistrictSettings.js` | 820 | CalendarManagement, Benchmarks, SystemBenchmarks |
| `ImportStateTestData.js` | 751 | Import* pages — verify all routes |
| `Rollover.js` | 803 | RolloverPage, StaffRolloverPage — verify |
| `SectionAssessmentResultList` routes | — | section-assessment-resultlist and KNTC variants |
| `IgAssessmentResultList` route | — | ig-assessment-resultlist |
| `BatchPrint` route | — | utility-batch-print → BatchPrintPage |
| Angular bundle removal | — | Vite config + wwwroot cleanup |

### 6.2 Out of Scope

| Item | Reason |
|------|--------|
| New backend API endpoints | .NET 10 API complete and stable |
| EF6 → EF Core migration | Separate initiative |
| SSR / Next.js | Additive; not required |
| New features not in OldNorthStar | Scope creep |
| i18n / localization | Not present in OldNorthStar |
| Design system re-skin | Maintain visual parity |
| Mobile-native apps | Not in scope |

---

## 7. Functional Requirements

### FR-1: API Response Casing Normalization
**FR-1.1** Configure `JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase` in `NS4.WebAPI` `Program.cs`.
**FR-1.2** All existing React components using `toLowerCase()` / `toUpperCase()` casing workarounds must be updated to use native camelCase property names.
**FR-1.3** All existing Playwright parity tests must pass after this change.

### FR-2: Shared Chart Primitive Library
**FR-2.1** Expose `<ResponsiveLineChart>`, `<ResponsiveStackedBarChart>`, `<ChartContainer>` with standardized props from `components/charts/`.
**FR-2.2** All chart components must use `ResponsiveContainer` from recharts.
**FR-2.3** All chart components must include ARIA labels and roles for accessibility.
**FR-2.4** All chart components must render correctly in print CSS (`@media print`).
**FR-2.5** Evaluate existing `components/charts/` files for reuse/replacement vs. new primitives.

### FR-3: SectionDataEntry Migration
**FR-3.1** All SectionDataEntry assessment variants (Generic, CAP, LID, HRSIW, AVMR, BAS-Alt, SpellV4p, InitialSound, Hrsiw2/3/4) must have React page feature parity.
**FR-3.2** Section-level data entry must autosave to the API at minimum every 30 seconds with a visible save indicator.
**FR-3.3** The URL must encode section, benchmark date, and assessment type to enable bookmarking and back-navigation without state loss.
**FR-3.4** Navigating away with unsaved changes must display a confirmation dialog.
**FR-3.5** All tests in `DataEntryPages.test.tsx` must pass.

### FR-4: SectionReports Migration
**FR-4.1** All section report sub-types must be served by React pages: WV, BAS, FP, HFW, CAP, LID, HRSIW, HRSIW2/3/4, AVMR (structured + detail), SPELL, KNTC, InitialSound.
**FR-4.2** The AVMR structured detail route (`/section-report-avmr-structured-detail/...`) must be handled by `SectionReportAvmrStructuredDetailPage.tsx`.
**FR-4.3** All section reports must be printable via browser print with correct rendering.
**FR-4.4** Section report calculations for WV band scoring, BAS level determination, and AVMR scoring must match OldNorthStar output exactly. `LEGACY_DOCUMENTATION.md` must be consulted before implementing.
**FR-4.5** A `SectionReportBase` shared component must be extracted for the common filter bar.

### FR-5: Chart Migrations (LineGraphs + StackedBarGraphs)
**FR-5.1** All Angular chart routes must be served by React pages using shared primitives (FR-2):
- `/stackedbargraph-groups/:autoload` → `StackedBarGraphGroupsPage`
- `/stackedbargraph-comparison/:autoload` → `StackedBarGraphComparisonPage`
- `/stacked-bar-graph-summary/:scoreGrouping/:testDueDate`
- All line graph routes → `StudentLineGraphPage`, `TrendLineGraphPage`, `InterventionGroupLineGraphPage`, `IGLineGraphPage`
**FR-5.2** StackedBarGraphs must support group and comparison display modes.
**FR-5.3** Line graphs must support benchmark overlay (horizontal dashed lines).
**FR-5.4** All charts must pass Playwright visual parity snapshot tests.

### FR-6: ObservationSummary Migration
**FR-6.1** Routes `/observation-summary-class/:benchmarkDateId` and `/observation-summary-filtered` must be served by `ObservationSummaryPage.tsx`.
**FR-6.2** Multi-step observation form state must persist across page navigations via React state / URL params.

### FR-7: InterventionToolkit Migration
**FR-7.1** `InterventionToolkitPage.tsx` must implement the full multi-step intervention workflow.
**FR-7.2** Step navigation must be URL-state-aware (step in URL param).

### FR-8: TeamMeetings Migration
**FR-8.1** All TeamMeeting Angular routes must be served by React:
- `/tm-attend/:teamMeeting` → `TeamMeetingAttendPage.tsx`
- `/tm-attend-notes/:teamMeetingId/:studentId` → `TeamMeetingNotesPage.tsx`
- `/tm-email-invite/:teamMeetingId/:tddid/:staffId` → `TeamMeetingInvitationsPage.tsx`

### FR-9: All Remaining Angular Route Coverage
**FR-9.1** `/student-dashboard/:id` and `/student-dashboard-printall/:id/:tab` → `StudentDashboardPage.tsx`
**FR-9.2** All ig-dashboard routes → `InterventionDashboardPage.tsx` (including printall variant)
**FR-9.3** `/section-assessment-resultlist/:assessmentId`, `/section-assessment-resultlist-kntc/:assessmentId` → respective result list pages
**FR-9.4** `/ig-assessment-resultlist/:assessmentId` → `IgAssessmentResultListPage.tsx`
**FR-9.5** `/utility-batch-print` → `BatchPrintPage.tsx`
**FR-9.6** All district/school admin routes (calendar, benchmarks, data export, import, dashboards, settings) → settings/* and admin pages

### FR-10: Angular Bundle Decommission
**FR-10.1** After all routes confirmed React-served, remove `angular.js`, `angular-route.js`, `angular-bootstrap.js`, D3 v3, jQuery 1.x from Vite bundle config.
**FR-10.2** `wwwroot/app/scripts/controllers/` must be emptied or removed from static path.
**FR-10.3** `npm run build` must produce a bundle with no Angular-related chunk.
**FR-10.4** Final route audit must confirm zero `ng-app` / `ng-controller` attributes in rendered HTML.

---

## 8. Non-Functional Requirements

### NFR-1: Performance
- **NFR-1.1** Initial page load (LCP) ≤ 2.5s on 4G after Angular bundle removal.
- **NFR-1.2** SectionDataEntry autosave completes within 3s; visible error if fails.
- **NFR-1.3** Chart renders complete within 1s for ≤ 30 students × 3 benchmark dates.

### NFR-2: Accessibility
- **NFR-2.1** All migrated pages meet WCAG 2.1 AA for color contrast and keyboard navigation.
- **NFR-2.2** All chart components expose ARIA labels readable by screen readers.
- **NFR-2.3** All data-entry forms have properly associated labels and error associations.

### NFR-3: Browser Support
- **NFR-3.1** Chrome 120+, Firefox 120+, Edge 120+, Safari 17+. (IE11 not required — Angular removal unblocks this.)
- **NFR-3.2** Print layout works in Chrome print preview.

### NFR-4: Test Coverage
- **NFR-4.1** Every new React page must have a Playwright parity test.
- **NFR-4.2** SectionDataEntry autosave must have unit test coverage.
- **NFR-4.3** Section report calculation logic (WV, BAS, AVMR) must have unit tests with known-good inputs.
- **NFR-4.4** All existing `NS4.WebAPI.Tests` must remain passing throughout.

### NFR-5: Security
- **NFR-5.1** No Angular.js post-EOL security surface retained after removal.
- **NFR-5.2** All new React pages use existing `RequireAuth` HOC for authenticated routes.
- **NFR-5.3** No sensitive student data stored in localStorage or sessionStorage.

### NFR-6: Maintainability
- **NFR-6.1** All new React pages use TypeScript with no untyped `any` except legacy API interfaces.
- **NFR-6.2** All new API calls use `@tanstack/react-query`.
- **NFR-6.3** All new forms use `react-hook-form`.
- **NFR-6.4** All charts use `recharts` only — no direct D3 usage in new code.

---

## 9. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| SectionReports scoring logic mismatch | **High** | **High** | Review LEGACY_DOCUMENTATION.md before coding; unit test with known-good production data |
| D3→recharts rewrite underestimated | **High** | **Medium** | Use shared chart primitives; budget 3–4x vs simple port for StackedBarGraphs |
| API camelCase change breaks existing React pages | **Medium** | **High** | Run full parity test suite before merging camelCase fix |
| Phase 2 velocity slower (teacher workflow complexity) | **Medium** | **High** | Seed autosave pattern early; identify second engineer if slipping |
| Undocumented Angular directive behavior (ObservationSummary) | **Medium** | **Medium** | Step through Angular source in dev browser; write behavior tests before migration |
| Browser-cached Angular routes after cutover | **Low** | **High** | Set `Cache-Control: no-cache` on `app.js`; invalidate service worker on deploy |

---

## 10. Dependencies

| Dependency | Owner | Status | Blocker For |
|------------|-------|--------|-------------|
| `.NET 10` API stable | Backend | ✅ Complete | — |
| Playwright parity test framework | QA / DevOps | ✅ Operational | All phases |
| NorthStarET.AppHost (Aspire) | Platform | ✅ Operational | Dev environment |
| `LEGACY_DOCUMENTATION.md` reviewed | Architect | ⏳ Required before Phase 4 | SectionReports complex scoring |
| Shared chart primitive library | Frontend | ⏳ Phase 1 | Phase 3 |
| API camelCase fix merged | Frontend/Backend | ⏳ Phase 1 | All new form work |
| Second engineer (optional) | Management | ❓ Decision needed | Phase 2+ velocity |

---

## 11. Success Criteria (Definition of Done)

| Criterion | Verification Method |
|-----------|-------------------|
| Zero `angular.js` in production bundle | `npm run build` → no Angular chunk in `dist/` |
| All routes serve React components | Playwright route audit: zero `ng-app` / `ng-controller` in HTML |
| All Playwright parity tests pass | CI: `NS4.Parity.Tests` 100% green |
| All unit tests pass | CI: `NS4.WebAPI.Tests` 100% green |
| SectionDataEntry autosave functional | Manual QA: data persists across browser refresh during active session |
| Section report calculations correct (WV, BAS, AVMR) | Unit test: 20 known-good inputs match OldNorthStar output |
| Zero new casing workarounds | Code review gate in PR template |
| Angular npm packages removed | `package.json` has no `angular*` dependencies |
| `wwwroot/app/scripts/controllers/` empty | File system check in CI |

---

## 12. Related Documents

- [Product Brief](./product-brief.md)
- [Research Summary](./research-summary.md)
- [Brainstorm Notes](./brainstorm-notes.md)
- [NorthStarET Architecture](../../../../lens-sync/NorthStarET/architecture.md)
- [Migration Map](../../../../lens-sync/NorthStarET/migration-map.md)
- [OldNorthStar Architecture](../../../../lens-sync/OldNorthStar/architecture.md)
- [OldNorthStar Migration Map](../../../../lens-sync/OldNorthStar/migration-map.md)
