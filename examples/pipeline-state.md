# HitForge — Pipeline State

_Auto-updated by every command. Human-readable snapshot of where each concept stands._

**Last updated:** 2026-07-20 (initial scaffold — no scans yet)

## Stages

`scanned → approved → spec → creatives`

## Concepts

| id | title | signal | status | stage | spec | creatives |
|----|-------|--------|--------|-------|------|-----------|
| _(none yet — run `/market-scan`)_ | | | | | | |

## Pending approvals

_None yet._

## Recent activity

- 2026-07-20 — Project scaffolded. Skills, commands, conventions, template, README in place.

---

### How to advance the pipeline
1. `/market-scan` → produces `pending` concept cards.
2. Approve in `docs/research/<date>-concepts.md` by setting `status: approved`.
3. `/spec <id>` → writes `docs/specs/<id>/spec.md`.
4. `/creatives <id>` → writes `docs/creatives/<id>/`.
5. `/pipeline-status` → refreshes this file and `docs/pipeline-dashboard.html`.
