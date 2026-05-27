# Draft Upstream Issue Text

**Copy the content below (starting from the line after this one) to post as a new issue in https://github.com/gannonh/kata**

---

**Title:**

Symphony GitHub adapter hard-requires a single repository even for Projects v2 boards that span multiple repos

**Body:**

### Problem

GitHub Projects v2 (especially organization-level projects) are deliberately repo-agnostic. You can create a project such as `EntityProcess/projects/3` that has no primary repository attached, and freely add issues and pull requests from any number of repositories the organization has access to.

Kata Symphony's current GitHub integration (`kind: github` + Projects v2) does not support this model. The tracker configuration requires both `repo_owner` and `repo_name`:

```yaml
tracker:
  kind: github
  repo_owner: ...
  repo_name: ...
  github_project_owner_type: org
  github_project_number: 3
```

When Symphony polls the project board, it only successfully processes items whose issue numbers exist in the single configured repository. Items from other repositories on the same board are either ignored (404) or never fetched properly.

See the detailed technical analysis here:
https://github.com/christso/kata/pull/1

### Current Behavior

- Symphony can read the list of items from a Projects v2 board.
- For every item, it calls the GitHub REST API against the single `repo_owner/repo_name` configured in the workflow.
- Any item belonging to a different repository is skipped (treated as missing).
- The Projects v2 GraphQL query used by Symphony does not request the repository for each item.
- The `GithubClient` and many helpers are fundamentally bound to one repository.

This means true multi-repository Projects v2 boards are effectively unusable with Symphony today.

### Expected Behavior

Symphony should be able to work with GitHub Projects v2 boards that contain work from multiple repositories, matching GitHub's own capabilities.

At minimum:
- `repo_owner` / `repo_name` should be optional when using `github_project_number`.
- Symphony should discover the actual repository for each board item.
- Subsequent operations (fetching issue details, creating branches/PRs, updating state, etc.) should be routed to the correct repository for that item.

### Why This Matters

Many teams want a single high-level project board for planning and visibility across multiple code repositories. Forcing either:
- one Symphony instance per repository, or
- a "meta" repository that holds all dispatchable issues

...adds unnecessary friction and goes against how GitHub Projects are designed to be used at organization scale.

### Possible Direction

A reasonable fix would involve:
- Updating the Projects v2 query to capture repository information per item
- Making the GitHub client/adapter capable of operating across multiple repositories (or resolving per-item clients)
- Updating config validation, helpers, workspace bootstrap logic, and observability to support per-item repositories
- Preserving backward compatibility for existing single-repo configurations

This is a non-trivial refactor but appears to be a natural evolution of the GitHub Projects v2 support.

---

**Suggested labels (if you have permission to set them when creating):**
- `enhancement`
- `symphony`
- `github`

**After posting**, you can link this new upstream issue back to your draft PR in christso/kata, and vice versa.
