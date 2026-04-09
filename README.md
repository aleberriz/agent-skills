# agent-skills

**Reusable AI skill definitions** for version control, analytics workflows, and semantic layer design.

---

## What This Is

Skills are structured instructions that AI agents load to perform specialized tasks consistently. Instead of re-explaining conventions in every session (commit message format, branching strategy, PR hygiene, Python tooling preferences), skills encode them once and apply everywhere.

This repository contains skills designed for **data analysts and analytics engineers** who work with AI coding assistants. The conventions here are tool-agnostic: they work in any IDE (Cursor, VS Code, Zed, JetBrains), with any AI model, and across any git-based workflow.

The skills follow the [Agent Skills](https://github.com/anthropics/skills) standard (SKILL.md format with YAML frontmatter) and are installable as a [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code/plugins). They are equally readable as plain markdown by any LLM or human.

---

## Prerequisites

### System requirements

- **Git** (2.30+) with a configured identity:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  ```
- **GitHub CLI** (`gh`), authenticated and configured for SSH:
  ```bash
  gh auth login
  gh auth setup-git
  ```
- **SSH key** registered with GitHub, typically at `~/.ssh/id_ed25519` or `~/.ssh/id_rsa`. Generate one if needed:
  ```bash
  ssh-keygen -t ed25519 -C "you@example.com"
  gh ssh-key add ~/.ssh/id_ed25519.pub --title "my-machine"
  ```
- **Platform**: Linux (Ubuntu or Fedora) or macOS. The conventions are OS-agnostic but the documented commands assume a Unix shell.

### Recommended tools

- [Cursor](https://www.cursor.com/) or any AI-enabled editor
- [Poetry](https://python-poetry.org/) for Python dependency management
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) for plugin installation (optional; skills are usable without it)

---

## Installation

### As a Claude Code plugin (recommended for Claude Code users)

Register the marketplace and install:

```bash
/plugin marketplace add aleberriz/agent-skills
/plugin install core-skills@agent-skills
```

After installation, skills activate automatically based on context. Mention git, commits, branching, PRs, or any covered topic and the relevant skill loads.

### In Cursor IDE

Symlink skills into your global `~/.cursor/skills/` directory (recommended) or copy them into a project's `.cursor/skills/` directory.

**Global symlink (recommended):** the symlink points at the live repo, so every `git pull` on this repo automatically updates the installed skill — no re-copy needed.

```bash
mkdir -p ~/.cursor/skills
ln -s "$(pwd)/plugins/core-skills/skills/process-git" ~/.cursor/skills/process-git
```

**Per-project copy** (useful when you want a pinned version or don't have this repo checked out locally):

```bash
mkdir -p /path/to/your-project/.cursor/skills
cp -r plugins/core-skills/skills/process-git /path/to/your-project/.cursor/skills/

# Install an entire category
cp -r plugins/core-skills/skills/process-* /path/to/your-project/.cursor/skills/
```

When using a copy rather than a symlink, re-run the `cp` command after pulling updates from this repo to keep the installed skill current.

### In any other tool (AGENTS.md reference)

Reference the skills from your project's `AGENTS.md`:

```markdown
## Conventions

This project follows the git workflow conventions defined in
[agent-skills/process-git](https://github.com/aleberriz/agent-skills/tree/main/plugins/core-skills/skills/process-git).
```

Any AI agent that reads `AGENTS.md` (Cursor, GitHub Copilot, OpenAI Codex, Google Jules, Aider, Zed, JetBrains Junie) will follow the link and apply the conventions.

### In a team project (Claude Code)

Add to the project's `.claude/settings.json` to make the skills available automatically for everyone on the team:

```json
{
  "extraKnownMarketplaces": {
    "agent-skills": {
      "source": {
        "source": "github",
        "repo": "aleberriz/agent-skills"
      }
    }
  },
  "enabledPlugins": ["core-skills@agent-skills"]
}
```

---

## The Workflow

### Creating a new repository

```bash
cd ~/repos
gh repo create <owner>/<repo-name> --public --clone -d "Short description"
cd <repo-name>
```

Commit the initial LICENSE and .gitignore directly on `main`:

```bash
git add LICENSE .gitignore
git commit -m "chore: initialize repository"
git push -u origin main
```

Then branch for all remaining setup:

```bash
git checkout -b chore/init-config
```

On this branch, add: `README.md`, `AGENTS.md`, project configuration (`.gitignore` extensions, `pyproject.toml`, `.cursor/rules/`, etc.). Open a PR, merge, and clean up:

```bash
git push -u origin chore/init-config
gh pr create --title "chore: add project configuration and documentation" --body "..."
# After merge:
git checkout main && git pull --ff-only && git branch -d chore/init-config && git fetch --prune
```

### Working on a branch

```bash
git checkout -b feat/my-new-feature
# ... work ...
git add -A && git commit -m "feat: add my new feature"
git push -u origin feat/my-new-feature
gh pr create --title "feat: add my new feature" --body "..."
```

### After merge

```bash
git checkout main
git pull --ff-only
git branch -d feat/my-new-feature
git fetch --prune
```

See the [process-git skill](plugins/core-skills/skills/process-git/SKILL.md) for the full specification including commit format, branching model, PR description conventions, and merge strategy.

---

## Skill Catalog

Skills use prefixed names that sort alphabetically into three groups, making it easy to browse, glob, and cherry-pick by category.

### `process-*` (cross-cutting)

| Skill | Status | Description |
|-------|--------|-------------|
| [process-git](plugins/core-skills/skills/process-git/SKILL.md) | **Active** | Branching model, conventional commits, PR hygiene, post-merge cleanup, repo initialization |
| [process-scaffold](plugins/core-skills/skills/process-scaffold/SKILL.md) | Planned | Full repo scaffolding recipes for analytics and data projects |

### `tooling-*` (languages and ecosystems)

| Skill | Status | Description |
|-------|--------|-------------|
| [tooling-python](plugins/core-skills/skills/tooling-python/SKILL.md) | Planned | Poetry, Jupyter, plotnine/Plotly, pandas conventions |
| [tooling-sql](plugins/core-skills/skills/tooling-sql/SKILL.md) | Planned | Dialect-aware SQL formatting, naming, performance patterns |

### `analytics-*` (domain knowledge)

| Skill | Status | Description |
|-------|--------|-------------|
| [analytics-descriptive](plugins/core-skills/skills/analytics-descriptive/SKILL.md) | Planned | EDA, analysis structure, stakeholder communication, dashboards |
| [analytics-experimentation](plugins/core-skills/skills/analytics-experimentation/SKILL.md) | Planned | A/B testing, power analysis, sequential testing, result interpretation |
| [analytics-ml](plugins/core-skills/skills/analytics-ml/SKILL.md) | Planned | Feature engineering, model evaluation, forecasting, deployment |
| [analytics-semantic-layer](plugins/core-skills/skills/analytics-semantic-layer/SKILL.md) | Planned | AI context fields, topic scoping, metric governance (OSI-aligned) |

### Complementary skills (external)

These skills are maintained by their respective projects and cover tool-specific mechanics. The skills in this repository encode the judgment layer on top.

| Skill | Maintained by | Description |
|-------|--------------|-------------|
| [omni-claude-skills](https://github.com/exploreomni/omni-claude-skills) | Omni | Omni REST API: model exploration, querying, content building, admin |
| [Anthropic example skills](https://github.com/anthropics/skills) | Anthropic | Reference implementations for document creation, design, development |

---

## Design Principles

- **Vendor-agnostic content**: Skills are plain markdown with YAML frontmatter. The `SKILL.md` format is readable by any LLM. The `.claude-plugin/` structure enables Claude Code installation but is not required for use.
- **Explain the why**: Instructions explain reasoning, not just rules. AI models perform better when they understand intent rather than following rigid constraints.
- **Practitioner-grade**: Written for working analysts and engineers, not as tutorials. Concrete examples, linked references, opinionated defaults.
- **Composable**: Skills are independent and can be used individually or together. The analytics-semantic-layer skill builds on top of omni-claude-skills; the process-scaffold skill builds on top of process-git. Each works alone.

---

## References

- [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
- [Agent Skills specification](https://github.com/anthropics/skills/tree/main/spec)
- [Agent Skills skill creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- [Open Semantic Interchange (OSI)](https://github.com/open-semantic-interchange/OSI)
- [omni-claude-skills](https://github.com/exploreomni/omni-claude-skills)

---

## About

Built by [Alejandro Berrizbeitia](https://github.com/aleberriz), Senior Data Analyst at [Kit](https://kit.com).

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
