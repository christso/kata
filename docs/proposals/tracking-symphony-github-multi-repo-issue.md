# Tracking Issue: GitHub Projects v2 Multi-Repository Support Gap in Symphony

**Status**: Open for discussion

## Summary

GitHub Projects v2 supports creating organization-level projects that are not tied to any single repository (e.g. `EntityProcess/projects/3`). These projects can aggregate issues and PRs from many repositories by design.

Kata Symphony's current implementation for `kind: github` + Projects v2 hard-requires a single `repo_owner` + `repo_name`. This prevents proper use of true multi-repo GitHub Project boards.

## Draft PR with Full Analysis

- https://github.com/christso/kata/pull/1
- `docs/proposals/symphony-github-multi-repo-gap.md`

## Proposed Paths Forward

1. **Implement proper support in kata** (recommended long-term)
   - Make repo fields optional when using Projects v2
   - Capture repository per item from the board
   - Support per-item workspace/PR routing

2. **Workarounds (current state)**
   - Use a dedicated "meta/workspace" repo that holds all dispatchable issues + `.symphony/` config
   - Run multiple Symphony instances (one per code repo) against the same board with scoping
   - Use Linear as the tracker for multi-repo cases (best current parity)

## Action Items

- [ ] Review gap analysis
- [ ] Decide priority vs other Symphony improvements
- [ ] If implementing: create upstream issue/PR against gannonh/kata

## References

- Upstream kata: https://github.com/gannonh/kata/tree/main/apps/symphony
- Internal discussion context available in orchestration workspace

**Note**: This file exists because Issues were disabled on the repository at the time of initial setup. Once Issues are enabled in repo settings, a real GitHub Issue should be created linking to this PR.