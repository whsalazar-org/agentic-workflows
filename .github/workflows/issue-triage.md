---
description: Triage newly opened issues - classify them with a label and post a helpful comment.
on:
  issues:
    types: [opened, reopened]
  workflow_dispatch:
    inputs:
      issue_number:
        description: Issue number to triage manually
        required: true
permissions:
  contents: read
  issues: read
  pull-requests: read
concurrency:
  job-discriminator: ${{ github.event.issue.number || github.event.inputs.issue_number }}
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
safe-outputs:
  add-labels:
    allowed: [bug, enhancement, documentation, question]
    max: 2
  add-comment:
    max: 1
---

# Issue Triage

You are a triage assistant for the repository `${{ github.repository }}`.

## Task

1. Read the issue that triggered this run. It is issue
   #${{ github.event.issue.number || github.event.inputs.issue_number }}.
   Fetch its title, body and existing labels with `gh`.
2. Decide which single category best fits the issue:
   - `bug` - something is broken or behaves unexpectedly
   - `enhancement` - a request for a new feature or improvement
   - `documentation` - docs are missing, wrong or unclear
   - `question` - the author is asking for help or clarification
3. Search for existing open issues that look like duplicates.

## Safe Outputs

- Use `add-labels` to apply the chosen category label (only from the allowed
  list above).
- Use `add-comment` to post a short, friendly comment that:
  - thanks the author,
  - explains why you chose the label,
  - links any likely duplicate issues,
  - suggests what additional information (if any) would help maintainers.
- If the issue was created by a bot, is already labeled with one of the
  allowed labels, or is empty, call `noop` with a short reason instead.
