# agentic-workflows

A hands-on demo of **writing and executing [GitHub Agentic Workflows](https://github.github.com/gh-aw/)** (`gh-aw`).

Agentic workflows are written in **Markdown**: YAML frontmatter configures the trigger, permissions,
tools and allowed outputs, and the Markdown body is a natural-language prompt for an AI agent.
The `gh aw` CLI compiles each `.md` file into a regular GitHub Actions workflow (`.lock.yml`) that
runs the agent in a sandboxed, read-only job. Any writes (issues, comments, labels, …) are done by
separate, validated **safe-outputs** jobs.

```
.github/workflows/my-workflow.md  ──gh aw compile──▶  .github/workflows/my-workflow.lock.yml  ──▶  GitHub Actions
   (you write this)                                      (generated, commit it too)
```

## What's in this repository

| Workflow (source) | Trigger | What the agent does | Safe outputs |
| --- | --- | --- | --- |
| [`hello-agentic-world.md`](.github/workflows/hello-agentic-world.md) | Manual (`workflow_dispatch`) | Explores the repo and opens an issue greeting you and describing the demo | `create-issue` |
| [`issue-triage.md`](.github/workflows/issue-triage.md) | Issue opened / reopened, or manual | Classifies the issue, looks for duplicates, and comments | `add-labels`, `add-comment` |
| [`daily-repo-status.md`](.github/workflows/daily-repo-status.md) | Weekday schedule, or manual | Summarizes the last 24h of activity in a status-report issue (older reports auto-closed) | `create-issue` |
| [`code-improvement.md`](.github/workflows/code-improvement.md) | New issue opened | Investigates actionable workflow/CI defects and proposes a focused, validated draft fix (one open proposal at a time) | `create-pull-request` |
| [`validate-agentic-workflows.yml`](.github/workflows/validate-agentic-workflows.yml) | PR / push touching workflows | Regular (non-agentic) CI: recompiles all agentic workflows and fails if `.lock.yml` files are stale | — |

Each `*.md` file has a matching, generated `*.lock.yml` file. `.gitattributes` marks the lock files as
generated so they are collapsed in diffs.

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`), authenticated with `gh auth login`
- The `gh-aw` extension:

  ```bash
  gh extension install github/gh-aw
  gh aw version
  ```

- A token for the AI engine. The demo workflows use the default engine, **GitHub Copilot**, which
  reads a fine-grained PAT from the `COPILOT_GITHUB_TOKEN` repository secret. Set it with the helper:

  ```bash
  gh aw secrets bootstrap        # interactive: detects and sets required secrets
  # or manually:
  gh secret set COPILOT_GITHUB_TOKEN
  ```

  See [engines](https://github.github.com/gh-aw/reference/engines/) for Claude, Codex, Gemini and others.

- To let `code-improvement` propose changes to workflow files, install a GitHub App
  with Contents, Issues, Pull requests, and Workflows write permissions. Set its client ID
  in `GH_AW_APP_CLIENT_ID` (repository variable) and private key in
  `GH_AW_APP_PRIVATE_KEY` (repository secret). The agent itself remains read-only;
  the app is used only by the `create-pull-request` safe output.

## 1. Write a workflow

Create a Markdown file under `.github/workflows/` (or scaffold one with `gh aw new my-workflow`):

```markdown
---
description: Say hello by opening an issue.
on:
  workflow_dispatch:
permissions:
  contents: read          # the agent job is read-only
tools:
  github:
    mode: gh-proxy        # the agent reads GitHub data with `gh`
    toolsets: [default]
safe-outputs:
  create-issue:           # the only write the agent is allowed to request
    title-prefix: "[hello] "
---

# Hello

Open an issue that greets the team and summarizes this repository.
If there is nothing to say, call `noop` with a short reason.
```

Key frontmatter fields:

- **`on:`** – any GitHub Actions trigger, plus friendly forms such as `schedule: daily on weekdays`.
- **`permissions:`** – keep these read-only; writes go through `safe-outputs`.
- **`tools:`** – what the agent can use (GitHub, web-fetch, Playwright, MCP servers, …).
- **`safe-outputs:`** – the allow-list of GitHub writes (create issue, add comment, add labels, open PR, …).
- **`engine:`** – optional; omit to use the default (Copilot).

Full reference: <https://github.github.com/gh-aw/reference/frontmatter/>

## 2. Compile it

```bash
gh aw compile                 # compile every .github/workflows/*.md
gh aw compile issue-triage    # compile one workflow
gh aw compile --actionlint    # also lint the generated YAML
```

Commit **both** the `.md` source and the generated `.lock.yml`. Re-run `gh aw compile` whenever you edit
the frontmatter. (Edits to the Markdown body only are picked up at runtime without recompiling.)

> This repository compiles with `--schedule-seed whsalazar-org/agentic-workflows` so that the
> "scattered" cron time chosen for `schedule: daily on weekdays` is deterministic and matches CI.

## 3. Run it

```bash
gh aw run hello-agentic-world                 # trigger a workflow_dispatch run
gh aw run hello-agentic-world --raw-field name=Octocat # pass inputs
gh aw run issue-triage --raw-field issue_number=1
```

…or use the **Actions** tab → select the workflow → **Run workflow**. To see `issue-triage` react
automatically, just open a new issue.

## 4. Inspect the results

```bash
gh aw status                   # list agentic workflows and their state
gh aw logs hello-agentic-world # download and summarize recent run logs (tokens, cost, tool calls)
gh aw audit <run-id-or-url>    # deep-dive into a single run
```

## Learn more

- Docs: <https://github.github.com/gh-aw/>
- Quick start: <https://github.github.com/gh-aw/setup/quick-start/>
- Examples: <https://github.github.com/gh-aw/examples/>
- Security architecture: <https://github.github.com/gh-aw/introduction/architecture/>
- Source: <https://github.com/github/gh-aw> · More sample workflows: <https://github.com/githubnext/agentics>

> ⚠️ Agentic workflows run AI agents in your repository. Review permissions, tools, network access and
> safe outputs before enabling them, and supervise their output.
