---
context: conversation
description: Smart Conversation Compression with Session Logging
model: opus
allowed-tools: Read, Write, Bash
---

# /compress - Smart Conversation Compression

Prepares preservation notes for conversation compaction AND saves the full session to searchable logs. Run this BEFORE `/compact`.

**Workflow:** `/preserve` (optional) → `/compress` → session saved → `/compact` (always last)

## Instructions for Claude

When the user runs `/compress`, follow these steps with no interactive prompts — analyze the conversation and proceed automatically.

### Step 1: Generate Topic Name

Analyze the conversation and generate a concise topic name (3-5 words, lowercase, hyphens). Use it directly without asking for confirmation.

### Step 2: Generate Session Log

Include all relevant categories that have content. Always include Quick Reference, Quick Resume Context, and Raw Session Log. Include any of the following that apply:

Create the session log content with this structure:

```markdown
# Session Log: DD-MM-YYYY HH:MM - {Topic Name}

## Quick Reference (for AI scanning)
**Confidence keywords:** {extracted keywords from conversation}
**Projects:** {project names or references mentioned}
**Outcome:** {1-sentence outcome summary}

## Decisions Made
- {Decision 1 with brief rationale}
- {Decision 2 with brief rationale}

## Key Learnings
- {Learning 1}
- {Learning 2}

## Solutions & Fixes
- {Solution 1}
- {Solution 2}

## Files Modified
- `{path/to/file}`: {what changed}

## Setup & Config
- {Config item if selected}

## Pending Tasks
- {Pending item if selected}

## Errors & Workarounds
- {Error and fix if selected}

## Key Exchanges
- {Notable exchange 1, brief summary}
- {Notable exchange 2, brief summary}

---

## Quick Resume Context
{2-3 sentences that would help resume this work in a future session}

---

## Raw Session Log

{FULL CONVERSATION - Copy the entire conversation history here, preserving all user messages and assistant responses. This is the searchable archive.}
```

**IMPORTANT:** Only include sections the user selected in Step 1. Always include:
- Quick Reference (for AI scanning)
- Quick Resume Context
- Raw Session Log

### Step 3: Detect Project Root & Save

**Generate filename:**
```
DD-MM-YYYY-HH_MM-{topic-name}.md
```
Example: `05-03-2026-17_30-api-auth-refactor.md`

**Detect project root:**

```
1. Get current working directory (pwd)
2. Find project root: walk up from pwd looking for CLAUDE.md or .git
3. If found: project_root = that directory
4. If not found: project_root = pwd
5. Session logs path: if {project_root}/.claude/ exists, use {project_root}/.claude/CC-Session-Logs/, else use {project_root}/CC-Session-Logs/
6. Create folder if it doesn't exist
7. Write session log there
```

**Save the session log:**
```bash
# Determine logs path
if [ -d "{project_root}/.claude" ]; then
  logs_path="{project_root}/.claude/CC-Session-Logs/"
else
  logs_path="{project_root}/CC-Session-Logs/"
fi

# Create folder if needed
mkdir -p "$logs_path"

# Write session log
Write tool -> $logs_path/{filename}
```

### Step 4: Confirm and Instruct

Output confirmation:

```markdown
## Session Saved Successfully

### File Created

**Session Log:**
`{project_root}/CC-Session-Logs/{filename}`

### Session Summary
- **Project:** {project_root basename}
- **Topic:** {topic-name}
- **Sections preserved:** {list of selected sections}
- **Keywords:** {confidence keywords}

---

**Next step:** Run `/compact` to compress the conversation context (always last).

The session log is saved locally. Use `/resume` to load context from recent sessions.
```

---

## Guidelines

- **Be concise:** Each bullet should be actionable or informative
- **Use code blocks** for commands, paths, and code snippets
- **Include file paths** with line numbers where relevant
- **Preserve exact values:** Don't paraphrase credentials, IDs, or specific configs
- **Link context:** If something depends on something else, note the relationship
- **Extract keywords:** The "Confidence keywords" field is critical for future AI scanning
- **Full raw log:** The Raw Session Log must contain the COMPLETE conversation for searchability

---

## Confidence Keywords Extraction

When generating the "Confidence keywords" field, extract:
- Project names (my-api, user-dashboard)
- Technical terms (auth, middleware, migration, deploy)
- Action types (refactor, fix, create, update, delete)
- Tool/framework names (React, PostgreSQL, Docker)
- People mentioned (if relevant to decisions)
- Specific identifiers (issue numbers, ticket IDs)

These keywords enable the `/resume` skill to find relevant sessions via search.

---

## Example Output

```markdown
## Session Saved Successfully

### File Created

**Session Log:**
`/home/user/my-project/CC-Session-Logs/05-03-2026-17_30-api-auth-refactor.md`

### Session Summary
- **Project:** my-project
- **Topic:** api-auth-refactor
- **Sections preserved:** Decisions Made, Key Learnings, Files Modified
- **Keywords:** auth, JWT, middleware, refresh-tokens, session-management
```
