---
doc_type: epic-party-mode-review
epic: EP-004
epic_title: "SectionReports Module"
phase: devproposal
initiative: finish-northstaret-migration-3a10d3
date: "2026-02-26"
lead: Winston (Architect)
participants:
  - Winston (Architect)
  - Mary (Analyst)
  - Sally (UX Designer)
readiness_verdict: PARTIAL — scope clarification required
party_verdict: CONDITIONAL PASS
blocking_issues: 1
---

# Epic Stress Gate: EP-004 — SectionReports Module

## Implementation Readiness Check (adversarial)

**Winston asks:** Is SectionReports scope bounded enough to estimate reliably?

| Check | Question | Answer |
|-------|---------|--------|
| Report types inventoried | Are all report views in legacy NS identified? | ✅ Yes — `SectionProgressReport`, `AttendanceReport`, `GradesOverTimeReport`, `ComparisonReport` |
| Print pipeline | Is CSS print media approach or PDF library chosen? | ⚠️ ADR-005 (tech-decisions.md) says "browser print CSS preferred" but does NOT confirm whether `@page` margin/header/footer spec exists |
| Chart dependencies | Does EP-004 safely depend on EP-003 primitives? | ✅ Yes — story sequencing in epics.md lists EP-003 as prerequisite |
| Export format | Is PDF export in-scope for this phase? | ⚠️ **AMBIGUOUS** — epics.md mentions "export to PDF" but the product brief scopes this as "print-ready view", not programmatic PDF generation |
| Date range filter | Is the date range picker component specified or sourced? | ✅ Yes — architecture §8.2: use `react-datepicker` with NS4 style overrides |
| Dual-axis charts | Are `GradesOverTimeReport` chart requirements confirmed (dual Y-axis)? | ✅ Yes — covered by EP-003 dual-axis story (per Mary's EP-003 note) |
| Scope risk | Could PDF generation add 2+ sprint weeks of effort? | ⚠️ YES — if PDF means `puppeteer`/`jsPDF` server-side generation, that is a separate epic |

**BLOCKING ISSUE:** "Export to PDF" scope must be clarified before story point estimation. If it means: (a) "browser File→Print→Save as PDF" = zero new work. If it means: (b) "Download PDF button generating a server-side PDF" = new backend story + CI/CD change.

**Readiness verdict: CONDITIONAL.** One scope clarification required. All other areas are story-derivable.

---

## Party Mode Review

**Winston (Architect — lead):**
The PDF scope ambiguity is my primary concern and it is a real blocker for story sizing. Here is my ruling: **EP-004 stories may be written for "print-ready view" (browser print, zero new backend), and a separate spike story `SPIKE: PDF Export Feasibility` is added to EP-004 with 1 sprint point and no dependency.** The spike runs concurrently with the print-ready stories. If the spike concludes "server-side PDF needed," a new story is appended to EP-004 before the epic review gate. This unblocks story writing now.

`@page` CSS spec gap: I reviewed the legacy Angular templates and the NorthStar print stylesheet uses `@media print { .section-report { page-break-inside: avoid; } }`. We must port this stylesheet into `SectionReport.print.css`. Add to acceptance criteria.

✅ Blocking issue resolved by spike approach — CONDITIONAL PASS elevated to PASS with spike story.

**Mary (Analyst):**
SectionReports is a high-visibility teacher feature — this is the view principals use for compliance reporting. Two acceptance criteria additions:

1. **Date range defaulting**: Reports must default to the current semester date range, not the calendar year. This matches teacher expectations from the legacy NS behavior.
2. **Print header**: Printed reports must include school name, section name, teacher name, and date range in the print header (`@page` margin content). This is a compliance need.

Both are verifiable and scoped. ✅ Not blocking.

**Sally (UX Designer):**
The report layout must follow a clear visual hierarchy: section summary bar → chart → detail table. This matches the current NorthStar print layout but must be preserved in React. Story acceptance criterion: "Report layout order: summary card → visualization → data table — must match legacy NorthStar report layout (visual regression snapshot required)."

Responsive print caveat: reports should be `max-width: 1100px` on screen (not full-bleed) and switch to full-width on print. Both are in print.css scope. ✅ Not blocking.

---

## Scope Clarification Resolution

PDF Scope ruling (Winston): EP-004 proceeds with **browser print path only**. Spike story `SPIKE: PDF Export Feasibility (1pt)` added. If server-side PDF confirmed needed, new story added before epic close.

---

## Verdict

| Reviewer | Vote | Notes |
|----------|------|-------|
| Winston | ✅ CONDITIONAL PASS → PASS | Spike story resolves ambiguity; print CSS spec added |
| Mary | ✅ PASS | Semester default + print header compliance criteria added |
| Sally | ✅ PASS | Layout order + max-width + visual regression snapshot |

**Result: ✅ READY (with spike story) — EP-004 clears stress gate.**

> **Artifact amendment:** `epics.md` EP-004 acceptance criteria must be updated with: (1) print-only scope note, (2) spike story, (3) semester date default, (4) print header compliance, (5) visual regression snapshot requirement.
