# PR Remediation Plugin

An AI agent plugin that reads open reviewer comments on a GitHub Pull Request, applies the requested code changes, and commits the fixes back to the PR branch — automatically.

## What it does

The plugin enables AI agents to handle the full PR review response loop:

- Reads all unresolved inline reviewer comments on a PR
- Understands the requested changes from natural language descriptions
- Applies the minimal code fix for each concern
- Commits all changes to the PR branch with a descriptive commit message
- Posts a reply comment on the PR summarizing what was addressed

## What's Inside

- [Skills](skills/) — reusable agent capabilities for PR remediation workflows
- [`.mcp.json`](.mcp.json) — preconfigured GitHub MCP Server with credential prompting

## Getting Started

### Prerequisites

This plugin requires the [GitHub MCP Server](https://github.com/modelcontextprotocol/servers/tree/main/src/github) (`@modelcontextprotocol/server-github`).

- **Node.js** v16 or later (for `npx`)
- A **GitHub Personal Access Token** with `repo` scope

> **Token scope:** The token needs read and write access to the repository containing the PR. Go to [GitHub Settings → Developer settings → Personal access tokens](https://github.com/settings/tokens) and create a token with the `repo` scope.

### Install

Install the plugin in Code Studio via the **Agent Customizations → Plugins** panel, or add this repository as a marketplace source:

```json
"chat.pluginMarketplaces": ["your-org/pr-remediation-plugin"]
```

During installation, Code Studio will automatically prompt you for your GitHub Personal Access Token. The token is stored encrypted — you will not be asked again.

### Usage

Ask your AI agent to address PR review comments in natural language:

> **You:** "Address the review comments on PR #42 in myorg/my-repo"
>
> **Agent:** Reads the PR, finds 3 unresolved comments, applies fixes, commits to the branch, and posts a reply.
>
> ✅ 3 review comments addressed. Commit pushed to `feature/auth-refactor`.

You can also target specific reviewers:

> **You:** "Fix the issues raised by @alice on PR #42 in myorg/my-repo"

Or preview without committing:

> **You:** "Show me what changes would be needed to address the review on PR #42 in myorg/my-repo, but don't commit yet"

## Skills

### `pr-remediation`

The core skill that orchestrates the full workflow:

1. Fetches PR metadata and all inline reviewer comments
2. Groups actionable concerns by file
3. Reads current file content from the PR branch
4. Applies minimal targeted fixes
5. Commits all changes with a structured commit message
6. Posts a review reply summarizing addressed and skipped comments

See [`skills/pr-remediation/SKILL.md`](skills/pr-remediation/SKILL.md) for full documentation.

## Security

- Your GitHub token is stored in **encrypted secret storage** by Code Studio — it is never written to disk in plaintext.
- The plugin only reads and writes to repositories you have explicit access to via your token.
- The agent will never force-push, rebase, or merge a PR — only push fix commits.

## Contributing

We welcome contributions! Please read our [Contributing Guide](CONTRIBUTING.md) for details.

## License

This project is licensed under the [MIT License](LICENSE).
