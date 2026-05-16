---
doc_type: epic-stress-gate-summary
phase: devproposal
initiative: finish-northstaret-migration-3a10d3
date: "2026-02-26"
lead: Winston (Architect)
participants:
  - Winston (Architect)
  - Mary (Analyst)
  - Sally (UX Designer)
epics_reviewed: 6
epics_passed: 6
blocking_issues_found: 1
blocking_issues_resolved: 1
---

# Epic Stress Gate Summary — Finish NorthStarET Migration

**Initiative:** finish-northstaret-migration-3a10d3
**Phase:** DevProposal — Step [2] of 4
**Date:** 2026-02-26
**Panel:** Winston (Architect, lead), Mary (Analyst), Sally (UX Designer)

---

## Summary Table

| Epic | Title | Verdict | Issues Found | Issues Resolved |
|------|-------|---------|-------------|----------------|
| EP-001 | Foundation: API + Shared Infrastructure | ✅ PASS | 2 (minor) | ✅ Added to AC |
| EP-002 | SectionDataEntry Module | ✅ PASS | 5 (minor) | ✅ Added to AC |
| EP-003 | Shared Chart Primitives | ✅ PASS | 4 (minor) | ✅ Added to AC |
| EP-004 | SectionReports Module | ✅ PASS (conditional → resolved) | 1 (blocking) + 3 (minor) | ✅ Spike story + scope gate |
| EP-005 | Admin & Settings Module | ✅ PASS | 4 (minor) | ✅ Added to AC |
| EP-006 | Angular Decommission & Cutover | ✅ PASS | 6 (minor) | ✅ Added to AC |

**Overall: 6/6 epics pass. 0 unresolved blockers. Ready for story generation.**

---

## Issue Log

### Blocking Issues (1 total — resolved)

| ID | Epic | Description | Resolution |
|----|------|-------------|-----------|
| BLK-001 | EP-004 | "Export to PDF" scope ambiguous — could mean browser print OR server-side PDF generation (2–3 sprint difference) | **Resolved:** Scope locked to browser print only (`File → Print → Save as PDF`). `SPIKE: PDF Export Feasibility (1pt)` story added. AC AT-037 added as scope gate. |

### Minor Issues (all resolved via acceptance criteria additions)

| ID | Epic | Issue | Resolution |
|----|------|-------|-----------|
| MIN-001 | EP-001 | W1 grep must search both `StudentId` AND `studentId` | AC expanded to both casing variants |
| MIN-002 | EP-001 | AutosaveIndicator timestamp must use local timezone | `toLocaleTimeString()` note in AC |
| MIN-003 | EP-002 | Sticky header row must not be part of virtual row pool | AC added: "Column header always visible during scroll" |
| MIN-004 | EP-002 | Partial saves must be stated as expected behavior | AC: "Partial saves persist entered cells without requiring full row completion" |
| MIN-005 | EP-002 | Focus ring spec missing | AC: 3px solid `#0066CC` focus ring on data cells |
| MIN-006 | EP-002 | Empty Grade cells should show `—` placeholder, not blank | AC: empty cells show `—` placeholder |
| MIN-007 | EP-002 | Minimum viewport not stated | AC: min viewport 1024px (matches legacy NorthStar) |
| MIN-008 | EP-003 | `ResponsiveContainer` can infinite-resize without parent min-height | AC: parent wrapper `min-height: 200px` |
| MIN-009 | EP-003 | print.css must replace `.d3-chart` selectors with `.recharts-wrapper` | AC: print.css selector audit story |
| MIN-010 | EP-003 | Dual-axis chart support must appear in EP-003 story (not just EP-004) | AC: dual-axis confirmed in chart primitives story |
| MIN-011 | EP-003 | Chart color tokens not locked (Recharts defaults) | AC: all series colors from design token constants |
| MIN-012 | EP-004 | EP-004 reports must default to current semester date range | AC AT-038 added |
| MIN-013 | EP-004 | Print header must include school name, section, teacher, date range | AC AT-039 added (compliance) |
| MIN-014 | EP-004 | Visual regression baseline needed before Angular route removal | AC AT-040 added |
| MIN-015 | EP-005 | `RequireRole` must use specific role constants (`school_admin`/`district_admin`) | AC: role constant specificity note |
| MIN-016 | EP-005 | Admin actions must write to NS4 audit table | AC: audit log acceptance criterion |
| MIN-017 | EP-005 | Deactivate button must use "Deactivate User" language | AC: UX copy specified |
| MIN-018 | EP-005 | Bulk deactivate story needed | Scope confirmed — one bulk action checkbox story |
| MIN-019 | EP-006 | Cutover timing not operationally gated | AC AT-056: school break or off-hours + 48h notification |
| MIN-020 | EP-006 | No lint rule to prevent Angular regression | AC AT-059: `no-angularjs-controller` ESLint rule |
| MIN-021 | EP-006 | Parity test count could decrease (suite erosion risk) | AC AT-060: test count gate |
| MIN-022 | EP-006 | Mobile Safari 14+ not in test matrix | AC AT-061: mobile Safari test requirement |
| MIN-023 | EP-006 | Release notes for 3 stakeholder audiences not scoped | AC AT-062: release notes story |
| MIN-024 | EP-006 | Hash route redirects not explicitly scoped | AC AT-057: 302 redirect for legacy hash routes |

---

## Amendments Applied to epics.md

All issues above have been applied to `epics.md` as:
- Acceptance criteria additions (AT-037 through AT-062 in EP-004 and EP-006)
- Spike story in EP-004
- Expanded Party Review Notes table (W1–W8, M1–M5, S1–S8)

No epic-level scope changes were made. The SPIKE story in EP-004 is the only new story item; all others are acceptance criterion additions to existing stories.

---

## Readiness Assessment

**EP-001:** ✅ Story-derivable. Two AC additions applied.  
**EP-002:** ✅ Story-derivable. Five AC additions applied. EP-001 prerequisite enforced.  
**EP-003:** ✅ Story-derivable. Four AC additions applied. EP-001 prerequisite enforced.  
**EP-004:** ✅ Story-derivable. PDF scope locked (browser print). Spike story added. Six AC additions applied.  
**EP-005:** ✅ Story-derivable. Four AC additions applied. Decoupled from EP-002/003/004.  
**EP-006:** ✅ Story-derivable. Six AC additions applied. Operational gates (cutover timing, release notes) added.

**Stress gate complete. Proceed to Step [3]: Story Generation.**
