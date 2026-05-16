---
doc_type: epics
phase: devproposal
initiative: finish-northstaret-migration-3a10d3
status: draft
author: "John (PM) — DevProposal Phase"
timestamp: "2026-02-26T17:00:00Z"
input_artifacts:
  - prd.md
  - architecture.md
  - tech-decisions.md
  - ux-brief.md
epic_count: 6
---

# Epics — Finish NorthStarET Migration

**Author:** John (PM)
**Phase:** DevProposal
**Initiative:** finish-northstaret-migration-3a10d3
**Date:** 2026-02-26

---

## Epic Summary

| Epic | Title | Phase Arch | Audience | Sprints | Gate |
|------|-------|-----------|----------|---------|------|
| EP-001 | Foundation: API + Shared Infrastructure | Phase 1 | Small → Medium | 1–2 | Parity suite green |
| EP-002 | Data Entry Complete | Phase 2 | Medium | 2–3 | DataEntry parity PASS |
| EP-003 | Charts & Dashboards | Phase 3 | Medium | 2–3 | Chart parity PASS |
| EP-004 | Reports & Observations | Phase 4 | Medium | 3–5 | Scoring tests + parity PASS |
| EP-005 | Admin, Settings & Cleanup | Phase 5 | Medium | 1–2 | Settings parity PASS |
| EP-006 | Angular Bundle Decommission | Phase 5 (final) | Large | 1 | Zero angular bundle verified |

**Total estimated sprints:** 10–15 (2-week sprints)
**Target:** Q3 2026 zero-Angular release

---

## EP-001 — Foundation: API + Shared Infrastructure

**Priority:** P0 — All other epics depend on this
**Architecture reference:** §11 Phase 1, TD-005, TD-004, TD-001
**Estimated effort:** 1–2 sprints

### Goal

Establish the technical foundation that all subsequent migration work requires. This epic contains no Angular.js route removals — it is purely infrastructure. However, it unlocks every other epic.

### Done State

- `chore/api-camelcase` PR merged; `NS4.Parity.Tests` 100% green post-merge
- `useAutosave` hook, `<AutosaveIndicator>`, `<UnsavedChangesDialog>` implemented and tested
- `<SectionBenchmarkSelector>` implemented with `syncToUrl` and `replace: true` behavior (W1 from party review)
- `ChartContainer` has ARIA `role="img"` + `aria-label`; `StackedBarChart` confirmed print-mode safe
- `lib/queryClient.ts` TTL constants codified (TD-015)
- Zero new Angular routes removed in this epic (removal starts EP-002+)

### Problem Statements

1. **API casing mismatch:** All API responses return PascalCase; React code should use camelCase TypeScript types. Every new feature requires manual `camelcase-keys` transforms or inconsistent type interfaces. Fix: `JsonNamingPolicy.CamelCase` on the server + update all `features/*/types.ts`.
2. **No autosave primitives:** SectionDataEntry requires save-as-you-type behavior. No `useAutosave` hook, no `<AutosaveIndicator>`, no `<UnsavedChangesDialog>` exists yet. Fix: implement as shared hooks/components before EP-002.
3. **SectionBenchmarkSelector missing:** Multiple pages share section + benchmark date selection. No shared component exists; each page reimplements independently. Fix: shared `<SectionBenchmarkSelector>` with URL sync.
4. **Chart library fragmentation risk:** `chartUtils.ts` exists but `StackedBarChart` has not been verified for print-mode compliance. ARIA on `ChartContainer` is missing. Fix: audit and bring to spec before EP-003.
5. **QueryClient TTL uncodified:** Default `staleTime: 0` causes excessive refetches. Fix: codify per-category TTLs per TD-015.

### Functional Requirements (from PRD)

- FR: API returns camelCase for all controller responses
- FR: `<AutosaveIndicator>` displays idle/saving/saved/error states with `aria-live="polite"`
- FR: `<SectionBenchmarkSelector>` syncs selection to URL via `replace: true` `setSearchParams`
- FR: `ChartContainer` passes axe scan (`role="img"`, `aria-label`)

### Acceptance Criteria

- [ ] `NS4.Parity.Tests` suite passes 100% against the camelCase API (AT-001)
- [ ] `useAutosave` hook: debounce 1.5s, interval fallback 30s, batch via `batchSave` endpoint (AT-002)
- [ ] `<AutosaveIndicator>` renders correct copy for all 4 statuses; axe scan passes (AT-003)
- [ ] `<SectionBenchmarkSelector>` filter changes do not add browser history entries (AT-004)
- [ ] `<StackedBarChart>`: `isPrintMode()` called; renders fixed dimensions in print mode (AT-005)
- [ ] `queryClient.ts` exports TTL constants; all existing `use*.ts` hooks reference them (AT-006)

### Tracked Notes from Party Review

- **W1:** Before merging `chore/api-camelcase`, run `grep -r "\.StudentId\|\.BenchmarkDate\|\.SectionId"` across Angular template files. Document findings in `docs/northstar/migration/api-camelcase-impact.md`.
- **S1:** `SectionBenchmarkSelector` must use `setSearchParams(updater, { replace: true })`.

### Dependencies

- None — this is the root epic

---

## EP-002 — Data Entry Complete

**Priority:** P1 — Core teacher workflow; highest usage
**Architecture reference:** §11 Phase 2, TD-003, TD-004, §4.3
**Estimated effort:** 2–3 sprints
**Depends on:** EP-001 complete (autosave hooks + camelCase API)

### Goal

Wire all remaining Data Entry Angular routes to their React page equivalents, implement the autosave flow on all `SectionDataEntry*` pages, and confirm parity for every DataEntryController-backed route. This is the highest-traffic teacher workflow.

### Done State

- All `SectionDataEntry*` routes removed from `wwwroot/app/app.js`
- All removed routes have passing parity tests in `src/parity/SectionDataEntry.parity.ts`
- Autosave active on all SectionDataEntry pages: debounce 1.5s, 30s interval, `BatchSave` endpoint
- `InterventionGroupDataEntry` route confirmed React-served
- Remaining DataEntry close-out routes (StudentConsolidation, StaffConsolidation, AssessmentSyndicate) verified React-served
- All 4 PRs from architecture §11 Phase 2 merged

### Problem Statements

1. **SectionDataEntry pages lack autosave:** Teachers lose score entries on accidental navigation. React pages exist but use a "Save" button. Requires: `useAutosave` (from EP-001) + `<AutosaveIndicator>` integrated per page.
2. **SectionDataEntry parity not verified:** React `SectionDataEntry*` pages exist but no parity test confirms they match OldNorthStar visual output. Routes cannot be removed from Angular without this.
3. **Close-out routes unverified:** StudentConsolidation, StaffConsolidation, AssessmentSyndicate routes exist in Angular `app.js` — React equivalents must be confirmed working.

### Functional Requirements (from PRD)

- FR-2.1: Autosave on every score entry field; save visible in `<AutosaveIndicator>`
- FR-2.2: Unsaved changes guard when navigating away mid-entry
- FR-2.3: Partial failure handling — surface per-student error if BatchSave returns partial failure
- FR-2.4: All SectionDataEntry Angular routes removed from `app.js`

### Acceptance Criteria

- [ ] Autosave fires within 1.5s of last keystroke; indicator shows "Saved HH:MM" on success (AT-010)
- [ ] Navigating away with unsaved changes shows `<UnsavedChangesDialog>` (AT-011)
- [ ] Parity test passes for all SectionDataEntry* page variants vs. OldNorthStar baseline (AT-012)
- [ ] `grep -r "section-data-entry\|sectiondataentry" wwwroot/app/app.js` returns nothing after PR merge (AT-013)
- [ ] Close-out routes (StudentConsolidation, StaffConsolidation, AssessmentSyndicate) parity test passes (AT-014)
- [ ] 30-day regression metric: no teacher-reported data entry regressions in post-launch monitoring (M1 from party review — acceptance criterion, not gate)

### Dependencies

- EP-001 complete: `useAutosave`, `<AutosaveIndicator>`, `<UnsavedChangesDialog>`, camelCase API

---

## EP-003 — Charts & Dashboards

**Priority:** P1 — Core reporting; second-highest teacher interaction
**Architecture reference:** §11 Phase 3, TD-001, TD-002, §3
**Estimated effort:** 2–3 sprints
**Depends on:** EP-001 complete (chart primitive audit, print-mode verified)

### Goal

Migrate all Angular chart routes (LineGraphs, StackedBarGraphs) and dashboard routes (InterventionDashboard print variants, StudentDashboard print-all) to React. All pages must be print-mode safe and return visually identical output to OldNorthStar for the same data inputs.

### Done State

- All LineGraph Angular routes removed from `app.js` — React `StudentLineGraphPage`, `TrendLineGraphPage`, `InterventionGroupLineGraphPage`, `IGLineGraphPage` confirmed via parity tests
- All StackedBarGraph Angular routes removed — `StackedBarGraphGroupsPage`, `StackedBarGraphComparisonPage` with recharts full implementation
- `ig-dashboard-printall/:schoolYear` and `student-dashboard-printall/:id/:tab` routes added to React router and parity tested
- Print mode (`?printmode=1`) verified: all chart pages render fixed-dimension charts via `isPrintMode()`
- All 3 PRs from architecture §11 Phase 3 merged

### Problem Statements

1. **LineGraph pages exist but routes not confirmed removed:** React line graph pages are stubbed but the corresponding Angular routes may still exist in `app.js`. Parity test is the gate before removal.
2. **StackedBarGraph is a recharts rewrite:** `StackedBarChart.tsx` exists but the Angular `StackedBarGraphs.js` has 2,813 lines including groups mode, comparison mode, and summary mode. The React component may not cover all three modes. Requires thorough comparison and possible `mode` prop addition.
3. **Dashboard print-all routes missing from React router:** `ig-dashboard-printall/:schoolYear` and `student-dashboard-printall/:id/:tab` are not in `routes.tsx` (architecture §6.3). Need to be added.
4. **Print mode gap risk:** Only `LineChartWithBenchmarks` currently implements `isPrintMode()`. Other chart components must be verified before chart pages are declared print-safe.

### Functional Requirements (from PRD)

- FR-5.1: All line graph routes → React pages using `MultiLineChart` / `LineChartWithBenchmarks`
- FR-5.2: All stacked bar routes → React pages using `StackedBarChart` (groups, comparison, summary modes)
- FR-5.3: All chart pages must be BatchPrint-compatible (`?printmode=1` renders fixed-size charts)
- FR-5.4: `ig-dashboard-printall` and `student-dashboard-printall` routes added to React router

### Acceptance Criteria

- [ ] Parity tests pass for all LineGraph page variants (student, trend, IG, intervention group) (AT-020)
- [ ] StackedBarChart: groups mode, comparison mode, summary mode all parity tested (AT-021)
- [ ] `?printmode=1` produces fixed-width/height chart rendering on all chart pages; BatchPrint worker compatible (AT-022)
- [ ] `ig-dashboard-printall` and `student-dashboard-printall` routes exist in `routes.tsx` and load without error (AT-023)
- [ ] `grep -r "stackedbargraph\|linegraph\|ig-dashboard" wwwroot/app/app.js` returns nothing after PR merge (AT-024)
- [ ] Performance: MultiLineChart with ≥30 data points has `isAnimationActive={false}` (AT-025)

### Dependencies

- EP-001 complete: chart primitive ARIA audit, `StackedBarChart` print-mode verification

---

## EP-004 — Reports & Observations

**Priority:** P1 — High trust risk; scoring correctness is non-negotiable
**Architecture reference:** §11 Phase 4, TD-010, §10.2
**Estimated effort:** 3–5 sprints
**Depends on:** EP-001 complete (camelCase API); scoring unit tests must pass before any route removals

### Goal

Migrate all SectionReport, ObservationSummary, and InterventionToolkit Angular routes to React. The scoring logic (WV band scoring, BAS level determination, AVMR structured scoring) must be unit-tested against 20+ known-good production fixtures **before** any Angular report route is removed. This is the highest teacher-trust risk area in the migration.

### Done State

- `feat/section-reports-scoring-tests`: 20+ scoring fixtures per algorithm (WV, BAS, AVMR) green in CI
- All SectionReport route variants (WV, BAS, AVMR structured detail) removed from `app.js` — React pages parity tested
- `ObservationSummaryPage` (class and filtered variants) parity tested and routes removed
- `InterventionToolkitPage` URL-state audit complete; route removed from Angular
- `<SectionReportFilterBar>` shared component implemented and integrated across all report pages
- All 4 PRs from architecture §11 Phase 4 merged

### Problem Statements

1. **Scoring correctness — critical:** SectionReports contain WV band scoring, BAS level determination, and AVMR structured detail scoring. If the React implementation produces different scores than OldNorthStar, teachers will notice immediately and lose trust in the system. Screenshot parity tests alone cannot catch score-level bugs.
2. **`<SectionReportFilterBar>` missing:** All report pages share a section/benchmark/report-type filter pattern. Without a shared component, each page reimplements it independently with diverging behavior.
3. **ObservationSummary route param mismatch:** Architecture §6.3 flags that the `/observation-summary-class/:benchmarkDateId` Angular route may not match the React route param naming. Must be verified before Angular route removal.
4. **InterventionToolkit URL state:** Angular controller uses `$routeParams` for toolkit state. React must use `useSearchParams` equivalents — needs audit before route removal.

### Functional Requirements (from PRD)

- FR-4.1: All SectionReport routes → `SectionReport*Page` variants; scoring output must match OldNorthStar for 20+ fixture inputs
- FR-4.2: AVMR structured detail route handled by `SectionReportAvmrStructuredDetailPage`
- FR-4.3: `ObservationSummaryPage` handles both class and filtered route variants
- FR-4.4: `InterventionToolkitPage` uses URL state for step/view; no data loss on browser back
- FR-4.5: `<SectionReportFilterBar>` extracted as shared component

### Acceptance Criteria

- [ ] Scoring unit tests: `getWvBand()`, `getBasLevel()`, `getAvmrScore()` pass 20+ fixture inputs each (AT-030)
- [ ] Scoring tests committed and green in CI **before** any section report Angular route is removed (AT-031 — gate, not just acceptance)
- [ ] All SectionReport page variants parity tested vs. OldNorthStar at `maxDiffPixelRatio: 0.01` (AT-032)
- [ ] `<SectionReportFilterBar>` axe scan passes; used on all SectionReport pages (AT-033)
- [ ] ObservationSummary class + filtered variants parity tested; route params verified (AT-034)
- [ ] InterventionToolkit URL-state audit documented; all state in `useSearchParams`, browser-back safe (AT-035)
- [ ] `grep -r "section-report\|observation-summary\|intervention-toolkit" wwwroot/app/app.js` returns nothing after PR merge (AT-036)
- [ ] 30-day metric: zero teacher-reported scoring discrepancy tickets post-Phase 4 (M1 from party review)
- [ ] **[SCOPE GATE — Stress Gate EP-004]** Report export scope = browser print only (`File → Print → Save as PDF`); no programmatic PDF generation endpoint ships in this epic (AT-037)
- [ ] Reports default to current semester date range on load (not calendar year) (AT-038)
- [ ] Printed report header includes: school name, section name, teacher name, and date range via `@page` margin content (AT-039 — compliance)
- [ ] Angular-era report screenshots captured as visual regression baseline **before** Angular routes removed; React implementations pass at 95% similarity threshold (AT-040)

### Spike Story (added by Epic Stress Gate)

- **SPIKE: PDF Export Feasibility (1pt):** Evaluate whether programmatic PDF generation (server-side `puppeteer` or `jsPDF`) is required by a stakeholder. If yes, create a follow-on story before EP-004 epic close. Spike runs in parallel with print-ready stories — does not block EP-004. Deliverable: one-page spike summary with recommendation.

### Tracked Notes

- **W-arch:** `LEGACY_DOCUMENTATION.md` must be reviewed by Architect before Phase 4 sprint begins (PRD §10 dependency table)
- **Stress Gate EP-004:** PDF scope ambiguity resolved — browser print only confirmed. See spike story above.

### Dependencies

- EP-001 complete (camelCase API)
- `LEGACY_DOCUMENTATION.md` reviewed (see PRD dependency table)
- Scoring fixtures: derived from OldNorthStar seeded test DB output (requires QA coordination)

---

## EP-005 — Admin, Settings & Cleanup

**Priority:** P2 — Teacher-adjacent; lower daily-driver frequency than EP-002–004
**Architecture reference:** §11 Phase 5 (pre-decommission), §6.3 (admin routes)
**Estimated effort:** 1–2 sprints
**Depends on:** EP-001 complete; can run in parallel with EP-003/004

### Goal

Wire all remaining admin and settings routes: TeamMeetings (attend, notes, invite), district/school calendar management, benchmark settings, data export/import, and assessment result list pages. These are lower-frequency pages but must be fully React-served before Angular bundle removal.

### Done State

- TeamMeeting routes (`tm-attend`, `tm-attend-notes`, `tm-email-invite`) removed from `app.js`, React pages parity tested
- All settings routes (calendar, benchmarks, system benchmarks, yearly assessment benchmarks) confirmed React-served
- Data export/import routes confirmed React-served
- Assessment result list routes (section, section-KNTC, IG variants) confirmed React-served
- BatchPrint utility route (`utility-batch-print`) confirmed React-served
- All PRs from architecture §11 Phase 5 (admin section) merged

### Problem Statements

1. **TeamMeeting route params diverge:** Angular routes use `/:teamMeeting` (string); React routes use `/:id` (number). If not reconciled, deep links from teacher email invitations will break.
2. **Settings pages fragmented:** 8+ settings routes scattered across Angular controllers. Most React equivalents likely exist but need route-in-`routes.tsx` verification — not all may be wired.
3. **Assessment result list pages:** Three variants (standard, KNTC, IG) — likely the simplest of the remaining pages but must each have parity verification.

### Functional Requirements (from PRD)

- FR-8.1: TeamMeeting attend, notes, and invite routes served by React; param migration documented
- FR-9.6: All district/school admin routes (calendar, benchmarks, data export, import) → settings/* pages
- FR-6.x: BatchPrint page functional via `utility-batch-print` route

### Acceptance Criteria

- [ ] TeamMeeting deep links from email invitations work with React route params (AT-040)
- [ ] All settings routes parity tested; breadcrumb navigation works correctly (AT-041)
- [ ] Assessment result list variants (standard, KNTC, IG) parity tested (AT-042)
- [ ] BatchPrint page: triggers `?batchprint=1` mode; charts render print-safe (AT-043)
- [ ] `grep -r "tm-attend\|tm-email\|district-calendar\|system-benchmarks\|batch-print" wwwroot/app/app.js` returns nothing after PR merge (AT-044)

### Dependencies

- EP-001 complete (camelCase API)
- Can be parallelised with EP-003 and EP-004

---

## EP-006 — Angular Bundle Decommission

**Priority:** P0 (final gate) — Cannot happen until EP-002 through EP-005 all complete
**Architecture reference:** §12, FR-10.1, FR-10.4
**Estimated effort:** 1 sprint
**Depends on:** EP-002, EP-003, EP-004, EP-005 all complete (all Angular routes removed)

### Goal

Remove the Angular.js bundle (angular.js, angular-route.js, angular-bootstrap.js, D3 v3, jQuery 1.x) from the Vite build, npm packages, and wwwroot. Confirm zero Angular in the production bundle. This is the finish line.

### Done State

- `npm run build` produces no chunk matching `/angular|d3|jquery/`
- `package.json` has no `angular*` dependencies
- `wwwroot/app/` directory removed (controllers, services, app.js)
- Script tags for Angular/jQuery/D3 removed from index.html / Razor views
- Final route audit: `playwright --grep "ng-app|ng-controller"` returns zero matches across all live pages
- `Cache-Control: no-cache` on `app.js` (handled by deploy config; verify with ops)
- G1 and G2 PRD measurable goals confirmed met

### Problem Statements

1. **Bundle contamination:** Until Angular is explicitly removed from Vite config and `package.json`, the Angular bundle continues to ship on every deploy, adding ~400KB to initial load for every teacher.
2. **Browser cache risk:** Teachers may have `app.js` cached. After Angular removal, any cached Angular route hit would 404. Mitigation: `Cache-Control: no-cache` on `app.js`; service worker cache bust on deploy.

### Functional Requirements (from PRD)

- FR-10.1: Remove `angular.js`, `angular-route.js`, `angular-bootstrap.js`, D3 v3, jQuery 1.x from Vite config
- FR-10.4: Final route audit confirms zero `ng-app` / `ng-controller` in rendered HTML
- G1: `npm ls angular` returns nothing
- G2: Route audit returns zero Angular in live HTML

### Acceptance Criteria

- [ ] Pre-removal checklist from architecture §12.1 fully checked (AT-050)
- [ ] `npm run build`: no chunk in `dist/assets/` matching `/angular|d3|jquery/` (AT-051)
- [ ] `npm ls angular` returns empty (AT-052)
- [ ] Playwright final route audit: all 50+ routes loaded, zero `ng-app`/`ng-controller` found (AT-053)
- [ ] Page load performance: LCP improvement ≥ 15% vs. pre-decommission baseline on sampled pages (AT-054)
- [ ] Zero 404s from Angular route hits in first 48h post-deploy monitoring window (AT-055)
- [ ] **[Stress Gate EP-006]** Cutover is scheduled during a school break period OR outside school hours (before 7AM / after 6PM local time); 48-hour teacher notification email sent before cutover (AT-056 — ops gate)
- [ ] Legacy Angular hash routes (`#/section-data`, `#/reports`, `#/admin`) return `302` redirect to React equivalents — no 404s (AT-057)
- [ ] Visual regression baseline screenshots captured from Angular version for all 4 major views; React equivalents pass at 95% similarity threshold (AT-058)
- [ ] ESLint rule `no-angularjs-controller` added to `.eslintrc` — CI fails on any new Angular controller file creation (AT-059)
- [ ] Parity test count at EP-006 gate ≥ parity test count at EP-001 gate (no tests deleted or `.skip()`-ed) (AT-060)
- [ ] Mobile Safari 14+ tested across all 4 major views with zero layout regressions (AT-061)
- [ ] Release notes approved by product owner for all 3 audiences: Teachers, School Admins, District Admins (AT-062)

### Dependencies

- EP-002 complete (verified): all DataEntry Angular routes removed
- EP-003 complete (verified): all Chart Angular routes removed
- EP-004 complete (verified): all Report Angular routes removed
- EP-005 complete (verified): all Admin Angular routes removed
- Pre-removal checklist signed off by Architect (W2 from party review: all route-verify stories must show ✅ before this epic begins)
- Cutover timing approved by school calendar (ops)
- Release notes approved (product owner)

---

## Epic Dependency Map

```
EP-001 (Foundation)
  ├── EP-002 (Data Entry)     ←─ main teacher workflow
  ├── EP-003 (Charts)         ←─ can start after EP-001
  ├── EP-004 (Reports)        ←─ scoring tests gate (can start after EP-001, gate before removal)
  └── EP-005 (Admin)          ←─ can run in parallel with EP-003/004
        │
        ▼ (all 4 complete)
     EP-006 (Decommission)    ←─ final epic, 1 sprint
```

**Parallelisation:** EP-002, EP-003, EP-004, EP-005 can all be worked in parallel sprints after EP-001 is done. EP-006 requires all four to be fully complete.

---

## Scope Exclusions (carried from PRD)

- No changes to .NET backend (except Phase 1 `JsonNamingPolicy.CamelCase` config)
- No new API endpoints
- No mobile-native app changes
- No Redux / state management library introduction
- No migration to `createBrowserRouter` (hash routing stays until post-migration)
- OldNorthStar read-only: used as parity baseline only, not modified

---

## Party Review Notes Carried Into Stories

These must appear as acceptance criteria or notes on individual stories generated in step [3]:

| Note | Source | Epic | Story target |
|------|--------|------|-------------|
| W1: camelCase grep searches both `StudentId` AND `studentId` casing variants | Winston | EP-001 | S-001: API camelCase migration |
| W2: route-verify story per phase before Angular removal | Winston | EP-002–005 | Per-phase verification stories |
| W3: sticky column header row not virtualized (always visible) | Winston | EP-002 | SectionDataEntry virtualization story |
| W4: DOM event protocol `northstar:sectionSaved` confirmed — no `$rootScope.$apply` coupling | Winston | EP-002 | SectionDataEntry strangler-fig story |
| W5: chart container parent must have `min-height: 200px` to prevent infinite resize loops | Winston | EP-003 | Chart primitives story |
| W6: print.css must replace `.d3-chart` selectors with `.recharts-wrapper` | Winston | EP-003 | Chart print-mode story |
| W7: cutover scheduled during school break or off-hours; 48h teacher notification | Winston | EP-006 | Cutover story |
| W8: ESLint `no-angularjs-controller` rule added before EP-006 scope closes | Winston | EP-006 | Lint rule story |
| M1: 30-day regression metric in Phase 2+4 | Mary | EP-002, EP-004 | Autosave + SectionReports stories |
| M2: partial saves allowed — teachers can save incomplete rows | Mary | EP-002 | SectionDataEntry autosave story |
| M3: reports default to current semester date range (not calendar year) | Mary | EP-004 | SectionReports filter story |
| M4: print header must include school name, section name, teacher name, date range | Mary | EP-004 | SectionReports print story |
| M5: release notes for 3 audiences (Teachers, School Admins, District Admins) | Mary | EP-006 | Release notes story |
| S1: `replace: true` in SectionBenchmarkSelector | Sally | EP-001 | S-SectionBenchmarkSelector |
| S2: AutosaveIndicator timestamp uses `toLocaleTimeString()` (local timezone, not UTC) | Sally | EP-001 | S-AutosaveIndicator |
| S3: focus ring 3px solid `#0066CC` on SectionDataEntry cells | Sally | EP-002 | SectionDataEntry keyboard nav story |
| S4: empty Grade cells show `—` placeholder (not blank) | Sally | EP-002 | SectionDataEntry cell render story |
| S5: min viewport 1024px (matches legacy NorthStar) | Sally | EP-002 | SectionDataEntry layout story |
| S6: chart series colors use design token constants — no Recharts defaults | Sally | EP-003 | Chart primitives story |
| S7: visual regression baseline from Angular before any cutover | Sally | EP-006 | Visual regression story |
| S8: legacy hash routes redirect 302 to React routes | Sally | EP-006 | Route redirect story |
