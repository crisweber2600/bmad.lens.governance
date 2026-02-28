---
layer: domain
name: NorthStar
---

## Preamble

The NorthStar domain governs all work related to the migration from OldNorthStar to NorthStarET — including the React/TypeScript frontend, the data-entry modules, chart rendering primitives, and the BatchPrint worker integration. This constitution ensures that all governed work upholds the architectural conventions established during the techplan phase, preserves compatibility with the BatchPrint worker service, and delivers a maintainable, testable codebase for long-term operations.

```yaml
permitted_tracks:
  - full
  - feature
  - tech-change
required_gates:
  - tests-pass
additional_review_participants: {}
```

---

## Articles

### Article 1: Feature Slice Architecture

**Rule:** All front-end feature work MUST follow the mandatory feature slice pattern. Each domain feature requires `features/{domain}/{domain}Api.ts`, `{domain}Types.ts`, and `use{Domain}.ts`. Pages MUST NOT call axios directly — all API access goes through the feature slice layer.

**Rationale:** Enforces consistent separation of concerns, makes API contracts explicit, and prevents the proliferation of ad-hoc data-fetching patterns that plagued OldNorthStar.

**Evidence Required:** Code review must verify that no page-level components import or call axios directly. All new features include the full slice triad before code review approval.

---

### Article 2: Print Mode via URL Parameter

**Rule:** All print functionality MUST use the URL parameter `?printmode=1` detected by `isPrintMode()` from `chartUtils.ts`. CSS media queries (`@media print`) and `window.matchMedia` MUST NOT be used for print detection.

**Rationale:** The BatchPrint worker service invokes pages with the `?printmode=1` URL parameter. Using CSS media queries breaks BatchPrint compatibility. This was established as TD-002 during the techplan phase.

**Evidence Required:** Any PR touching chart or print functionality must confirm no `window.matchMedia` or `@media print` usage for print detection. Integration test with BatchPrint worker must pass.

---

### Article 3: API Mutation Pattern

**Rule:** Bulk data saves MUST use the existing `BatchSave` endpoint pattern. No new PATCH endpoints for individual-record mutations when a batch endpoint exists. New endpoints require explicit tech-decision approval.

**Rationale:** Simplifies backend contract, reduces API surface, and avoids duplication of persistence logic. Established as TD-004 during the techplan phase. New endpoints introduce coordination overhead across backend and frontend teams.

**Evidence Required:** API change PRs must reference a tech decision record approving any new endpoint. Bulk operations that bypass `BatchSave` must include a documented exception with approval.

---

### Article 4: Migration Compatibility Gate

**Rule:** All changes to shared data models, API contracts, or chart primitives MUST verify that OldNorthStar parity tests (`NS4.Parity.Tests`) remain 100% green before merge.

**Rationale:** NorthStarET is a migration target. Breaking parity with OldNorthStar's behavior blocks the migration for active users and teams. The parity gate was established as a Phase 1 requirement.

**Evidence Required:** CI must run `NS4.Parity.Tests` on every PR touching data models, API types, or chart rendering. PRs may not merge while parity tests are red. Gate status is tracked in Domain.yaml.

---

### Article 5: Scope Discipline for PDF/Print

**Rule:** PDF export scope is limited to browser-print-only (via `window.print()`). Server-side PDF generation, headless-browser rendering, and third-party PDF libraries are out of scope for EP-004 and require a new initiative with a spike story before implementation.

**Rationale:** Scope creep in EP-004 (Reports and PDF) was identified as a top risk during preplan. Keep the delivery unit cohesive and ship print functionality that can be tested without infrastructure complexity.

**Evidence Required:** Any PR introducing a PDF library dependency (`puppeteer`, `wkhtmltopdf`, `pdf-lib`, etc.) must be accompanied by a spike story acceptance record from a new initiative. Browser-print stories must have a spike story (S-024) cleared first.
