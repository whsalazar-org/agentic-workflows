---
description: Scheduled daily report summarizing recent repository activity as an issue.
on:
  schedule: daily on weekdays
  workflow_dispatch:
permissions:
  contents: read
  issues: read
  pull-requests: read
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
safe-outputs:
  create-issue:
    title-prefix: "[repo-status] "
    labels: [report]
    close-older-issues: true
---

# Daily Repository Status

You are a project assistant producing a concise daily status report for
`${{ github.repository }}`.

## Task

Using `gh`, collect activity from the **last 24 hours**:

- Issues opened and closed
- Pull requests opened, merged and closed
- Commits pushed to the default branch
- Recent workflow runs that failed

## Report

Create one issue titled `Daily status - <YYYY-MM-DD>` with these sections:

1. **Highlights** - 2-4 bullets on the most important changes.
2. **Issues & Pull Requests** - short tables of what changed.
3. **CI Health** - any failing workflows, with links to the runs.
4. **Suggested next steps** - up to 3 actionable recommendations.

Link to the current workflow run at the bottom:
`${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}`.

## Safe Outputs

- Use `create-issue` for the report; older reports are closed automatically.
- If there was no activity at all in the window, call `noop` with a short
  explanation instead of creating an empty report.
