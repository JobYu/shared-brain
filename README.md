# shared-brain

Sync long-term memory across multiple AI entry points (Cursor, Gemini CLI, Codex, and bots).

## What This Does

When you work with multiple AI tools (Cursor, Gemini CLI, Codex, IM bots, etc.), each tool usually has its own context window and memory. This means you have to re-explain yourself every time you switch tools.

**shared-brain** creates a single set of memory files that all your AI tools read from and write to. Switch tools without losing context.

## Key Features

- 🧠 **Single shared memory** — One main file, one identity, one set of rules
- 📂 **Smart archiving** — Yearly indexes + monthly archives keep memory manageable
- 🔌 **Model-agnostic** — Works with Cursor, Gemini CLI, Codex, Copilot, Windsurf, IM bots, and more
- ✍️ **Append-only** — Safe for multi-tool concurrent use
- 📏 **Clear boundaries** — Memory vs. knowledge base, well-defined
- 🛡️ **Anti-chain-reaction** — Lazy-read design prevents LLMs from spiraling into infinite planning loops

## Installation

### For Antigravity / Gemini CLI Users

Copy this folder into your project's `.agent/skills/` directory:

```bash
cp -r shared-brain /path/to/your-project/.agent/skills/
```

### For Claude Code Users

```bash
# Install via Claude Code slash command
/install-skill https://github.com/JobYu/shared-brain
```

Or manually copy `SKILL.md` into your project's skill directory.

### For Other AI Tools

Read `SKILL.md` and follow the Phase 2 instructions to create the appropriate rule file for your tool.

## Usage

Tell your AI assistant:

> "Set up shared memory for this project"

or

> "I want all my AI tools to share the same memory"

The skill will guide you through setup step by step.

## ⚠️ Anti-Chain-Reaction Design

The #1 gotcha with shared memory systems: telling an LLM to "read memory on every conversation" causes **infinite planning loops**, especially when combined with other automation skills.

**shared-brain** uses **lazy-read** instead of eager-read:

| Pattern | Behavior |
|---------|----------|
| ❌ Eager-read | Read all 4 memory files at every conversation start → task list explodes → model narrates its planning → infinite loop |
| ✅ Lazy-read | Read memory **only** when the task actually needs past context. Self-contained tasks (git push, code edit) skip memory entirely |

See `SKILL.md` → "Anti-Chain-Reaction Rules" for the full 5-rule guardrail system.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions (the AI reads this) |
| `README.md` | This file (for humans) |
| `templates/` | Template files for initializing shared memory |

## Supported AI Tools

| Tool | Rule File |
|------|-----------|
| Cursor | `.cursor/rules/shared-memory.mdc` |
| Gemini CLI | `GEMINI.md` |
| Codex | `AGENTS.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Windsurf | `.windsurfrules` |
| Cline / Roo Code | `.clinerules` |
| IM Bots | System prompt injection |

## License

MIT — Use it, modify it, share it.
