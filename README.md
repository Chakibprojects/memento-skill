# Memento

**Cognitive persistence for Claude Code.** Never lose your reasoning again.

## The Problem

You close your Claude Code session. Tomorrow, your AI assistant is a stranger again.

- The code is on Git. The decisions? Lost.
- "Why did we choose this architecture?"
- "What did we try before?"

## The Solution

Memento makes Claude write its own "work memories" for its future self.

### The Graveyard 🪦

A cumulative file of rejected approaches — with **WHY** they failed.

```markdown
## Native Unity Physics for Jump
**Why it failed:** Lag at high velocities, inconsistent ground detection
**Do instead:** Custom raycast system
```

In 6 months, when you return to the project:
- Claude reads the Graveyard
- It **never** suggests those approaches again
- You don't waste 2 hours repeating the same mistakes

## Installation

```bash
npx skills add https://github.com/Chakibprojects/memento-skill --skill memento
```

## Usage

```bash
# Start of session — restore cognitive context
/memento hydrate

# End of session — save decisions & learnings
/memento checkpoint

# View current state (read-only)
/memento status
```

## What Gets Captured

| Element | Description |
|---------|-------------|
| **Decisions** | Architectural choices with reasoning |
| **Graveyard** | Rejected approaches + WHY (cumulative) |
| **Tech Debt** | Conscious compromises to revisit later |
| **User Context** | Your preferences, skill level |
| **Next Actions** | Where to pick up |

## File Structure

```
.claude/
└── chronicles/
    ├── latest.md      # Current state (overwritten)
    ├── graveyard.md   # Forbidden approaches (append-only)
    └── sessions/      # Archived chronicles
```

## Philosophy

- **Graveyard is absolute** — Never suggest a rejected approach
- **Silence is Golden** — Minimal terminal output, detailed files
- **Signal over noise** — Distilled reasoning, not raw chat logs

## License

MIT
