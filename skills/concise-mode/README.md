# concise-mode

A Claude Code skill that toggles a low-verbosity conversation mode. Cuts filler, narration, and closings — keeps full technical accuracy.

## What it does

Once active, Claude:
- Fires tool calls without narrating them ("Now let me...", "I'll check...")
- Responds in 1–3 sentences for questions
- Returns code + 1 line for code tasks
- Skips preambles ("Sure!", "Great question!") and closings ("Let me know if...")
- Never truncates code or omits critical warnings

## Usage

```
concise mode        → activates (stays on for entire conversation)
verbose mode        → deactivates
deactivate concise  → deactivates
```

## Install

```bash
claude skill install concise-mode.skill
```

## Source

See [SKILL.md](SKILL.md) for the full skill definition.
