# Handoff: Symphony GitHub Multi-Repo Projects v2 Support

**Status**: PR 1 merged into `develop` (internal use); continuing stacked implementation

## Goal

Enable Kata Symphony to work properly with GitHub Projects v2 boards that span multiple repositories (including repo-agnostic org-level projects).

Upstream contribution will be done via **stacked PRs**. Internally we maintain this `develop` branch with all changes merged so the capability can be used immediately.

## Key Documents

- [Gap Analysis & Learnings](docs/proposals/symphony-github-multi-repo-gap.md)
- [Stacked PR Plan & Strategy](docs/proposals/stacked-pr-plan-symphony-multi-repo.md)
- PR 1 scope (historical): `docs/PR1-config-layer-scope.md`

## Current State (as of 2026-05)

- `main` branch tracks upstream (`gannonh/kata`)
- `develop` branch = working fork with merged stack (for immediate internal use on multi-repo boards)
- PR 1 complete & merged: **Config layer** — `repo_owner`/`repo_name` now optional when `github_project_number` is present (Projects v2). Doctor updated to be friendly about the new mode. Tests added/updated.
  - Merge commit on develop: cb954cb
  - Draft upstream-style PR on fork: christso/kata#2
- PR 2 complete & merged: **Projects v2 query** — now captures per-item `repository` (nameWithOwner + owner login + name) from the GraphQL response. `ProjectItem` carries optional `repo_owner`/`repo_name`. All existing tests pass; query fragment asserted.
  - Latest merge on develop: 6cf3af1
- PR 3 (partial, on develop): **GithubClient** — added `get_issue_in_repo(owner, repo, ...)` and `list_issues_in_repo(...)` (old methods delegate for compat).
- Adapter bridge (landed on develop): main Projects v2 fetch paths now call the per-item repo variants when `ProjectItem` carries repo data from the query. This is a major step — Symphony can now *fetch* issues that live in secondary repos on a real multi-repo board. Helper layer (PR5) and workspace/PR creation still need work for full dispatch.

## How to Continue

1. Work is now done directly against (or merged into) `develop`.
2. Next up: PR 2 — extend the Projects v2 GraphQL query to capture per-item repository information (`apps/symphony/src/github/projects_v2.rs` + `ProjectItem*` structs).
3. Create focused feature branches off `develop` for each remaining PR in the stack (see plan).
4. Merge each PR into `develop` as it lands (for internal testing).
5. Rebase `develop` onto latest upstream `main` periodically.
6. Update this HANDOFF.md after each merge.

## Important Context

- The limitation is in **Symphony**, not GitHub Projects v2.
- GitHub natively supports multi-repo (even repo-agnostic) Projects.
- We are using stacked PRs for upstream because the repo culture favors focused changes.
- Full backward compatibility for existing single-repo users is mandatory.
- Early PRs give partial functionality; full end-to-end multi-repo dispatch requires PRs 2–5 at minimum.

## Next Agent Responsibilities

- Continue the stack on `develop` (PR 2 next: query repo-per-item capture)
- Keep `develop` green and usable for real multi-repo boards as layers land
- Add characterization / E2E tests as soon as the core (query + client + adapter + helpers) supports them
- Update this HANDOFF.md as the plan evolves
- When ready, open focused PRs from the feature branches to upstream

---

*Last updated after PR 1 merge into develop (May 2026)*
