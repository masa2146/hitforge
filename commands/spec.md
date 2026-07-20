---
description: Turn an approved concept into a buildable Unity spec. Usage: /spec <concept-id>. Requires the concept's status to be approved.
---

Invoke the `concept-spec` skill (`${CLAUDE_PLUGIN_ROOT}/skills/concept-spec/SKILL.md`) for concept id
`$ARGUMENTS` and follow it exactly.

If no id is given, list the `approved` concepts from `docs/research/idea-history.jsonl` and
ask which one. Enforce the approval gate before brainstorming.

When finished, update `docs/pipeline-state.md`.
