---
doc_type: epic-party-mode-review
epic: EP-002
epic_title: "SectionDataEntry Module (Highest Priority)"
phase: devproposal
initiative: finish-northstaret-migration-3a10d3
date: "2026-02-26"
lead: Winston (Architect)
participants:
  - Winston (Architect)
  - Mary (Analyst)
  - Sally (UX Designer)
readiness_verdict: READY
party_verdict: PASS
blocking_issues: 0
---

# Epic Stress Gate: EP-002 — SectionDataEntry Module

## Implementation Readiness Check (adversarial)

**Winston asks:** Can every story in this epic be code-reviewed without resolving unknown architecture decisions?

| Check | Question | Answer |
|-------|---------|--------|
| Section types bounded | Is the list of section types complete? | ✅ Yes — `ISectionDataEntrySection` includes Attendance, Grades, Notes, Rubric, Roster |
| Props interface stable | Is `SectionDataEntryContainerProps` fully specced? | ✅ Yes — architecture §5.1: sectionId, gradeLevel, date, onSave, onNavigate, isLoading |
| Keyboard nav scoped | Are all keyboard interactions from the Angular source mapped? | ✅ Yes — Tab/Shift-Tab column traverse, Enter/Space toggle, Ctrl+S save (same as original) |
| Tab persistence mechanism | Is the tabIndex persistence architecture decided? | ✅ Yes — sessionStorage per sectionId, cleared on unmount |
| Performance gate | Is the virtualization threshold defined for large rosters? | ✅ Yes — row-virtualization via `react-window` when `students.length > 50` |
| Offline/error | Is optimistic update rollback specified? | ✅ Yes — architecture §7.2: rollback on 4xx, toast on 5xx, no rollback |
| Migration compatibility | Can old Angular route `#/section-data` run in parallel? | ✅ Yes — strangler fig: React mounted as Angular directive `<northstar-section-data-entry>` in Phase 1 |
| A11y | Any WCAG critera not met by keyboard nav spec? | ⚠️ Missing: `aria-live` region for autosave status updates (screen reader parity) |

**Readiness verdict: READY.** One a11y gap (aria-live on autosave indicator) must be added to acceptance criteria — not a blocker.

---

## Party Mode Review

**Winston (Architect — lead):**
The strangler-fig embedding (`<northstar-section-data-entry>` as an Angular directive wrapping the React mount point) is the right call. Adversarial challenge: when React and Angular are sharing the same DOM during Phase 1, who owns the `$rootScope.$apply` digest cycle for Angular-side watchers triggered by React state changes? The answer from architecture §9.1 is: React fires a custom DOM event (`northstar:sectionSaved`) and Angular listens with `$document.on('northstar:sectionSaved', ...)`. This is correct and confirmed. No hidden coupling concern.

Second adversarial challenge: `react-window` virtualization with sticky header row. The first row (column headers) must not be virtualized. Spec is silent on this. Story acceptance criterion should include: "Column header row is always visible during scroll (sticky header, not part of virtual row pool)." ✅ One sentence addition — not blocking.

**Mary (Analyst):**
This is the highest-priority teacher workflow. The data completeness validator ("flag incomplete cells") is listed but the threshold rule is not: does a section require 100% completion or is partial save acceptable? From the Angular source, partial saves are allowed (teachers save as they go). Story acceptance criterion should confirm: "Partial saves persist entered cells without requiring full row completion." ✅ Not blocking — clarifies existing behavior.

**Sally (UX Designer):**
The keyboard navigation spec is strong. Three UX additions for acceptance criteria:
1. **Focus indicator visibility**: Focus ring must be 3px solid `#0066CC` (matching existing NS4 design tokens) — not browser-default.
2. **Empty cell CTA**: Empty cells in Grades should show a subtle placeholder (`—`) not a blank, so teachers know cells are interactive.
3. **Mobile/tablet**: Pinch-zoom must not break the sticky column layout. Spec should state minimum supported viewport: 1024px (same as legacy NorthStar).

All three are one-liner acceptance criteria additions. ✅ Not blocking.

**aria-live gap (detail from Winston):**
AutosaveIndicator must expose `role="status"` and `aria-live="polite"`. Already specified in EP-001 but EP-002 stories that embed `<AutosaveIndicator>` must reference EP-001 as a prerequisite. Story sequencing enforces this.

---

## Verdict

| Reviewer | Vote | Notes |
|----------|------|-------|
| Winston | ✅ PASS | Add sticky header note + DOM event protocol is confirmed |
| Mary | ✅ PASS | Partial saves must be stated in acceptance criterion |
| Sally | ✅ PASS | Focus ring spec + empty cell placeholder + min viewport 1024px |

**Result: ✅ READY — no blocking issues. EP-002 clears stress gate.**
