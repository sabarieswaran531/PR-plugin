# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| 1.x     | ✅ Yes    |

## Reporting a Vulnerability

If you discover a security vulnerability in this plugin, please **do not** open a public GitHub issue.

Instead, please report it privately via [GitHub Security Advisories](https://github.com/your-org/pr-remediation-plugin/security/advisories/new) or email the maintainers directly.

We will acknowledge your report within 48 hours and aim to release a fix within 7 days for critical issues.

## Security Considerations

- **GitHub Token**: Your Personal Access Token is stored in Code Studio's encrypted secret storage. It is never written to disk in plaintext and never included in commits or logs.
- **Scope**: The plugin only accesses repositories that your token has explicit permission to read and write. Use a token with the minimum required scope (`repo`).
- **No force-push**: The plugin will never force-push, rebase, or alter commit history. It only appends new commits to the existing PR branch.
- **No auto-merge**: The plugin never merges a PR. All merges remain under human control.
