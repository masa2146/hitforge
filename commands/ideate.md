---
description: Diverge from a market scan into 3-5 original, not-yet-proven concepts. Usage: /ideate [scan-date or "latest"]. Autonomous — runs no question loop.
---

Invoke the `ideate` skill (`${CLAUDE_PLUGIN_ROOT}/skills/ideate/SKILL.md`) for target
`$ARGUMENTS` (default `latest`) and follow it exactly.

ideate is the divergence step: it takes a scan's cards as raw material and pours out
`ORIGINAL` cards. It runs **no** question loop.

When finished, update `docs/pipeline-state.md` and print the Next step block.
