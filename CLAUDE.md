# Agent Instructions

These instructions apply to all coding agents in this repository (Cursor, Claude Code, Codex).

## Pull Request issues

**Failed tests**: When the user mentions failed tests, fetch the PR's failed checks and inspect the relevant workflow runs. The `link` field contains the run ID in the format `.../actions/runs/<RUN_ID>/jobs/...`:

```bash
gh pr checks <PR_NUMBER> --json name,state,link
gh run view <RUN_ID> --log-failed
```

**Review comments**: When the user mentions comments, use `gh api graphql` to query `pullRequest.reviewThreads` and focus on threads where `isResolved` is `false`. The REST API does not expose resolution state. GraphQL `PullRequestReviewThread.isResolved` is required.
