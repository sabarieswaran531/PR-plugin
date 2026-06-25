# Support

## How to get help

- **Bug reports & feature requests**: Open an issue on the [GitHub repository](https://github.com/your-org/pr-remediation-plugin/issues).
- **Questions**: Use [GitHub Discussions](https://github.com/your-org/pr-remediation-plugin/discussions) for general questions and usage help.

## Before opening an issue

1. Check the [README](README.md) for setup and usage instructions.
2. Verify your GitHub token has the `repo` scope.
3. Confirm the GitHub MCP Server is running (`npx @modelcontextprotocol/server-github` requires Node.js v16+).
4. Check that the PR is open and on a branch you have push access to.

## Common Issues

| Problem | Solution |
|---|---|
| "No open review comments found" | The PR may have all comments resolved already, or the PR number/repo is incorrect |
| "Push failed" | Check your token has write access; check for branch protection rules |
| Token prompt not appearing | Uninstall and reinstall the plugin to re-trigger credential collection |
| MCP server not starting | Ensure Node.js v16+ is installed: `node --version` |
