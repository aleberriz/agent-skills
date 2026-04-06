---
name: process-git
description: Git workflow conventions covering branching model, conventional commits, PR hygiene, post-merge cleanup, and repository initialization. Use this skill whenever working with git in any capacity, from committing changes and creating branches to writing commit messages, opening pull requests, merging, pruning branches, initializing new repos, or any other version control operation. Apply these conventions by default even when the user doesn't explicitly mention them, because consistent version control practices are foundational to every project. Also use this skill when the user asks about commit message format, branching strategy, PR descriptions, or repository setup.
---

# Git Workflow

Conventions for version control that make commit history, pull requests, and branch management consistently useful for human collaborators, for AI agents, and for the author's future self.

These conventions follow [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) and are designed for solo practitioners and small teams where the analyst or engineer is the primary committer, with AI agents assisting as drafters. The principles are tool-agnostic: they apply in any IDE, any git client, any CI/CD system.

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

### Why branch for everything

The branch-per-topic model creates a clean PR history where each merge represents a complete, reviewable unit of work. This matters more in an AI-assisted workflow because the PR description is where human judgment is documented. The branch and its PR are the unit of accountability.

## Conventional Commits

Every commit message follows [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).

### Format

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

### When to add a body

Most commits need only the subject line. The PR description carries the rationale.

Add a commit body only when all three conditions are true:
1. The *why* behind this specific commit is non-obvious
2. The PR description doesn't already cover it (because the PR has many commits and this one needs its own context)
3. Someone might accidentally undo this change without understanding the constraint it addresses

Separate the body from the subject with a blank line. Write in plain prose, not bullet points. Do not restate what the diff already shows.

```
fix: cap LOD expression at account grain

The original expression computed at transaction grain, which produced correct totals but inflated per-account averages by 3-4x when joined to the accounts topic. Capping at account grain matches the documented metric definition.
```

### When to add a footer

Use `BREAKING CHANGE:` in the footer when a commit changes something that downstream consumers depend on, such as a renamed skill, a removed field, or a changed API. This is rare in documentation and analytics repos but important when it happens.

## Pull Request Discipline

Open a PR for every branch, even in solo work. The PR is where the *why* lives.

### PR title

Use the same conventional commit format as the primary commit:

```
feat: add git-workflow skill with conventional commits and branching model
```

If the PR contains a single commit, the PR title matches the commit message. If it contains multiple commits, the PR title summarizes the overall change.

### PR description

Write in markdown. Do not insert manual line breaks to wrap text. GitHub's UI handles wrapping, and hard-wrapped text renders poorly on different screen widths.

Structure the description around rationale, not mechanics. The diff shows *what* changed; the description explains *why* and provides context that the diff cannot convey.

For substantive PRs, use this structure:

```markdown
## Summary

[1-3 sentences: what this PR does and why it matters]

## Context

[What prompted this change: a problem observed, a new requirement, a design decision]

## Changes

[Brief list of what was added/changed/removed, only when the diff is large enough that a guide is useful]

## References

[Links to issues, docs, specs, prior art: anything that informed the decisions]
```

For small PRs (config changes, typo fixes, single-file additions), a one-line summary is sufficient. Do not pad small changes with boilerplate sections.

### Merge strategy

Use the merge strategy that preserves the most useful history for the project:

- **Squash merge** when the branch has many small work-in-progress commits that are individually meaningless. The squashed commit message should match the PR title.
- **Merge commit** when the branch has well-structured commits that each represent a meaningful step. The merge commit preserves the full commit sequence.

Default to squash merge for most analytical work. The PR description captures the rationale, and a single clean commit per feature keeps the main branch scannable.

## Post-Merge Hygiene

After a PR is merged on the remote, clean up immediately. Stale branches create noise and ambiguity.

```bash
git checkout main
git pull --ff-only
git branch -d <merged-branch>
git fetch --prune
```

This sequence: switches to main, pulls the merged changes, deletes the local branch (which is now fully merged), and prunes remote-tracking references to branches that no longer exist on the remote.

`--ff-only` enforces that main is never diverged locally. If this command fails, it signals an unexpected state — investigate rather than overriding it. In this workflow, a fast-forward pull is always valid because main receives no direct commits after initialization.

If the local branch wasn't fully merged (e.g., it was squash-merged), use `git branch -D <branch>` instead of `-d`.

### Never force-push to main

Force-pushing to `main` rewrites shared history. There is no valid reason to do this in a properly managed repository. If `main` has a bad commit, revert it with a new commit. The revert itself is a documented decision.

## Authorship

Commits are authored by the human. AI agents draft content, suggest commit messages, and prepare PR descriptions, but the human reviews, edits, and commits.

When assisting a user with git operations:

- Draft commit messages following these conventions, then present them for review
- Draft PR descriptions, then let the user edit and submit
- Never auto-commit without explicit user instruction
- Never change `git config user.name` or `git config user.email`
- When the user says "commit this", prepare the `git add` and `git commit` commands with the drafted message and execute only after the user confirms or the conversation context makes their intent unambiguous

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

After merge:

```bash
git checkout main
git pull --ff-only
git branch -d chore/init-config
git fetch --prune
```

### Why this pattern

The init branch creates a PR that documents the foundational decisions (license choice, tooling choices, conventions) in a reviewable, linkable artifact. Starting with a clean PR history from commit zero signals intentional project management.

## References

- [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/): the specification this skill implements
- [Anthropic Agent Skills](https://github.com/anthropics/skills): the skills ecosystem and best practices for skill design
- [Open Semantic Interchange (OSI)](https://github.com/open-semantic-interchange/OSI): vendor-agnostic semantic model standard, referenced here because these git conventions were designed with OSI-aligned analytics projects in mind
- [fvadicamo/dev-agent-skills](https://github.com/fvadicamo/dev-agent-skills): community git workflow skills for Claude Code (software-dev focused; monitor for complementary patterns)
