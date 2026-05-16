---
doc_type: epic-party-mode-review
epic: EP-006
epic_title: "Angular Decommission & Cutover"
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

# Epic Stress Gate: EP-006 — Angular Decommission & Cutover

## Implementation Readiness Check (adversarial)

**Winston asks:** Is the decommission plan reversible at each step? Can we rollback during cutover?

| Check | Question | Answer |
|-------|---------|--------|
| Feature flag architecture | Is a feature flag system specced for controlled cutover? | ✅ Yes — architecture §10.1: `useFeatureFlag('react_modules')` from NS4 config API |
| Angular removal sequence | Is the removal order defined to avoid cascading breaks? | ✅ Yes — tech-decisions.md ADR-008: remove routes one at a time, verify React parity per route, then remove Angular controller |
| Rollback window | Is there a rollback mechanism post-cutover? | ✅ Yes — feature flag can revert within same deploy; hard rollback = git revert + redeploy (<15 min) |
| Bundle size validation | Is there a bundle size gate? | ✅ Yes — epics.md: "React bundle must be < 150kB gzipped before Angular removal" |
| NS4.Parity.Tests | Does parity test suite cover all 4 epics' routes? | ✅ Yes — parity tests are gate for EP-001 AND EP-006 close |
| Orphan Angular code | Is there a lint rule that fails on new Angular controller creation? | ⚠️ Not specced — add ESLint/TSLint rule: `no-angularjs-controller` after Phase 2 |
| Training/comms | Is user-facing change communication in scope? | ✅ Yes — epics.md: "Produce release notes for admin, teacher, district admin audiences" |
| Canary rollout | Is a canary/gradual rollout strategy defined? | ✅ Yes — architecture §10.2: school-by-school enablement via feature flag; 3-school pilot before full rollout |
| Mobile Safari | Any known mobile regression risks? | ⚠️ Angular IE11 polyfills removed in React version — confirm mobile Safari 14+ is tested (no IE11 concern but old Safari has edge cases) |

**Readiness verdict: READY.** Two additions: lint rule story + mobile Safari in test matrix.

---

## Party Mode Review

**Winston (Architect — lead):**
This is the highest-risk epic operationally, even though it is technically the simplest. My adversarial concern is the **cutover coordination window.** Cutover can't happen mid-semester without communication. Acceptance criterion for the cutover story: "Cutover is scheduled during a school break period OR during non-school hours (before 7AM / after 6PM local time) AND a 48-hour teacher notification email is sent."

Second concern: the `no-angularjs-controller` lint rule must be a story in EP-006 — not just an ADR note. Lint rule enforces the decommission is permanent and prevents regression. ✅ One story added.

Third: the NS4.Parity.Tests suite must be maintained as a first-class gate. If any parity test is deleted or marked `.skip()` during EP-006, that is a failing acceptance criterion. Story: "Parity test count must equal or exceed count from EP-001 gate."

**Mary (Analyst):**
Three stakeholder groups need different release notes:
1. **Teachers**: "What changed in your daily workflow" — e.g., new keyboard shortcuts, autosave, updated chart views.
2. **Admins**: "New admin panel walkthrough" — settings, user management.
3. **District admins**: "Data access and compliance report changes" — SectionReports PDF/print changes.

Release notes are not code but are an acceptance criterion for EP-006 close. Story acceptance criterion: "Release notes for all three audiences reviewed and approved by product owner before cutover."

**Sally (UX Designer):**
The cutover moment is a UX risk. Teachers who have used NorthStar for years will notice visual changes. Two acceptance criteria:
1. **Visual regression baseline**: Before Angular removal, capture full-page screenshots of all 4 major views in the Angular version. React versions must pass visual regression comparison (95% similarity threshold via `jest-image-snapshot` or `Chromatic`).
2. **400/redirected routes**: When old Angular hash routes (`#/section-data`, `#/reports`) are accessed after cutover, they must redirect to the React equivalents with a `window.location` redirect — not a 404. Story: "Legacy hash routes return 302 redirect to React route."

Both are already implied by architecture §10.3 but must be explicit acceptance criteria. ✅ Not blocking.

---

## Verdict

| Reviewer | Vote | Notes |
|----------|------|-------|
| Winston | ✅ PASS | Lint rule story added; parity test count gate; cutover timing criterion |
| Mary | ✅ PASS | Release notes (3 audiences) are EP-006 close gate |
| Sally | ✅ PASS | Visual regression baseline + hash-route redirect story |

**Result: ✅ READY — no blocking issues. EP-006 clears stress gate.**
