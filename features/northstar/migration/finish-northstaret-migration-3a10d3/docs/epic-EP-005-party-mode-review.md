---
doc_type: epic-party-mode-review
epic: EP-005
epic_title: "Admin & Settings Module"
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

# Epic Stress Gate: EP-005 — Admin & Settings Module

## Implementation Readiness Check (adversarial)

**Winston asks:** Are admin features isolated enough to avoid blocking or being blocked by other epics?

| Check | Question | Answer |
|-------|---------|--------|
| Role-based access | Is admin role check specced at the component level? | ✅ Yes — architecture §9.3: `<RequireRole role="admin">` wrapper HOC; non-admin sees 403 page |
| Settings persistence | Are admin settings stored in API or LocalStorage? | ✅ Yes — all admin settings via `NS4 /api/v1/admin/settings` — no localStorage |
| User management scope | Is the user CRUD fully scoped? | ✅ Yes — Create, Read, Update, Deactivate (soft-delete only — no hard delete per business rule) |
| School year config | Is "school year" a string or a structured date range? | ✅ Yes — architecture §9.2: `{ schoolYear: string, startDate: Date, endDate: Date }` — typed, not free text |
| Import/export admin data | Is bulk user import (CSV) in scope? | ✅ Yes — scoped as one story: CSV upload → validation → preview → confirm |
| Permission gate | Who controls admin role assignment in the system? | ✅ Yes — district admin can promote/demote school admins; school admins cannot self-promote |
| Dependency | Does EP-005 depend on any other epic? | ✅ EP-001 only (foundation auth + API) — no dependency on EP-002, EP-003, EP-004 |
| Parallelism | Can EP-005 run in parallel after EP-001 ships? | ✅ Yes — deliberately decoupled |

**Readiness verdict: READY.** Admin module is self-contained after EP-001.

---

## Party Mode Review

**Winston (Architect — lead):**
Adversarial challenge: district admin vs school admin permission boundary. The `RequireRole` HOC checks for `"admin"` generically — but district admin and school admin are different permission levels. Architecture §9.3 must clarify: `<RequireRole role="school_admin" />` vs `<RequireRole role="district_admin" />`. Story acceptance criteria must reference the specific role constant, not the generic `"admin"`. ✅ One-line addition per admin story — not blocking.

**Mary (Analyst):**
Admin module is essential for school setup at the start of each year. Two acceptance criteria:
1. **Onboarding sequence**: First-time admins should see a guided setup wizard (school year, sections, teachers). This is not the same as user management. If this wizard doesn't exist in legacy NS, flag it as a new feature outside scope. ← Per the Angular source audit, legacy NS has a simple text-form for this, no wizard. React version matches legacy — no wizard needed.
2. **Audit log**: Admin actions (user create/deactivate, settings change) must be logged to the NS4 audit table. This is a backend concern — confirm with backend team that the audit table exists and the endpoint captures admin actor ID. ✅ One acceptance criterion: "Admin actions write to audit log via NS4 API."

**Sally (UX Designer):**
Admin UI is secondary to teacher UX in priority, but principals are a key user type. Three UX notes:
1. **Deactivate vs delete**: The UI must use "Deactivate" language — not "Delete" or "Remove" — to correctly reflect soft-delete semantics. Story acceptance criterion: "User deactivation button reads 'Deactivate User' with confirmation modal stating 'This user will be hidden from all active sections.'"
2. **Bulk actions**: The user management table should support checkbox-bulk-deactivate. Scoped as one UX story.
3. **Settings discard guard**: If admin navigates away from settings with unsaved changes, show a `<DiscardChangesDialog>` (same component used in EP-002). Confirm `<DiscardChangesDialog>` is in EP-001 shared components scope. ✅ It is — per architecture §4.1.

---

## Verdict

| Reviewer | Vote | Notes |
|----------|------|-------|
| Winston | ✅ PASS | Role constant specificity — use `school_admin`/`district_admin` |
| Mary | ✅ PASS | Audit log acceptance criterion added |
| Sally | ✅ PASS | Deactivate language, bulk action, discard guard |

**Result: ✅ READY — no blocking issues. EP-005 clears stress gate.**
