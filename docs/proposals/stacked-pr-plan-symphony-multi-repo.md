# Proposed Stacked PR Plan: Symphony Multi-Repo GitHub Projects v2 Support

## Goal

Enable Symphony to properly work with GitHub Projects v2 boards that contain issues from multiple repositories (including repo-agnostic org projects like `EntityProcess/projects/3`).

## Strategy

We will use **stacked PRs** for the upstream contribution. This aligns with the observed culture in `gannonh/kata` of preferring focused, reviewable changes.

In parallel, we will maintain a `develop` branch in `christso/kata` that merges all layers so the feature can be used internally immediately.

## Proposed Stack (Logical Order)

### PR 1: Config Layer – Make repo fields optional for Projects v2
- Relax the hard requirement on `repo_owner` + `repo_name` when `github_project_number` is present.
- Update validation in `config.rs` and `doctor.rs`.
- Add tests for the new optional behavior.
- Keep full backward compatibility when the fields are provided.

**Size**: Small–Medium

### PR 2: Projects v2 Query – Capture repository per item
- Update `QUERY_PROJECT_ITEMS` in `projects_v2.rs` to request repository information.
- Extend `ProjectItemContent` / `ProjectItem` structs.
- Update deserialization and any related logic.

**Size**: Small

### PR 3: Core Client – Support multi-repo operations
- Refactor or extend `GithubClient` to support operations across different repositories (either by making methods repo-aware or introducing a client factory).
- This is the highest-risk technical change.

**Size**: Medium–Large

### PR 4: Adapter – Per-item repository resolution
- Update `GithubAdapter` to resolve the correct client/repository for each board item.
- Handle state mode, URL construction, etc.

**Size**: Medium

### PR 5: Helper Layer – Per-issue repo context
- Update `github_adapter_inputs` and all dependent helper functions to accept or derive repository context per call.
- Critical for agent-side operations (PR creation, reviews, comments, etc.).

**Size**: Medium–Large (many functions)

### PR 6: Orchestrator, Workspace, Doctor, Logging, and Polish
- Update `orchestrator.rs`, workspace bootstrap (especially Docker path), doctor checks, TUI/logging, and any remaining surfaces.
- Add end-to-end tests for multi-repo scenarios.

**Size**: Medium

### PR 7 (Optional): Documentation & Examples
- Update README, WORKFLOW-REFERENCE.md, and add a multi-repo example.

**Size**: Small

## Upstream Contribution Flow

1. Open PR 1 as a normal PR (or draft).
2. Once merged, rebase PR 2 on top and open it.
3. Continue the chain.
4. Use clear commit messages and reference previous PRs in the stack.

## Internal Usage (christso/kata)

- Maintain a `develop` branch that merges all the above layers (rebased as upstream PRs land).
- This branch can be used immediately for real work against multi-repo boards.
- Periodically rebase `develop` onto the latest upstream `main`.

## Benefits of This Approach

- Matches the repo's observed preference for focused PRs.
- Much higher chance of thorough review than one giant PR.
- Allows early feedback on individual layers.
- Gives the team immediate internal value via the `develop` branch.
- Clear, bisectable history.

## Risks & Mitigations

- **Rebase pain**: Mitigate by keeping stacks relatively shallow and rebasing frequently.
- **Upstream movement**: The `develop` branch in our fork will absorb this.
- **Partial functionality**: Early PRs in the stack may not be usable alone — document this clearly.

## Next Steps

- Agree on this stack (or adjust layering).
- Draft the actual first PR (config layer) in the christso fork.
- Open the first upstream PR when ready.

---

*This document lives in the draft analysis branch and can be updated as the plan evolves.*
