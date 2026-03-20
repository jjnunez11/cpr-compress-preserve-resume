# CPR — Compress, Preserve & Resume for Claude Code

**Persistent memory across sessions. Never lose context again.**

Three custom skills for Claude Code that save, search, and restore your conversation context — so you can pick up exactly where you left off.

---

## What Are Skills?

Skills are custom slash commands for Claude Code. You type `/compress`, `/resume`, or `/preserve` in your conversation and Claude follows the instructions defined in a markdown file — no code, no plugins, just a `.md` file in the right folder.

Claude Code loads skills from two locations:
- **Global:** `~/.claude/commands/*.md` — available in every project
- **Per-project:** `{project}/.claude/commands/*.md` — available only in that project

Each `.md` file becomes a `/command` you can run. That's it. CPR is three of these files.

---

## The Problem

Claude Code has **no memory between sessions**. When you close a conversation, everything is gone — decisions, solutions, context, all of it.

It gets worse:

- **Auto-compacting loses details.** When the context window fills up, Claude Code silently compresses your conversation history. Specific values, file paths, and nuanced decisions get flattened into vague summaries.
- **Long sessions lose early context.** That critical decision you made in the first 10 minutes? Gone by hour two.
- **New session = blank slate.** Every time you start fresh, you're re-explaining your project, re-discovering file paths, re-making decisions you already made.
- **Past work is unsearchable.** There's no way to look up what you discussed three sessions ago.
- **CLAUDE.md isn't enough.** It's a static file. It doesn't capture the flow of decisions, the specific errors you hit, or the solutions you found along the way.

## The Solution

Three skills (custom slash commands) that work together to give Claude Code a memory:

```
Session Work ──> /compress ──> Session Log saved ──> /compact
                                        │
               /preserve ──> Updates CLAUDE.md with key learnings
                                        │
New Session ──> /resume ──> Loads CLAUDE.md + recent logs ──> Full context restored
```

| Skill | What it does |
|-------|-------------|
| **`/compress`** | Captures the full session (decisions, solutions, files, errors) into a structured, searchable log file. Run this BEFORE `/compact`. |
| **`/preserve`** | Updates CLAUDE.md with key learnings. Keeps it lean (under 280 lines) with automatic archiving. |
| **`/resume`** | Loads CLAUDE.md + last N session log summaries when starting a new session. Supports topic search across all past sessions. |

---

## Critical: Disable Auto-Compacting

Claude Code's auto-compact feature automatically compresses your conversation when the context window fills up. This is the enemy — it throws away details before you can save them.

**Disable it:**

```bash
claude config set --global autoCompact false
```

With auto-compact off, **you** control when compression happens:

1. `/compress` — save the session log (preserves everything)
2. `/compact` — then compress the context (safe, because you already saved)

This gives you a clean, explicit workflow instead of silent data loss.

---

## The Session Log System

Every `/compress` creates a structured markdown file in `CC-Session-Logs/` at your project root.

**Filename format:** `DD-MM-YYYY-HH_MM-topic-name.md`

**Log structure:**

```markdown
# Session Log: 05-03-2026 14:20 - api-auth-refactor

## Quick Reference (for AI scanning)
**Confidence keywords:** auth, JWT, refresh-tokens, middleware
**Projects:** my-saas-app
**Outcome:** Replaced cookie-based auth with JWT + refresh tokens

## Decisions Made
- JWT over session cookies — stateless scales better

## Key Learnings
- Redis EX flag is cleaner than separate EXPIRE calls

## Solutions & Fixes
- Login race condition fixed with SETNX

## Files Modified
- `src/middleware/auth.ts` — JWT verification

## Pending Tasks
- [ ] Add refresh token rotation

---
## Quick Resume Context
2-3 sentence summary for fast loading in /resume.

---
## Raw Session Log
{Full conversation archive — searchable but never loaded by /resume}
```

**The key insight:** `/resume` only reads the summary sections (everything above "Raw Session Log"). The raw conversation is there for searchability, but it never wastes tokens during context loading.

See [`examples/session-log-example.md`](examples/session-log-example.md) for a complete example.

---

## Installation

### 1. Get the files

```bash
git clone https://github.com/eliaalberti/cpr-compress-preserve-resume.git
cd cpr-compress-preserve-resume
```

### 2. Install the skills

Skills are `.md` files that go in a `commands/` folder. Pick one:

**Global install** (available in all projects):

```bash
mkdir -p ~/.claude/commands
cp commands/*.md ~/.claude/commands/
```

**Per-project install** (available only in that project):

```bash
mkdir -p /path/to/your/project/.claude/commands
cp commands/*.md /path/to/your/project/.claude/commands/
```

### 3. Restart Claude Code

Skills are loaded at startup. Restart for the new `/compress`, `/preserve`, and `/resume` commands to appear.

**Model:** All three skills default to **Claude Opus** (`model: opus` in frontmatter) for maximum context understanding and output quality. If you don't have Opus access, change `model: opus` to `model: sonnet` in each file's frontmatter — Sonnet 4.6 works well as a fallback.

### 4. Disable auto-compacting

```bash
claude config set --global autoCompact false
```

### 5. Test

```
You: /compress
```

If you see the preservation question, it's working.

---

## Usage

### End of session — save your work

```
You: /compress
Claude: What would you like to preserve? [multi-select]
  1. Key Learnings
  2. Solutions & Fixes
  3. Decisions Made
  4. Files Modified
  ...
You: 1, 2, 3, 4
Claude: Anything specific to highlight? (Type 'skip' to continue)
You: skip
Claude: Suggested topic: api-auth-refactor. Accept or type your own:
You: ok
Claude: Session saved to CC-Session-Logs/05-03-2026-17_30-api-auth-refactor.md
        Run /compact to compress conversation context.
```

### Starting a new session — restore context

```
You: /resume
Claude:
══════════════════════════════════════════════
 RESUMING: my-saas-app
══════════════════════════════════════════════

CONTEXT:
- JWT auth flow implemented, tests passing
- Redis used for refresh token storage

MOST RECENT SESSION: 05-03-2026 17:30
Topic: api-auth-refactor
...

READY TO:
- Add refresh token rotation
- Set up token blacklist for logout
══════════════════════════════════════════════
```

### Search past sessions by topic

```
You: /resume auth
Claude: [Shows recent sessions + RELATED SESSIONS matching "auth"]

══════════════════════════════════════════════
 RELATED SESSIONS (Topic: "auth")
══════════════════════════════════════════════

- 05-03-2026: api-auth-refactor — JWT + refresh tokens
- 28-02-2026: oauth-google-setup — Google OAuth integration
══════════════════════════════════════════════
```

### After major decisions — update CLAUDE.md

```
You: /preserve
Claude: What should be preserved? [multi-select]
You: 2, 6 (Key Decisions, Next Steps)
Claude: CLAUDE.md Updated
        Preserved:
        - Added JWT auth decision rationale
        - Updated next steps with token rotation
        CLAUDE.md is now 185 lines (target: <280)
```

---

## Recommended Workflow

```
┌─────────────────────────────────────────────────┐
│  1. Start session                               │
│     └── /resume          Load context           │
│                                                 │
│  2. Do work...                                  │
│     └── (normal Claude Code usage)              │
│                                                 │
│  3. Before ending or when context is filling up  │
│     ├── /compress        Save session log       │
│     └── /compact         Compress context       │
│                                                 │
│  4. Optionally                                  │
│     └── /preserve        Update CLAUDE.md       │
└─────────────────────────────────────────────────┘
```

**When to `/compress`:**
- Before ending a session
- Before context fills up (if you notice responses getting less specific)
- After completing a significant chunk of work

**When to `/preserve`:**
- After making important architectural decisions
- When you discover patterns you'll need in future sessions
- When project phase/status changes

---

## Customisation

### Session log storage path

By default, logs go to `{project_root}/CC-Session-Logs/`. The project root is detected by walking up from your current directory looking for `CLAUDE.md` or `.git`.

To change this, edit the path detection logic in `commands/compress.md` and `commands/resume.md` (Step 5 / Step 3 respectively).

### CLAUDE.md line target

The default target is 280 lines. Change this in `commands/preserve.md` (Step 6) by modifying the threshold value.

### Protected sections

Mark any section in your CLAUDE.md as immune to archiving:

```markdown
## My Important Section (PROTECTED)
This will never be suggested for archiving.
```

Or mark sections as safe to archive:

```markdown
## Old Notes (ARCHIVABLE)
This will be auto-suggested for archiving when CLAUDE.md gets too long.
```

### Core sections

Edit the "CORE Sections" list in `commands/preserve.md` to match your CLAUDE.md structure. These sections are never suggested for archiving.

---

## How It Scales

| Session Logs | Behaviour |
|-------------|-----------|
| < 100 | Direct file listing + grep search — fast and simple |
| >= 100 | Grep-based search for topic matching — still fast |
| Any count | `/resume` reads summaries only, never raw logs — token-efficient at any scale |

The raw session logs can grow large (full conversation archives), but `/resume` never reads past the `## Raw Session Log` marker. Only the structured summary header is loaded.

---

## FAQ

**Do I need all three skills?**
`/compress` + `/resume` is the minimum viable setup. `/preserve` is optional but recommended — it keeps your CLAUDE.md up to date without manual editing.

**Where are logs stored?**
`{project_root}/CC-Session-Logs/`. Project root is the nearest parent directory containing `CLAUDE.md` or `.git`. If neither is found, it falls back to the current working directory.

**Will this work with any project?**
Yes. The skills auto-detect your project root and create the `CC-Session-Logs/` folder on first use. No configuration needed.

**How big do logs get?**
A full session log with the raw conversation can be several hundred KB. But `/resume` only reads the summary header (typically 30-80 lines), so token usage stays low regardless of log size.

**Should I commit session logs to git?**
Up to you. They're useful for team knowledge sharing but can be large. Consider adding `CC-Session-Logs/` to `.gitignore` if you prefer to keep them local. Alternatively, commit just the summary sections.

**What if I forget to `/compress` before `/compact`?**
The compacted context will still work, but you'll lose the detailed session log. Make `/compress` a habit before `/compact`.

**Can I use this with CLAUDE.md files in subdirectories?**
The skills look for CLAUDE.md at the project root. If you have multiple CLAUDE.md files (e.g., monorepo), run from the relevant subdirectory.

---

## License

MIT — see [LICENSE](LICENSE).
