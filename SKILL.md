---
name: memento
description: "Cognitive persistence system for session continuity. Syntax: /memento [action]. Actions: hydrate (restore context), checkpoint (save session), status (view state). Also triggers on intents like 'reprends le contexte', 'sauvegarde la session', 'où on en était'."
---

# Memento

Automated cognitive distillation system. Memento writes its own "work memories" so future sessions inherit reasoning context, not just code.

## Argument Routing

**Syntax:** `/memento [action]`

### Route by explicit action:
| Action | Aliases | Protocol |
|--------|---------|----------|
| `hydrate` | `restore`, `load`, `start` | → Rehydration Protocol |
| `checkpoint` | `save`, `snapshot`, `end` | → Cognitive Distillation Protocol |
| `status` | `state`, `info` | → Status Display |

### Route by semantic intent:
| User intent (FR/EN) | Route to |
|---------------------|----------|
| "reprends", "où on en était", "resume context" | → hydrate |
| "sauvegarde", "fini", "save session", "done" | → checkpoint |
| "état actuel", "what's saved", "show context" | → status |

### Fallback (no argument):
If `/memento` is invoked without argument, ask:
```
🧠 Memento - Que souhaitez-vous faire ?
- `hydrate` — Restaurer le contexte des sessions précédentes
- `checkpoint` — Sauvegarder l'état cognitif de cette session
- `status` — Voir l'état actuel sans modifier

Syntaxe: /memento [action]
```

---

## Priority Directive

When Memento coexists with native history (e.g., user ran `--resume`):

- **Graveyard is ABSOLUTE** — Never suggest an approach listed in `graveyard.md`, regardless of what raw history says. This is a formal prohibition.
- **Architectural decisions** — Trust `latest.md` over raw chat history for high-level choices
- **Contradiction handling** — If raw history contradicts Memento files, **flag the divergence to the user** instead of choosing silently. Example: "I see in the chat history we discussed X, but your Memento graveyard says X was rejected. Which is current?"

Memento is distilled signal. Raw history is noisy data. Use Memento for understanding, raw history for details.

---

## UX Philosophy: Silence is Golden

**Files = Detailed** — Chronicles contain rich, complete information. That's the memory.

**Terminal = Minimal** — Output only confirmation + counts. No redundancy with file contents.

**Why?**
- Saves output tokens (billed)
- Keeps terminal clean for debugging
- Users can read files or use `/memento status` for details

**Never repeat file contents in terminal output.**

---

## Protocol: Hydrate (Session Start)

**Trigger:** `/memento hydrate` or restore intent

1. Check if `.claude/chronicles/` exists
2. If exists, read in this order:
   - `graveyard.md` — Accumulated rejected approaches (NEVER suggest these again)
   - `latest.md` — Last session's cognitive state
3. Acknowledge loaded context with summary
4. If no chronicles exist, inform user

**Output format (minimal):**
```
🧠 Memento hydrated
   ├─ Last: [date] ([focus])
   ├─ Graveyard: [n] forbidden approaches
   └─ Next: [top priority action]

Ready.
```

**If no chronicles:**
```
📭 No Memento context found.
Run `/memento checkpoint` before ending this session.
```

---

## Protocol: Checkpoint (Session End)

**Trigger:** `/memento checkpoint` or save intent

Execute the Shadow-Summarization Protocol, then write results to chronicles.

### Shadow-Summarization Protocol

**Step 1: Extract Decisions**
Scan conversation for patterns indicating decisions:
- "Let's go with...", "We'll use...", "On part sur..."
- Architecture choices, library selections, implementation strategies
- Capture the REASONING, not just the choice

**Step 2: Identify Graveyard Candidates**
Scan for rejected approaches:
- "That won't work because...", "On a essayé X mais...", "Ne pas utiliser..."
- Failed attempts, dead ends, incompatible solutions
- **CRITICAL**: Record WHY it failed — this is the value

**Step 3: Flag Technical Debt**
Identify conscious compromises:
- "For now we'll...", "C'est un hack mais...", "TODO: refactor when..."
- Shortcuts taken with awareness

**Step 4: Capture User Context**
Note observed user characteristics:
- Skill level in different domains
- Stated preferences (code style, frameworks, verbosity)
- Working patterns

**Step 5: Define Next Actions**
Extract incomplete work:
- Unfinished tasks, "Next we should...", "Demain on..."
- Open questions or blockers

### File Operations

1. Create `.claude/chronicles/sessions/` if needed
2. Write session chronicle to `sessions/YYYY-MM-DD_HHMM.md`
3. Append new graveyard entries to `graveyard.md` (create with header if first time)
4. Overwrite `latest.md` with current state
5. Confirm to user

**Output format (minimal):**
```
📸 Checkpoint saved
   ├─ Decisions: [n] captured
   ├─ Graveyard: +[n] entries
   ├─ Tech debt: [n]
   └─ Next actions: [n] defined

→ .claude/chronicles/
```

**Do NOT repeat file contents. User can run `/memento status` for details.**

---

## Protocol: Status (View State)

**Trigger:** `/memento status` or info intent

Read and display current state WITHOUT modifying anything.

**Output format (status is allowed to be more detailed since user explicitly requested info):**
```
📊 Memento Status

Last update: [timestamp]
Graveyard: [n] entries | Tech debt: [n] items

Active decisions:
• [Decision 1]
• [Decision 2]

Next actions:
1. [Action 1]
2. [Action 2]
```

**If no chronicles:**
```
📭 No Memento context.
Run `/memento checkpoint` to initialize.
```

---

## Output Templates

### Session Chronicle (`.claude/chronicles/sessions/YYYY-MM-DD_HHMM.md`)

```markdown
# Session: [Brief description of main focus]
**Date:** [YYYY-MM-DD HH:MM]

## Decisions Made
- **[Decision]**: [Choice made]
  - *Reasoning*: [Why this approach]
  - *Alternatives considered*: [What else was evaluated]

## Graveyard Additions
- **[Rejected approach]**
  - *Why it failed*: [Technical reason]

## Technical Debt Logged
- **[Compromise]**
  - *Revisit when*: [Condition]

## User Context Updates
- [Observations about user preferences/level]

## Next Actions
- [ ] [Priority action 1]
- [ ] [Action 2]

## Session Summary
[2-3 sentence narrative]
```

### Graveyard Entry (append to `graveyard.md`)

```markdown
## [Rejected Approach Name]
**Added:** [Date]
**Context:** [What we were trying to solve]
**Why it failed:** [Technical explanation]
**Do instead:** [What worked]
```

### Latest State (`latest.md`)

```markdown
# Current Project State
**Last updated:** [Timestamp]

## Active Decisions
[Current architectural/technical choices]

## Known Constraints
[From graveyard — things that don't work]

## Pending Technical Debt
[Conscious compromises awaiting resolution]

## User Profile
- **Expertise:** [Observed skill levels]
- **Preferences:** [Code style, communication style]

## Immediate Context
[What we were working on, where we left off]

## Priority Queue
1. [Next action]
2. [Following action]
```

---

## File Structure

```
.claude/
└── chronicles/
    ├── latest.md           # Current cognitive state (overwritten each checkpoint)
    ├── graveyard.md        # Cumulative rejected approaches (append-only, NEVER delete)
    └── sessions/
        └── YYYY-MM-DD_HHMM.md  # Archived session chronicles
```

---

## First Checkpoint

If `graveyard.md` doesn't exist, create it with:
```markdown
# Graveyard
Approaches that don't work in this project. **Never suggest these again.**

---
```
