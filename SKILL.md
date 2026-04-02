---
name: makino-distilled
invocation: user
description: "Distilled — Daily AI digest in your terminal. 130+ sources scored and structured into JSON. No API keys, no dependencies — just curl."
version: "3.4"
last_updated: "2026-04-02"
---

# Distilled — Don't scroll. Distill.

You are a terminal-based reader for the Distilled AI daily feed.
You do NOT generate, score, or process any content — you fetch and render.
The website is the SSOT. Your job is to present the data in terminal format.

Core value: help users proactively manage AI information, keep up with developments, and reduce information anxiety.

Data updates twice daily (09:25 + 20:25 Beijing time) via VPS crontab.
All times in this skill are Beijing time (UTC+8).
Base URL: `https://feed.makinote.cn`

**Data freshness**: Pipeline runs at 09:25 and 20:25 Beijing time.
If you run before 09:25, you get yesterday's evening data.
If you run between 09:25-20:25, you get today's morning data.
If you run after 20:25, you get today's evening data.

When fetching data, always append `?c=skill` to the URL for analytics.

Endpoints:

| Endpoint | Use for |
|----------|---------|
| `lists/watchlist.json?c=skill` | Watching section (30-day window) + header stats |

SSOT for entity list: `watchlist.json` -> `curated_ids` (sourced from VPS `config/watchlist.json` -> `llm_enabled`).
Do not hardcode entity count or IDs. Read dynamically from `meta` and `curated_ids`.

## Commands

| Command | What it shows |
|---------|--------------|
| `/makino-distilled` | Full digest — Watching all curated entities |
| `/makino-distilled <entity>` | Single entity deep dive (e.g. `/makino-distilled claude`) |

## Execution

### Step 1: Fetch all data

Fetch one endpoint using `curl -s`:
- `lists/watchlist.json?c=skill`

Also fetch the remote SKILL.md to check for updates:
- `curl -s https://raw.githubusercontent.com/makinotes/makino-distilled/main/SKILL.md | head -6`
- Extract the `version:` line from remote, compare with local version `3.4`
- If remote version > local version, prepend this notice before the header:

```
[UPDATE] Distilled v{remote} available (you have v3.4). Run: cd ~/.claude-internal/skills/makino-distilled && git pull
```

- If versions match or curl fails: show nothing, skip silently.

### Step 2: Render header

```
DISTILLED · {MM-DD}
Don't scroll. Distill.
Proactively manage AI info · Keep up with developments · Reduce anxiety

{meta.article_total} articles · {meta.entity_curated} entities · data from {generated_at_beijing}
```

Where:
- `{generated_at_beijing}` = from `watchlist.json` field `generated_at`, convert to Beijing time, format as "MM-DD HH:MM"
- `{meta.*}` = from `watchlist.json` -> `meta` object
- No freshness warning needed. The data timestamp speaks for itself.

### Step 3: Render WATCHING

```
━━━ WATCHING ━━━
```

Source: `watchlist.json`

Show ALL curated entities from `curated_ids` that have content (narrative.sections with articles > 0).
Sort by article count descending. Mark all with `◆`.

`article_count` = sum of articles across all `narrative.sections` (NOT the `total_articles` field).

Per entity, show summary + per-section top 3 articles:

```
◆ {display}  [{type}]  {article_count} articles
  {narrative.summary — smart truncated at sentence boundary, ~180 chars max}

  ── {section.topic} ({section_article_count}) ──
  [{score}] {title}
       {link}  ({date MM-DD})
  [{score}] {title}
       {link}  ({date})
  [{score}] {title}
       {link}  ({date})

  ── {section.topic} ({section_article_count}) ──
  ...

────────────────────────────────────────────────────────────────────
```

Formatting rules:
- Summary: truncate at sentence boundary (。. ；——) near 180 chars. Do NOT cut mid-sentence.
- Title: truncate at word boundary for English (~60 chars). Do NOT cut mid-word.
- Score + title on one line, link + date on next line (compact two-line per article).
- Section header shows total article count in parens, e.g. `(6)`.
- Each section separated by blank line.
- Entity separator: `────` line (68 chars) after each entity.

Per section: show up to 3 articles sorted by score descending.
If a section has fewer than 3, show all.
Skip sections with 0 articles.

---

### Step 4: Render footer

```
--------------------------------------------------------------------
distilled.makinote.cn · manage AI info proactively · by makino
```

### Step 5: Auto-save to local file

After rendering, save the same content to a local markdown file.

**File naming**: `distilled-{YYYY-MM-DD}.md`

For entity detail: `distilled-{YYYY-MM-DD}-{entity_id}.md`

**Date source**: use `date "+%Y-%m-%d"`, do NOT hardcode.

**Save location**: current working directory (`./`).

**File format**: plain text in a .md file (no frontmatter, no markdown headers). Content identical to terminal output.

**After saving**, print:

```
Saved to ./distilled-{YYYY-MM-DD}.md
```

## Entity Detail (`/makino-distilled <entity>`)

Fetch: `lists/watchlist.json`

Find entity by entity_id (case-insensitive match on entity_id or display).
Show full narrative.summary (no truncation) + all sections with ALL articles.

```
◆ {display}  [{type}]  {article_count} articles · last updated {last_updated}

{narrative.summary — full text, no truncation}

  ── {section.topic} ({section_article_count}) ──
  [{score}] {title}
       {link}  ({date MM-DD})
  [{score}] {title}
       {link}  ({date})
  ...

  ── {section.topic} ({section_article_count}) ──
  ...
```

Show ALL articles in each section (no top-3 limit), sorted by score descending.

## Error Handling

- If curl fails or returns empty: "Data unavailable. Try again later or visit distilled.makinote.cn"
- If entity not found: "Entity '{name}' not found. Available: {list of entity displays}"
- If a section has no data: skip that section silently, do NOT show empty headers.

## Notes

- All data is pre-computed. Do NOT add your own analysis, scoring, or commentary.
- Do NOT modify, filter, or re-rank articles. Show them as-is from the JSON.
- Output is plain text for terminal readability. No markdown headers, no bold, no emoji.
- Keep output clean and scannable. Align columns where practical.

## Consumed Fields (API Contract)

This skill consumes JSON from a public API endpoint (`feed.makinote.cn`).
The upstream pipeline produces data; this skill is a read-only consumer.
If any required field is missing, show `[WARN] Missing field: {path}` and continue gracefully.

```
GET https://feed.makinote.cn/lists/watchlist.json

watchlist.json
├── version                   (string, e.g. "2.0")
├── generated_at              (string, ISO 8601 with timezone)
├── default_pins              (string[], default pinned entity IDs)
├── curated_ids               (string[], all curated entity IDs)
├── meta
│   ├── window_days           (int, e.g. 30)
│   ├── article_total         (int)
│   ├── entity_total          (int)
│   ├── entity_curated        (int)
│   ├── entity_with_narrative (int)
│   └── entity_types          (object, e.g. {"Product": 18, "Concept": 27, ...})
└── entities[]
    ├── entity_id             (string)
    ├── display               (string)
    ├── type                  (string, e.g. "Product", "Concept", "Company", "Person", "Tool")
    ├── total_articles        (int, total articles for this entity)
    ├── last_updated          (string, ISO 8601)
    ├── trend                 (object, trend data)
    ├── boards                (object, board/category data)
    └── narrative
        ├── summary           (string)
        └── sections[]
            ├── topic         (string)
            ├── summary       (string, section-level summary)
            ├── keywords      (string[], topic keywords)
            └── articles[]
                ├── title     (string)
                ├── score     (int, 0-100 relevance score)
                ├── date      (string, YYYY-MM-DD)
                ├── link      (string, URL)
                └── summary   (string, article summary)
```

Fields used by this skill: `generated_at`, `curated_ids`, `meta.article_total`, `meta.entity_curated`,
`entities[].{entity_id, display, type, last_updated, narrative.summary, narrative.sections[].topic, narrative.sections[].articles[].{title, score, date, link}}`.

Other fields (`version`, `default_pins`, `boards`, `trend`, `section.summary`, `section.keywords`, `article.summary`) are available but not currently rendered. Future versions may use them.

Upstream: VPS pipeline at `/opt/feed-pipeline/` → JSON published to `feed.makinote.cn` via Vercel CDN.
