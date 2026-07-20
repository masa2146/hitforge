---
description: Summarize the whole pipeline — which concept is at which stage and which approvals are pending — and render docs/pipeline-dashboard.html.
---

Report the state of the HitForge pipeline.

First read `${CLAUDE_PLUGIN_ROOT}/skills/shared/conventions.md` (§4 status, §5 single-file HTML pattern +
dashboard JSON schema, §8 Next step).

Steps:

1. Scan `docs/`:
   - `docs/research/idea-history.jsonl` → every concept, its `status` and `signal`
     (including `ORIGINAL` cards from `/ideate` and `SYNTHESIS` cards).
   - `docs/research/*-concepts.md` **and** `docs/research/*-ideas.md` → authoritative
     `status:` (source of truth for approvals).
   - `docs/specs/<id>/spec.md` present? → stage `spec`.
   - `docs/creatives/<id>/` present? → stage `creatives`.
   - Determine each concept's furthest `stage`: `scanned → approved → spec → creatives`.

2. Compute each concept's **`next`** action (conventions §8):
   - `pending` → `approve` (awaiting your decision)
   - `approved` + no spec → `/spec`
   - spec present + no creatives → `/creatives`
   - creatives present → `fake-ad test`

3. Print a concise summary in the user's session language: each concept with its stage,
   status, signal, and its **next** action; plus a **"pending approvals"** list.

4. Render `docs/pipeline-dashboard.html` using the single-file HTML pattern — fill **only**
   the embedded JSON block using the dashboard schema in conventions §5 (include the `next`
   field per concept, shown as a **"next" badge** on each row). If the dashboard file does
   not exist yet, create it from that pattern (self-contained, offline, one JSON block).

5. Update `docs/pipeline-state.md` and tell the user the dashboard path.
