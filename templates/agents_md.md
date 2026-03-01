# AGENTS.md — Shared Memory Section

<!-- 
  Add this section to your project's AGENTS.md for Codex integration.
  Replace {memory_root} and {user_id} with actual paths.
-->

## Shared Memory System

### Required Reading (every session)
- `{memory_root}/shared_memory.md` — Long-term context
- `{memory_root}/shared_assistant_identity.md` — Identity definition
- `{memory_root}/core_rules.md` — Behavior rules
- `{memory_root}/{user_id}/profile.md` — User profile

### Archive Lookup
When main memory is insufficient:
1. Read `{memory_root}/shared_memory/{YYYY}.md` (yearly keyword index)
2. Read matching `{memory_root}/shared_memory/{YYYY-MM}.md` (monthly archive)

### Write Protocol
- Append-only. No deletions, no overwrites, no reordering.
- Format: `[YYYY-MM-DD HH:MM] summary`
- Shared memory = user preferences, identity, long-term rules
- Knowledge base = engineering details, bug fixes, workflows
