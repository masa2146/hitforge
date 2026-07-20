---
name: creative-prompts
description: Use when the user runs /creatives <concept-id> or asks for ad-creative prompts (images + video) for a concept. Requires docs/specs/<id>/spec.md to exist. Writes a fixed style-core plus 8-10 image prompts and a second-by-second video ad plan, all consistent with the spec. Never promises mechanics that are not in the spec.
---

# creative-prompts

Produce a **consistent image + video ad-creative prompt pack** for one concept, anchored to
its spec.

**First:** read `${CLAUDE_PLUGIN_ROOT}/skills/shared/conventions.md` for the idea-history schema (§2) and
language rule (§1). **Language:** write human-facing text in the user's session language;
ad copy is delivered in **both TR and EN** regardless.

## Precondition — spec gate

If `docs/specs/<id>/spec.md` does not exist, **STOP** and tell the user to run
`/spec <id>` first. Read the spec before writing any prompt — every creative must match the
mechanics actually in the spec.

## Golden rule — no misleading ads

**A mechanic that is not in the spec may not be shown or promised in any creative.** The ad
must depict the real game. This applies to every image and every second of the video.

## Output — write three files under `docs/creatives/<id>/`

### 1. `style-core.md` — the fixed core (consistency anchor)
The constant every visual prompt starts from:
- **Color palette** with **hex codes**.
- Art-style definition (render style, line, shading).
- Asset descriptions (characters, props, environment).
- Camera angle and lighting.

Every image prompt begins by restating this core — that is how visual consistency is held.

### 2. `image-prompts.md` — 8–10 numbered prompts
Each prompt = `style-core` + a scene-specific description. Cover:
- **Gameplay screen ×4** — different level states.
- **Near-loss moment ×1.**
- **Win moment ×1.**
- **UI / store screen ×2.**
- **Icon ×2.**

### 3. `video-plan.md` — 15–20s ad
- **Second-by-second storyboard:** 0–3s = mechanic + hook; middle = near-failure;
  end = satisfaction + logo.
- Per frame: which image to use, a **CapCut motion instruction** (zoom / pan / cursor),
  and an **audio suggestion**.
- **One single-shot consistent genAI video prompt** (optional path B).
- **3 ad-copy variants in TR + EN.**

## After writing

Update the concept's row in `docs/research/idea-history.jsonl` (stage is now "creatives")
and update `docs/pipeline-state.md`. Tell the user the folder path, then print the Next step.

## Next step (handoff — conventions §8)

> **Next step:** produce the images (`image-prompts.md`), cut the video (`video-plan.md`),
> and run the fake-ad test. When you have the result, update state with `/pipeline-status`.
