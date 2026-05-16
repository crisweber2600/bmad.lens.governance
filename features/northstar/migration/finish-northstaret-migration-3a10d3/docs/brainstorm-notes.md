---
stepsCompleted: [1, 2, 3, 4, 5]
initiative: finish-northstaret-migration-3a10d3
phase: preplan
audience: small
session_topic: "How do we finish the NorthStarET migration from Angular.js → React 19?"
session_goals:
  - Ordered list of features/pages to migrate next
  - Risk-mitigated delivery plan
  - Team capacity / sprint breakdown
  - Feed into Product Brief
selected_approach: AI-Recommended + Progressive Flow
techniques_used:
  - Brain Dump (divergent ideation)
  - Constraint Inversion
  - Risk Reversal
  - Velocity-First Prioritization
  - Anti-Bias Domain Rotation
ideas_generated: 67
generated_at: "2026-02-26T00:00:00Z"
generated_by: "Carson (Brainstorming Coach) + Mary (Analyst)"
---

# Brainstorming Session — Finish NorthStarET Migration

**Topic:** How do we prioritize, sequence, and de-risk the remaining 55% of the NorthStarET Angular.js → React 19 migration?

**Current State Snapshot:**
- Migration progress: ~45% complete (251 TSX components, 35 controllers migrated)
- Remaining Angular.js: ~14,300 lines across 12 modules
- Biggest pending items: SectionReports (4,500 lines), StackedBarGraphs (3,800), ObservationSummary (3,200), SectionDataEntry (3,000)

---

## 🧠 BRAIN DUMP — 67 Raw Ideas

*Anti-bias protocol applied: rotated domains every ~10 ideas*

---

### Domain: Sequencing & Prioritization (Ideas 1–12)

1. Prioritize by **user-facing impact** — SectionDataEntry first because it blocks teacher daily workflow
2. Prioritize by **code complexity inverse** — tackle small wins (LineGraphs 2,100 lines) before monsters (SectionReports 4,500)
3. **Freeze the Angular.js modules** — stop bug fixes in legacy code, redirect effort to React
4. **Parallel workstreams**: one team does SectionReports decomposition while another finishes Student/Staff CRUD
5. Build a **dependency graph** of which React components unblock which others before writing a single line
6. Tackle **SectionReports as micro-services** — split into 4 separate React components (WV, BAS, FP, HFW) and migrate one per sprint
7. **Kill ObservationSummary first** — remove it from scope if it has <5% usage
8. **Mirror the production URL structure** in React router from day 1 — avoids routing seams during transition period
9. Use **feature flags** so React and Angular components can run side-by-side per route
10. Migrate **chart library first** (D3 → recharts) as a shared primitive, then StackedBar and LineGraph cost nothing
11. **Throttle migration rate** to no more than 3 new React pages per sprint to avoid QA bottleneck
12. Write a **migration script** that auto-generates React component stubs from Angular controller signatures

---

### Domain: Risk & Quality (Ideas 13–25)

13. Every new React page requires a **passing visual parity test before merge** — no exceptions
14. Set up **Playwright screenshot diffing** against production not localhost — localhost has data variance
15. Create a **migration acceptance checklist** per page: routing, API calls, auth guards, error states, empty states, mobile breakpoints
16. **Shadow mode deployment** — run React and Angular in parallel, log discrepancies before cutover
17. Add **contract tests** between React frontend and .NET API controllers to catch API shape changes early
18. Risk: SectionReports.js has 4,500 lines — **complexity estimate is likely wrong**, budget 3x
19. Risk: D3.js chart logic is **stateful and imperative** — recharts idioms are different, expect rewrites not ports
20. Risk: Angular `$scope` watchers in Assessment.js likely hide **subtle timing dependencies** not obvious in porting
21. Risk: `ng-grid` → `@tanstack/react-table` is an **API philosophy change**, not just a swap
22. Create a **"migration debt register"** — track every Angular pattern used in remaining files to pre-identify surprises
23. **Canary deploy each migrated page** to 5% of production traffic before full cutover
24. Add **Sentry error tracking** per migrated page — React errors will surface differently than Angular errors
25. Write an **E2E smoke test suite** that exercises the critical path (login → section → data entry → save) on every deploy

---

### Domain: Technical Architecture (Ideas 26–38)

26. Extract **SectionReports into a standalone React micro-frontend** — load it lazy, decouple from main bundle
27. The `.referenceSrc/` folder in NorthStarET repo is gold — use it as the **canonical spec** for every migration, not the running Angular app
28. Build a **shared React context** for school/district/section state — currently each Angular controller re-fetches it independently
29. Replace **Angular `$http` calls** with a centralized Axios interceptor that matches the Angular `$httpProvider` behavior during transition
30. Create a **React component library** (`/components/shared`) for buttons, tables, modals — migrate once, use everywhere
31. Use **React Suspense boundaries** at the page level to get built-in loading states — eliminates Angular's `ng-loading` pattern
32. The current API returns **PascalCase** in some endpoints and **camelCase** in others (bug3240 history) — fix at API level, not per-component
33. **SSR (Server-Side Rendering)** opportunity — Aspire + .NET 10 could serve initial HTML, dramatically improve perceived load time
34. Implement **optimistic UI updates** in data entry forms — current Angular version blocks the UI during saves
35. Replace Angular `$routeProvider` with React Router `<Outlet>` nesting — model the nested layout once, reuse everywhere
36. **Web Workers** for SectionReports PDF generation — offload heavy computation from the main thread
37. Consider **tRPC** or **OpenAPI codegen** to auto-generate TypeScript types from .NET controller signatures
38. The Entity Framework + .NET API layer is **stable and complete** — zero need to touch backend during frontend migration

---

### Domain: User Experience & Product (Ideas 39–50)

39. **Don't port SectionReports — reimagine it.** The Angular version has 4,500 lines because it's unmaintainable. React version should be <800 lines.
40. The current Angular data entry forms have **no autosave** — add autosave as a React migration improvement
41. Visual parity tests expose a truth: **production has inconsistent CSS** — standardize in React, document the decision
42. Add **keyboard navigation** to all migrated pages — the Angular version has known a11y gaps
43. **URL-shareable filter state** — teachers frequently share report URLs, the Angular version drops filter params on reload
44. Consolidate 3 separate "Rollover" UX flows into one **unified Rollover wizard** in React
45. **Print stylesheets** — SectionReports are printed weekly; React migration is the chance to fix print layout
46. Add **loading skeletons** instead of spinners — reduces perceived latency on data-heavy pages
47. **Mobile responsiveness** — the Angular version degrades on tablets; bake in responsive design during migration
48. "Empty state" design — most Angular pages show nothing when no data; React version should show helpful CTAs
49. **Breadcrumb navigation** — migrated pages inconsistently implement breadcrumbs; standardize with React Router
50. **Contextual help tooltips** on complex data entry fields — one-time cost during migration, high teacher value

---

### Domain: Delivery & Process (Ideas 51–62)

51. Define **"done" explicitly** — migration is complete when: (a) all routes return React, (b) Angular bundle removed from build, (c) no `ng-` prefixed directives in codebase
52. Create a **public migration dashboard** visible to stakeholders — real-time progress tracker tied to GitHub
53. Run a **"migration hackathon"** sprint — 2 days, whole team, knock out all medium-complexity pages
54. **Pair programming** on SectionReports — it's too risky for solo development
55. Set a **hard cutover date** (e.g., Q2 2026) and work backward — forces prioritization
56. Define **feature-freeze policy** on Angular.js modules: no new features after migration sprint begins
57. Use **GitHub Projects board** to track Angular component → React component migration status
58. **Monthly demo to stakeholders** of newly migrated pages — builds confidence and surfaces feedback early
59. Write **ADRs (Architecture Decision Records)** for every major technical choice during migration
60. **Hire or contract** a React specialist specifically for SectionReports and charting — highest-risk modules
61. Define **rollback procedure** for each page cutover — if React page breaks in production, how quickly can we revert?
62. Treat each migrated page as a **mini-release** with its own changelog entry

---

### Domain: Black Swan / Edge Cases (Ideas 63–67)

63. What if SectionReports.js has **undocumented business logic** that exists nowhere else in the codebase? Audit first.
64. What if the Angular `ng-grid` in Assessment pages is **rendering server-side paginated data** in a way that breaks with `@tanstack/react-table`'s assumptions?
65. What if **users have browser-cached Angular routes** and get 404s after React cutover? Need cache headers + service worker strategy.
66. What if the OldNorthStar `.referenceSrc` reference implementation diverges from what's actually running in production? Validate against live screenshots.
67. What if migration completion reveals **hidden EF6 N+1 query problems** that only appear at scale? Plan a load test sprint before final cutover.

---

## 🔍 Cluster Analysis

After 67 ideas, three clear themes emerge:

### Theme A: The Big 4 Are the Real Migration
The remaining 55% is dominated by 4 modules:
- SectionReports (4,500 lines) — decompose into 4 sub-components
- StackedBarGraphs (3,800 lines) — recharts rewrite, not port
- ObservationSummary (3,200 lines) — audit for usage before committing
- SectionDataEntry (3,000 lines) — priority #1, blocks teacher daily workflow

### Theme B: Build Shared Primitives First
Charts, tables, data-entry patterns, and routing are re-invented per page in Angular. Building React shared primitives (chart wrapper, data-table, form context, autosave hook) once will cut per-page migration cost by ~40%.

### Theme C: Test Infrastructure is the Multiplier
The visual parity testing framework already works. Extending it to cover the Big 4 **before** migration starts is the single highest-leverage investment.

---

## ✅ Recommended Prioritization (Draft)

| Priority | Module | Lines | Rationale |
|----------|--------|-------|-----------|
| 1 | SectionDataEntry.js | 3,000 | Blocks daily teacher workflow |
| 2 | Shared chart primitives | ~500 | Enables #3 and #4 cheaply |
| 3 | LineGraphs.js | 2,100 | Uses chart primitives, moderate complexity |
| 4 | StackedBarGraphs.js | 3,800 | Uses chart primitives, high complexity |
| 5 | SectionReports.js | 4,500 | Decompose into 4 sub-components |
| 6 | ObservationSummary.js | 3,200 | Audit usage first |
| 7 | InterventionToolkit.js | 1,100 | Multi-step but bounded |
| 8 | Assessment full CRUD | ~930 | ~50% done, finish it |
| 9 | Student/Staff full CRUD | ~1,200 | ~75-85% done, close them out |
| 10 | Admin pages | ~2,900 | Lower risk, lower traffic |

---

## 🚫 Top Risks Identified

1. **SectionReports scope underestimate** — 4,500 lines with complex state; budget 3x time
2. **D3.js → recharts is a philosophy change** — not a library swap; expect rewrites
3. **Undocumented business logic** in legacy Angular — audit `.referenceSrc` before migrating
4. **casing mismatch** in API responses (established pattern from bug3240 history)
5. **Missing "done" definition** — without it, migration will never be declared complete

---

## 📋 Feeding Into Product Brief

**Problem Statement (draft):** NorthStarET's Angular.js frontend creates maintenance burden, onboarding friction, and user experience debt. 55% remains unmigrated, concentrated in 4 high-complexity modules.

**Vision (draft):** A fully React 19 front-end with zero Angular.js dependencies, deployed to production by end of Q2 2026, with all pages passing visual parity tests and having improved UX (autosave, URL-shareable state, mobile responsiveness).

**Scope:**
- In: SectionDataEntry, Charts (Line + StackedBar), SectionReports, ObservationSummary, InterventionToolkit, completing Student/Staff/Assessment CRUD
- Out: Backend changes, EF6 migration, new features not in OldNorthStar
