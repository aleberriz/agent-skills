---
name: process-git
description: Git workflow conventions covering branching model, conventional commits, PR hygiene, post-merge cleanup, and repository initialization. Use this skill whenever working with git in any capacity, from committing changes and creating branches to writing commit messages, opening pull requests, merging, pruning branches, initializing new repos, or any other version control operation. Apply these conventions by default even when the user doesn't explicitly mention them, because consistent version control practices are foundational to every project. Also use this skill when the user asks about commit message format, branching strategy, PR descriptions, or repository setup.
---

# Git Workflow

Conventions for version control that make commit history, pull requests, and branch management consistently useful for human collaborators, for AI agents, and for the author's future self.

These conventions follow [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) and are designed for solo practitioners and small teams where the analyst or engineer is the primary committer, with AI agents assisting as drafters.

## Pre-Work Check

Before making any changes, always verify:

1. What branch are you on? Run `git branch` or `git status`.
2. Is the branch up to date with remote main? Run `git fetch origin` then `git log HEAD..origin/main --oneline` to check for upstream commits you haven't pulled.

If on `main` and upstream has commits you don't have, pull first: `git pull --ff-only`. Then create a new branch for the work.

## Branching Model

Create a branch for every unit of work that produces a meaningful diff. Working directly on `main` is reserved for the initial repository commit only (LICENSE + .gitignore).

### Branch naming

```
type/short-description
```

Use the same type prefixes as commits:

| Prefix | Use when |
|--------|----------|
| `feat/` | Adding new functionality, a new skill, a new framework document |
| `fix/` | Correcting an error: broken logic, wrong formula, bad reference |
| `docs/` | Writing or revising documentation, READMEs, AGENTS.md |
| `chore/` | Maintenance that doesn't change content meaning: config, tooling, CI, dependency updates |
| `refactor/` | Restructuring without changing behavior: renaming, reorganizing, splitting files |
| `research/` | Exploratory work: notebooks, literature review, prototyping (common in data/analytics repos) |
| `test/` | Adding or improving tests, evals, validation scripts |

Keep descriptions in kebab-case: `feat/power-calculator`, `docs/semantic-layer-fundamentals`, `chore/init-config`.

## Conventional Commits

Every commit message follows [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).

### Format

Commit messages are always a single line. Details belong in the PR description, not the commit body.

```
type: imperative description of what changed
```

Or with optional scope:

```
type(scope): imperative description of what changed
```

**Example 1:**
Input: Added a new power calculator notebook for A/B testing
Output: `feat: add power calculator notebook for A/B testing`

**Example 2:**
Input: Fixed the broken link in the reading list
Output: `fix: correct broken link in reading list`

**Example 3:**
Input: Updated the gitignore to exclude workspace artifacts
Output: `chore: exclude workspace artifacts from gitignore`

**Example 4:**
Input: Restructured the semantic layer folder to separate Omni from dbt content
Output: `refactor(semantic-layer): separate Omni and dbt content into subfolders`

### Rules for the subject line

- Use imperative mood ("add", "fix", "update", not "added", "adds", "adding")
- Lowercase after the type prefix; no capital letter on the description
- No period at the end
- Keep it under 72 characters total
- One logical change per commit; if a commit message needs "and", consider splitting it
- Never add a commit body; all rationale goes in the PR description

## Pull Request Discipline

Open a PR for every branch, even in solo work. The PR is where the *why* lives.

### PR title

Use the same conventional commit format as the primary commit:

```
feat: add git-workflow skill with conventional commits and branching model
```

If the PR contains a single commit, the PR title matches the commit message. If it contains multiple commits, the PR title summarizes the overall change.

### PR description

Write in markdown. Do not insert manual line breaks to wrap text — GitHub's UI handles wrapping, and hard-wrapped text renders poorly on different screen widths.

Style rules:
- Prefer bullet points over prose
- Avoid em-dashes; use commas or split into separate sentences
- Do not add a "Testing" section unless the user explicitly requests it
- Keep writing concise; omit filler and boilerplate

Structure the description around rationale, not mechanics. The diff shows *what* changed; the description explains *why* and provides context the diff cannot convey.

For substantive PRs, use this structure:

```markdown
## Summary

[1-3 sentences: what this PR does and why it matters]

## Context

[What prompted this change: a problem observed, a new requirement, a design decision]

## Changes

[Brief bullet list of what was added/changed/removed, only when the diff is large enough that a guide is useful]

## References

[Links to issues, docs, specs, prior art: anything that informed the decisions]
```

For small PRs (config changes, typo fixes, single-file additions), a one-line summary is sufficient. Do not pad small changes with boilerplate sections.

### Merge strategy

- **Squash merge** when the branch has many small work-in-progress commits that are individually meaningless. GitHub auto-generates the squash commit title from the PR title — leave the extended description field blank; the PR description already carries the rationale.
- **Merge commit** when the branch has well-structured commits that each represent a meaningful step.

Default to squash merge for most analytical work. A single clean commit per feature keeps the main branch scannable.

When squash-merging on GitHub, the platform appends the PR number to the commit message (e.g., `fix: use --ff-only for post-merge pulls on main (#2)`). Leave this suffix — it creates a permanent cross-reference between the commit and the PR.

## Post-Merge Hygiene

After a PR is merged and the remote branch is deleted, ask the user before running local cleanup:

> "PR is merged. Want me to delete the local branch and pull the latest main?"

If the user confirms:

```bash
git checkout main
git pull --ff-only
git branch -d <merged-branch>
git fetch --prune
```

This sequence: switches to main, pulls the merged changes fast-forward only, deletes the local branch, and prunes remote-tracking references.

If the local branch was squash-merged, use `git branch -D <branch>` instead of `-d`.

`--ff-only` enforces that main is never diverged locally. If this command fails, it signals unexpected state — investigate rather than overriding.

### Never force-push to main

Force-pushing to `main` rewrites shared history. If `main` has a bad commit, revert it with a new commit. The revert itself is a documented decision.

## Authorship

Commits are authored by the human. AI agents draft content, suggest commit messages, and prepare PR descriptions, but the human reviews, edits, and commits.

When assisting a user with git operations:

- Draft the commit message (one-liner) and present it for explicit review before running any `git commit` command. Wait for the user to confirm or edit the message.
- Draft PR title and description and present them for review before running `gh pr create`. Never open a PR without the user having seen and approved the title and body.
- Never auto-commit or push without explicit user instruction
- Never change `git config user.name` or `git config user.email`

## Repository Initialization

The first branch on any new repo is `chore/init-config`. This establishes the skeleton that makes every subsequent branch productive.

### The workflow

```bash
cd ~/repos
gh repo create <owner>/<repo-name> --public --clone -d "Short description"
cd <repo-name>
```

Commit LICENSE and .gitignore directly on `main` (the only direct-to-main commit):

```bash
git add LICENSE .gitignore
git commit -m "chore: initialize repository"
git push -u origin main
```

Then branch for all configuration work:

```bash
git checkout -b chore/init-config
```

On this branch, create:

| File | Purpose |
|------|---------|
| `README.md` | What this project is, how to set it up, how to contribute |
| `AGENTS.md` | Context for AI agents: who owns the repo, conventions, repo map |
| `.gitignore` | (extend if the initial one was minimal) |
| Project config | `pyproject.toml`, `.cursor/rules/`, or whatever the project stack requires |

Commit, push, open a PR, merge, then clean up:

```bash
git push -u origin chore/init-config
gh pr create --title "chore: add project configuration and documentation" --body "..."
```

After merge, ask before local cleanup (see Post-Merge Hygiene).

## References

- [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/): the specification this skill implements
- [Git Workflow Guidelines for LLMs](https://gist.github.com/aleberriz/64b52cb082480f3edbcd6ddb17015333): the personal conventions this skill is aligned with
