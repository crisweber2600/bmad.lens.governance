---
generated: true
phase: preplan
initiative: finish-northstaret-migration-3a10d3
doc_type: research-summary
timestamp: "2026-02-26T00:00:00Z"
sources:
  - docs/lens-sync/NorthStarET/migration-map.md
  - docs/lens-sync/NorthStarET/architecture.md
  - TargetProjects/northstar/migration/NorthStarET (git log --since 2026-02-02)
  - TargetProjects/northstar/migration/OldNorthStar/.referenceSrc
---

# Research Summary — Finish NorthStarET Migration

## 1. Current Migration State (as of 2026-02-26)

| Dimension | Current | Target | Notes |
|-----------|---------|--------|-------|
| React components | 251 | ~420 (est.) | Count of all React components in the repo (including shared and WIP); PRD baseline of 171 counts only end-user-facing screens. |
| Angular.js lines remaining | ~14,300 | 0 | Approx. Angular.js LOC in `app/src` for NorthStarET (excludes tests, admin, and reference sources); PRD baseline of 26,092 includes those additional modules. |
| Migration % | 45% | 100% | Derived from React vs. Angular LOC using the narrower ~14,300 LOC scope above. |
| Visual parity tests passing | 12 modules | All modules | Based on automated visual checks for NorthStarET user flows only. |
| Controllers migrated | 35 / 90 total | 35 (backend complete) | Backend-only count; aligns with the 90-controller scope defined in the PRD. |

## 2. Remaining Work Inventory

### High Complexity (Critical Path)

| Module | Legacy Lines | Complexity Driver | Estimated React Effort |
|--------|-------------|-------------------|----------------------|
| SectionReports.js | 4,500 | 4 report sub-types, print layout, date formatting | 8–12 weeks |
| StackedBarGraphs.js | 3,800 | D3.js stateful imperative rendering | 6–8 weeks |
| ObservationSummary.js | 3,200 | Complex state, nested forms | 5–7 weeks |
| SectionDataEntry.js | 3,000 | Nested forms, autosave, validation | 4–6 weeks |

### Medium Complexity

| Module | Legacy Lines | Complexity Driver | Estimated React Effort |
|--------|-------------|-------------------|----------------------|
| LineGraphs.js | 2,100 | D3.js, sharable with chart primitives | 3–4 weeks |
| Assessment full CRUD | ~930 | ~50% exists, API casing mismatch | 2–3 weeks |
| Student/Staff CRUD | ~1,200 | ~75–85% exists, close-out work | 1–2 weeks |
| InterventionToolkit.js | 1,100 | Multi-step workflow | 2–3 weeks |

### Low Complexity

| Module | Legacy Lines | Complexity Driver | Estimated React Effort |
|--------|-------------|-------------------|----------------------|
| SchoolDashboards.js | 980 | Dashboard, moderate data binding | 1–2 weeks |
| DataExport.js | 950 | File generation | 1 week |
| DistrictSettings.js | 980 | Admin screen, simple forms | 1 week |
| ImportStateTestData.js | 950 | File upload | 1 week |

**Total estimated remaining effort: 35–55 weeks (solo) → 18–28 weeks (2-person)**

## 3. Technology Assessment

### React 19 / Vite 7 Stack — Fit for Migration
- ✅ `@tanstack/react-query 5.90.12` — replaces Angular `$http` + `ngResource` patterns cleanly
- ✅ `react-hook-form 7.68.0` — replaces Angular form directives for SectionDataEntry
- ✅ `recharts 3.5.1` — modern, React-idiomatic replacement for D3.js (note: API is declarative vs. D3 imperative)
- ✅ `@tanstack/react-table` — replaces `ng-grid`
- ⚠️ No shared chart wrapper exists yet — must be built before StackedBar/LineGraph migrations
- ⚠️ `recharts` PDF/print output is limited — SectionReports print requirement needs investigation

### API Layer — Stable, One Known Issue
- `.NET 10` ASP.NET Core API is complete and stable
- **Known issue**: API response casing inconsistency (PascalCase vs camelCase) — documented via bug3240 history
  - Recommendation: Fix at API serialization level (`JsonSerializerOptions.PropertyNamingPolicy`) rather than per-component adapters
- Entity Framework 6.5.1 data layer is production-ready; no migration scope needed

### Test Infrastructure — Strong Foundation
- Playwright visual parity tests operational for 12+ migrated pages
- Screenshot comparison targets production (not localhost) — correct approach
- Existing test patterns can be extended to all new pages at low marginal cost

## 4. OldNorthStar Reference Analysis

The `.referenceSrc/OldNorthStar` directory within NorthStarET provides the canonical legacy implementation. Key findings:

- **NS4.Angular/** contains all Angular.js controllers (primary migration source)
- **NS4.WebAPI/** contains legacy API endpoints (already migrated to .NET 10)
- **NorthStar4_Framework46.sln** — legacy solution; Worker and Rollover services already migrated
- **LEGACY_DOCUMENTATION.md** in OldNorthStar repo — contains undocumented business rules for SectionReports calculation logic
  - ⚠️ **This must be read before migrating SectionReports** — complex scoring algorithms present

## 5. Velocity Analysis (Last 4 Weeks)

Based on git log since 2026-02-02, the team completed 12 pages in ~24 days:
- Average: **1 page per ~2 days** for medium-complexity pages
- Extrapolating to remaining 12 modules: **~24–48 days** for medium + low complexity
- High-complexity modules (Big 4) will take significantly longer — estimate 3–5x multiplier

**Realistic completion timeline: Q3 2026** (if current velocity maintained, no team additions)
**Aggressive completion timeline: End of Q2 2026** (requires 2 dedicated engineers, pre-built primitives)

## 6. Key Research Conclusions

1. **SectionDataEntry is the right first target** — blocks teacher workflow, React form architecture is well-suited, no chart dependencies
2. **Build a shared chart primitive layer before any chart page** — 5–10 days investment that saves ~15–20 days across StackedBar + LineGraph + SectionReports
3. **Read LEGACY_DOCUMENTATION.md before SectionReports** — undocumented scoring logic risk is HIGH
4. **Fix API casing at serialization level** — the per-component workarounds (seen in bug3240 fixes) are accumulating tech debt
5. **Q2 2026 deadline requires a second engineer** — current solo velocity won't reach 100% by Q2
