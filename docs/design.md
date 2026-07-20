# HitForge — Design Document

**Date:** 2026-07-20
**Status:** Approved (build in progress)

## What HitForge is

A generic, game-agnostic **mobile-game concept-production pipeline** built as Claude Code
skills and slash commands. It runs a repeatable, human-gated chain:

```
market-scan  →  (human approval)  →  concept-spec  →  creative-prompts
  research         status: approved      spec.md          image/video prompts
```

The project is not tied to any single game. Every user grows their own idea pool from
their own scans. The pipeline has an **idea memory** (`idea-history.jsonl`) so the same
mechanic cannot silently return under a new theme.

## Conventions (language, source-of-truth, gating)

- **Language of committed files** (skills, commands, README, templates): **English.**
- **Language of runtime-generated content** (questions asked to the user, report bodies,
  spec bodies): **matches the language the user is speaking in that session.** Every
  SKILL.md carries this instruction.
- **Source of truth for approvals: markdown.** HTML reports are read-only. The user
  approves/rejects by editing the `status:` field in the concepts markdown; skills read
  status from there and mirror it into `idea-history.jsonl`.
- Every command updates `docs/pipeline-state.md` when it finishes.

## Directory layout

```
.claude/skills/shared/conventions.md          shared contract (single source)
.claude/skills/market-scan/SKILL.md
.claude/skills/concept-spec/SKILL.md
.claude/skills/creative-prompts/SKILL.md
.claude/commands/{market-scan,spec,creatives,pipeline-status}.md
docs/research/idea-history.jsonl               idea memory (one concept per line)
docs/research/template.html                    single-file report template
docs/research/YYYY-MM-DD-concepts.md           per-scan concept cards (approval lives here)
docs/research/YYYY-MM-DD-report.html           rendered from template
docs/specs/<id>/spec.md
docs/creatives/<id>/{style-core,image-prompts,video-plan}.md
docs/pipeline-state.md
docs/pipeline-dashboard.html
README.md
```

## Shared conventions file (`.claude/skills/shared/conventions.md`)

Common procedures live in **one** place, not copied into three SKILL.md files. Each
SKILL.md says *"read `.claude/skills/shared/conventions.md` and follow it"* at the
relevant step. The README **links** to this file and does not repeat the schema.

`conventions.md` defines:

1. **`idea-history.jsonl` schema** — field by field.
2. **`mechanic_fingerprint` normalization** + **Jaccard comparison** with numeric
   thresholds:
   - `>= 0.5` → **same idea**. If existing status is `rejected`, do not produce it (log
     under "eliminated"); if there is concrete new market signal, produce as `revisit`
     with "why rejected before / what changed now".
   - `0.3–0.5` → **similar idea**. May be produced, but the card must name which existing
     `id` it resembles, flagged as a similarity warning.
   - `< 0.3` → **new idea**.
   - Re-suggesting `approved` or `pending` concepts is always forbidden.
3. **`status` lifecycle** — `pending → approved/rejected → revisit`.
4. **Single-file HTML pattern** — zero external deps, inline CSS + vanilla JS, data as a
   single embedded JSON block at end of file; filling = editing only the JSON. Includes
   the JSON block schema.

## Skill 1 — market-scan

- **Step 0 (memory gate):** create `idea-history.jsonl` if missing; read it; compare each
  candidate fingerprint via the Jaccard rules above.
- **Step 1 (momentum):** three time lenses (30d momentum / 3–6mo trend / 12mo durability);
  dated evidence required; signal tag per card: NEW-HOT / RISING / EVERGREEN /
  COOLING (COOLING → not carded, goes to "eliminated").
- **Step 2 (quality gate):** a card enters only if ≥2 independent dated sources (≥1 from
  last 30 days) + sum of 4 scores ≥ 13/20 + no single score = 1 + a one-sentence,
  evidence-backed "why now?". No filler: if only 3 ideas pass, produce 3 and say so.
  No score without justification; no unsupported "this will hit" claims.
- **Step 3 (report):** markdown (source of truth) + `YYYY-MM-DD-report.html` rendered from
  `template.html`.
- **Gates:** if web search is unavailable → **stop and warn** (never fabricate evidence).
  `--manual` mode (only on explicit request) → user supplies source link + publish date;
  cards get `source: "manual entry"` and stay separate from auto scans; same quality gate.

## Skill 2 — concept-spec (`/spec <id>`)

- **Precondition:** the card's `status: approved`; otherwise stop and tell the user.
- **Enhanced single-question loop** (order): core loop → fail-state/difficulty →
  level structure → meta layer → monetization (title level) → out-of-scope (YAGNI).
  One question per turn; after each answer give a short "here's what I understood"
  summary and confirm; concretize vague answers; offer at least one alternative and
  challenge at least one weak point ("is this readable in 3s in an ad?"). `core loop` and
  `out-of-scope` must be answered before writing the spec. If superpowers is installed,
  offer an optional deeper session at loop start; convert its output into our spec.md
  template (do not follow it into writing-plans).
- **spec.md:** core loop; MVP has / deliberately does-not-have (scope lock); level rules;
  Unity architecture (scenes, managers, ScriptableObject schemas, folders); phased
  roadmap (Phase 1 core + 10 levels, Phase 2 fail/win + progression, Phase 3 test-build
  polish) each with acceptance criteria, split into single-session tasks. Monetization/meta
  = Phase 4+ headings only.
- Updates the concept's `idea-history.jsonl` row when done.

## Skill 3 — creative-prompts (`/creatives <id>`)

- **Precondition:** `docs/specs/<id>/spec.md` exists.
- Writes three files: `style-core.md` (fixed core — hex palette, art style, assets, camera,
  light) → `image-prompts.md` (8–10 prompts: gameplay x4, near-loss x1, win x1, UI/store x2,
  icon x2; each = core + scene) → `video-plan.md` (15–20s second-by-second storyboard,
  CapCut motion + audio, optional single-shot genAI video prompt, 3 ad-copy variants TR+EN).
- **Rule:** no mechanic absent from the spec may be promised in the ad (no misleading ads).
- Updates the concept's `idea-history.jsonl` row when done.

## Commands

`/market-scan [--manual]`, `/spec <id>`, `/creatives <id>`, `/pipeline-status`
(scans `docs/`, summarizes which concept is at which stage + pending approvals, and
renders `pipeline-dashboard.html` with the same single-file HTML pattern). Every command
updates `docs/pipeline-state.md`.

## README

GitHub-quality, English: what HitForge is, install, command table, flow diagram, example
run, and the note that **superpowers is optional** (offers a deeper brainstorm if
installed). README links to `conventions.md`; it does not repeat the schema.

## Post-install steps

1. Summarize every file created.
2. Render `template.html` once with sample fake data and show the path.
3. Finalize the README command + flow sections.
4. Give first-run guidance for `/market-scan`.
