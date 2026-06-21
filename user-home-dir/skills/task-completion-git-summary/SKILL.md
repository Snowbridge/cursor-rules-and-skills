---
name: task-completion-git-summary
description: At task completion, lists git repos with uncommitted changes and drafts a commit message per repo in the user's format. Use when finishing tasks that modified files, after implementation/fixes, or when the user asks for commit message summaries.
---

# Task completion: changed repos and commit messages

## When to apply

At the **end of the response** if the task **modified files** in one or more git repos.

Skip if:
- read-only / Q&A / plan-only, no edits;
- no file changes.

## Steps

1. Find affected repo roots (`git rev-parse --show-toplevel` from changed paths).
2. Per repo: `git status --short` (unstaged, staged, untracked).
3. Omit repos with no changes.
4. Per changed repo: skim `git diff` + `git diff --cached`; draft commit message; **do not commit** unless asked.

## Chat output format

Append a section:

```markdown
## Changed repositories

### `<repo name or path>`

```md
**short title**
- change bullet
- ...
```
```

Rules:
- one `###` per repo;
- one commit-message block inside;
- 1–5 `-` bullets (not numbered);
- title: bold, one line;
- message language: match user's task language (default Russian);
- include section even for a single repo.

## Commit message template

Use this structure verbatim:

```md
**short title**
One to five change bullets
```

Bullets use `-`, not numbers.

## Example

```markdown
## Changed repositories

### `application/product-factory/front`

```md
**Fix classifier-linked currency edit**
- Lock charCode/numCode on edit when classifier-linked
- PUT sends only title and enactedDt — fixes HTTP 409
```
```

## Notes

- Draft only — no `git commit` without explicit request.
- Multi-repo workspace: list each repo separately.
- Mention version bumps (`package.json`, etc.) if part of the task.
