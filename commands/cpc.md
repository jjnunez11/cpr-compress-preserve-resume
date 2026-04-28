---
context: conversation
description: Compress + Preserve + Compact — end-of-session cleanup in one command
model: opus
allowed-tools: Read, Edit, Write, Bash
---

# /cpc - Compress, Preserve, Compact

Runs the full end-of-session cleanup: saves a session log, updates CLAUDE.md if warranted, then prompts for `/compact`. Fully automated — no interactive prompts.

## Instructions for Claude

Work through the phases below in order. Decide independently whether each phase applies; skip it with a one-line note if not.

---

## Phase 1: Assess

Before doing anything, assess the session:

**Should compress run?**
Run if: the session has substantial content — files created/modified, decisions made, problems solved, workflows established, or anything a future session might need to search for.
Skip if: the session was trivial (a single quick question with no real work done).

**Should preserve run?**
Run if: the session introduced something new that belongs in CLAUDE.md — a new workflow, a corrected assumption, a new pattern, a standing rule, a new section of the vault structure, etc.
Skip if: everything done was routine operations already covered in CLAUDE.md, or the only changes were to content files (emails, meeting notes, project files) rather than to how the vault/assistant should work.

State your assessment briefly before proceeding:
```
Compress: yes / no — [one-line reason]
Preserve: yes / no — [one-line reason]
```

---

## Phase 2: Compress (if warranted)

Follow these steps with no interactive prompts.

### 2a: Generate Topic Name

Generate a concise topic name (3-5 words, lowercase, hyphens). Use it directly.

### 2b: Generate Session Log

Include all relevant sections that have content. Always include Quick Reference, Quick Resume Context, and Raw Session Log.

```markdown
# Session Log: DD-MM-YYYY HH:MM - {topic-name}

## Quick Reference (for AI scanning)
**Confidence keywords:** {extracted keywords}
**Projects:** {project names or references mentioned}
**Outcome:** {1-sentence outcome summary}

## Key Learnings
- {Learning}

## Decisions Made
- {Decision with rationale}

## Solutions & Fixes
- {Solution}

## Files Modified
- `{path}`: {what changed}

## Setup & Config
- {Config item}

## Pending Tasks
- {Pending item}

## Errors & Workarounds
- {Error and fix}

---

## Quick Resume Context
{2-3 sentences that would help resume this work in a future session}

---

## Raw Session Log

{FULL CONVERSATION — all user messages and assistant responses, verbatim}
```

Only include sections that have content. Omit empty ones.

### 2c: Detect Project Root & Save

```
1. pwd to get working directory
2. Walk up looking for CLAUDE.md or .git — that is project_root
3. If project_root/.claude/ exists: logs_path = project_root/.claude/CC-Session-Logs/
   Otherwise: logs_path = project_root/CC-Session-Logs/
4. mkdir -p logs_path
5. Filename: DD-MM-YYYY-HH_MM-{topic-name}.md
6. Write the session log
```

**Confidence keywords to extract:** project names, technical terms, action types, tool/framework names, people mentioned, specific identifiers.

---

## Phase 3: Preserve (if warranted)

Follow these steps with no interactive prompts. Analyze the conversation and include all relevant categories.

### 3a: Find CLAUDE.md

Look for `CLAUDE.md`, `Claude.md`, or `.claude/CLAUDE.md` from the project root. If not found, skip preserve and note it.

### 3b: Read CLAUDE.md

Read the current file to understand structure, existing sections, and what needs updating vs. adding.

### 3c: Generate and Apply Updates

Identify what from the session is worth adding or updating. Follow these signal rules:

**Include (high signal):**
- New standing rules or workflow steps
- Corrections to existing instructions
- New patterns or naming conventions
- New sections of vault structure or tooling
- Decisions with lasting impact on how Claude should behave

**Exclude (low signal):**
- Content-level work (emails filed, meeting notes written, project files updated)
- Routine operations already documented
- Implementation details that live in the files themselves
- Session-specific context that won't recur

**Format rules:**
- Match the existing CLAUDE.md style
- Single-line entries, not paragraphs
- Point to files rather than duplicating content
- Target: CLAUDE.md under 280 lines

Edit CLAUDE.md directly.

### 3d: Check Line Count

```bash
wc -l CLAUDE.md
```

If over 280 lines, identify auto-archivable content (`## Session Notes (DATE)` older than 7 days, `## Completed Projects`) and archive to `{project_root}/.claude/CLAUDE-Archive.md` (or `{project_root}/CLAUDE-Archive.md` if no `.claude/` folder). Only archive non-core sections — never archive structural or workflow sections.

---

## Phase 4: Final Report

Output a single summary:

```
## /cpc Complete

**Compress:** [done — saved to path/filename.md] / [skipped — reason]
**Preserve:** [done — updated N lines in CLAUDE.md] / [skipped — reason]

**Next step:** Run `/compact` to compress the conversation context.
```
