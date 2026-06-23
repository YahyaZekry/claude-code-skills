# project-knowledge

A Claude Code skill that maintains a living `PROJECT_KNOWLEDGE.md` file at your project root. Auto-scans your codebase on first run, tracks changes each session, and keeps AI context accurate across conversations.

## What it does

**On first run (CREATE mode):**
- Scans file tree, detects stack, frameworks, and entry points
- Greps for systems: auth, DB, email, payments, storage, queues, realtime, AI
- Inventories every server action / API route, hook / service, and component
- Extracts all environment variables and dev commands
- Asks only what it couldn't infer, then writes the full file

**Each subsequent session (UPDATE mode):**
- Detects changed files since last update
- Infers likely changes from file names and git log
- Appends a session log entry with date and summary
- Never deletes history — marks deprecated items with strikethrough

**New session orientation (READ mode):**
- Reads the file and gives a <150-word briefing
- Covers: what the project is, stack, what's built, last session, next step

## Usage

Trigger phrases:
```
update project docs
log what we did
sync project knowledge
start a new session
what have we done so far
```

Also triggers automatically at session end when you say `we're done` or `that's it for now`.

## Output

Creates/updates `PROJECT_KNOWLEDGE.md` at your project root with sections:
- Tech Stack, Project Structure, Database Schema
- Server Actions / API Routes, Hooks & Services, Component Inventory
- Environment Variables, Dev Commands
- External Integrations, Systems, Features, Workflows
- Removed, Fixed, Known Issues, Decisions, Session Log

## Install

```bash
claude skill install project-knowledge.skill
```

## Source

See [SKILL.md](SKILL.md) for the full skill definition.
