# HitForge — Shared Conventions

**This file is the single source of truth for procedures shared across all HitForge
skills.** Skills reference it instead of duplicating logic. When a SKILL.md says
*"read `${CLAUDE_PLUGIN_ROOT}/skills/shared/conventions.md` and follow it"*, apply the relevant section
below exactly.

Do not copy these procedures into individual skills. If a rule changes, it changes here.

---

## 1. Language convention

- All **committed files** (skills, commands, README, HTML templates) are written in
  **English**.
- All **runtime-generated content** — questions asked to the user, report prose, concept
  card bodies, spec bodies, creative prompts' human-facing text — is written in **the
  language the user is speaking in the current session**. Detect it from the user's
  messages; default to English only if unclear.
- Machine-readable keys stay English regardless of language: JSON field names,
  `mechanic_fingerprint` keywords, `id` slugs, `status` values, signal tags.

---

## 2. `idea-history.jsonl` schema

The idea memory. One concept = one JSON line (JSONL). Located at
`docs/research/idea-history.jsonl`. Create it as an empty file if missing (do **not** run
`mkdir` blindly — the `docs/research/` directory already exists in an installed project).

Each line has exactly these fields:

| field | type | notes |
|-------|------|-------|
| `id` | string | kebab-case unique id, e.g. `conveyor-sort-shooter`. Stable across the whole pipeline. |
| `date` | string | `YYYY-MM-DD`, the scan date this concept was produced. |
| `mechanic_fingerprint` | string[] | 3–6 normalized keywords (see §3). |
| `status` | string | one of `pending` / `approved` / `rejected` / `revisit` (see §4). |
| `scores` | object | `{ "video_clarity": n, "production_ease": n, "competition_gap": n, "meta_potential": n }`, each 1–5. |
| `summary` | string | one-line human summary of the concept (in session language is fine). |
| `signal` | string | signal tag: `NEW-HOT` / `RISING` / `EVERGREEN` / `REVISIT`. |
| `source` | string | `"auto"` (default) or `"manual entry"` for `--manual` scans. |
| `reject_reason` | string? | present only when `status` is `rejected` or `revisit`; why it was rejected. |
| `revisit_reason` | string? | present only when `status` is `revisit`; what concrete new signal changed. |

Example line:

```json
{"id":"conveyor-sort-shooter","date":"2026-07-20","mechanic_fingerprint":["conveyor","shoot","sort"],"status":"pending","scores":{"video_clarity":5,"production_ease":4,"competition_gap":3,"meta_potential":3},"summary":"Sort items on a conveyor by shooting them into color bins.","signal":"RISING","source":"auto"}
```

**Reading:** always read the whole file at the start of a scan and before updating any row.
**Writing a new concept:** append one line. **Updating status:** rewrite the matching
line in place (match by `id`); never duplicate an `id`.

---

## 3. `mechanic_fingerprint` normalization + Jaccard comparison

### Normalization
A fingerprint is **3–6 lowercase English keywords** that capture the *mechanic*, not the
theme. Rules:
- Describe the verb/system, not the skin: `["merge","number","chain"]`, not
  `["candy","kitchen","cute"]`.
- Singular, lowercase, no punctuation. Prefer canonical mechanic words:
  `merge, sort, stack, shoot, dodge, run, draw, pull, pour, fold, tap, idle, tycoon,
  conveyor, physics, crowd, parkour, aim, match, slice`.
- Strip theme/art words (colors, characters, settings). Two games that play identically
  but look different must yield near-identical fingerprints — that is the whole point.

### Jaccard comparison
For two fingerprints A and B (as sets):

```
J(A, B) = |A ∩ B| / |A ∪ B|
```

Compare each **candidate** against **every existing** fingerprint in `idea-history.jsonl`.
Use the highest J found against any existing concept, and apply these **numeric
thresholds**:

| J value | meaning | action |
|---------|---------|--------|
| **≥ 0.5** | **same idea** | If the matched concept is `rejected`: **do not produce**; list it under "Eliminated" as *"already rejected (date)"*. If `rejected` **but** there is concrete new market signal (new chart entry, new hit game): may produce as **`revisit`** with `reject_reason` + `revisit_reason`. If matched concept is `approved` or `pending`: **never re-suggest** (forbidden). |
| **0.3 – 0.5** | **similar idea** | May be produced, but the card **must name the resembling existing `id`** and carry a "similar to `<id>` (J≈0.xx)" warning. |
| **< 0.3** | **new idea** | Produce normally. |

`approved` and `pending` concepts are **never** re-suggested regardless of J.

---

## 4. `status` lifecycle

```
pending ──approved──▶ approved
   │
   └──rejected──▶ rejected ──new signal──▶ revisit
```

- **`pending`** — freshly produced by a scan; awaiting the user's decision.
- **`approved`** — the user approved it (edited `status:` in the concepts markdown).
  Only `approved` concepts may enter `/spec`.
- **`rejected`** — the user rejected it; carries a `reject_reason`. Never re-suggested as-is.
- **`revisit`** — a previously `rejected` concept resurfaced with concrete new market
  signal; carries both `reject_reason` and `revisit_reason`.

**Approval mechanism:** the user edits the `status:` field in the per-scan concepts
markdown (`docs/research/YYYY-MM-DD-concepts.md`). Skills read status from the markdown and
mirror it into `idea-history.jsonl`. **HTML is read-only** and never authoritative.

---

## 5. Single-file HTML pattern

The report shell **ships with the plugin** at `${CLAUDE_PLUGIN_ROOT}/skills/shared/template.html`.
market-scan renders a report by **copying that shell** and replacing only its JSON block — it
does not regenerate the HTML/CSS/JS. The dashboard (`docs/pipeline-dashboard.html`, built by
`/pipeline-status`) follows the same single-file pattern. Both obey:

- **One file, zero external dependencies.** No CDN, no external CSS/JS/fonts (use a
  system-font stack). Opens offline by double-click.
- **Inline `<style>` + vanilla `<script>`.** No frameworks.
- **Data lives in a single embedded JSON block** near the end of the file:
  ```html
  <script id="hitforge-data" type="application/json">
  { ... }
  </script>
  ```
  The JS reads `document.getElementById('hitforge-data').textContent`, `JSON.parse`s it,
  and renders. **To fill a report you edit only this JSON block** — never the CSS/JS.
- **Responsive + readable contrast**; must look correct on mobile widths.

### Report JSON block schema (`template.html`)
```json
{
  "scan_date": "YYYY-MM-DD",
  "summary": { "card_count": 0, "eliminated_count": 0,
               "signals": { "NEW-HOT": 0, "RISING": 0, "EVERGREEN": 0 } },
  "cards": [
    {
      "id": "kebab-id",
      "title": "Human title",
      "status": "pending|approved|rejected|revisit",
      "signal": "NEW-HOT|RISING|EVERGREEN",
      "signal_evidence": "dated one-liner justifying the tag",
      "mechanic": "2-sentence mechanic summary",
      "fingerprint": ["kw1","kw2","kw3"],
      "reference_games": ["Game A", "Game B"],
      "scores": { "video_clarity": 5, "production_ease": 4,
                  "competition_gap": 3, "meta_potential": 3 },
      "cpi_range": "$0.30–0.60",
      "target_market": "US / Tier-1",
      "risks": ["risk 1", "risk 2"],
      "sources": [ { "label": "Sensor Tower — ...", "url": "https://...", "date": "YYYY-MM-DD" } ],
      "source": "auto|manual entry",
      "similar_to": null
    }
  ],
  "eliminated": [ { "id": "kebab-id", "reason": "already rejected (2026-05-01)" } ]
}
```

### Signal badge colors (consistent everywhere)
- `RISING` → green
- `NEW-HOT` → orange
- `EVERGREEN` → blue
- `REVISIT` → purple

### Dashboard JSON block schema (`pipeline-dashboard.html`)
```json
{
  "generated": "YYYY-MM-DD",
  "concepts": [
    { "id": "kebab-id", "title": "Human title", "status": "pending|approved|rejected|revisit",
      "stage": "scanned|approved|spec|creatives",
      "has_spec": false, "has_creatives": false,
      "signal": "RISING", "date": "YYYY-MM-DD" }
  ],
  "pending_approvals": ["kebab-id", "..."]
}
```
The `stage` field is the furthest pipeline step reached:
`scanned` → `approved` → `spec` (spec.md exists) → `creatives` (creatives written).
