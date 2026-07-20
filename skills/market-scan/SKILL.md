---
name: market-scan
description: Use when the user runs /market-scan, asks for a "market scan / pazar taraması", wants fresh trending mobile-game concepts to build, or asks "what should I build next". Researches the app/game market across three time lenses, checks the idea memory to avoid repeats, and produces scored concept cards as a markdown report plus a single-file HTML report. Supports --manual mode when no web search is available.
---

# market-scan

Produce a batch of **scored, non-repeating mobile-game concept cards** from live market
research, gated by evidence quality and idea memory.

**First:** read `${CLAUDE_PLUGIN_ROOT}/skills/shared/conventions.md` and follow it for the idea-history
schema (§2), fingerprint + Jaccard rules (§3), status lifecycle (§4), and the single-file
HTML pattern (§5). Everything below assumes those rules.

**Language:** write the report and all card prose in the user's session language; keep
machine keys (ids, fingerprints, signal tags, JSON) in English (conventions §1).

## Web-search gate (do this before anything else)

This skill requires **live, dated web evidence**. Check whether you can perform web search
(WebSearch/WebFetch or an equivalent browsing tool):

- **Web search available →** proceed with the automatic scan below.
- **Web search NOT available and no `--manual` flag →** **STOP.** Tell the user:
  *"I can't reach live web search, so I can't gather dated evidence. Fabricating market
  claims is not allowed. Re-run with web search enabled, or run `/market-scan --manual` to
  supply sources yourself."* Do not produce any cards.
- **`--manual` requested →** run the Manual mode section at the bottom instead.

Never invent chart positions, dates, CPI figures, or source links. Unsupported "this will
hit" claims are forbidden.

## Step 0 — Memory gate (mandatory first step)

1. Ensure `docs/research/idea-history.jsonl` exists (create empty if missing).
2. Read the whole file.
3. As you form candidate concepts, compute each candidate's `mechanic_fingerprint` and
   compare it against **every** existing fingerprint using the Jaccard thresholds
   (conventions §3):
   - **≥ 0.5 & existing is `rejected`** → do not produce; add to the report's "Eliminated"
     section as *"already rejected (date)"*. Exception: concrete new signal → produce as
     `revisit` with both `reject_reason` and `revisit_reason`.
   - **≥ 0.5 & existing is `approved`/`pending`** → never re-suggest (skip silently or note
     in Eliminated as *"already in pipeline"*).
   - **0.3–0.5** → may produce, but the card must name the resembling `id` with a
     "similar to `<id>` (J≈0.xx)" warning.
   - **< 0.3** → new idea.

## Step 1 — Time-windowed research (momentum analysis)

Research the market through **three lenses**, classifying every source by publish date:

- **Last 30 days → momentum** (chart moves, new entries).
- **Last 3–6 months → trend direction** (rising or plateau).
- **Last 12 months → durability** (a year on the charts, or seasonal?).

Source priorities: top download/revenue chart analyses (Sensor Tower, AppMagic, data.ai),
rising-mechanic breakdowns, and ad-creative trends on Meta Ad Library / TikTok. **Always
put the current month/year in your queries.** A source with an unclear/absent publish date
does **not** count as momentum evidence.

Give every card a **signal tag** backed by dated evidence inside the card:

| tag | meaning |
|-----|---------|
| **NEW-HOT** | blew up in the last 30 days — risky but fast opportunity |
| **RISING** | climbing for 3–6 months — ideal entry point |
| **EVERGREEN** | solid 12+ months — heavy competition |
| **COOLING** | peaked 6+ months ago, declining → **no card**; list under Eliminated |

## Step 2 — Card production + quality gate

Target **5–8 cards**, but **no filler**: if only N ideas clear the bar, produce N cards and
say so explicitly in the report.

A candidate becomes a card **only if all hold**:
- **≥ 2 independent sources** (link + date), **at least one from the last 30 days**.
- **Sum of the 4 scores ≥ 13/20 AND no single score = 1.**
- A one-sentence, evidence-backed answer to **"why now?"**.

Card fields (write into `docs/research/YYYY-MM-DD-concepts.md`):
- `id`: kebab-case unique id
- `status: pending` (the user manually sets approved/rejected)
- signal tag + its dated evidence
- mechanic summary (2 sentences) + `mechanic_fingerprint`
- reference games + chart evidence (with source link)
- scores (1–5), **each with a one-line justification**: `video_clarity` /
  `production_ease` / `competition_gap` / `meta_potential`
- estimated CPI range + target market
- 2–3 risks
- if 0.3–0.5 Jaccard match: the "similar to `<id>`" warning

**Rules:** no score without justification; no unsupported "it'll hit" claim; state
uncertainty explicitly in the card. Candidates that fail the bar go in a one-line
**"Eliminated"** section at the end with the reason.

## Step 3 — Output

1. Write the **markdown report** `docs/research/YYYY-MM-DD-concepts.md` — this is the
   **source of truth**; approvals happen here by editing `status:`.
2. **Append** each produced card as one line to `docs/research/idea-history.jsonl`
   (schema §2), `status: pending`.
3. **Render HTML:** copy `docs/research/template.html` to
   `docs/research/YYYY-MM-DD-report.html` and fill **only** the embedded JSON block
   (conventions §5). If `template.html` is missing, create it first from the pattern in
   conventions §5.
4. Update `docs/pipeline-state.md` (add the scan + its pending concepts).
5. Tell the user the exact file paths and remind them: **approve/reject by editing the
   `status:` field in the markdown**, then run `/spec <id>` on an approved concept.

## Manual mode (`--manual`, only on explicit request)

When web search is unavailable and the user explicitly asks for manual mode:
- Ask the user to supply, per candidate, **source link(s) + publish date(s)**.
- Fill cards **only** with the user-provided evidence — invent nothing.
- Tag these cards `source: "manual entry"` in both the markdown and the HTML/JSON so they
  never mix with automatic scans.
- The **same quality gate applies**: undated or linkless evidence is rejected; ideas that
  fail the bar cannot become cards.
- Default behavior is always the automatic scan; manual mode runs only when explicitly
  requested.
