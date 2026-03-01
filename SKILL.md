---
name: shared-brain
description: |
  Set up a unified long-term memory system shared across multiple AI entry points (IM Bot, Cursor, Gemini CLI, Codex, etc.).
  TRIGGER: Only when user explicitly asks to "set up shared memory", "add a new entry point", or "migrate memory".
  DO NOT TRIGGER: For normal tasks like git push, bug fixes, code edits, or Q&A. This skill is a one-time setup tool, not a per-conversation background process.
---

# Shared Brain Skill

Set up a **unified long-term memory** architecture so that multiple AI entry points — IM bots, Cursor, Gemini CLI, Codex, or any future tool — read from and write to the **same** memory, identity, and rules. No more "amnesia" when switching tools.

## Core Philosophy

1. **Single Source of Truth** — One main memory file. Not two, not three. One.
2. **Append-Only** — All entry points only append. Never overwrite, never delete, never reorder.
3. **Separation of Concerns** — Identity, rules, user profile, and memory are separate files.
4. **Memory ≠ Knowledge** — Shared memory stores *who you are and what you decided*. Engineering knowledge (bug fixes, scripts, workflows) goes to a separate knowledge base.

---

## ⚠️ Anti-Chain-Reaction Rules (CRITICAL)

**This section prevents LLMs from entering infinite planning loops. Every entry point MUST follow these rules.**

### Rule 1: This Skill Is a Setup Tool, Not a Background Daemon

This skill should only be invoked when the user explicitly asks to set up, migrate, or configure shared memory. It should **never** auto-trigger during normal tasks (git push, bug fix, code edit, Q&A).

### Rule 2: Lazy-Read, Not Eager-Read

Do **NOT** read all memory files at the start of every conversation. Instead:
- Read memory files **only when the current task requires context** that you don't already have.
- If the user's request is self-contained (e.g., "push to GitHub"), do NOT read memory files at all.
- If the user asks about past decisions or preferences, THEN read memory files.

### Rule 3: No Internal Monologue Leaks

When deciding whether to read/write memory, do it **silently**. Never output your planning process (e.g., "I'll now check the shared memory...", "Let me read the archive..."). Just do it or don't.

### Rule 4: One Task at a Time

Never chain multiple background tasks together. If the user asks to "push to GitHub":
- ✅ Just push to GitHub.
- ❌ Do NOT also: read memory → check archives → update memory → capture knowledge → update KIs → ...

### Rule 5: Write Throttle

Only write to shared memory when **ALL** of these are true:
1. The conversation has genuinely concluded.
2. A new **long-term** preference or decision was explicitly confirmed by the user.
3. The information is NOT already in the memory file.

Do **NOT** write to shared memory for: routine tasks, bug fixes, git operations, or file edits.

---

## Architecture Overview

```mermaid
flowchart TD
    U[User]

    IM[IM Bot]
    CUR[Cursor]
    GEM[Gemini CLI]
    COD[Codex]
    OTHER[Other AI Tools]

    MEM["memory/shared_memory.md<br/>Main long-term memory"]
    MEMA["memory/shared_memory/YYYY.md<br/>Yearly keyword index"]
    MEMM["memory/shared_memory/YYYY-MM.md<br/>Monthly archive"]
    ID["memory/shared_assistant_identity.md<br/>Assistant identity"]
    RULE["core_rules.md<br/>Core behavior rules"]
    PROFILE["memory/{user}/profile.md<br/>User profile & tone"]
    KNOW["knowledge/*.md<br/>Engineering knowledge"]

    U --> IM
    U --> CUR
    U --> GEM
    U --> COD
    U --> OTHER

    IM --> MEM
    CUR --> MEM
    GEM --> MEM
    COD --> MEM
    OTHER --> MEM

    IM --> MEMA
    CUR --> MEMA
    GEM --> MEMA
    COD --> MEMA

    IM --> MEMM
    CUR --> MEMM
    GEM --> MEMM
    COD --> MEMM

    IM --> ID
    CUR --> ID
    GEM --> ID
    COD --> ID

    IM --> RULE
    CUR --> RULE
    GEM --> RULE
    COD --> RULE

    IM --> PROFILE
    CUR --> PROFILE
    GEM --> PROFILE
    COD --> PROFILE

    IM -.reuse experience.-> KNOW
    CUR -.reuse experience.-> KNOW
    GEM -.reuse experience.-> KNOW
    COD -.reuse experience.-> KNOW
    OTHER -.reuse experience.-> KNOW
```

Every entry point converges on the same set of files. `shared_memory.md` holds the current long-term context. Archives hold history. `shared_assistant_identity.md` defines *who the assistant is*. `core_rules.md` defines *how to behave*. `profile.md` defines *how to talk to this user*. `knowledge/*.md` stores engineering-specific reusable knowledge separately.

---

## Phase 0: Detect Mode

Determine what the user needs:

| Mode | When | Next Phase |
|------|------|------------|
| **A: Fresh Setup** | No shared memory exists yet | Phase 1 |
| **B: Migration** | User has scattered memory files, wants to unify | Phase 1 (with import) |
| **C: Add Entry Point** | Shared memory exists, adding a new AI tool | Phase 2 |

---

## Phase 1: Initialize Shared Memory

### Step 1.1: Choose Memory Root

Ask the user where to place the memory directory. Common patterns:

| Project Type | Recommended Root |
|-------------|-----------------|
| Single monorepo | `./memory/` in project root |
| Multi-project | A dedicated shared directory (e.g. `~/my-project/memory/`) |
| Personal assistant | Under the bot/assistant project (e.g. `~/Bot-Project/memory/`) |

### Step 1.2: Create Directory Structure

Create the following directory tree. Use the templates from the `templates/` folder in this skill:

```
{memory_root}/
├── shared_memory.md                    # Main long-term memory (from templates/shared_memory.md)
├── shared_assistant_identity.md        # Assistant identity (from templates/shared_assistant_identity.md)
├── shared_memory/                      # Archive directory
│   ├── {YYYY}.md                       # Yearly keyword index (create when needed)
│   └── {YYYY-MM}.md                    # Monthly archive (create when needed)
├── {user_id}/                          # Per-user directory (optional)
│   └── profile.md                      # User profile & tone preferences
└── core_rules.md                       # Core behavior rules
```

### Step 1.3: Populate Initial Content

For each file, read the corresponding template and customize it:

1. **`shared_memory.md`** — Start with a header and the first entry:
   ```md
   # Shared Long-Term Memory

   <!-- All entries use append-only format: [YYYY-MM-DD HH:MM] summary -->

   [YYYY-MM-DD HH:MM] Initialized shared memory system.
   ```

2. **`shared_assistant_identity.md`** — Ask the user about their assistant's personality, name, tone, and thinking style. Fill in the template.

3. **`core_rules.md`** — Ask the user about their hard rules (e.g., "always execute commands directly", "use Chinese for responses", "never overwrite files without asking"). Fill in the template.

4. **`profile.md`** — Ask the user about their personal preferences, communication style, and any context the assistant should always remember.

### Step 1.4: Migration (Mode B Only)

If the user has existing memory files:

1. Scan for existing memory/context files in the project.
2. Present a list of found files and ask which to import.
3. Merge content into `shared_memory.md` in chronological order.
4. Archive older entries into `{YYYY-MM}.md` files as appropriate.
5. **Do not delete** the original files — let the user do that manually.

---

## Phase 2: Configure Entry Points

For each AI tool the user wants to connect, set up explicit rules that tell the tool where to find and how to use the shared memory. The key principle is: **never rely on the tool "just knowing"** — always write explicit rules.

### Entry Point: Cursor (via `.cursor/rules/`)

Create `.cursor/rules/shared-memory.mdc` in the project root:

```markdown
---
description: Shared memory location and rules (lazy-read, append-only)
globs:
alwaysApply: true
---

## Shared Memory System

### Memory File Locations
- `{memory_root}/shared_memory.md` — Main long-term memory
- `{memory_root}/shared_assistant_identity.md` — Identity definition
- `{memory_root}/core_rules.md` — Behavior rules
- `{memory_root}/{user_id}/profile.md` — User preferences & tone

### When to Read (Lazy-Read — NOT every conversation)
- Read ONLY when the current task requires context you don't have.
- Self-contained requests (git push, bug fix, code edit) → do NOT read memory.
- User asks about past decisions or preferences → read memory.

### Archive Lookup (only when main file doesn't have enough info)
1. Check `{memory_root}/shared_memory/{YYYY}.md` for yearly keyword index
2. Open the matching `{memory_root}/shared_memory/{YYYY-MM}.md` for full context

### Write Rules
- **Append only** — never delete, overwrite, or reorder existing entries
- **Format**: `[YYYY-MM-DD HH:MM] (ToolName) one-line summary` — always include which tool wrote the entry
- **Write ONLY when**: A new long-term preference or decision was explicitly confirmed by the user
- **Do NOT write for**: Routine tasks, bug fixes, git operations, file edits
- **What to write**: User preferences, identity rules, long-term decisions
- **What NOT to write** (use `knowledge/` instead): Bug fixes, scripts, workflows, environment gotchas

### ⚠️ Anti-Chain-Reaction
- Never auto-trigger background tasks (memory read/write, knowledge capture) from a simple user command.
- Never output your internal planning about memory operations. Just do it silently or skip it.
- One task at a time. Finish the user's request first, then consider memory updates.
```

### Entry Point: Gemini CLI (via `GEMINI.md` or global rules)

Create a `GEMINI.md` in the project root, or add to the existing global rules:

```markdown
## Shared Memory System

### Memory File Locations
1. `{memory_root}/shared_memory.md`
2. `{memory_root}/shared_assistant_identity.md`
3. `{memory_root}/core_rules.md`
4. `{memory_root}/{user_id}/profile.md`

### When to Read (Lazy-Read)
- Read ONLY when the current task requires past context you don't have.
- Do NOT read for self-contained tasks (git push, code edit, bug fix).

### Archive Lookup (fallback only)
1. Check `{memory_root}/shared_memory/{YYYY}.md` for yearly keyword index
2. Open the matching `{memory_root}/shared_memory/{YYYY-MM}.md`

### Write Rules
- Append only. Never delete or overwrite.
- Format: `[YYYY-MM-DD HH:MM] (ToolName) one-line summary`
- Write ONLY new long-term preferences or decisions confirmed by the user.
- Do NOT write for routine tasks, bug fixes, or git operations.
- Engineering details → `knowledge/`, not shared memory.

### Anti-Chain-Reaction
- Complete the user's request FIRST. Consider memory updates only after.
- Never narrate your memory operations. Do them silently or skip them.
- One task at a time. No cascading background tasks.
```

### Entry Point: Codex (via `AGENTS.md`)

Create or update `AGENTS.md` in the project root:

```markdown
## Shared Memory System

### Memory File Locations
- `{memory_root}/shared_memory.md` — Long-term context
- `{memory_root}/shared_assistant_identity.md` — Identity definition
- `{memory_root}/core_rules.md` — Behavior rules
- `{memory_root}/{user_id}/profile.md` — User profile

### When to Read (Lazy-Read)
- Read ONLY when the task needs past context. Skip for self-contained tasks.

### Archive Lookup (fallback only)
1. Read `{memory_root}/shared_memory/{YYYY}.md` (yearly keyword index)
2. Read matching `{memory_root}/shared_memory/{YYYY-MM}.md` (monthly archive)

### Write Protocol
- Append-only. No deletions, no overwrites, no reordering.
- Format: `[YYYY-MM-DD HH:MM] (ToolName) summary`
- Write ONLY new long-term decisions confirmed by user.
- Do NOT write for routine tasks or git operations.
- Shared memory = user preferences, identity, long-term rules
- Knowledge base = engineering details, bug fixes, workflows

### Anti-Chain-Reaction
- Finish the user's request first. Memory is secondary.
- No internal monologue about memory operations.
```

### Entry Point: IM Bot (Telegram, Discord, etc.)

For bot frameworks, the shared memory is injected at the prompt assembly stage. The same **lazy-read** principle applies:

1. **Always read** `shared_assistant_identity.md` and `core_rules.md` — these are small, static files that define who the bot is. Loading them is cheap.
2. **Lazy-read** `shared_memory.md` and `profile.md` — only inject these when the conversation topic needs past context (e.g., user references a past decision). For simple requests, skip them.
3. **Write with source tag** — when appending to memory, always include the tool name: `[YYYY-MM-DD HH:MM] (TelegramBot) summary`.
4. **For archive lookup**, implement the same fallback: main file → yearly index → monthly archive.

Example pseudo-code (adapt to your bot framework):

```python
import os
from datetime import datetime

# Static files: small, rarely change. Safe to always load.
STATIC_FILES = ["shared_assistant_identity.md", "core_rules.md"]

# Dynamic files: can be large. Load only when needed.
DYNAMIC_FILES = ["shared_memory.md"]  # + profile.md per user


def build_system_prompt(memory_root, user_id, needs_memory_context=False):
    """
    Build system prompt with shared memory context.
    
    Args:
        needs_memory_context: Set to True only when the user's message
        references past decisions, preferences, or history. For simple
        self-contained requests, set to False to avoid bloating the prompt.
    """
    parts = []

    # 1. Always read static files (identity + rules)
    for filename in STATIC_FILES:
        filepath = os.path.join(memory_root, filename)
        if os.path.exists(filepath):
            parts.append(read_file(filepath))

    # 2. Read user profile (small, per-user)
    profile_path = os.path.join(memory_root, user_id, "profile.md")
    if os.path.exists(profile_path):
        parts.append(read_file(profile_path))

    # 3. Lazy-read: only load shared memory when the task needs it
    if needs_memory_context:
        memory_path = os.path.join(memory_root, "shared_memory.md")
        if os.path.exists(memory_path):
            parts.append(read_file(memory_path))

    return "\n\n---\n\n".join(parts)


def append_to_memory(memory_root, summary, tool_name="Bot"):
    """
    Append a new entry to shared memory.
    Always includes the source tool name for traceability.
    Append-only — never overwrite.
    """
    memory_path = os.path.join(memory_root, "shared_memory.md")
    timestamp = datetime.now().strftime("[%Y-%m-%d %H:%M]")
    with open(memory_path, "a") as f:
        f.write(f"\n{timestamp} ({tool_name}) {summary}\n")


def lookup_archive(memory_root, query_keywords):
    """Fallback: search yearly index, then open monthly archive."""
    year = datetime.now().strftime("%Y")
    index_path = os.path.join(memory_root, "shared_memory", f"{year}.md")
    if os.path.exists(index_path):
        index_content = read_file(index_path)
        # Find which month(s) match the keywords
        # Then read the corresponding YYYY-MM.md
        # Return the relevant context
    return ""
```

### Entry Point: Generic / Other AI Tools

For any AI tool that supports project-level instructions or system prompts, the pattern is always the same:

1. **Tell it which files to read** at session start.
2. **Tell it the archive lookup order**: main → yearly index → monthly archive.
3. **Tell it the write rules**: append-only, timestamped, one-line summaries.
4. **Tell it the boundary**: shared memory vs. knowledge base.

Create the appropriate config file for the tool (e.g., `.windsurfrules`, `rules.md`, `.github/copilot-instructions.md`, etc.) using the same content structure as the Cursor or Gemini examples above.

---

## Phase 3: Set Up Archive Mechanism

### Why Archive Early

Don't wait until the main memory file is too long. Set up the archive structure from day one.

### Archive Structure

```
shared_memory/
├── 2026.md          # Yearly keyword index
├── 2026-01.md       # January archive
├── 2026-02.md       # February archive
└── ...
```

### Yearly Index Format (`YYYY.md`)

```markdown
# 2026 Keyword Index

| Month | Keywords |
|-------|----------|
| 01 | project setup, assistant identity, initial rules |
| 02 | payment flow, PWA deployment, Facebook scraper |
| 03 | shared memory, multi-tool integration |
```

### Monthly Archive Format (`YYYY-MM.md`)

```markdown
# 2026-03 Archive

[2026-03-01 08:00] Initialized shared memory system.
[2026-03-01 14:30] Confirmed assistant identity: Lucy, warm tone, uses ～ and 喔.
[2026-03-05 10:15] Added recurring payment feature to offering flow.
```

### Archive Rules

1. **When to archive**: When the main memory file exceeds ~500 entries.
2. **Who archives**: Archiving is **user-initiated** — e.g., the user says "archive old memory entries". AI tools should NOT auto-archive in the middle of a task. They may *suggest* archiving if they notice the file is large.
3. **How to archive**: Move older entries (keeping the most recent ~100) to the appropriate `YYYY-MM.md`.
4. **Update the yearly index** after each archive operation.
5. **The main file should always stay lightweight** — it represents *current* context, not full history.
6. **Over-limit behavior**: If the main file exceeds ~1000 entries, the AI should append a warning line `[YYYY-MM-DD HH:MM] (System) ⚠️ Memory file is getting long (~N entries). Consider archiving.` and suggest archiving to the user. It should NOT refuse to write or silently auto-archive.

### Lookup Logic (Hard Rule — Only When Lazy-Read Is Triggered)

This lookup logic applies **only** when the task actually requires past context (per lazy-read rules). For self-contained tasks, skip entirely.

```
0. Is this task self-contained? (git push, code edit, bug fix) → SKIP ALL LOOKUPS
1. Read shared_memory.md (main file)
2. If the information you need is NOT in the main file:
   a. Read shared_memory/{current_year}.md (yearly keyword index)
   b. Find which month(s) might contain the info
   c. Read shared_memory/{YYYY-MM}.md (monthly archive)
3. Only after exhausting archives, ask the user
```

---

## Phase 4: Define Write Boundaries

### What Goes WHERE

This is the most important design decision. Without clear boundaries, memory files will become chaotic.

| Content Type | Write To | Example |
|-------------|----------|---------|
| User preference | `shared_memory.md` | "User prefers dark mode in all UIs" |
| Long-term decision | `shared_memory.md` | "Decided to use Next.js for the main site" |
| Identity / tone rule | `shared_assistant_identity.md` | "Speak warmly, use ～ at end of sentences" |
| Hard behavior rule | `core_rules.md` | "Always execute commands directly, don't ask user to run them" |
| User background | `profile.md` | "User is a designer who codes, prefers visual explanations" |
| Bug fix process | `knowledge/*.md` | "Fixed FB timestamp by adjusting timezone offset" |
| Script / workflow | `knowledge/*.md` | "Steam scraper requires specific User-Agent header" |
| Environment gotcha | `knowledge/*.md` | "Node 20 breaks the legacy build, pin to Node 18" |

### The Rule of Thumb

> **If the future value is "remember this person / this habit / this long-term rule" → shared memory.**
>
> **If the future value is "remember how this system works / how to fix this" → knowledge base.**

### Dual-Write Pattern

When something has both types of value:

1. Write a **one-line conclusion** to `shared_memory.md`
2. Write the **full details** to `knowledge/*.md`

Example:
- Memory: `[2026-03-01 14:00] (Cursor) Decided to use append-only pattern for all shared memory writes.`
- Knowledge: Full article on why append-only, how to implement it, and edge cases.

---

## Phase 5: Verify & Deliver

### Verification Checklist

After setup, verify each entry point:

| Check | How |
|-------|-----|
| Files exist | `ls {memory_root}/` should show all expected files |
| Main memory readable | Open the AI tool and ask "what do you know about me?" |
| Identity works | Ask the AI tool "who are you?" — it should respond with the defined identity |
| Rules work | Test a rule (e.g., "run this command" should execute, not instruct) |
| Archive lookup works | Put info only in an archive file, ask the AI about it |
| Write works | Complete a task, check if a new entry was appended |
| Boundary respected | Fix a bug, verify the details went to `knowledge/` not `shared_memory.md` |

### Delivery Summary

After verification, present to the user:

```
✅ Shared Brain setup complete!

📁 Memory root: {memory_root}
🧠 Main memory: shared_memory.md
🪪 Identity: shared_assistant_identity.md
📋 Rules: core_rules.md
👤 Profile: {user_id}/profile.md
📦 Archives: shared_memory/

Connected entry points:
- {list of configured tools}

Remember:
- All writes are append-only
- Archive when main file gets long (~100 entries)
- Engineering knowledge goes to knowledge/*.md, not shared memory
```

---

## Troubleshooting

### Common Issues

**🔴 Infinite loop / chain reaction / internal monologue leak:**
This is the #1 issue. The model receives a simple command (e.g., "push to GitHub") but spirals into reading memory → checking archives → planning writes → narrating all of it.
- **Root cause**: The entry-point rules tell the model to read memory on every conversation, and the model tries to plan all tasks at once.
- **Fix**: Switch to **lazy-read** rules (see Anti-Chain-Reaction section). Memory should only be read when the task actually needs past context.
- **Quick patch**: Add this line to the top of your entry-point rule file:
  ```
  CRITICAL: Do NOT read memory files unless the user's request requires past context. For self-contained tasks (git push, code edit, bug fix), skip all memory operations entirely. Never narrate your internal planning.
  ```

**AI tool ignores shared memory:**
- Verify the rule file is in the correct location for that tool
- Check if the tool requires restarting to pick up new rules
- Ensure the rule file uses the correct format for that tool (e.g., `.mdc` for Cursor)

**Memory file grows too fast:**
- Review what's being written — engineering details should go to `knowledge/`
- Archive more aggressively (keep only ~20-30 recent entries in main file)
- Check if multiple entry points are writing duplicate entries

**Archive lookup not working:**
- Verify the yearly index file exists and contains keyword entries
- Check if the rule explicitly says "must fall back to archive" (not just "may")
- Test by putting unique info only in an archive, then asking for it

**Conflicting writes from multiple tools:**
- Append-only design prevents destructive conflicts
- If entries are duplicated, it's annoying but not harmful — add a de-dup rule
- Consider adding the tool name to each entry: `[2026-03-01 14:00] (Cursor) summary`

**Identity drift across tools:**
- Re-read `shared_assistant_identity.md` — make sure each tool's rules point to it
- Check if any tool has its own identity override that conflicts

---

## Supported AI Tools

This skill is designed to be **model-agnostic** and works with any AI coding assistant. Here's a quick reference for where to place the rule file:

| Tool | Rule File Location |
|------|-------------------|
| Cursor | `.cursor/rules/shared-memory.mdc` |
| Gemini CLI | `GEMINI.md` or `~/.gemini/settings.json` |
| OpenAI Codex | `AGENTS.md` in project root |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Windsurf | `.windsurfrules` in project root |
| Cline / Roo Code | `.clinerules` in project root |
| Aider | `.aider.conf.yml` or `CONVENTIONS.md` |
| Any IM Bot | Inject via system prompt at runtime |
| Other tools | Check the tool's docs for project-level instruction files |

---

## Example Session Flow

1. User: "Set up shared memory for my project"
2. Skill detects Mode A (fresh setup)
3. Asks user for memory root location
4. Creates directory structure with templates
5. Asks user about assistant identity, rules, and profile
6. Populates template files with user's input
7. Asks which AI tools to configure
8. Creates rule files for each tool (Cursor, Gemini, Codex, etc.)
9. Sets up archive structure
10. Runs verification checks
11. Delivers summary

---

## Related Concepts

- **Knowledge Items (KIs)** — For engineering knowledge, use a separate knowledge capture system (e.g., the `auto-knowledge-capture` skill). Shared memory and KIs serve different purposes.
- **Project-level rules** — Each AI tool has its own mechanism for project-level instructions. This skill generates the appropriate format for each tool.
- **Conversation logs** — Some tools (like Antigravity) maintain their own conversation logs. Shared memory is a *curated subset* of what matters long-term, not a conversation transcript.
