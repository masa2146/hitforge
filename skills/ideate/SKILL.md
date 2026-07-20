---
name: ideate
description: Use when the user runs /ideate [scan-date or "latest"] or asks to diverge from a market scan into original, not-yet-proven concepts ("brainstorm original ideas", "what could we invent from this scan"). Autonomous divergence step — the mirror of market-scan: the scan finds what is proven, ideate produces what is possible. Takes a scan's cards as raw material, applies divergence operators, and writes 3-5 ORIGINAL cards. Runs NO question loop with the user.
---

# ideate

The **divergence** step. market-scan finds *what is proven*; ideate produces *what is
possible*. **The two never mix.**

**First:** read `${CLAUDE_PLUGIN_ROOT}/skills/shared/conventions.md` and follow §7 (ORIGINAL
cards + divergence operators + anti-convergence bans + scoring), §3 (fingerprint/Jaccard
dedupe), §2 (idea-history), §5 (HTML shell), and §8 (Next step).

**Autonomous:** ideate runs **no question loop** with the user. It takes the raw material,
applies the operators, and pours out cards. (The interactive brainstorm lives only in
`concept-spec`.)

**Language:** write card prose in the user's session language; keep ids, fingerprints,
`operator` values, and JSON keys in English (conventions §1).

## Step 1 — Load raw material

1. Resolve the target scan from the argument: `latest` (default) → the most recent
   `docs/research/YYYY-MM-DD-concepts.md`; or an explicit `YYYY-MM-DD`. If none exists,
   stop and tell the user to run `/market-scan` first.
2. Read that scan's cards — these are the **raw material** (proven mechanic families).
3. Read `docs/research/idea-history.jsonl` for dedupe.

## Step 2 — Diverge (apply operators)

Produce **3–5** `ORIGINAL` cards. Each card **applies at least one** divergence operator
(conventions §7); **5 cards may not all use the same operator** — spread them.

Operators: **TRANSPLANT** / **INVERT** / **CONSTRAINT-SWAP** / **THEME-MECHANIC FUSION** /
**AUDIENCE-SHIFT**.

Enforce the **anti-convergence bans**:
- theme-only change is not an idea;
- the literal sum of two games is not an idea unless the `leap` says something concretely new;
- "X but better" is not an idea;
- **naming test:** describe the idea in 2 sentences **without any existing game's name**.

Run the **dedupe** (§3) on each card's fused `mechanic_fingerprint`; a `≥0.5` match against a
non-parent concept blocks the card.

## Step 3 — Each ORIGINAL card carries (mandatory)

- `id` (kebab-case), `title`, `status: pending`, `signal: "ORIGINAL"`
- `parents` — the raw-material parent id(s)
- `operator` — the operator applied
- `leap` — one concrete sentence: what is DIFFERENT from the parent
- `mechanic` — 2 sentences, **no existing game name**
- `mechanic_fingerprint` — the new/fused set
- `similarity_check` — `"partly exists"` (+ link) or `"not found"`
- `test_hypothesis` — the 3-second fake-ad moment
- `signal_evidence` — the fixed label `market-unvalidated — a fake-ad test will produce the evidence`
- `scores` — `video_clarity` / `production_ease` / **`novelty_risk`** (5 = very new/risky) /
  `meta_potential`. **No quality bar** — instead:
- `why_might_hit` **and** `why_might_not` — the honest two-sided pair

Optional: `target_market`, `images` (never auto-downloaded, conventions §5).

## Step 4 — Output

1. Write `docs/research/YYYY-MM-DD-ideas.md` — the **source of truth**; approvals happen
   here by editing `status:`.
2. **Append** each ORIGINAL card to `docs/research/idea-history.jsonl` (schema §2),
   `status: pending`, with its `parents`.
3. **Render HTML:** copy `${CLAUDE_PLUGIN_ROOT}/skills/shared/template.html` to
   `docs/research/YYYY-MM-DD-ideas-report.html` and fill **only** the embedded JSON block
   (the shell renders `ORIGINAL` cards in magenta). Never rewrite the HTML/CSS/JS.
4. Update `docs/pipeline-state.md`.
5. **Next step** (conventions §8): tell the user to review the ORIGINAL cards, set one to
   `approved`, then run `/spec <id>` — and that `/spec`'s first brainstorm question will be
   the card's `leap`.
