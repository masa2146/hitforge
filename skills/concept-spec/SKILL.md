---
name: concept-spec
description: Use when the user runs /spec <concept-id> or asks to turn an approved market-scan concept into a buildable spec. Requires the concept's status to be approved. Runs an enhanced one-question-at-a-time brainstorm (core loop, fail-state, difficulty, meta, ad-readability) and then writes docs/specs/<id>/spec.md with a Unity architecture and a phased roadmap. Optionally uses the superpowers brainstorming skill for a deeper session.
---

# concept-spec

Turn one **approved** concept into a buildable, test-build-oriented spec.

**First:** read `${CLAUDE_PLUGIN_ROOT}/skills/shared/conventions.md` and follow it for status (§4) and
idea-history (§2). **Language:** conduct the brainstorm and write spec prose in the user's
session language; keep ids, schema keys, and code identifiers in English (conventions §1).

## Precondition — approval gate

1. Find the concept `<id>` in `docs/research/*-concepts.md` and `idea-history.jsonl`.
2. If its `status` is **not `approved`**, **STOP** and tell the user:
   *"Concept `<id>` is `<status>`, not `approved`. Approve it first by setting
   `status: approved` in its concepts markdown, then re-run `/spec <id>`."*
3. Only proceed when `status: approved`.

## Step 1 — Enhanced single-question brainstorm

Run the brainstorm **before** writing any spec. This is the built-in loop; it is the
primary path.

**If the superpowers plugin is installed**, at the very start offer, in one line:
*"If you want, I can open a deeper superpowers brainstorming session instead of my built-in
loop — say the word."* If the user accepts, invoke `superpowers:brainstorming`, but when it
finishes **do not follow it into writing-plans** — take its output and convert it into our
`spec.md` template yourself (Step 2). If the user declines or superpowers is absent, run the
built-in loop below.

### Built-in loop rules
- **One question per turn.** Do not advance until the current answer arrives.
- **Order:** core loop → fail-state / difficulty curve → level structure → meta layer →
  monetization (title level only) → out-of-scope (YAGNI).
- After **each** answer: give a short *"here's what I understood"* summary and have the
  user confirm the assumption. **Do not accept vague answers — concretize them.**
- **At least once**, offer an alternative: *"we could also build it this way — your
  preference?"*
- **At least once**, challenge a weak point: *"is this mechanic readable in 3 seconds in an
  ad?"*
- The user may say **"go to spec"** to end early — **but** the `core loop` and
  `out-of-scope` questions must be answered before any spec is written.

## Step 2 — Write `docs/specs/<id>/spec.md`

The spec targets a **test build**. Include:

- **Core loop** — step by step, one paragraph.
- **Mechanics list** — what's **in the MVP** vs. what is **deliberately NOT** in it
  (scope lock).
- **Level design rules** — hand-made or procedural, and the difficulty curve.
- **Unity architecture** — scene list, managers, data model (ScriptableObject schemas),
  folder structure.
- **Phased roadmap** with acceptance criteria per phase ("phase ends when ..."):
  - **Phase 1** — working core loop + 10 levels.
  - **Phase 2** — fail/win flow + simple progression.
  - **Phase 3** — test-build polish.
  - Split each phase into tasks that finish in a **single AI-assisted session**.
- **Phase 4+** — monetization and meta as **headings only** (no detail).

## Step 3 — Update memory

Update the concept's row in `docs/research/idea-history.jsonl` (its stage is now "spec").
Update `docs/pipeline-state.md`. Tell the user the spec path and that the next step is
`/creatives <id>`.
