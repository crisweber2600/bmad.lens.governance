---
doc_type: ux_brief
phase: businessplan
initiative: finish-northstaret-migration-3a10d3
status: draft
author: "Sally (UX Designer) — BusinessPlan Phase"
timestamp: "2026-02-26T13:00:00Z"
related_prd: docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/prd.md
---

# UX Brief
## Finish NorthStarET Migration — Angular.js → React 19

**Author:** Cris Weber / Sally (UX)
**Date:** 2026-02-26
**Phase:** BusinessPlan
**Initiative:** finish-northstaret-migration-3a10d3

---

## 1. UX Overview & Philosophy

This migration is a **visual parity initiative with targeted UX upgrades**. The guiding rule is:

> **Keep what works. Fix what's broken. Don't redesign.**

Teachers and literacy coaches have established muscle memory with NorthStarET. Arbitrary visual changes cause rework and training costs. The migration will:

1. **Mirror** the OldNorthStar visual layout as the default baseline for every migrated screen.
2. **Apply** targeted UX improvements only where the Angular version has documented UX defects (data loss, broken print, inaccessible charts).
3. **Maintain** all navigation patterns (section picker, benchmark date picker, report type tabs) exactly.

---

## 2. Page Inventory

### 2.1 Currently Serving React (Complete / Maintenance Mode)

These pages are already in React and are not part of the migration scope. They are referenced here as pattern sources.

| Page File | Route Pattern | User Type | Notes |
|-----------|--------------|-----------|-------|
| `DashboardPage.tsx` | `/` | All | Home dashboard, React complete |
| `ClassroomDashboardPage.tsx` | `/classroom-dashboard` | Teacher | Multiple variants (filtered, multi-column, multi-date) |
| `InterventionDashboardPage.tsx` | `/ig-dashboard/*` | Interventionist | ⚠️ Angular ig-dashboard routes partially remaining |
| `StudentListPage.tsx` | `/students` | Admin | React complete |
| `StudentEditPage.tsx` | `/students/:id` | Admin | React complete |
| `StaffListPage.tsx` | `/staff` | Admin | React complete |
| `StaffEditPage.tsx` | `/staff/:id` | Admin | React complete |
| `SectionListPage.tsx` | `/sections` | Teacher | React complete |
| `SectionEditPage.tsx` | `/sections/:id` | Teacher | React complete |
| `InterventionGroupListPage.tsx` | `/ig` | Interventionist | Close-out remaining flows |
| `InterventionGroupEditPage.tsx` | `/ig/:id` | Interventionist | Close-out remaining flows |
| `ObservationSummaryPage.tsx` | `/observation-summary*` | Coach | ⚠️ Angular routes still active |
| `StackedBarGraphGroupsPage.tsx` | `/stackedbargraph-groups/:autoload` | Coach | ⚠️ Angular route still active |
| `StackedBarGraphComparisonPage.tsx` | `/stackedbargraph-comparison/:autoload` | Coach | ⚠️ Angular route still active |
| `StudentDashboardPage.tsx` | `/student-dashboard/:id` | Coach | ⚠️ printall route still Angular |
| All `import*/export*` Pages | `/settings/import*`, `/settings/export*` | Admin | React complete or near-complete |
| All `settings/*` Pages | `/settings/*` | Admin | Most done; see §2.3 |
| All `TeamMeeting*` Pages | `/team-meetings/*` | Admin | ⚠️ Angular attend/notes/invite routes remain |

### 2.2 Migration Targets (Angular → React)

Ordered by priority per PRD.

#### Phase 1 — Foundation (No New Pages)
No new pages. Work is API-layer and chart primitives.

#### Phase 2 — Data Entry Completion

| Target | Angular Controller | Lines | Status | UX Priority |
|--------|-------------------|-------|--------|------------|
| SectionDataEntry (all variants) | `SectionDataEntry.js` | 2,303 | Page files exist — add autosave/URL state | **P1 — Critical** |
| InterventionDataEntry | `InterventionDataEntry.js` | 500 | `InterventionGroupDataEntryPage.tsx` exists | Verify completeness |
| Student close-out | `Student.js` | 703 | `StudentConsolidationPage.tsx` — close-out | Medium |
| Staff close-out | `Staff.js` | 516 | `StaffConsolidationPage.tsx` — close-out | Medium |
| Assessment close-out | `Assessment.js` | 886 | `AssessmentSyndicatePage.tsx` — close-out | Medium |

#### Phase 3 — Charts

| Target | Angular Controller | Lines | React Page | Status |
|--------|-------------------|-------|-----------|--------|
| Line Graphs | `LineGraphs.js` | 1,493 | `StudentLineGraphPage`, `TrendLineGraphPage`, `IGLineGraphPage`, `InterventionGroupLineGraphPage` | Page stubs exist — connect to chart primitives |
| Stacked Bar Graphs | `StackedBarGraphs.js` | 2,813 | `StackedBarGraphGroupsPage`, `StackedBarGraphComparisonPage` | Page stubs exist — full recharts rewrite |
| Intervention close-out | `InterventionGroups.js` | 657 | `InterventionGroupListPage`, `InterventionGroupEditPage` | Finish remaining flows |
| Student Dashboard | `StudentDashboard.js` | 675 | `StudentDashboardPage` | Complete modal + print-all route |

#### Phase 4 — Reports & Observations

| Target | Angular Controller | Lines | React Page | Status |
|--------|-------------------|-------|-----------|--------|
| Section Reports (WV, BAS, FP, HFW, etc.) | `SectionReports.js` | 3,461 | Multiple `*SectionReportPage.tsx` files | Most variants exist; WV/BAS/AVMR need verification |
| Observation Summary | `ObservationSummary.js` | 1,903 | `ObservationSummaryPage.tsx` | Exists — verify Angular route removal |
| Intervention Toolkit | `InterventionToolkit.js` | 1,274 | `InterventionToolkitPage.tsx` | Exists — URL-state audit |
| Intervention Dashboard | `InterventionDashboard.js` | 500 | `InterventionDashboardPage.tsx` | ⚠️ /ig-dashboard/* routes still Angular |

#### Phase 5 — Admin Cleanup

| Target | Angular Controller | Lines | Status |
|--------|-------------------|-------|--------|
| Team Meetings | `TeamMeetings.js` | 1,038 | `TeamMeetingAttendPage`, `TeamMeetingNotesPage`, `TeamMeetingInvitationsPage` exist — verify routes wired |
| School/District Dashboards | `SchoolDashboards.js` | 712 | `SchoolDashboardPage`, `DistrictDashboardPage` exist |
| Data Export | `DataExport.js` | 742 | `settings/DataExportPage.tsx` exists |
| District Settings | `DistrictSettings.js` | 820 | Settings pages exist — verify all sub-routes |
| Import State Tests | `ImportStateTestData.js` | 751 | Import* pages exist — verify sub-routes |
| Rollover | `Rollover.js` | 803 | `settings/RolloverPage.tsx`, `rollover/StaffRolloverPage.tsx` exist |
| Batch Print | — | — | `BatchPrintPage.tsx` — verify route wired |
| Angular bundle removal | — | — | Last step |

### 2.3 Settings Pages Audit

Settings sub-pages that need route verification:

| File | Route | Area |
|------|-------|------|
| `settings/AssessmentConfigPage.tsx` | `/settings/assessment-config` | Admin |
| `settings/DataExportPage.tsx` | `/settings/data-export` | Admin |
| `settings/DistrictAccessPage.tsx` | `/settings/district-access` | District Admin |
| `settings/DistrictBenchmarkDatesPage.tsx` | `/settings/benchmark-dates` | District Admin |
| `settings/DistrictBenchmarksPage.tsx` | `/settings/benchmarks` | District Admin |
| `settings/DistrictCalendarPage.tsx` | `/settings/calendar` | District Admin |
| `settings/DistrictHfwListPage.tsx` | `/settings/hfw-list` | District Admin |
| `settings/DistrictInterventionsPage.tsx` | `/settings/interventions` | District Admin |
| `settings/DistrictSpedLabelsPage.tsx` | `/settings/sped-labels` | District Admin |
| `settings/DistrictYearlyBenchmarksPage.tsx` | `/settings/yearly-benchmarks` | District Admin |
| `settings/PersonalAssessmentsPage.tsx` | `/settings/my-assessments` | Teacher |
| `settings/PersonalFieldsPage.tsx` | `/settings/my-fields` | Teacher |
| `settings/RolloverPage.tsx` | `/settings/rollover` | Admin |
| `settings/StudentAttributesPage.tsx` | `/settings/student-attributes` | Admin |
| `settings/UserPasswordPage.tsx` | `/settings/password` | Any |
| `settings/UserProfilePage.tsx` | `/settings/profile` | Any |
| `settings/UserUsernamePage.tsx` | `/settings/username` | Any |
| `SystemBenchmarksPage.tsx` | `/settings/system-benchmarks` | Super Admin |

---

## 3. Interaction Patterns

### 3.1 Data Entry Autosave (NEW — SectionDataEntry)

**Problem:** Teachers lose entire data-entry sessions when the browser tab is closed or navigated away. Angular version has no save guard.

**Design:**
```
┌─────────────────────────────────────────────────────────────┐
│ Section: Wilson A · Fall 2025 · Word Vocabulary             │
│                                                [● Saving...] │
├─────────────────────────────────────────────────────────────┤
│  Student Name        │ Score │ Notes │ Level │              │
│  ───────────────────────────────────────────────────────     │
│  Adams, Jaylen       │  [14] │       │  [C]  │              │
│  Brown, Mia         │  [22] │       │  [D]  │              │
│  ...                                                         │
└─────────────────────────────────────────────────────────────┘
```

- **Autosave trigger:** Debounced 1.5s after last keystroke, AND interval every 30s.
- **Save indicator states:** `● Saving...` → `✓ Saved 2:14 PM` → `⚠ Save failed` (with retry button).
- **Unsaved-changes guard:** React Router `beforeunload` handler + custom `<UnsavedChangesDialog>` on navigation-away.
- **URL state:** `/section-data-entry/:sectionId/:benchmarkDateId/:assessmentType` — all three params in URL so back/forward/bookmark works.

### 3.2 URL-State-Aware Navigation (All Complex Pages)

Any page with filter state (benchmark date picker, report type, student selector) must encode that state in the URL.

```
/section-reports/:sectionId/:benchmarkDateId/:reportType
/observation-summary/:benchmarkDateId?filter=...
/ig-dashboard/:schoolYear/:igId
/student-dashboard/:studentId?tab=progress
```

**Pattern:** `useSearchParams` for filter state, `useParams` for entity IDs.
**Goal:** Browser back/forward works predictably; teachers can share links.

### 3.3 Print Layout

Multiple pages require print-safe layouts:
- SectionReports (all types)
- StudentDashboardPrintAll
- BatchPrint
- IGDashboardPrintAll

**Pattern:**
```tsx
// Every printable page uses this structure:
<div className="screen-only">
  {/* filters, nav, controls */}
</div>
<div className="print-content">
  {/* only printed content; charts via recharts with print CSS */}
</div>
```

Recharts `ResponsiveContainer` wraps must be replaced with `width={600} height={300}` (fixed) for print — dynamic containers collapse in print media.

### 3.4 Confirmation Dialogs

Standard pattern for destructive / unsaved-changes scenarios:

```tsx
<ConfirmDialog
  open={showUnsavedDialog}
  title="Leave without saving?"
  message="You have unsaved data entry. Your changes will be lost."
  confirmLabel="Leave"
  cancelLabel="Stay"
  onConfirm={() => navigate(pendingPath)}
  onCancel={() => setShowUnsavedDialog(false)}
/>
```

Use existing `ConfirmDialog` component — do not create per-page dialogs.

### 3.5 Section/Benchmark Date Pickers

The section + benchmark date picker combo appears on every data entry and report page. It is the primary navigation control for teachers and coaches.

**Current state in React pages:** Inconsistently implemented — some use `SectionPicker`, some have inline `<select>` elements.

**Target:** Extract `<SectionBenchmarkSelector>` shared component that:
- Accepts `sectionId`, `benchmarkDateId`, `onSectionChange`, `onBenchmarkDateChange`
- Handles fetch + loading states internally
- Syncs to URL params automatically

---

## 4. Chart UX

### 4.1 Existing Chart Primitives

Located at `components/charts/`:

| File | Type | Status |
|------|------|--------|
| `ChartContainer.tsx` | Wrapper | ✅ Exists — use as outer shell |
| `BarChart.tsx` | Basic bar | ✅ Exists |
| `StackedBarChart.tsx` | Stacked bar | ✅ Exists — basis for StackedBarGraphs migration |
| `LineChart.tsx` | Basic line | ✅ Exists |
| `LineChartWithBenchmarks.tsx` | Line + overlay | ✅ Exists — key for SectionDataEntry trend |
| `MultiLineChart.tsx` | Multi-series line | ✅ Exists — key for IGLineGraph |

### 4.2 Chart Requirements by Page

| Page | Chart Type | Benchmark Overlay | Print | Accessibility |
|------|-----------|-------------------|-------|--------------|
| `StudentLineGraphPage` | Multi-line (student progress) | Yes (dashed lines per benchmark) | Required | ARIA labels per data point |
| `TrendLineGraphPage` | Multi-line (section trend) | Yes | Required | ARIA labels |
| `IGLineGraphPage` | Multi-line (IG avg vs. benchmark) | Yes | Required | ARIA labels |
| `InterventionGroupLineGraphPage` | Line (single IG) | Yes | Required | ARIA labels |
| `StackedBarGraphGroupsPage` | Stacked bar (group view) | No | Required | ARIA per segment |
| `StackedBarGraphComparisonPage` | Stacked bar (comparison) | No | Required | ARIA per segment |
| Section Report charts (WV, BAS) | Bar / level grid | No | Required | ARIA per bar |

### 4.3 D3 → Recharts Migration Rules

1. **No new D3 code.** All new chart code uses recharts only.
2. **ResponsiveContainer note:** Use for screen; switch to fixed dimensions for print (`@media print`).
3. **Color palette:** Maintain OldNorthStar color bands (benchmark level colors) — extract to `chartColors.ts`.
4. **Performance:** For ≥ 30 students, disable animations (`isAnimationActive={false}`) to prevent jank.
5. **Tooltip:** All charts expose a custom recharts `<CustomTooltip>` showing student name + score + level where applicable.

---

## 5. Accessibility

### 5.1 Minimum Requirements (WCAG 2.1 AA)

- All interactive elements reachable by Tab/Shift+Tab.
- All form fields have associated `<label>` or `aria-label`.
- Color is never the sole differentiator (benchmark level bands need text + icon in addition to color).
- Screen readers must announce save state changes (`aria-live="polite"` region for autosave indicator).
- Charts expose `role="img"` with `aria-label` describing the chart content (e.g., "Line graph of student [Name] reading level progress, Fall 2023 to Spring 2025").

### 5.2 Data Entry Grid Accessibility

SectionDataEntry renders a student × assessment-field grid. Requirements:
- Grid cells are `<td>` with `<th scope="row">` for student names and `<th scope="col">` for field headers.
- Number inputs include `aria-label="[Student Name] [Field Name] score"`.
- Invalid score ranges announce error inline (no separate error summary).

---

## 6. UX Approval Gates

Before each phase's implementation:

| Gate | Check |
|------|-------|
| Phase 2 | SectionDataEntry wireframe review with one teacher (if available) |
| Phase 4 | SectionReports — verify print output matches OldNorthStar printed report format exactly |
| Phase 5 | Route audit confirms no Angular HTML in any response |

---

## 7. Component Candidates for Extraction

These are shared UX patterns that appear across ≥3 pages and should be extracted:

| Candidate | Appears In | Priority |
|-----------|-----------|---------|
| `<SectionBenchmarkSelector>` | All data entry + report pages | **P1** |
| `<SectionReportFilterBar>` | All `*SectionReport*` pages | **P1** |
| `<AutosaveIndicator>` | SectionDataEntry + InterventionDataEntry | **P1** |
| `<UnsavedChangesDialog>` | SectionDataEntry + any multi-field form | **P1** |
| `<PrintWrapper>` | All printable pages | **P2** |
| `<ChartLegend>` | All chart pages | **P2** |
| `<BenchmarkLevelBadge>` | Reports + Dashboard pages | **P2** |
| `<StudentAvatarCell>` | Data entry grid + report tables | **P3** |

---

## 8. Related Documents

- [PRD](./prd.md)
- [Architecture Brief](./architecture-brief.md)
- [NorthStarET Architecture](../../../../lens-sync/NorthStarET/architecture.md)
- [Migration Map](../../../../lens-sync/NorthStarET/migration-map.md)
