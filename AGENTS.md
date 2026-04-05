# AGENTS.md

Context file for AI agents working in this repository.
Supported by Cursor, OpenAI Codex, GitHub Copilot, Google Jules, Aider, Zed, JetBrains Junie, and others.

---

## Who Owns This Repo

**Alejandro Berrizbeitia**, Senior Data Analyst at Kit (formerly ConvertKit).
GitHub: [@aleberriz](https://github.com/aleberriz)

---

## What This Repo Is

A public collection of reusable AI skill definitions following the [Agent Skills](https://github.com/anthropics/skills) standard. Skills are markdown files with YAML frontmatter that teach AI agents how to perform specialized tasks consistently.

The skills here are designed for data analysts and analytics engineers. They encode workflow conventions (git practices, Python tooling, semantic layer authoring, analytical methodology) so that they don't need to be re-explained in every session.

The repo doubles as a Claude Code plugin marketplace (`.claude-plugin/` structure) but every skill is also usable as standalone markdown by any tool.

---

## Repository Map

```
agent-skills/
├── .claude-plugin/
│   └── marketplace.json                              # Claude Code marketplace catalog
├── plugins/
│   └── core-skills/
│       ├── .claude-plugin/
│       │   └── plugin.json                           # Plugin manifest listing all skills
│       └── skills/
│           ├── analytics-descriptive/SKILL.md        # Domain: EDA, dashboards, comms (planned)
│           ├── analytics-experimentation/SKILL.md    # Domain: A/B testing, power analysis (planned)
│           ├── analytics-ml/SKILL.md                 # Domain: prediction, evaluation (planned)
│           ├── analytics-semantic-layer/SKILL.md     # Domain: AI context, OSI, governance (planned)
│           ├── process-git/SKILL.md                  # Cross-cutting: branching, commits, PRs
│           ├── process-scaffold/SKILL.md             # Cross-cutting: repo initialization (planned)
│           ├── tooling-python/SKILL.md               # Tooling: Poetry, Jupyter, viz (planned)
│           └── tooling-sql/SKILL.md                  # Tooling: formatting, naming, dialects (planned)
├── README.md
├── AGENTS.md                                         # This file
├── LICENSE                                           # Apache 2.0
└── .gitignore
```

Skills are organized with prefixes along three axes. The prefix groups skills alphabetically in file listings and makes category-level operations simple (`cp process-* ...`, reference all `analytics-*` in AGENTS.md):

- **`process-`**: cross-cutting conventions that apply to any project regardless of domain
- **`tooling-`**: universal Python and SQL conventions, independent of domain
- **`analytics-`**: subject-specific judgment and methodology

---

## Conventions

### Git

Follow the [process-git skill](plugins/core-skills/skills/process-git/SKILL.md), the authoritative source for this repo's own git practices:

- **Branches**: `type/short-description`, one branch per unit of work
- **Commits**: [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/), `type: imperative description`, single subject line, body only when the *why* is non-obvious
- **PRs**: always opened, even for solo work; PR description carries the rationale
- **Authorship**: all commits authored by Alejandro. AI agents draft content; the human reviews and commits. Never auto-commit.

### Content

- Skills follow the [SKILL.md format](https://github.com/anthropics/skills/tree/main/spec): YAML frontmatter (`name`, `description`) plus markdown body
- Descriptions should be explicit about triggering contexts, per [skill-creator guidance](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- Keep SKILL.md files under 500 lines; use bundled `references/` for overflow
- Write in imperative form; explain the *why* behind instructions rather than relying on rigid rules
- No proprietary content: general frameworks and public references only

### Skill development

When creating or improving skills in this repo, consult the [Anthropic skill-creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) for current best practices on structure, testing, description optimization, and iterative improvement.

---

## Working in This Repo

When drafting skill content, default to:

- Concise, practitioner-grade writing, not academic, not tutorial-style
- Concrete examples over abstract principles
- Linking to primary sources rather than paraphrasing them
- Vendor-agnostic framing: tools and IDEs are examples, not requirements

When suggesting commits or PR descriptions, draft them for Alejandro to review and edit. Do not auto-commit.
