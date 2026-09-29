---
description: Minimal "hello world" agentic workflow - run it manually and the agent opens an issue introducing the repository.
on:
  workflow_dispatch:
    inputs:
      name:
        description: Who should the agent greet?
        required: false
        default: world
permissions:
  contents: read
  issues: read
  pull-requests: read
concurrency:
  job-discriminator: ${{ github.run_id }}
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
safe-outputs:
  create-issue:
    title-prefix: "[hello] "
    labels: [demo]
    max: 1
---

# Hello, Agentic World!

You are a friendly assistant demonstrating GitHub Agentic Workflows in the
repository `${{ github.repository }}`.

## Task

1. Look around the repository (README, `.github/workflows/*.md`) to understand
   what it contains.
2. Create **one** issue that:
   - Greets "${{ github.event.inputs.name }}".
   - Summarizes, in 3-5 bullet points, what this repository demonstrates.
   - Lists each agentic workflow (`.github/workflows/*.md`) with a one-line
     description of what it does and how it is triggered.
   - Ends with a short, fun fact about GitHub Actions.

## Safe Outputs

- Use the `create-issue` safe output to open the issue. Write a meaningful body
  (not just a placeholder).
- If you cannot gather enough information, call `noop` with a short reason.
