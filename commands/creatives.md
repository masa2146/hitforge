---
description: Generate the image + video ad-creative prompt pack for a concept. Usage: /creatives <concept-id>. Requires docs/specs/<id>/spec.md to exist.
---

Invoke the `creative-prompts` skill (`${CLAUDE_PLUGIN_ROOT}/skills/creative-prompts/SKILL.md`) for
concept id `$ARGUMENTS` and follow it exactly.

If no id is given, list concepts that have a `spec.md` and ask which one. Enforce the spec
gate and the no-misleading-ads rule.

When finished, update `docs/pipeline-state.md`.
