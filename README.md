# 🔨 HitForge

**A repeatable, human-gated mobile-game concept-production pipeline — packaged as a Claude
Code plugin (skills + slash commands).**

HitForge takes you from **market research → an approved concept → a buildable spec →
ad-creative prompts**, in one continuous chain. It is **game-agnostic**: it is not tied to
any single game. Every user grows their **own** idea pool from their **own** scans, and an
**idea memory** stops the same mechanic from quietly resurfacing under a new theme.

```
  /market-scan ──[approve]──▶ /spec <id> ──▶ /creatives <id> ──▶ fake-ad test
   (proven cards)     ▲          (Unity spec)   (image + video,
        │             │                          no misleading ads)
        │  (optional) │
        └─▶ /ideate ──┘        idea memory blocks repeats (Jaccard fingerprints)
            ORIGINAL
            ideas (magenta)

  /pipeline-status  ·  dashboard of every concept, its stage & next step
```

market-scan finds **what is proven**; `/ideate` diverges into **what is possible** (both
paths lead to `/spec`). Nothing advances without your approval in the markdown.

---

## Why HitForge

- **Human-gated.** Nothing advances without you. You approve concepts by hand; skills never
  self-promote an idea.
- **Idea memory.** `docs/research/idea-history.jsonl` remembers every concept by its
  *mechanic fingerprint*, so a rejected idea can't come back re-skinned. Comparison is
  numeric (Jaccard), not vibes.
- **Evidence-gated.** A concept becomes a card only with dated sources and a passing score.
  No web search → the scan **stops** rather than inventing market claims.
- **Repeatable.** Run it weekly. Each scan builds on the last.
- **Two languages.** All committed files are English; everything HitForge *generates at
  runtime* (questions, reports, specs) comes out in the language you're speaking. Ad copy is
  always delivered TR + EN.

---

## Install

HitForge is a Claude Code plugin. This repository is its own marketplace, so installing is
two commands inside Claude Code:

```
/plugin marketplace add https://github.com/masa2146/hitforge
/plugin install hitforge@hitforge
```

Then restart/reload Claude Code so it picks up the skills and commands. The commands below
become available as `/hitforge:market-scan`, `/hitforge:spec`, etc. (Claude Code also
accepts the short form `/market-scan` when the name is unambiguous.)

> Prefer not to use a marketplace? Clone the repo and copy `skills/` and `commands/` into
> your project's `.claude/` folder — but then the internal `${CLAUDE_PLUGIN_ROOT}` paths in
> the skills won't resolve. The plugin install is the supported path.

**Requirements:** Claude Code with web search enabled (for `/market-scan`). No other
dependencies; the HTML reports are self-contained and open offline (system-font stack, no
CDN, no external fonts).

### Optional: superpowers plugin
The [superpowers](https://github.com/anthropics) `brainstorming` skill is **optional**. If
it's installed, `/spec` offers to open a deeper brainstorming session; if it isn't, `/spec`
runs its own built-in one-question-at-a-time loop, which is fully sufficient. HitForge works
either way.

---

## Commands

| Command | What it does |
|---------|--------------|
| `/market-scan [--manual]` | Research trending mobile-game mechanics across three time lenses, skip repeats via the idea memory, and produce scored concept cards — a markdown report (**source of truth**) + a self-contained HTML report. `--manual`: supply sources yourself when web search is unavailable. |
| `/ideate [date\|latest]` | **Divergence step** (autonomous, no questions): take a scan's cards as raw material, apply divergence operators, and pour out 3–5 `ORIGINAL` (not-yet-proven) idea cards → `docs/research/<date>-ideas.md` + HTML. Optional middle layer between scan and spec. |
| `/spec <concept-id>` | Turn an **approved** concept into a buildable spec. Runs a guided brainstorm (core loop, fail-state, difficulty, meta, ad-readability), then writes a Unity architecture + a phased roadmap to `docs/specs/<id>/spec.md`. |
| `/creatives <concept-id>` | Generate the ad-creative prompt pack for a concept with a `spec.md`: a fixed `style-core`, 8–10 image prompts, and a 15–20s video ad plan. Never promises mechanics absent from the spec. |
| `/pipeline-status` | Summarize which concept is at which stage and which approvals are pending, and render `docs/pipeline-dashboard.html`. |

Each command updates `docs/pipeline-state.md` when it finishes.

---

## The flow, step by step

1. **`/market-scan`** → produces `pending` concept cards in
   `docs/research/<date>-concepts.md` (+ an HTML report).
2. **(optional) `/ideate latest`** → diverges into 3–5 `ORIGINAL` idea cards in
   `docs/research/<date>-ideas.md` (+ an HTML report). Autonomous; runs no questions.
3. **You approve.** Open the markdown (concepts or ideas) and set `status: approved` (or
   `rejected`). **The markdown is the source of truth — the HTML is read-only.**
4. **`/spec <id>`** → only runs on an `approved` concept. Answers a short brainstorm (for an
   `ORIGINAL` card the first question is its `leap`), then writes `docs/specs/<id>/spec.md`.
5. **`/creatives <id>`** → writes `docs/creatives/<id>/{style-core,image-prompts,video-plan}.md`.
6. **Run the fake-ad test** with those creatives — that is the evidence an ORIGINAL idea was
   always waiting for.
7. **`/pipeline-status`** anytime → refreshes `docs/pipeline-state.md` and
   `docs/pipeline-dashboard.html`, and tells you each concept's **next step**.

Every skill ends with a **Next step** block, so you always know the next move.

---

## Proven vs. possible: `/market-scan` and `/ideate`

The two idea sources are deliberately separate and never mix:

- **`/market-scan`** is **evidence-gated** — a card needs dated sources and a passing score.
  It finds *what is proven*.
- **`/ideate`** is the **mirror**: it takes a scan's proven mechanics as raw material and
  applies **divergence operators** (transplant, invert, constraint-swap, theme-mechanic
  fusion, audience-shift) to produce *what is possible*. `ORIGINAL` cards carry **no evidence
  obligation** — instead each states its `leap` from the parent, a `similarity_check`, a
  3-second `test_hypothesis`, an honest *why it might hit / might not* pair, and the label
  *"market-unvalidated — a fake-ad test will produce the evidence."* They score `novelty_risk`
  in place of `competition_gap`, are never the "top pick", and feed the same `/spec` flow
  once approved.

---

## How concepts are scored

Each card carries four 1–5 scores, **each with a written justification**:

- **Video clarity** — is the mechanic obvious in a 3-second ad?
- **Production ease** — how fast can a small/AI-assisted team build it?
- **Competition gap** — is there room, or is the genre saturated?
- **Meta potential** — how much depth/monetization can sit on top later?

A candidate becomes a card **only if**: ≥ 2 independent dated sources (≥ 1 from the last
30 days) **and** score sum ≥ 13/20 **and** no single score = 1 **and** a one-sentence,
evidence-backed *"why now?"*. **No filler** — if only three ideas clear the bar, you get
three cards.

Every card gets a **signal tag**: `NEW-HOT` (just exploded), `RISING` (climbing 3–6mo,
the ideal entry), `EVERGREEN` (solid 12mo+, crowded), `COOLING` (declining → not carded),
or `SYNTHESIS` (a fused concept — see below).

---

## Synthesis cards & card images

After the evidence-gated cards, `/market-scan` runs a **synthesis** step (Step 2.5): it
crosses the fingerprints of the passing cards to forge 1–2 **`SYNTHESIS`** concepts — a
proven mechanic family plus a differentiating layer (meta, theme, or a second mechanic)
from another card. A synthesis card is honest about being unproven: it carries `parents`,
a one-paragraph `why_combination`, a `visual_preview`, and the fixed note
*"market-unvalidated combination — evidence inherited from parents."* It is scored but is
**never the "top pick"**, and if no sensible combination exists the step produces **zero**
synthesis cards.

Each **reference game** carries its **Google Play / App Store link** (the scan adds it) and
an optional strip of **store slides** shown under the game name. Any card may also carry an
optional card-level **`images`** array (`{src, caption}`) that renders as a thumbnail strip.
Both open in a new tab on click.

**HitForge never auto-downloads or bundles store screenshots** — copyright, and your repo
may be public; the store *link* is enough. Every image `src` is something you add yourself
or a visual produced during `/creatives`. The bundled examples use clearly-labelled
illustrative placeholders, not real store assets.

---

## Repository layout

```
.claude-plugin/
  plugin.json                    Plugin manifest
  marketplace.json               Marketplace manifest (this repo is its own marketplace)
skills/
  shared/conventions.md          Shared contract — schema, fingerprint/Jaccard, HTML pattern
  shared/template.html           Report shell — copied + JSON-filled for every scan/ideate
  market-scan/SKILL.md
  ideate/SKILL.md                Divergence step → ORIGINAL idea cards
  concept-spec/SKILL.md
  creative-prompts/SKILL.md
commands/
  market-scan.md  ideate.md  spec.md  creatives.md  pipeline-status.md
examples/                        Rendered samples of what HitForge produces in *your* project
  2026-07-20-report.html         Rendered sample scan report
  2026-07-20-ideas-report.html   Rendered sample ORIGINAL-ideas report (/ideate)
  2026-07-20-ideas.md            Sample ideas markdown (source of truth)
  idea-history.jsonl             Sample idea memory
  pipeline-state.md              Sample stage snapshot
docs/
  design.md                      Design document
README.md  LICENSE  .gitignore
```

When you run the commands, HitForge writes its output into **your** project (not the
plugin): `docs/research/<date>-concepts.md` + `<date>-report.html`, `docs/specs/<id>/spec.md`,
`docs/creatives/<id>/…`, `docs/research/idea-history.jsonl`, and `docs/pipeline-state.md`.
The `examples/` folder shows what those look like.

## The idea memory & fingerprints

The single contract for how ideas are stored, fingerprinted, and de-duplicated lives in
**[`skills/shared/conventions.md`](skills/shared/conventions.md)** — the
`idea-history.jsonl` schema, the `mechanic_fingerprint` normalization, the numeric Jaccard
thresholds (`≥0.5` same idea · `0.3–0.5` similar · `<0.3` new), the `status` lifecycle, and
the single-file HTML pattern. All skills read from that one file; this README does not
repeat it.

---

## Example

```
> /market-scan
  … researches, checks memory, writes docs/research/2026-07-20-concepts.md
    + docs/research/2026-07-20-report.html  (5 cards, 2 eliminated)

> (edit the markdown: set status: approved on "magnet-crowd-runner")

> /spec magnet-crowd-runner
  … guided brainstorm, then docs/specs/magnet-crowd-runner/spec.md

> /creatives magnet-crowd-runner
  … docs/creatives/magnet-crowd-runner/{style-core,image-prompts,video-plan}.md

> /pipeline-status
  … refreshes docs/pipeline-dashboard.html
```
