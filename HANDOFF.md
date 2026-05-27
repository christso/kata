# Handoff: Symphony GitHub Multi-Repo Projects v2 Support

**Status**: Pre-implementation handoff

## Goal

Enable Kata Symphony to work properly with GitHub Projects v2 boards that span multiple repositories (including repo-agnostic org-level projects).

Upstream contribution will be done via **stacked PRs**. Internally we maintain this `develop` branch with all changes merged so the capability can be used immediately.

## Key Documents

- [Gap Analysis & Learnings](docs/proposals/symphony-github-multi-repo-gap.md)
- [Stacked PR Plan & Strategy](docs/proposals/stacked-pr-plan-symphony-multi-repo.md)

## Current State

- `main` branch tracks upstream (`gannonh/kata`)
- `develop` branch = our working fork with all planned changes
- First PR branch created: `feat/symphony-config-optional-repo-projects-v2`

## How to Continue

1. Start with the first PR branch:
   ```bash
   git checkout feat/symphony-config-optional-repo-projects-v2
   ```

2. Read the scope document on that branch:
   `docs/PR1-config-layer-scope.md`

3. This PR is the safest, smallest first step (making repo fields optional for Projects v2 only).

4. Follow the stacked plan in `docs/proposals/stacked-pr-plan-symphony-multi-repo.md`.

## Important Context

- The limitation is in **Symphony**, not GitHub Projects v2.
- GitHub natively supports multi-repo (even repo-agnostic) Projects.
- We are using stacked PRs for upstream because the repo culture favors focused changes.
- Full backward compatibility for existing single-repo users is mandatory.

## Next Agent Responsibilities

- Implement PR 1 (config layer) following the scope doc
- Maintain clean, atomic commit history within each PR
- Keep `develop` updated as upstream PRs land
- Update this HANDOFF.md as the plan evolves

---

*Last updated during initial handoff (May 2026)*
