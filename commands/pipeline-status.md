---
description: Summarize the whole pipeline — which concept is at which stage and which approvals are pending — and render docs/pipeline-dashboard.html.
---

Report the state of the HitForge pipeline.

First read `${CLAUDE_PLUGIN_ROOT}/skills/shared/conventions.md` (§4 status, §5 single-file HTML pattern,
dashboard JSON schema).

Steps:

1. Scan `docs/`:
   - `docs/research/idea-history.jsonl` → every concept, its `status` and `signal`.
   - `docs/research/*-concepts.md` → authoritative `status:` (source of truth for approvals).
   - `docs/specs/<id>/spec.md` present? → stage `spec`.
   - `docs/creatives/<id>/` present? → stage `creatives`.
   - Determine each concept's furthest `stage`: `scanned → approved → spec → creatives`.

2. Print a concise summary in the user's session language: each concept with its stage and
   status, plus a **"pending approvals"** list (concepts still `pending`).

3. Render `docs/pipeline-dashboard.html` using the single-file HTML pattern — fill **only**
   the embedded JSON block using the dashboard schema in conventions §5. If the dashboard
   file does not exist yet, create it from that pattern (self-contained, offline, one JSON
   block, concepts and their stages at a glance).

4. Update `docs/pipeline-state.md` and tell the user the dashboard path.
