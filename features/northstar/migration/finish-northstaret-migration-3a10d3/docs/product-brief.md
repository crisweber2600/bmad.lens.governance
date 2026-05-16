---
generated: true
phase: preplan
initiative: finish-northstaret-migration-3a10d3
doc_type: product-brief
timestamp: "2026-02-26T00:00:00Z"
author: "Mary (Analyst) — PrePlan Phase"
status: draft
---

# Product Brief — Finish NorthStarET Migration

## Executive Summary

Complete the migration of NorthStarET's Angular.js 1.4.5 front-end to React 19, eliminating all legacy Angular dependencies by end of Q2–Q3 2026. The remaining 55% of the migration is concentrated in four high-complexity modules that require a structured, risk-managed approach.

---

## 1. Problem Statement

NorthStarET's Angular.js 1.4.5 front-end creates three compounding problems:

**1. Maintenance burden.** Angular.js reached End-of-Life in December 2021. Security vulnerabilities are no longer patched upstream. The framework is also difficult to onboard new engineers onto.

**2. User experience debt.** The legacy Angular UI lacks autosave, URL-shareable state, mobile responsiveness, and consistent empty/error states — all of which teachers and administrators need for daily workflow.

**3. Developer velocity drag.** Every new feature must be implemented twice (Angular + React) or deferred until migration is complete. The longer migration runs, the larger this overhead grows.

The 45% already migrated demonstrates the approach is sound. The remaining 55% is harder, not fundamentally different.

---

## 2. Vision

A fully React 19 NorthStarET front-end with:
- Zero Angular.js dependencies in the production bundle
- All pages passing automated visual parity tests against production
- Improved UX across the migrated modules (autosave, URL state, mobile, print)
- Clean TypeScript types auto-generated from .NET API contracts
- No regression in any currently-passing feature

---

## 3. Target Users

| User | Role | Migration Impact |
|------|------|-----------------|
| Teachers | Daily data entry (SectionDataEntry, DataEntryForms) | **High** — blocked by legacy daily |
| Literacy Coaches | Section reports, intervention tracking | **High** — SectionReports is primary tool |
| Administrators | Rollover, Staff management, District settings | Medium — rollover already migrated |
| Developers | Engineering team | High — maintenance burden, new feature pipeline |

---

## 4. Scope

### In Scope

| Module | Why |
|--------|-----|
| SectionDataEntry.js (3,000 lines) | Priority #1 — daily teacher workflow blocker |
| Shared chart primitive layer | Enables LineGraph + StackedBar at low marginal cost |
| LineGraphs.js (2,100 lines) | Progress monitoring — medium complexity |
| StackedBarGraphs.js (3,800 lines) | Assessment visualization — high complexity |
| SectionReports.js (4,500 lines) | Core reporting tool — decompose into 4 sub-components |
| ObservationSummary.js (3,200 lines) | Complete after SectionReports (shares state patterns) |
| InterventionToolkit.js (1,100 lines) | Bounded scope, high user value |
| Close out Student/Staff/Assessment CRUD | ~75–85% done — finish the in-progress work |
| Admin pages (SchoolDashboards, DataExport, DistrictSettings, Import) | Lower complexity, clean up last angular routes |
| Fix API response casing at serialization level | Eliminates class of bugs across all pages |

### Out of Scope

| Item | Reason |
|------|--------|
| Backend changes | .NET 10 API is complete and stable |
| EF6 → EF Core migration | Separate initiative; no dependency on frontend migration  |
| New features not in OldNorthStar | Scope creep risk; defer to post-migration backlog |
| SSR / Server-Side Rendering | Desirable but additive; not required for migration completion |
| i18n / localization | Not present in OldNorthStar; defer |

---

## 5. Success Criteria

| Criterion | Measurement |
|-----------|-------------|
| Zero Angular.js in production bundle | `npm ls angular` returns nothing; `angular.js` removed from Vite config |
| All routes return React components | Route audit: zero `ng-app` / `ng-controller` in live HTML |
| Visual parity tests pass | All Playwright parity tests pass in CI against production screenshots |
| No regression on currently passing tests | CI green on `NS4.WebAPI.Tests/` and `NS4.Parity.Tests/` |
| SectionDataEntry autosave works | Manual QA checklist: data persists across browser refresh mid-session |
| API casing consistent | Zero `toLowerCase()` / `toUpperCase()` workarounds new-added after migration |

---

## 6. Delivery Phases (Proposed)

### Phase 1: Foundation (Weeks 1–3)
- Fix API response casing at serialization level (`JsonSerializerOptions.PropertyNamingPolicy = CamelCase`)
- Build shared React chart primitive library (recharts wrapper, responsive container, tooltip standard)
- Audit LEGACY_DOCUMENTATION.md for SectionReports scoring logic
- Extend Playwright parity test harness to cover new modules

### Phase 2: High-Value Data Entry (Weeks 4–9)
- Migrate SectionDataEntry.js → React (autosave, validation, URL state)
- Complete Student Management CRUD (80% → 100%)
- Complete Staff Management CRUD (70% → 100%)
- Complete Assessment CRUD (50% → 100%)

### Phase 3: Charts (Weeks 10–17)
- Migrate LineGraphs.js using shared chart primitives
- Migrate StackedBarGraphs.js using shared chart primitives
- Visual parity tests for both

### Phase 4: Reports & Observation (Weeks 18–28)
- Decompose SectionReports.js into 4 sub-components (WV, BAS, FP, HFW)
- Migrate ObservationSummary.js
- Print layout testing for all report pages

### Phase 5: Cleanup & Cutover (Weeks 29–32)
- Migrate InterventionToolkit, SchoolDashboards, DataExport, DistrictSettings, Import
- Remove Angular.js bundle from Vite config
- Final route audit; remove all `ng-` directives
- Declare migration complete; deprecate `.referenceSrc` reference

---

## 7. Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| SectionReports scope underestimate | High | High | Decompose first, estimate per sub-component; pair program |
| D3→recharts is a rewrite not a port | High | Medium | Build chart primitives before migrating; budget 3x vs. simple port |
| Undocumented scoring logic in SectionReports | Medium | High | Read LEGACY_DOCUMENTATION.md; write unit tests before migration |
| API casing bugs cause test failures | Medium | Medium | Fix at serialization level in Phase 1 |
| Q2 deadline requires extra capacity | High | High | Identify second engineer now; plan 6-week critical path |
| Browser-cached Angular routes after cutover | Low | High | Set cache-busting headers; service worker invalidation |

---

## 8. Dependencies

| Dependency | Owner | Status |
|------------|-------|--------|
| .NET 10 API stable | Backend | ✅ Complete |
| Playwright parity test framework | QA / DevOps | ✅ Operational |
| NorthStarET.AppHost (Aspire) | Platform | ✅ Operational |
| Shared chart primitive library | Frontend engineer | ⏳ Must build (Phase 1) |
| LEGACY_DOCUMENTATION.md review | Architect | ⏳ Must complete before Phase 4 |
| Second engineer (optional but recommended) | Management | ❓ Decision needed |

---

## 9. Related Documents

- [Brainstorm Notes](./brainstorm-notes.md)
- [Research Summary](./research-summary.md)
- [Architecture](../../../../../docs/lens-sync/NorthStarET/architecture.md)
- [Migration Map](../../../../../docs/lens-sync/NorthStarET/migration-map.md)
- [OldNorthStar Architecture](../../../../../docs/lens-sync/OldNorthStar/architecture.md)
