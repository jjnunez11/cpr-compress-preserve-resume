<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-Skills-5A67D8?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyem0wIDE4Yy00LjQxIDAtOC0zLjU5LTgtOHMzLjU5LTggOC04IDggMy41OSA4IDgtMy41OSA4LTggOHoiLz48L3N2Zz4=&logoColor=white" alt="Claude Code Skills" />
  <img src="https://img.shields.io/badge/Model-Opus%204.6-E74C3C?style=for-the-badge" alt="Opus 4.6" />
  <img src="https://img.shields.io/badge/Skills-3%20Included-blue?style=for-the-badge" alt="3 Skills" />
  <img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge" alt="MIT License" />
</p>

<h1 align="center">CPR for Claude Code</h1>
<h3 align="center"><em>Compress, Preserve & Resume</em></h3>

<p align="center">
  <strong>Persistent memory across sessions. Never lose context again.</strong>
</p>

<p align="center">
  Three custom skills that save, search, and restore your conversation context - so you can pick up exactly where you left off.
</p>

<p align="center">
  <a href="#the-problem">Problem</a> &bull;
  <a href="#the-solution">Solution</a> &bull;
  <a href="#installation">Installation</a> &bull;
  <a href="#usage">Usage</a> &bull;
  <a href="#recommended-workflow">Workflow</a> &bull;
  <a href="#how-it-scales">Scaling</a> &bull;
  <a href="#faq">FAQ</a>
</p>

---

## What Are Skills?

Skills are **custom slash commands** for Claude Code. You type `/compress`, `/preserve`, or `/resume` in your conversation and Claude follows the instructions defined in a markdown file - no code, no plugins, just a `.md` file in the right folder.

Claude Code loads skills from two locations:

| Location | Scope |
|----------|-------|
| `~/.claude/commands/*.md` | Global - available in every project |
| `{project}/.claude/commands/*.md` | Per-project - available only in that project |

Each `.md` file becomes a `/command` you can run. That's it. CPR is three of these files.

---

## The Problem

Claude Code has **no memory between sessions**. When you close a conversation, everything is gone - decisions, solutions, context, all of it.

It gets worse:

| Problem | Impact |
|---------|--------|
| **Auto-compacting loses details** | Context window fills up, Claude silently compresses history - specific values, file paths, nuanced decisions get flattened |
| **Long sessions lose early context** | That critical decision from the first 10 minutes? Gone by hour two |
| **New session = blank slate** | Re-explaining your project, re-discovering paths, re-making decisions every time |
| **Past work is unsearchable** | No way to look up what you discussed three sessions ago |
| **CLAUDE.md isn't enough** | Static file - doesn't capture the flow of decisions, errors, or solutions |

---

## The Solution

Three skills that work together to give Claude Code a memory:

```
Session Work ──> /compress  ──> Session Log saved
                 /preserve  ──> CLAUDE.md updated (interchangeable order)
                      |
                 /compact   ──> Context compressed (always LAST)
                      |
New Session  ──> /resume   ──> Loads CLAUDE.md + recent logs ──> Full context restored
```

<table>
  <tr>
    <th>Skill</th>
    <th>What it does</th>
  </tr>
  <tr>
    <td><strong><code>/compress</code></strong></td>
    <td>Captures the full session (decisions, solutions, files, errors) into a structured, searchable log file. Run this (and/or <code>/preserve</code>) BEFORE <code>/compact</code>.</td>
  </tr>
  <tr>
    <td><strong><code>/preserve</code></strong></td>
    <td>Updates CLAUDE.md with key learnings from the session. Keeps it lean (under 280 lines) with automatic archiving when it gets too long.</td>
  </tr>
  <tr>
    <td><strong><code>/resume</code></strong></td>
    <td>Loads CLAUDE.md + last N session log summaries when starting a new session. Supports topic search across all past sessions.</td>
  </tr>
</table>

---

## Critical: Disable Auto-Compacting

Claude Code's auto-compact feature automatically compresses your conversation when the context window fills up. This is the enemy - it throws away details before you can save them.

**Disable it:**

1. Type `/config` inside Claude Code
2. Set **Auto-Compact** to **false** (it's the first option)

Or via CLI:

```bash
claude config set --global autoCompact false
```

With auto-compact off, **you** control when compression happens:

1. `/compress` - save the session log (preserves everything)
2. `/preserve` - update CLAUDE.md with key learnings (optional)
3. `/compact` - compress the context (always last, because you already saved)

This gives you a clean, explicit workflow instead of silent data loss.

---

## The Session Log System

Every `/compress` creates a structured markdown file in `CC-Session-Logs/` at your project root.

**Filename format:** `DD-MM-YYYY-HH_MM-topic-name.md`

<details>
<summary><strong>Log structure (click to expand)</strong></summary>

```markdown
# Session Log: 05-03-2026 14:20 - api-auth-refactor

## Quick Reference (for AI scanning)
**Confidence keywords:** auth, JWT, refresh-tokens, middleware
**Projects:** my-saas-app
**Outcome:** Replaced cookie-based auth with JWT + refresh tokens

## Decisions Made
- JWT over session cookies - stateless scales better

## Key Learnings
- Redis EX flag is cleaner than separate EXPIRE calls

## Solutions & Fixes
- Login race condition fixed with SETNX

## Files Modified
- `src/middleware/auth.ts` - JWT verification

## Pending Tasks
- [ ] Add refresh token rotation

---
## Quick Resume Context
2-3 sentence summary for fast loading in /resume.

---
## Raw Session Log
{Full conversation archive - searchable but never loaded by /resume}
```

</details>

**The key insight:** `/resume` only reads the summary sections (everything above "Raw Session Log"). The raw conversation is there for searchability, but it never wastes tokens during context loading.

See [`examples/session-log-example.md`](examples/session-log-example.md) for a complete example.

---

## Installation

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)

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

### 4. Model

All three skills default to **Claude Opus** (`model: opus` in frontmatter) for maximum context understanding and output quality. If you don't have Opus access, change `model: opus` to `model: sonnet` in each file's frontmatter - Sonnet 4.6 works well as a fallback.

### 5. Disable auto-compacting

Type `/config` inside Claude Code and set **Auto-Compact** to **false** (first option).

Or via CLI:

```bash
claude config set --global autoCompact false
```

**This step is critical.** Without it, Claude Code will silently compress your context before you get a chance to save it with `/compress` or `/preserve`.

### 6. Test

```
You: /compress
```

If you see the preservation question, it's working.

---

## Usage

### End of session - save your work

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
        Run /preserve to update CLAUDE.md, then /compact to compress context.
```

### After major decisions - update CLAUDE.md

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

### Starting a new session - restore context

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

- 05-03-2026: api-auth-refactor - JWT + refresh tokens
- 28-02-2026: oauth-google-setup - Google OAuth integration
══════════════════════════════════════════════
```

---

## Recommended Workflow

```
┌──────────────────────────────────────────────────────┐
│  1. Start session                                    │
│     └── /resume          Load context                │
│                                                      │
│  2. Do work...                                       │
│     └── (normal Claude Code usage)                   │
│                                                      │
│  3. Before ending or when context is filling up      │
│     ├── /compress        Save session log            │
│     ├── /preserve        Update CLAUDE.md (optional) │
│     └── /compact         Compress context (LAST)     │
└──────────────────────────────────────────────────────┘
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

<details>
<summary><strong>Session log storage path</strong></summary>

By default, logs go to `{project_root}/CC-Session-Logs/`. The project root is detected by walking up from your current directory looking for `CLAUDE.md` or `.git`.

To change this, edit the path detection logic in `commands/compress.md` and `commands/resume.md` (Step 5 / Step 3 respectively).

</details>

<details>
<summary><strong>CLAUDE.md line target</strong></summary>

The default target is 280 lines. Change this in `commands/preserve.md` (Step 6) by modifying the threshold value.

</details>

<details>
<summary><strong>Protected sections</strong></summary>

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

</details>

<details>
<summary><strong>Core sections</strong></summary>

Edit the "CORE Sections" list in `commands/preserve.md` to match your CLAUDE.md structure. These sections are never suggested for archiving.

</details>

---

## How It Scales

| Session Logs | Behaviour |
|-------------|-----------|
| < 100 | Direct file listing + grep search - fast and simple |
| >= 100 | Grep-based search for topic matching - still fast |
| Any count | `/resume` reads summaries only, never raw logs - token-efficient at any scale |

The raw session logs can grow large (full conversation archives), but `/resume` never reads past the `## Raw Session Log` marker. Only the structured summary header is loaded.

---

## FAQ

<details>
<summary><strong>Do I need all three skills?</strong></summary>

`/compress` + `/resume` is the minimum viable setup. `/preserve` is optional but recommended - it keeps your CLAUDE.md up to date without manual editing.

</details>

<details>
<summary><strong>Where are logs stored?</strong></summary>

`{project_root}/CC-Session-Logs/`. Project root is the nearest parent directory containing `CLAUDE.md` or `.git`. If neither is found, it falls back to the current working directory.

</details>

<details>
<summary><strong>Will this work with any project?</strong></summary>

Yes. The skills auto-detect your project root and create the `CC-Session-Logs/` folder on first use. No configuration needed.

</details>

<details>
<summary><strong>How big do logs get?</strong></summary>

A full session log with the raw conversation can be several hundred KB. But `/resume` only reads the summary header (typically 30-80 lines), so token usage stays low regardless of log size.

</details>

<details>
<summary><strong>Should I commit session logs to git?</strong></summary>

Up to you. They're useful for team knowledge sharing but can be large. Consider adding `CC-Session-Logs/` to `.gitignore` if you prefer to keep them local.

</details>

<details>
<summary><strong>What if I forget to /compress before /compact?</strong></summary>

The compacted context will still work, but you'll lose the detailed session log. Always run `/compress` and/or `/preserve` before `/compact`, since `/compact` clears the entire context window.

</details>

<details>
<summary><strong>Can I use this with CLAUDE.md files in subdirectories?</strong></summary>

The skills look for CLAUDE.md at the project root. If you have multiple CLAUDE.md files (e.g., monorepo), run from the relevant subdirectory.

</details>

---

## Credits

Created by [Elia Alberti](https://github.com/EliaAlberti). Built with and for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

---

## License

MIT - see [LICENSE](LICENSE).
