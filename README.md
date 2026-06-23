# Claude Code Skills

A collection of custom skills for [Claude Code](https://claude.ai/code) — the AI coding CLI by Anthropic.

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [concise-mode](skills/concise-mode/SKILL.md) | Reduces token usage ~75% by stripping filler — narration, preambles, closings — without touching reasoning, tool use, or code | [Download](concise-mode.skill) |
| [project-knowledge](skills/project-knowledge/SKILL.md) | Maintains a living `PROJECT_KNOWLEDGE.md` file — auto-scans your codebase, tracks changes, and keeps AI sessions context-aware across conversations | [Download](project-knowledge.skill) |

## Installation

### Option 1 — Install from this repo
```bash
# Clone the repo
git clone https://github.com/YahyaZekry/claude-code-skills.git

# Install a skill (from the repo root)
claude skill install ./claude-code-skills/concise-mode.skill
```

### Option 2 — Direct download
Download the `.skill` file and install:
```bash
claude skill install concise-mode.skill
```

## Usage

Once installed, invoke a skill by name in Claude Code:

```
concise mode
```
```
update project knowledge
```

## Skills Overview

### `concise-mode`
Reduces token usage ~75% by stripping verbal filler from Claude's responses — without changing how Claude reasons, uses tools, or handles code. Cuts narration, preambles, and closings. The answers stay identical; you just stop paying for the wrapper around them.

**Trigger:** say `concise mode`  
**Deactivate:** say `verbose mode` or `deactivate concise`

---

### `project-knowledge`
Scans your project (file tree, stack, routes, hooks, components, env vars) and builds a `PROJECT_KNOWLEDGE.md` file at the root. Updates it as you work. Orients new sessions instantly without re-explaining context.

**Trigger:** say `update project docs`, `log what we did`, `sync project knowledge`, or `start a new session`

---

## Contributing

PRs welcome. Each skill lives in `skills/<name>/SKILL.md`. The `.skill` files at the root are the packaged installable versions (ZIP archives).
