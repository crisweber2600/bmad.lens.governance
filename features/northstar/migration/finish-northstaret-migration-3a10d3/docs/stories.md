---
doc_type: stories
phase: devproposal
initiative: finish-northstaret-migration-3a10d3
status: draft
author: "John (PM) + Winston (Architect) — DevProposal Phase"
timestamp: "2026-02-26T19:00:00Z"
input_artifacts:
  - epics.md
  - architecture.md
  - tech-decisions.md
  - epic-stress-gate-summary.md
story_count: 38
total_estimated_points: 118
---

# Stories — Finish NorthStarET Migration

**Author:** John (PM) + Winston (Architect)
**Phase:** DevProposal — Step [3] of 4
**Initiative:** finish-northstaret-migration-3a10d3
**Date:** 2026-02-26

> Stories are sized in modified Fibonacci (1, 2, 3, 5, 8, 13). Points are rough estimates for sprint planning; teams should re-estimate in sprint planning.
> All acceptance criteria inherit stress gate amendments from `epic-stress-gate-summary.md`.

---

## Story Map

| Story ID | Title | Epic | Points | Priority |
|----------|-------|------|--------|----------|
| S-001 | API CamelCase Migration | EP-001 | 3 | P0 |
| S-002 | useAutosave Hook | EP-001 | 3 | P0 |
| S-003 | AutosaveIndicator Component | EP-001 | 2 | P0 |
| S-004 | UnsavedChangesDialog Component | EP-001 | 2 | P0 |
| S-005 | SectionBenchmarkSelector Shared Component | EP-001 | 3 | P0 |
| S-006 | ChartContainer ARIA + StackedBarChart Print Audit | EP-001 | 2 | P0 |
| S-007 | QueryClient TTL Constants | EP-001 | 1 | P0 |
| S-008 | SectionDataEntry Autosave Integration | EP-002 | 5 | P1 |
| S-009 | SectionDataEntry Parity Tests | EP-002 | 5 | P1 |
| S-010 | SectionDataEntry Keyboard Navigation | EP-002 | 3 | P1 |
| S-011 | SectionDataEntry Virtualization (Large Rosters) | EP-002 | 3 | P1 |
| S-012 | Close-Out Routes Verification | EP-002 | 2 | P1 |
| S-013 | SectionDataEntry Angular Route Removal | EP-002 | 2 | P1 |
| S-014 | Chart Primitives: LineGraph Recharts Implementation | EP-003 | 5 | P1 |
| S-015 | Chart Primitives: StackedBarGraph Recharts Implementation | EP-003 | 5 | P1 |
| S-016 | Chart Print Mode Verification | EP-003 | 2 | P1 |
| S-017 | Dashboard Print-All Routes | EP-003 | 2 | P1 |
| S-018 | Chart Angular Route Removal | EP-003 | 2 | P1 |
| S-019 | Scoring Unit Tests (WV / BAS / AVMR) | EP-004 | 5 | P0-gate |
| S-020 | SectionReportFilterBar Component | EP-004 | 3 | P1 |
| S-021 | SectionReport Pages (WV, BAS, AVMR) | EP-004 | 8 | P1 |
| S-022 | ObservationSummary Pages | EP-004 | 3 | P1 |
| S-023 | InterventionToolkit URL-State Audit | EP-004 | 3 | P1 |
| S-024 | SPIKE: PDF Export Feasibility | EP-004 | 1 | P2 |
| S-025 | Report Angular Route Removal | EP-004 | 2 | P1 |
| S-026 | TeamMeeting Routes | EP-005 | 3 | P2 |
| S-027 | Settings Routes Verification | EP-005 | 2 | P2 |
| S-028 | Admin User Management (CRUD) | EP-005 | 5 | P2 |
| S-029 | Assessment Result List Pages | EP-005 | 2 | P2 |
| S-030 | BatchPrint Utility Route | EP-005 | 1 | P2 |
| S-031 | Admin Angular Route Removal | EP-005 | 1 | P2 |
| S-032 | Visual Regression Baseline Capture | EP-006 | 2 | P0-gate |
| S-033 | Legacy Hash Route Redirects | EP-006 | 2 | P0 |
| S-034 | Angular Bundle Removal | EP-006 | 3 | P0 |
| S-035 | ESLint no-angularjs-controller Rule | EP-006 | 1 | P0 |
| S-036 | Final Route Audit | EP-006 | 2 | P0 |
| S-037 | Release Notes (3 Audiences) | EP-006 | 2 | P0-gate |
| S-038 | Cutover & Go-Live | EP-006 | 3 | P0 |

**Total:** 38 stories, ~118 points

---

## EP-001 Stories — Foundation: API + Shared Infrastructure

---

### S-001 — API CamelCase Migration

**Epic:** EP-001
**Points:** 3
**Priority:** P0

**Story:**
As a developer working on any NorthStarET React feature, I want the NS4 API to return camelCase JSON so that I can use TypeScript types without manual field-casing transforms.

**Acceptance Criteria:**

- [ ] `JsonNamingPolicy.CamelCase` added to `NS4.WebAPI/Program.cs` JSON options
- [ ] All `features/*/types.ts` updated to camelCase field names
- [ ] `NS4.Parity.Tests` suite runs 100% green against the updated API (AT-001)
- [ ] Pre-merge: `grep -ri "\.StudentId\|\.studentId\|\.BenchmarkDate\|\.benchmarkDate\|\.SectionId\|\.sectionId"` run across all Angular template files; findings documented in `docs/northstar/migration/api-camelcase-impact.md` (**W1 — both casing variants, per stress gate MIN-001**)
- [ ] `api-camelcase-impact.md` reviewed by architect before merge

**Dependencies:** None (root story)

---

### S-002 — useAutosave Hook

**Epic:** EP-001
**Points:** 3
**Priority:** P0

**Story:**
As a teacher entering scores, I want my work automatically saved so that I never lose data from accidental navigation.

**Acceptance Criteria:**

- [ ] `useAutosave(saveFn, { debounceMs: 1500, intervalMs: 30000 })` hook implemented in `features/shared/hooks/useAutosave.ts`
- [ ] Debounce fires after 1.5s of inactivity and calls `saveFn`
- [ ] Interval fallback fires every 30s regardless of activity
- [ ] `saveFn` is called with the current dirty state payload
- [ ] Hook exposes `{ status, lastSavedAt, save }` — `status` is `'idle' | 'saving' | 'saved' | 'error'`
- [ ] On unmount, pending debounce is cancelled (no memory leak)
- [ ] Unit tests cover all 4 status transitions (AT-002)

**Dependencies:** S-001 (camelCase API for `batchSave` response type)

---

### S-003 — AutosaveIndicator Component

**Epic:** EP-001
**Points:** 2
**Priority:** P0

**Story:**
As a teacher, I want to see the save status of my data entry so that I know my work is being preserved.

**Acceptance Criteria:**

- [ ] `<AutosaveIndicator status={status} lastSavedAt={lastSavedAt} />` renders correct copy for all 4 statuses: `idle` → no text, `saving` → "Saving…", `saved` → "Saved HH:MM", `error` → "Save failed — retry?"
- [ ] "Saved HH:MM" uses `new Date(lastSavedAt).toLocaleTimeString()` — local timezone, not UTC (**S2 — per stress gate MIN-002**)
- [ ] Component has `role="status"` and `aria-live="polite"` for screen reader parity
- [ ] axe scan passes (AT-003)
- [ ] Visual snapshot test for all 4 states

**Dependencies:** S-002

---

### S-004 — UnsavedChangesDialog Component

**Epic:** EP-001
**Points:** 2
**Priority:** P0

**Story:**
As a teacher navigating away from data entry, I want to be warned if I have unsaved changes so that I can decide whether to save or abandon my work.

**Acceptance Criteria:**

- [ ] `<UnsavedChangesDialog isOpen={isDirty} onSave={...} onDiscard={...} onCancel={...} />` implemented
- [ ] Dialog shows: "You have unsaved changes. Save before leaving?" with Save / Discard / Cancel buttons
- [ ] `onCancel` keeps user on the current page (does not navigate)
- [ ] axe scan passes
- [ ] Dialog is reusable across EP-002 (DataEntry) and EP-005 (Admin Settings) contexts

**Dependencies:** S-002 (dirty state from useAutosave)

---

### S-005 — SectionBenchmarkSelector Shared Component

**Epic:** EP-001
**Points:** 3
**Priority:** P0

**Story:**
As a teacher viewing reports or data entry, I want a consistent section + benchmark date selector so that all pages show matching student lists for my chosen benchmark period.

**Acceptance Criteria:**

- [ ] `<SectionBenchmarkSelector sectionId={...} benchmarkDateId={...} onChange={...} />` implemented
- [ ] Filter changes use `setSearchParams(updater, { replace: true })` — no new browser history entries (**S1 — per stress gate**)
- [ ] Selector reads initial state from URL search params on mount
- [ ] Filter changes do not add browser history entries (AT-004)
- [ ] Unit test: changing section doesn't push to history stack

**Dependencies:** S-001 (camelCase API for sections/benchmarks endpoints)

---

### S-006 — ChartContainer ARIA + StackedBarChart Print Audit

**Epic:** EP-001
**Points:** 2
**Priority:** P0

**Story:**
As a teacher printing a report, I want charts to render correctly on paper and be accessible to screen readers so that all printed documents and assistive technology users get correct output.

**Acceptance Criteria:**

- [ ] `<ChartContainer>` has `role="img"` and `aria-label` derived from chart title
- [ ] `<StackedBarChart>` calls `isPrintMode()` and sets `isAnimationActive={!isPrintMode()}` (AT-005)
- [ ] `<StackedBarChart>` renders fixed dimensions (width/height) when `isPrintMode()` returns `true`
- [ ] axe scan passes on `<ChartContainer>`
- [ ] Parent wrapper for all charts has `min-height: 200px` to prevent `ResponsiveContainer` infinite resize (**W5 — per stress gate MIN-008**)

**Dependencies:** None

---

### S-007 — QueryClient TTL Constants

**Epic:** EP-001
**Points:** 1
**Priority:** P0

**Story:**
As a developer, I want QueryClient stale-time constants defined per data category so that we avoid excessive API refetches on pages with frequently-accessed, slow-changing data.

**Acceptance Criteria:**

- [ ] `lib/queryClient.ts` exports TTL constants: `STUDENT_TTL`, `SECTION_TTL`, `BENCHMARK_TTL`, `REPORT_TTL` per TD-015
- [ ] All existing `use*.ts` hooks updated to reference these constants (AT-006)
- [ ] No hook uses `staleTime: 0` unless explicitly needed

**Dependencies:** None

---

## EP-002 Stories — SectionDataEntry Module

---

### S-008 — SectionDataEntry Autosave Integration

**Epic:** EP-002
**Points:** 5
**Priority:** P1

**Story:**
As a teacher entering scores, I want each `SectionDataEntry` page to auto-save my work so that I never lose score data from accidental navigation.

**Acceptance Criteria:**

- [ ] `useAutosave` (S-002) integrated on all `SectionDataEntry*` page variants
- [ ] Autosave fires within 1.5s of last keystroke; `<AutosaveIndicator>` shows "Saved HH:MM" on success (AT-010)
- [ ] Navigating away with unsaved changes shows `<UnsavedChangesDialog>` (AT-011)
- [ ] Partial saves are allowed — teachers can save incomplete rows and return later (**M2 — per stress gate MIN-004**)
- [ ] `BatchSave` endpoint called with only dirty rows (not full section payload on every debounce)
- [ ] Partial `BatchSave` failure (per-student error) surfaces as inline error on the failed student row
- [ ] `<AutosaveIndicator>` renders via EP-001 `<AutosaveIndicator>` component (EP-001 prerequisite enforced)

**Dependencies:** S-002, S-003, S-004 (EP-001 complete)

---

### S-009 — SectionDataEntry Parity Tests

**Epic:** EP-002
**Points:** 5
**Priority:** P1

**Story:**
As a developer removing Angular routes, I want parity tests for every SectionDataEntry page variant so that I can confirm zero visual or behavioral regression before Angular route removal.

**Acceptance Criteria:**

- [ ] Parity test file `src/parity/SectionDataEntry.parity.ts` covers all page variants: Rubric, Grades, Attendance, Notes, Roster (AT-012)
- [ ] Each parity test compares React page output vs. OldNorthStar baseline at `maxDiffPixelRatio: 0.01`
- [ ] Tests run in CI and are green before S-013 (Angular route removal) is started
- [ ] Parity test count verified ≥ 5 (one per section type)

**QuickDev Integration:** Run `/quickdev` with `prod-local-parity-v2` agent to generate parity evidence. Reports saved to `parity-reports/` directory.

**Dependencies:** S-008

---

### S-010 — SectionDataEntry Keyboard Navigation

**Epic:** EP-002
**Points:** 3
**Priority:** P1

**Story:**
As a teacher entering scores, I want full keyboard navigation in data entry grids so that I can work without using a mouse and match the keyboard behavior I'm used to from the old system.

**Acceptance Criteria:**

- [ ] Tab / Shift-Tab traverse columns left/right; Enter moves to next row
- [ ] Ctrl+S triggers immediate save (bypasses debounce)
- [ ] Focus ring on active cell: 3px solid `#0066CC` (NS4 design token — not browser default) (**S3 — per stress gate MIN-005**)
- [ ] Empty Grade cells show `—` placeholder (not blank) so teachers know cells are interactive (**S4 — per stress gate MIN-006**)
- [ ] Keyboard nav spec verified against legacy Angular `SectionDataEntry.js` behavior — zero behavioral regression
- [ ] axe scan passes on full data grid

**Dependencies:** S-008

---

### S-011 — SectionDataEntry Virtualization (Large Rosters)

**Epic:** EP-002
**Points:** 3
**Priority:** P1

**Story:**
As a teacher with a large class (50+ students), I want the data entry grid to scroll smoothly so that I can work without browser performance degradation.

**Acceptance Criteria:**

- [ ] `react-window` virtualization enabled when `students.length > 50`
- [ ] Column header row is always visible during scroll (sticky header — not part of the virtual row pool) (**W3 — per stress gate MIN-003**)
- [ ] First column (student names) is sticky and does not scroll horizontally
- [ ] Minimum supported viewport: 1024px; layout must not break below this (**S5 — per stress gate MIN-007**)
- [ ] Performance test: 100-student roster renders initial paint < 300ms

**Dependencies:** S-009

---

### S-012 — Close-Out Routes Verification

**Epic:** EP-002
**Points:** 2
**Priority:** P1

**Story:**
As a developer, I want to verify that StudentConsolidation, StaffConsolidation, and AssessmentSyndicate close-out routes are React-served before removing them from Angular.

**Acceptance Criteria:**

- [ ] Parity tests pass for StudentConsolidation, StaffConsolidation, AssessmentSyndicate page variants (AT-014)
- [ ] Each route loads the React page (not Angular fallback) with a known test dataset
- [ ] Route-verify matrix document updated for these 3 routes

**Dependencies:** None (independent verification story)

---

### S-013 — SectionDataEntry Angular Route Removal

**Epic:** EP-002
**Points:** 2
**Priority:** P1

**Story:**
As a developer completing the DataEntry migration, I want all SectionDataEntry Angular routes removed from `app.js` so that the React routes are the only active entry points.

**Acceptance Criteria:**

- [ ] S-009 (parity tests) green in CI before this story is started
- [ ] S-012 (close-out routes) verified before this story is started
- [ ] `grep -r "section-data-entry\|sectiondataentry" wwwroot/app/app.js` returns nothing after PR merge (AT-013)
- [ ] Strangler-fig custom DOM event: when React saves data, `northstar:sectionSaved` DOM event fires so Angular observers (if any are still live) can update (**W4 — per stress gate MIN-003**)
- [ ] Zero console errors after Angular route removal verified by Playwright smoke test

**Dependencies:** S-009, S-012, S-010, S-011

---

## EP-003 Stories — Shared Chart Primitives

---

### S-014 — Chart Primitives: LineGraph Recharts Implementation

**Epic:** EP-003
**Points:** 5
**Priority:** P1

**Story:**
As a teacher viewing student progress, I want line graph charts rendered by the React implementation so that I can track performance trends with the same visual fidelity as the previous system.

**Acceptance Criteria:**

- [ ] `MultiLineChart` and `LineChartWithBenchmarks` implemented using Recharts `<LineChart>`, `<Line>`, `<XAxis>`, `<YAxis>`
- [ ] Dual Y-axis support: `<YAxis yAxisId="left" />` and `<YAxis yAxisId="right" orientation="right" />` both available on `<LineChartWithBenchmarks>` (**M from stress gate MIN-010**)
- [ ] Custom tooltip via `renderCustomTooltip` render prop per architecture §6.3
- [ ] All line graph routes parity tested vs. OldNorthStar baseline (AT-020)
- [ ] `isAnimationActive={false}` when `students.length ≥ 30` or `isPrintMode()` (AT-025)
- [ ] All chart series colors sourced from NS4 design token constants — no Recharts defaults and no hardcoded hex (**S6 — per stress gate MIN-011**)
- [ ] `<AccessibleLegend>` component used for legend (keyboard-navigable, not Recharts default Legend)

**Dependencies:** S-006 (EP-001 chart audit)

---

### S-015 — Chart Primitives: StackedBarGraph Recharts Implementation

**Epic:** EP-003
**Points:** 5
**Priority:** P1

**Story:**
As a teacher viewing group comparison data, I want stacked bar charts rendered correctly in groups, comparison, and summary modes so that I can analyze performance across sections.

**Acceptance Criteria:**

- [ ] `<StackedBarChart>` supports three modes via `mode` prop: `'groups' | 'comparison' | 'summary'`
- [ ] All three modes parity tested vs. legacy `StackedBarGraphs.js` (2,813-line Angular controller) (AT-021)
- [ ] D3 custom operations replaced: `scaleBand()` → Recharts `<BarChart>`, `axisBottom()`/`axisLeft()` → `<XAxis>`/`<YAxis>`, `d3-tip` tooltip → `renderCustomTooltip` prop
- [ ] Chart series colors from design token constants (same as S-014)
- [ ] Print mode: `isAnimationActive={!isPrintMode()}`, fixed dimensions in print mode (AT-022)

**Dependencies:** S-006, S-014

---

### S-016 — Chart Print Mode Verification

**Epic:** EP-003
**Points:** 2
**Priority:** P1

**Story:**
As a teacher printing a batch report, I want all chart pages to render correctly in print mode so that printed pages look identical to the on-screen charts.

**Acceptance Criteria:**

- [ ] `?printmode=1` query param produces fixed-width/height chart rendering on all chart pages (AT-022)
- [ ] `print.css` updated: `.d3-chart` selectors replaced with `.recharts-wrapper` (**W6 — per stress gate MIN-009**)
- [ ] BatchPrint worker compatibility confirmed (print worker can render each chart page headlessly)
- [ ] Manual print QA: chart pages reviewed in browser print preview

**Dependencies:** S-014, S-015

---

### S-017 — Dashboard Print-All Routes

**Epic:** EP-003
**Points:** 2
**Priority:** P1

**Story:**
As a teacher printing all student dashboards, I want the `ig-dashboard-printall` and `student-dashboard-printall` routes to exist in the React router so that batch printing works without Angular.

**Acceptance Criteria:**

- [ ] `ig-dashboard-printall/:schoolYear` route added to `routes.tsx`
- [ ] `student-dashboard-printall/:id/:tab` route added to `routes.tsx`
- [ ] Both routes load without error with test data (AT-023)
- [ ] Parity test against OldNorthStar for both route variants

**Dependencies:** S-016

---

### S-018 — Chart Angular Route Removal

**Epic:** EP-003
**Points:** 2
**Priority:** P1

**Story:**
As a developer completing the Charts migration, I want all Angular chart routes removed from `app.js` so that React is the sole chart renderer.

**Acceptance Criteria:**

- [ ] S-009, S-014, S-015, S-016, S-017 all green before this story starts
- [ ] `grep -r "stackedbargraph\|linegraph\|ig-dashboard" wwwroot/app/app.js` returns nothing after PR merge (AT-024)
- [ ] Zero console errors in Playwright smoke test after removal

**Dependencies:** S-014, S-015, S-016, S-017

---

## EP-004 Stories — SectionReports Module

---

### S-019 — Scoring Unit Tests (WV / BAS / AVMR)

**Epic:** EP-004
**Points:** 5
**Priority:** P0-gate (must be merged before S-021)

**Story:**
As a developer migrating scoring logic, I want unit tests for all scoring algorithms against known-good fixtures so that I can confirm zero scoring regression before Angular report routes are removed.

**Acceptance Criteria:**

- [ ] `getWvBand()`: 20+ fixture inputs pass (AT-030)
- [ ] `getBasLevel()`: 20+ fixture inputs pass (AT-030)
- [ ] `getAvmrScore()`: 20+ fixture inputs pass (AT-030)
- [ ] Fixtures derived from OldNorthStar seeded test DB — production data approximation, not invented values
- [ ] Tests committed and green in CI before S-021 (SectionReport pages) story is started (AT-031 — gate)
- [ ] QA reviews fixtures for completeness (edge cases: boundary values, minimum/maximum scores)

**Dependencies:** S-001 (camelCase API types)

---

### S-020 — SectionReportFilterBar Component

**Epic:** EP-004
**Points:** 3
**Priority:** P1

**Story:**
As a teacher viewing any section report, I want a consistent section/benchmark/report-type filter bar so that I can navigate between different report views without losing my current context.

**Acceptance Criteria:**

- [ ] `<SectionReportFilterBar>` shared component implemented; used on all SectionReport pages
- [ ] Filter changes persist to URL via `setSearchParams` (consistent with S-005 pattern)
- [ ] axe scan passes (AT-033)
- [ ] Reports filtered by `<SectionReportFilterBar>` default to current semester date range on initial load (**M3 — per stress gate MIN-012**)
- [ ] Date range picker uses `react-datepicker` with NS4 style overrides (architecture §8.2)

**Dependencies:** S-005 (SectionBenchmarkSelector pattern), S-019

---

### S-021 — SectionReport Pages (WV, BAS, AVMR)

**Epic:** EP-004
**Points:** 8
**Priority:** P1

**Story:**
As a teacher, I want all SectionReport page variants (WV, BAS, AVMR structured detail) implemented in React so that scoring reports are available without Angular.

**Acceptance Criteria:**

- [ ] S-019 (scoring tests) green in CI before this story starts (AT-031 gate)
- [ ] `SectionReportWvPage`, `SectionReportBasPage`, `SectionReportAvmrPage`, `SectionReportAvmrStructuredDetailPage` all implemented
- [ ] Parity tested vs. OldNorthStar at `maxDiffPixelRatio: 0.01` (AT-032)
- [ ] Printed report header includes: school name, section name, teacher name, date range via `@page` margin CSS (**M4 — per stress gate MIN-013; compliance requirement**)
- [ ] Report layout order: summary card → visualization → data table (matches legacy NorthStar layout)
- [ ] Screen/on-page: `max-width: 1100px`; print: full-width
- [ ] **Scope gate:** Report export = browser `File → Print → Save as PDF` only. No programmatic PDF generation endpoint ships in this story (AT-037)

**Dependencies:** S-019, S-020

---

### S-022 — ObservationSummary Pages

**Epic:** EP-004
**Points:** 3
**Priority:** P1

**Story:**
As a teacher reviewing observation data, I want the ObservationSummary class and filtered route variants to work correctly in React so that I can review observation records without Angular.

**Acceptance Criteria:**

- [ ] `ObservationSummaryPage` handles both route variants: class and filtered (AT-034)
- [ ] Route params verified: `/observation-summary-class/:benchmarkDateId` param name matches React router definition
- [ ] Parity test for both variants vs. OldNorthStar baseline

**Dependencies:** S-020

---

### S-023 — InterventionToolkit URL-State Audit

**Epic:** EP-004
**Points:** 3
**Priority:** P1

**Story:**
As a teacher navigating the Intervention Toolkit, I want my tool state preserved in the URL so that browser back/forward navigation works correctly.

**Acceptance Criteria:**

- [ ] URL-state audit document created: all `$routeParams` from Angular `InterventionToolkitCtrl.js` mapped to `useSearchParams` equivalents
- [ ] All toolkit step/view state stored in URL search params (AT-035)
- [ ] Browser-back returns to the previous toolkit step without data loss
- [ ] Audit document committed to `docs/northstar/migration/intervention-toolkit-param-map.md`

**Dependencies:** S-001

---

### S-024 — SPIKE: PDF Export Feasibility

**Epic:** EP-004
**Points:** 1
**Priority:** P2

**Story:**
As a product owner, I want to know whether programmatic PDF generation (server-side) is required so that we can decide whether to add a "Download PDF" button in a future sprint.

**Acceptance Criteria:**

- [ ] Spike deliverable: one-page `docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/pdf-export-spike.md`
- [ ] Spike evaluates: (a) stakeholder demand for PDF button, (b) technical approach (`puppeteer` vs. `jsPDF` vs. browser print)
- [ ] Recommendation: yes/no programmatic PDF needed
- [ ] If yes: follow-on story created and added to EP-004 before epic close

**Dependencies:** None (parallel to S-021)

---

### S-025 — Report Angular Route Removal

**Epic:** EP-004
**Points:** 2
**Priority:** P1

**Story:**
As a developer completing the Reports migration, I want all SectionReport, ObservationSummary, and InterventionToolkit Angular routes removed from `app.js`.

**Acceptance Criteria:**

- [ ] S-019, S-021, S-022, S-023 all green and merged before this story starts
- [ ] `grep -r "section-report\|observation-summary\|intervention-toolkit" wwwroot/app/app.js` returns nothing after PR merge (AT-036)
- [ ] Visual regression baseline screenshots captured from Angular version before routes are removed (**S7 — per stress gate MIN-014**)
- [ ] React versions pass visual regression comparison at 95% similarity threshold (AT-040)

**Dependencies:** S-021, S-022, S-023

---

## EP-005 Stories — Admin & Settings Module

---

### S-026 — TeamMeeting Routes

**Epic:** EP-005
**Points:** 3
**Priority:** P2

**Story:**
As a teacher receiving a team meeting email invitation, I want the meeting links to open the correct React page so that email deep links continue to work after Angular removal.

**Acceptance Criteria:**

- [ ] TeamMeeting attend (`/tm-attend/:id`), notes (`/tm-attend-notes/:id`), email invite (`/tm-email-invite/:id`) routes served by React; parity tested (AT-040)
- [ ] Route param migration: Angular `:teamMeeting` (string) → React `:id` (number) documented; redirect or ID-coercion applied
- [ ] Acceptance: email invitation links from production team meeting notifications correctly load the React page

**Dependencies:** S-001

---

### S-027 — Settings Routes Verification

**Epic:** EP-005
**Points:** 2
**Priority:** P2

**Story:**
As a developer, I want all 8+ settings routes verified as React-served so that settings pages are removed from Angular without 404 risk.

**Acceptance Criteria:**

- [ ] Route-verify matrix created for: calendar, benchmarks, system-benchmarks, yearly-assessment-benchmarks, data-export, data-import, district-calendar (AT-041)
- [ ] All routes parity tested; breadcrumb navigation works correctly
- [ ] `<UnsavedChangesDialog>` (S-004) wired on settings pages with unsaved form state

**Dependencies:** S-004

---

### S-028 — Admin User Management (CRUD)

**Epic:** EP-005
**Points:** 5
**Priority:** P2

**Story:**
As a school admin, I want to create, view, edit, and deactivate user accounts so that I can manage teacher and staff access to NorthStarET.

**Acceptance Criteria:**

- [ ] User management table: list users, search by name, filter by role
- [ ] Create user: form with validation; calls `NS4 /api/v1/admin/users`
- [ ] Edit user: update name, email, role (not password — separate flow)
- [ ] Deactivate user: soft-delete only; button reads "Deactivate User" (**S from stress gate MIN-017**); confirmation modal states "This user will be hidden from all active sections"
- [ ] Bulk deactivate: checkbox selection + bulk "Deactivate Selected" action
- [ ] Permission gate: `<RequireRole role="school_admin" />` — non-school-admin user sees 403 page; district admin sees all schools (**W from stress gate MIN-015**)
- [ ] Admin actions (create, edit, deactivate) write to NS4 audit table via API (**M from stress gate MIN-016**)
- [ ] axe scan passes on user management table

**Dependencies:** S-001

---

### S-029 — Assessment Result List Pages

**Epic:** EP-005
**Points:** 2
**Priority:** P2

**Story:**
As a teacher, I want assessment result list pages (standard, KNTC, IG variants) to load correctly in React so that accessing these pages does not fall back to Angular.

**Acceptance Criteria:**

- [ ] Standard, KNTC, and IG assessment result list page variants parity tested (AT-042)
- [ ] Each variant loads without error with a seeded test dataset

**Dependencies:** S-001

---

### S-030 — BatchPrint Utility Route

**Epic:** EP-005
**Points:** 1
**Priority:** P2

**Story:**
As a teacher using batch printing, I want the `utility-batch-print` route to function in React so that bulk print jobs still work.

**Acceptance Criteria:**

- [ ] `utility-batch-print` route confirmed in `routes.tsx` and loads correctly
- [ ] `?batchprint=1` mode triggers print-safe chart rendering (AT-043)

**Dependencies:** S-016

---

### S-031 — Admin Angular Route Removal

**Epic:** EP-005
**Points:** 1
**Priority:** P2

**Story:**
As a developer completing the Admin migration, I want all admin and settings Angular routes removed from `app.js`.

**Acceptance Criteria:**

- [ ] S-026, S-027, S-028, S-029, S-030 all green and merged before this story starts
- [ ] `grep -r "tm-attend\|tm-email\|district-calendar\|system-benchmarks\|batch-print" wwwroot/app/app.js` returns nothing after PR merge (AT-044)

**Dependencies:** S-026, S-027, S-028, S-029, S-030

---

## EP-006 Stories — Angular Decommission & Cutover

---

### S-032 — Visual Regression Baseline Capture

**Epic:** EP-006
**Points:** 2
**Priority:** P0-gate (must happen before S-034)

**Story:**
As a developer, I want visual regression baseline screenshots of all 4 major views captured from the Angular version before removal so that the React cutover has a provable quality gate.

**Acceptance Criteria:**

- [ ] Baseline screenshots captured for: SectionDataEntry, LineGraph, StackedBarGraph, SectionReport
- [ ] Screenshots stored in `tests/regression/baselines/angular/`
- [ ] React equivalents pass `jest-image-snapshot` or `Chromatic` comparison at 95% similarity (AT-040, AT-058)
- [ ] Mobile Safari 14+ tested: all 4 views render without layout regression (**per stress gate MIN-022**)

**QuickDev Integration:** Use `/quickdev` workflow to run parity agents per page and capture baseline evidence. Reports in `parity-reports/` provide the audit trail for this gate.

**Dependencies:** All EP-002, EP-003, EP-004 route removal stories complete

---

### S-033 — Legacy Hash Route Redirects

**Epic:** EP-006
**Points:** 2
**Priority:** P0

**Story:**
As a teacher who has bookmarked old `#/` Angular routes, I want those bookmarks to redirect to the correct React pages so that no bookmark or shared link breaks after cutover.

**Acceptance Criteria:**

- [ ] Legacy hash routes `#/section-data`, `#/reports`, `#/admin`, `#/settings`, `#/section-data-entry` return 302 redirect to React equivalents (**per stress gate MIN-024; AT-057**)
- [ ] Redirect implemented via `window.location` in a catch-all hash route handler
- [ ] Playwright test: navigate to 5 legacy hash routes, confirm each lands on correct React page

**Dependencies:** S-031 (all Angular routes removed)

---

### S-034 — Angular Bundle Removal

**Epic:** EP-006
**Points:** 3
**Priority:** P0

**Story:**
As a developer completing the migration, I want the Angular.js bundle removed from the Vite build, npm packages, and wwwroot so that the production bundle no longer ships Angular code.

**Acceptance Criteria:**

- [ ] Pre-removal checklist from architecture §12.1 fully checked (AT-050)
- [ ] `angular*`, `d3` (v3), `jquery` removed from `package.json` and `node_modules`
- [ ] `npm run build`: no chunk in `dist/assets/` matching `/angular|d3|jquery/` (AT-051)
- [ ] `npm ls angular` returns empty (AT-052)
- [ ] Script tags for Angular/jQuery/D3 removed from `index.html` / Razor views
- [ ] `wwwroot/app/` directory removed
- [ ] React bundle gzipped size < 150kB (per epics.md gate)

**Dependencies:** S-032, S-033

---

### S-035 — ESLint no-angularjs-controller Rule

**Epic:** EP-006
**Points:** 1
**Priority:** P0

**Story:**
As a developer, I want an ESLint rule that fails on new Angular controller file creation so that no one accidentally re-introduces Angular code after decommission.

**Acceptance Criteria:**

- [ ] Custom ESLint rule `no-angularjs-controller` added to `.eslintrc` (AT-059)
- [ ] Rule pattern: any file containing `angular.module(` or `.controller(` causes CI lint failure
- [ ] Rule applied to `src/**` (not `wwwroot/app/**` — that directory is deleted)
- [ ] CI pipeline runs lint as a required gate

**Dependencies:** S-034

---

### S-036 — Final Route Audit

**Epic:** EP-006
**Points:** 2
**Priority:** P0

**Story:**
As a developer and product owner, I want a final Playwright audit confirming zero Angular in all live rendered pages so that we can sign off on the migration with confidence.

**Acceptance Criteria:**

- [ ] Playwright test suite loads all 50+ React routes
- [ ] Zero `ng-app`, `ng-controller`, `ng-model` attributes found in any rendered HTML (AT-053)
- [ ] Test report attached to PR description
- [ ] Parity test count at this gate ≥ parity test count at EP-001 gate (no `.skip()` regressions) (AT-060)
- [ ] Page load performance: LCP improvement ≥ 15% vs. pre-decommission baseline (AT-054)

**Dependencies:** S-034, S-035

---

### S-037 — Release Notes (3 Audiences)

**Epic:** EP-006
**Points:** 2
**Priority:** P0-gate (must be approved before S-038)

**Story:**
As a product owner, I want release notes prepared for teachers, school admins, and district admins so that every stakeholder group understands what changed before cutover.

**Acceptance Criteria:**

- [ ] Release notes document created at `docs/northstar/migration/feature/finish-northstaret-migration-3a10d3/release-notes.md`
- [ ] Teacher section: "What changed in your daily workflow" — autosave, updated keyboard shortcuts, new chart views
- [ ] School Admin section: "New admin panel walkthrough" — settings, user management, deactivation flow
- [ ] District Admin section: "Data access and compliance report changes" — SectionReports print, role changes
- [ ] Release notes reviewed and approved by product owner before S-038 (AT-062)

**Dependencies:** S-021, S-028

---

### S-038 — Cutover & Go-Live

**Epic:** EP-006
**Points:** 3
**Priority:** P0

**Story:**
As a product owner, I want to schedule and execute the Angular-to-React cutover so that all teachers begin using the React-only system at a safe, communicated time.

**Acceptance Criteria:**

- [ ] S-032, S-033, S-034, S-035, S-036, S-037 all complete before this story opens
- [ ] Cutover scheduled during a school break period OR outside school hours (before 7AM / after 6PM local time) (**W7 — per stress gate MIN-019; AT-056**)
- [ ] 48-hour teacher notification email sent before cutover date
- [ ] Feature flag `react_modules` flipped school-by-school using 3-school pilot before full rollout
- [ ] Rollback plan: feature flag reverts within same deploy window (<15 min); documented in runbook
- [ ] Zero 404s from Angular route hits in first 48h post-deploy monitoring window (AT-055)
- [ ] G1: `npm ls angular` returns nothing (confirmed post-deploy)
- [ ] G2: Playwright final route audit zero Angular in live HTML (confirmed post-deploy)
- [ ] `Cache-Control: no-cache` on `app.js` confirmed with ops

**Dependencies:** S-036, S-037

---

## Sprint Sequencing Recommendation

```
Sprint 1–2 (EP-001):
  S-001, S-002, S-003, S-004, S-005, S-006, S-007

Sprint 3–4 (EP-002 + EP-005 parallel start):
  S-008, S-009, S-010, S-011, S-012        [EP-002]
  S-026, S-027                              [EP-005 - parallel]

Sprint 5–6 (EP-002 close + EP-003 + EP-005):
  S-013                                     [EP-002 close]
  S-014, S-015, S-016, S-017               [EP-003]
  S-028, S-029, S-030, S-031               [EP-005 close]

Sprint 7–8 (EP-003 close + EP-004 start):
  S-018                                     [EP-003 close]
  S-019, S-020, S-024 (spike)              [EP-004 gate stories]

Sprint 9–11 (EP-004 main):
  S-021, S-022, S-023, S-025               [EP-004]

Sprint 12 (EP-006 setup):
  S-032, S-033, S-035, S-037               [EP-006 gates]

Sprint 13 (EP-006 final):
  S-034, S-036, S-038                      [EP-006 — cutover]
```

**Total:** ~13 sprints (2-week each) = ~26 weeks ≈ Q3 2026 target ✅

---

## QuickDev Workflow Integration

The `/quickdev` BMAD workflow enables rapid, agent-driven parity verification throughout the migration. It delegates to target-project agents (`visual-parityV2`, `prod-local-parity-v2`, `sprint-task-picker`) and captures structured parity reports back in the planning repo.

**Key touchpoints:**
- **S-009** (SectionDataEntry Parity Tests) — `/quickdev` generates parity evidence per SectionDataEntry variant
- **S-032** (Visual Regression Baseline Capture) — `/quickdev` captures baseline screenshots and comparison reports
- **All EP-002 through EP-005 stories** — any story with a parity gate can use `/quickdev` to generate the required evidence

**Parity reports directory:** `parity-reports/`
**Prompt:** `lens-work.quickdev.prompt.md`
