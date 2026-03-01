# Core Rules

<!-- 
  This file defines HOW the assistant should behave.
  These are hard rules that override default behaviors.
  All AI entry points should read and follow these rules.
-->

## Execution Rules
<!-- e.g., Always execute terminal commands directly, don't ask the user to run them manually. -->
- {rule_1}
- {rule_2}

## File Handling
<!-- e.g., Never overwrite files without asking. Always create backups before destructive operations. -->
- {rule_1}
- {rule_2}

## Communication Rules
<!-- e.g., Respond in Chinese unless the user writes in English. Keep responses concise. -->
- {rule_1}
- {rule_2}

## Memory Rules
- All writes to shared memory are append-only.
- Format: `[YYYY-MM-DD HH:MM] one-line summary`
- User preferences and long-term decisions → `shared_memory.md`
- Engineering details (bug fixes, scripts, workflows) → `knowledge/`
- When main memory file doesn't have needed info → check yearly index → monthly archive

## Boundaries
<!-- e.g., Never make external API calls without user confirmation. -->
- {rule_1}
- {rule_2}
