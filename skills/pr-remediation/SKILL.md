---
name: pr-remediation
description: Read open reviewer comments on a GitHub Pull Request, apply the requested code changes, and commit the fixes back to the PR branch.
allowed-tools: Bash(git:*) Read
---

# PR Remediation Skill

## Overview

This skill reads unresolved reviewer comments on a GitHub Pull Request, understands the requested changes, applies the minimal code fixes, and commits them back to the PR branch. It uses the GitHub MCP Server for all GitHub API operations.

**Important**: Only use this skill when a user explicitly asks to address PR review comments. Do not run this unprompted or as part of general workflows.

---

## Prerequisites

- The **GitHub MCP Server** must be configured and running (provided by `.mcp.json` in this plugin).
- The GitHub token must have `repo` scope (read + write access to the repository).
- The PR must be open and on a branch you have push access to.

---

## Common Scenarios

| User goal | How to respond | Tools needed |
|---|---|---|
| Address all review comments on a PR | Fetch PR + comments → apply fixes → commit | GitHub MCP |
| Address a specific reviewer's comments | Filter comments by reviewer → apply targeted fixes → commit | GitHub MCP |
| Preview what changes would be made | Fetch PR + comments → describe planned changes, do NOT commit | GitHub MCP |
| Reply to reviewer after fixing | Apply fixes → commit → post review reply | GitHub MCP |

---

## Workflow

### Step 1 — Gather PR Context

Use the GitHub MCP Server tools to collect all necessary information:

1. Call `get_pull_request` with `owner`, `repo`, and `pullNumber` to read:
   - PR title, description, base branch, head branch
   - Current PR state (must be `open`)

2. Call `list_pull_request_comments` to retrieve all inline review comments.
   - Focus on comments that are **not yet resolved** and contain actionable change requests.
   - Ignore: praise, questions without a clear action, automated bot comments.

3. Call `get_pull_request_files` to get the list of changed files and their patches.

**If the user has not provided `owner`, `repo`, or `pullNumber`, ask for them before proceeding.**

---

### Step 2 — Parse and Group Reviewer Concerns

For each unresolved comment:

- Extract: `path` (file), `line` (line number), `body` (the concern text), `user.login` (reviewer)
- Classify the concern:
  - **Actionable** — reviewer explicitly requests a code change (rename, refactor, fix logic, add null check, etc.)
  - **Question** — reviewer asks for clarification; post a reply instead of changing code
  - **Nitpick** — minor style suggestion; apply only if the user confirms

Group actionable concerns by file path for efficient processing.

---

### Step 3 — Read Current File Content

For each file with actionable concerns:

1. Call `get_file_contents` with `owner`, `repo`, `path`, and `ref` set to the PR head branch.
2. Decode the base64 content to get the current file text.
3. Identify the exact lines referenced by each comment.

---

### Step 4 — Apply Fixes

For each actionable concern:

- Make the **minimal change** that directly addresses the reviewer's request.
- Do **not** change unrelated code, formatting, or logic outside the concern's scope.
- If a concern is ambiguous or could be interpreted multiple ways, choose the most conservative interpretation and note it in the commit message.
- If a concern requires understanding broader context (e.g., a refactor spanning many files), describe the plan to the user and ask for confirmation before proceeding.

**Hard rules:**
- Never delete files unless the reviewer explicitly requested it.
- Never change files not referenced in any reviewer comment.
- Never rebase, force-push, or alter commit history.
- Never merge the PR — only push fix commits to the branch.

---

### Step 5 — Commit the Changes

Use `push_files` (or `create_or_update_file` for single-file changes) to commit all fixes to the PR's head branch.

**Commit message format:**
```
fix: address PR review comments

- <file1>: <one-line summary of what was changed and why>
- <file2>: <one-line summary>

Addresses comments by: @<reviewer1>, @<reviewer2>
PR: #<pullNumber>
```

---

### Step 6 — Post a Review Reply

After committing, call `create_pull_request_review` to post a summary comment on the PR:

```
Addressed the following review comments:

- **<file>** (line <N>): <what was changed> — @<reviewer>
- **<file>** (line <N>): <what was changed> — @<reviewer>

Changes committed to branch `<head-branch>` in commit <sha>.
```

If any comments were **not** addressed (ambiguous, out of scope, or require human decision), list them explicitly:

```
The following comments were not automatically addressed and require your attention:
- <file> line <N>: <reason why it was skipped>
```

---

## Error Handling

| Situation | Action |
|---|---|
| PR is not open (merged/closed) | Inform the user, do not proceed |
| No unresolved comments found | Inform the user: "No open review comments found on PR #N" |
| File not found on head branch | Skip the file, report it in the reply comment |
| Push fails (permissions, branch protection) | Report the error clearly; suggest the user check token scope or branch protection rules |
| Ambiguous comment | Ask the user for clarification before applying any change |

---

## Example Usage

> **User:** "Address the review comments on PR #42 in owner/my-repo"
>
> **Agent:**
> 1. Fetches PR #42 — open, branch `feature/auth-refactor`
> 2. Finds 3 unresolved comments across 2 files
> 3. Reads both files from the head branch
> 4. Applies fixes: renames variable, adds null check, removes unused import
> 5. Commits to `feature/auth-refactor` with message `fix: address PR review comments`
> 6. Posts review reply listing all addressed comments
>
> ✅ 3 review comments addressed. Commit pushed to `feature/auth-refactor`.

---

## Required GitHub MCP Tools

| Tool | Purpose |
|---|---|
| `get_pull_request` | Read PR metadata (title, branch, state) |
| `list_pull_request_comments` | Fetch all inline reviewer comments |
| `get_pull_request_files` | List changed files and their diffs |
| `get_file_contents` | Read current file content from the head branch |
| `push_files` | Commit one or more file changes to the branch |
| `create_or_update_file` | Commit a single file change |
| `create_pull_request_review` | Post a summary reply on the PR |
