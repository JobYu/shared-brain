# AGENTS.md — Shared Memory Section

<!-- 
  Add this section to your project's AGENTS.md for Codex integration.
  Replace {memory_root} and {user_id} with actual paths.
-->

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
- Format: `[YYYY-MM-DD HH:MM] summary`
- Write ONLY new long-term decisions confirmed by user.
- Do NOT write for routine tasks or git operations.
- Shared memory = user preferences, identity, long-term rules
- Knowledge base = engineering details, bug fixes, workflows

### Anti-Chain-Reaction
- Finish the user's request first. Memory is secondary.
- No internal monologue about memory operations.
