# bmad.lens.governance

Universal governance artifacts for BMAD lens-work. This repo is decoupled from the control repo (`bmad.lens.release`) and all target project repos so that cross-initiative rules — constitutions, team roster, and org policies — are never captured on an initiative branch and never lost.

## Structure

```
constitutions/
  org/           ← Org-level constitutions (highest precedence, additive root)
  domain/        ← Domain-level constitutions (one subfolder per domain)
roster/          ← Team member records (committed, not git-ignored)
policies/        ← Org-wide process and coding policies
repo-inventory.yaml  ← Canonical discovered-repo inventory
```

## Data Zones

| Zone | Location | Git Status | Branch Strategy |
|---|---|---|---|
| `governance` | This repo | committed | `universal/{slug}` → `main` |
| `initiative` | `bmad.lens.release/_bmad-output/lens-work/initiatives/` | committed | initiative branch hierarchy |
| `personal` | `bmad.lens.release/_bmad-output/lens-work/personal/` | git-ignored | n/a (local only) |

## Local Clone Path

All lens-work tooling expects this repo cloned to:
```
TargetProjects/lens/lens-governance/
```
(sibling to `TargetProjects/lens/lens-work/`)

## Writing to This Repo

- **Anyone** may open a PR.
- **Governance roles** (Tech Lead, Architect) are required reviewers via CODEOWNERS.
- The `amendment-propagation` workflow is the only automated process that opens PRs here.
- All PRs target `main`. Branch name must match `universal/{slug}`.

## Constitution Hierarchy

Constitutions are additive — lower layers can only ADD rules, never remove them.

```
org  →  domain  →  service  →  repo
```

See `constitutions/` for authored constitution files. The `@lens/constitution` skill (`_bmad/lens-work/skills/constitution.md` in the control repo) contains all resolution logic.
