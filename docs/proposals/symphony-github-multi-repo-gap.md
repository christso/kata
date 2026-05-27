# Kata Symphony – GitHub Multi-Repository Gap Analysis

**Conclusion**: The limitation is **not** in GitHub Projects v2.  
GitHub Projects v2 fully supports issues from multiple repositories on a single board.  
The gap exists entirely in **Kata Symphony's GitHub implementation**.

---

## Core Problem

Symphony can query a GitHub Project board and receive items from many repos, but it is architecturally unable to process any item that does not belong to the single repository configured in the tracker.

---

## Primary Gap Locations (as of current codebase)

### 1. Projects v2 Query Does Not Capture Repository

**File**: `apps/symphony/src/github/projects_v2.rs`

```graphql
content {
  ... on Issue {
    number                    # Only the issue number — no repository information
    blockedBy { ... }
  }
}
```

Corresponding struct:

```rust
#[derive(Debug, Deserialize)]
struct ProjectItemContent {
    number: Option<u64>,      // No repo owner/name captured
    blocked_by: Option<...>,
}
```

Even if the GraphQL response contained repository data, Symphony would not deserialize or use it.

### 2. All Project Items Are Fetched Against a Single-Repo Client

**File**: `apps/symphony/src/github/adapter.rs` (multiple call sites)

After successfully reading items from the board via `query_items_by_status(...)`:

```rust
for item in project_items {
    let issue = match self.client.get_issue(item.issue_number).await {
        Ok(issue) => issue,
        Err(SymphonyError::GithubApiStatus { status: 404, .. }) => {
            // Silently skipped
            continue;
        }
        ...
    };
    ...
}
```

`self.client` is a `GithubClient` bound to one `(repo_owner, repo_name)` pair.  
Any issue whose number does not exist in that specific repo is 404'd and ignored.

Affected methods include:
- `fetch_candidate_issues_projects_v2`
- `fetch_issues_by_states_projects_v2`
- `fetch_issue_states_by_ids_projects_v2`
- `update_issue_state_projects`

### 3. GithubClient Is Fundamentally Single-Repo

**File**: `apps/symphony/src/github/client.rs`

```rust
pub struct GithubClient {
    pub repo_owner: String,
    pub repo_name: String,
    ...
}
```

All REST paths are constructed relative to this fixed repo (issues, pulls, labels, comments, etc.).

### 4. Hard Validation Enforces Single-Repo for GitHub

**File**: `apps/symphony/src/config.rs`

```rust
if matches!(tracker_kind.as_deref(), Some("github")) {
    require repo_owner
    require repo_name
    require github_project_owner_type
    require github_project_number
}
```

**File**: `apps/symphony/src/helper.rs`

`github_adapter_inputs()` performs the same enforcement (used by all agent helpers for PRs, reviews, comments, etc.).

**File**: `apps/symphony/src/doctor.rs`

`check_github()` also requires both repo fields.

---

## Why This Matters for Multi-Repo Boards

You can correctly have this situation on GitHub:

- Project #1 (EntityProcess) contains issues from `agentv`, `ai-research`, `platform`, etc.
- The board itself knows the correct repository for every item.

But when Symphony is pointed at that board with:

```yaml
tracker:
  kind: github
  repo_owner: EntityProcess
  repo_name: some-meta-repo          # or any single repo
  github_project_number: 1
```

It will only ever successfully process items that actually live in `some-meta-repo`. Everything else on the board is invisible or skipped.

---

## Relationship to the "Meta/Workspace Repo" Proposal

Using a dedicated meta repo as the home for `.symphony/` configuration + issues is a **workaround**, not a full solution to the gap.

It works around the limitation by ensuring all dispatchable issues live in the one repo Symphony is configured to watch. This is a valid pragmatic pattern, but it means you are not actually using GitHub Projects v2's native multi-repo support from Symphony's perspective.

---

## Is This Fixable?

Yes. This is an implementation gap, not a fundamental architectural blocker.

Fixing it would require:

- Updating the Projects v2 GraphQL query to request repository per item
- Making `GithubClient` / adapter able to operate across multiple repos (or resolve per-item clients)
- Updating helpers, workspace logic, config validation, doctor, etc.
- Adding tests and backward compatibility

It is a meaningful refactor but entirely within scope for the kata repository.

---

**References** (current as of analysis):
- `apps/symphony/src/github/projects_v2.rs`
- `apps/symphony/src/github/adapter.rs` (lines ~397–465, ~510+, ~580+, ~660+)
- `apps/symphony/src/github/client.rs`
- `apps/symphony/src/config.rs` (validation + RawTrackerConfig)
- `apps/symphony/src/helper.rs` (github_adapter_inputs)
- `apps/symphony/src/doctor.rs`

---

## Strong Confirmation: Org-Level Repo-Agnostic Projects

You are correct.

GitHub allows you to create an organization-level Project (e.g. `https://github.com/orgs/EntityProcess/projects/3`) that is **not attached to any repository at all**.

Such projects are multi-repository **by default**. You can freely add issues and pull requests from any repositories the organization (or its members) have access to. There is no requirement to designate a "primary" or "home" repository for the project.

This is the modern, intended usage of GitHub Projects v2 at organization scale.

### Current Symphony Behavior vs GitHub Reality

| Aspect                              | GitHub Projects v2 (Reality)                  | Kata Symphony (Current)                          | Gap? |
|-------------------------------------|-----------------------------------------------|--------------------------------------------------|------|
| Can a Project exist without being tied to one repo? | Yes (org-level projects are repo-agnostic)   | No — `repo_owner` + `repo_name` are mandatory for `kind: github` | Yes |
| Can one Project contain items from 10 different repos? | Yes, natively                                 | Symphony will only see/process items from the single configured repo | Yes |
| Does the Project board know which repo each item belongs to? | Yes                                           | Symphony never asks for or uses this information | Yes |
| Should you need a "meta" repo just to satisfy the tracker? | No                                            | Effectively required today as a workaround       | Yes |

### What "Feature Parity" Would Look Like

For proper parity with GitHub, Symphony should support this configuration:

```yaml
tracker:
  kind: github
  api_key: $GH_TOKEN
  github_project_owner_type: org
  github_project_number: 3          # A repo-agnostic org project
  # repo_owner and repo_name should be optional here
```

And then be able to:

- Discover the actual repository for each item on the board
- Route workspace creation, git operations, and PRs to the correct repository per issue
- Support multiple repositories without forcing users into multiple Symphony instances or meta-repo workarounds

Currently, the code makes this impossible.

---

## Summary

This is not a limitation of GitHub.

It is a limitation in Symphony's GitHub Projects v2 implementation — specifically the assumption that every GitHub-backed Symphony instance must be permanently bound to one code repository.

Your observation about private org-level projects that are multi-repo by default is accurate and highlights the gap clearly.

This document was generated from direct source inspection in the kata monorepo.
