# Claude Code Skills

A collection of custom skills for [Claude Code](https://claude.ai/code) — the AI coding CLI by Anthropic.

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [concise-mode](skills/concise-mode/SKILL.md) | Reduces token usage ~75% by stripping filler — narration, preambles, closings — without touching reasoning, tool use, or code | [Download](concise-mode.skill) |
| [project-knowledge](skills/project-knowledge/SKILL.md) | Maintains a `.project-knowledge/` folder — one file per concern, a master index for navigation. Auto-scans on first run, orients new sessions, updates only what changed | [Download](project-knowledge.skill) |

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
project knowledge
```

## Skills Overview

### `concise-mode`
Reduces token usage ~75% by stripping verbal filler — without changing how Claude reasons, uses tools, or handles code. Cuts narration before/after tool calls and agent spawning, preambles, and closings. The answers stay identical; you just stop paying for the wrapper around them.

**Trigger:** `concise mode`
**Deactivate:** `verbose mode` or `deactivate concise`

---

### `project-knowledge`
Maintains a `.project-knowledge/` folder at your project root — one focused file per concern (`schema.md`, `routes.md`, `hooks.md`, `roadmap.md`, etc.) with a master `INDEX.md` that tells any AI session which files to load for a given task. Keeps sessions context-aware without re-explaining the project from scratch.

Four modes:
- **MIGRATE** — old `PROJECT_KNOWLEDGE.md` found → splits it into the new folder structure
- **CREATE** — no folder yet → scans codebase, generates all relevant files
- **ORIENT** — fresh session → loads `INDEX.md`, picks relevant files, gives a brief
- **UPDATE** — mid-session → detects what changed, updates only affected files

**Trigger:** `project knowledge`

---

## Contributing

PRs welcome. Each skill lives in `skills/<name>/SKILL.md`. The `.skill` files at the root are the packaged installable versions (ZIP archives).
