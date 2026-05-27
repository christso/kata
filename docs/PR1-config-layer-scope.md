# PR 1 Scope: Config Layer – Optional repo fields for Projects v2

## Goal of this PR

Make `repo_owner` and `repo_name` optional in the GitHub tracker config **only when** `github_project_number` is provided (i.e., when using GitHub Projects v2).

This is the first and safest step in the stacked PR series for multi-repo support.

## Why This First?

- Lowest risk of regression.
- Unblocks later layers.
- Small, focused change.

## Detailed Scope

### Changes Expected

1. **config.rs**
   - Update `from_workflow` so that for `kind: github`, `repo_owner` and `repo_name` are only required if `github_project_number` is **not** present.
   - Keep the requirement for label-based GitHub usage.

2. **Validation (config.rs + doctor.rs)**
   - Adjust the hard validation rules.
   - Add clear error messages distinguishing the two modes.

3. **Tests**
   - Add tests for the new optional behavior with Projects v2.
   - Ensure all existing single-repo tests continue to pass.

### Out of Scope for PR 1

- Changing `GithubClient` or adapter logic
- Projects v2 query changes
- Helper layer changes
- Workspace or Docker changes

## Handoff Notes for Next Agent

- Full context and learnings are in:
  - `docs/proposals/symphony-github-multi-repo-gap.md`
  - `docs/proposals/stacked-pr-plan-symphony-multi-repo.md`

- The overall strategy is stacked PRs for upstream + `develop` branch in this fork for immediate use.

- This PR should be opened against `develop` (or rebased onto the latest `develop` before opening upstream).

## Suggested Commit Structure (for clean history)

1. Relax repo field requirement in config parsing
2. Update validation logic + error messages
3. Add tests for optional repo with Projects v2
4. Update doctor checks
5. Ensure no regression on existing single-repo configs

## Current Status

This branch was created as the minimal handoff point before the original agent handed off.

Next agent should start implementation from here.
