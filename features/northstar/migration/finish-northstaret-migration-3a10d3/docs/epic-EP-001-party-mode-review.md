---
doc_type: epic-party-mode-review
epic: EP-001
epic_title: "Foundation: API + Shared Infrastructure"
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

# Epic Stress Gate: EP-001 — Foundation: API + Shared Infrastructure

## Implementation Readiness Check (adversarial)

**Winston asks:** Can this epic be built starting on day 1 with no unresolved decisions?

| Check | Question | Answer |
|-------|---------|--------|
| Technically specified | Is `JsonNamingPolicy.CamelCase` a single known config change? | ✅ Yes — one line in `NS4.WebAPI/Program.cs` |
| Test gate defined | Is the Phase 1 gate measurable? | ✅ Yes — `NS4.Parity.Tests` 100% green |
| Autosave interface complete | Are `useAutosave`, `AutosaveIndicator` fully specced in architecture §4.3 / §7.1? | ✅ Yes — debounce 1.5s, interval 30s, BatchSave endpoint, 4 status states |
| Endpoint verified | Does `BatchSave` already exist with typed request/response? | ✅ Yes — `BatchSaveRequest`/`BatchSaveResponse` in `sectionDataEntryApi.ts` |
| Chart audit scoped | Is the StackedBarChart print-mode verification bounded? | ✅ Yes — add `isPrintMode()` call + `isAnimationActive` guard |
| Data-only risk | Any database or .NET controller changes needed? | ✅ No — only `Program.cs` JSON options |
| Breaking change blast radius | What breaks when camelCase lands? | ⚠️ Angular templates with PascalCase bindings — mitigated by W1 grep |

**Readiness verdict: READY.** Epic is fully story-derivable. One pre-work action (W1 grep) must be a story gate.

---

## Party Mode Review

**Winston (Architect — lead):**
This epic is correctly scoped as infrastructure. My one adversarial challenge: the camelCase PR impact on Angular templates is non-trivially checkable by grep alone — Angular's `$scope.studentId` vs `$scope.StudentId` binding depends on how each controller was written. The W1 grep should search both casing variants. Story acceptance criterion must be: "grep confirms all Angular template bindings use camelCase OR a documented list of PascalCase bindings exists with migration plan." ✅ No block — adds one sentence to story acceptance criterion.

**Mary (Analyst):**
The analytics impact of this epic is zero-visible to teachers during EP-001 — it's all under the hood. This is correct and I support the sequencing. No concerns.

**Sally (UX Designer):**
`AutosaveIndicator` state machine is well-specified. One note: the "Saved HH:MM" format should show the time in the teacher's local timezone, not UTC. This is a one-liner in the implementation (`new Date().toLocaleTimeString()`). Add to story acceptance criteria. ✅ Not blocking.

---

## Verdict

| Reviewer | Vote | Notes |
|----------|------|-------|
| Winston | ✅ PASS | Expand W1 grep to cover both casing variants |
| Mary | ✅ PASS | — |
| Sally | ✅ PASS | AutosaveIndicator time in local timezone |

**Result: ✅ READY — no blocking issues. EP-001 clears stress gate.**
