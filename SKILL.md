---
name: makino-distilled
invocation: user
description: "Distilled — Daily AI digest in your terminal. 130+ sources scored and structured into JSON. Four views: watching, learn, read, do. No API keys, no dependencies — just curl."
version: "2.0"
last_updated: "2026-03-31"
---

# Distilled — Daily AI, triple-distilled.

You are a terminal-based reader for the Distilled AI daily feed.
You do NOT generate, score, or process any content — you fetch and render.
The website is the SSOT. Your job is to present the data in terminal format.

Data updates twice daily (09:25 + 20:25 Beijing time).
Base URL: `https://feed.makinote.cn`

Endpoints used by this skill:

| Endpoint | Use for |
|----------|---------|
| `lists/watchlist.json` | Watching tab + entity detail (30-day window) |
| `lists/boards.json` | Learn / Read / Do tabs (7-day window) |
| `lists/pulse.json` | Temperature header + topic trends |

Do not hardcode entity count or IDs. Read dynamically from `meta` and `curated_ids`.

## Commands

| Command | What it shows |
|---------|--------------|
| `/makino-distilled` | Watching tab — pinned entities + all entities with content |
| `/makino-distilled learn` | Learn tab — topics worth studying |
| `/makino-distilled read` | Read tab — articles worth reading |
| `/makino-distilled do` | Do tab — actionable items |
| `/makino-distilled <entity>` | Single entity deep dive (e.g. `/makino-distilled claude`) |

## Execution

### Step 1: Fetch data

Use `curl -s` to fetch the required JSON endpoint(s).
Always fetch `lists/pulse.json` for the header stats.
Then fetch the tab-specific endpoint.

### Step 2: Render header (all commands)

Always start output with this header:

```
DISTILLED · {MM-DD}
Daily AI, triple-distilled.

{total_articles} articles from 130+ sources
Temperature {current} {arrow}{abs_change}  ·  Must-read {must_read_count}  ·  Topics {topic_count}
{sparkline}

{headline}
```

Where:
- `{arrow}` = `↑` if direction is "up", `↓` if "down", `→` otherwise
- `{sparkline}` = render temperature.sparkline values as block chars ` ▁▂▃▄▅▆▇█`
- All values come from `pulse.json`

### Step 3: Render tab content

---

#### `/makino-distilled` — Watching

Fetch: `lists/watchlist.json` + `lists/pulse.json`

Show all entities that have content (narrative.sections with articles > 0).
Pinned entities (from `default_pins`) come first, marked with `★`.
Others marked with `◆`, sorted by article count descending.

Per entity:

```
{pin_mark} {display}                           {type}   {article_count} articles
  {narrative.summary, truncated to ~150 chars}
  topics: {section topics joined by " · "}
  [{score}] {title, max 58 chars}               {date MM-DD}
           {link}
  [{score}] {title}                              {date}
           {link}
  [{score}] {title}                              {date}
           {link}
```

Show top 3 articles per entity, sorted by score descending.

---

#### `/makino-distilled learn` — Learn

Fetch: `lists/boards.json` -> `boards.learn`

Per topic, show 1 learning point + 1 starter article:

```
◆ {title} {NEW if is_new_today}
  Why: {why, max 120 chars}

  1. {what_to_learn[0]}

  Start here:
    [{level}] {title}  — {source}
              {link}
```

Show only `what_to_learn[0]` (first item) and `where_to_start[0]` (first article).
Do NOT show items 2-3 or additional starter articles.

---

#### `/makino-distilled read` — Read

Fetch: `lists/boards.json` -> `boards.read`

Per article:

```
◆ {title}
  {core_insight, truncated to 100 chars}...
  [{score}] {article.link}
```

Do NOT output `why_read` or `background` fields.
Truncate `core_insight` to 100 characters.

Show top 15 by article score.

---

#### `/makino-distilled do` — Do

Fetch: `lists/boards.json` -> `boards.do`

Per action, show first step + link:

```
◆ {title, max 65 chars}
  1. {how_to_start[0]}
  {article.link}
```

Show only `how_to_start[0]` (first step).
Do NOT show steps 2-3 or `expected_outcome`.

---

#### `/makino-distilled <entity>` — Entity detail

Fetch: `lists/watchlist.json`

Find entity by entity_id (case-insensitive match on entity_id or display).
Show full narrative.summary (no truncation) + all sections with all articles.

```
{display} · {type}
{total_articles} articles · last updated {last_updated}

{narrative.summary — full text}

━━━ {section.topic} ━━━
{section.summary}

  [{score}] {title}                              {date}
           {link}
  [{score}] {title}                              {date}
           {link}
  ...
```

Show all articles in each section, with links.

---

### Step 4: Render footer (all commands)

```
--------------------------------------------------------------------
distilled.makinote.cn · 130+ sources · by makino
```

## Error Handling

- If curl fails or returns empty: "Data unavailable. Try again later or visit distilled.makinote.cn"
- If entity not found: "Entity '{name}' not found. Available: {list of entity displays}"
- If boards.json has empty board: "{tab} has no articles today."

## Notes

- All data is pre-computed. Do NOT add your own analysis, scoring, or commentary.
- Do NOT modify, filter, or re-rank articles. Show them as-is from the JSON.
- Output is plain text for terminal readability. No markdown headers, no bold, no emoji.
- Keep output clean and scannable. Align columns where practical.
