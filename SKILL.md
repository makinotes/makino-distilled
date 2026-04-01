---
name: makino-distilled
invocation: user
description: "Distilled — Daily AI digest in your terminal. 130+ sources scored and structured into JSON. No API keys, no dependencies — just curl."
version: "3.0"
last_updated: "2026-04-01"
---

# Distilled — Daily AI, triple-distilled.

You are a terminal-based reader for the Distilled AI daily feed.
You do NOT generate, score, or process any content — you fetch and render.
The website is the SSOT. Your job is to present the data in terminal format.

Data updates twice daily (09:25 + 20:25 Beijing time).
Base URL: `https://feed.makinote.cn`

When fetching data, always append `?c=skill` to the URL for analytics.

Endpoints:

| Endpoint | Use for |
|----------|---------|
| `lists/pulse.json?c=skill` | Temperature header |
| `lists/watchlist.json?c=skill` | Watching section (30-day window) |
| `lists/boards.json?c=skill` | Learn / Read / Do sections (7-day window) |

Do not hardcode entity count or IDs. Read dynamically from `meta` and `curated_ids`.

## Commands

| Command | What it shows |
|---------|--------------|
| `/makino-distilled` | Full digest — Watching + Read + Learn + Do in one page |
| `/makino-distilled <entity>` | Single entity deep dive (e.g. `/makino-distilled claude`) |

## Execution

### Step 1: Fetch all data

Fetch all three endpoints in parallel using `curl -s`:
- `lists/pulse.json?c=skill`
- `lists/watchlist.json?c=skill`
- `lists/boards.json?c=skill`

Also fetch the remote SKILL.md to check for updates:
- `curl -s https://raw.githubusercontent.com/makinotes/makino-distilled/main/SKILL.md | head -6`
- Extract the `version:` line from remote, compare with local version `3.0`
- If remote version > local version, prepend this notice before the header:

```
[UPDATE] Distilled v{remote} available (you have v3.0). Run: cd ~/.claude-internal/skills/makino-distilled && git pull
```

- If versions match or curl fails: show nothing, skip silently.

### Step 2: Render header

```
DISTILLED · {MM-DD}
Daily AI, triple-distilled.

{total_articles} articles from 130+ sources · updated {generated_at}
Temperature {current} {arrow}{abs_change}  ·  Must-read {must_read_count}  ·  Topics {topic_count}
{sparkline}

{headline}
```

Where:
- `{generated_at}` = from `pulse.json` field `generated_at`, format as "HH:MM UTC+8"
- `{arrow}` = `↑` if direction is "up", `↓` if "down", `→` otherwise
- `{sparkline}` = render temperature.sparkline values as block chars ` ▁▂▃▄▅▆▇█`

### Step 3: Render all sections

Output four sections top-to-bottom, separated by section headers.

---

#### Section 1: WATCHING

```
━━━ WATCHING ━━━
```

Source: `watchlist.json`

Show all curated entities that have content (narrative.sections with articles > 0).
Pinned entities (from `default_pins`) come first, marked with `★`.
Others marked with `◆`, sorted by article count descending.

Per entity, show summary + top 3 articles:

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

---

#### Section 2: READ

```
━━━ READ — articles worth reading ━━━
```

Source: `boards.json` -> `boards.read`

Show top 10 by article score:

```
◆ {title}
  {core_insight, truncated to 100 chars}...
  [{score}] {article.link}
```

Do NOT output `why_read` or `background` fields.

---

#### Section 3: LEARN

```
━━━ LEARN — topics worth studying ━━━
```

Source: `boards.json` -> `boards.learn`

Per topic, show 1 learning point + 1 starter article:

```
◆ {title} {NEW if is_new_today}
  Why: {why, max 120 chars}
  1. {what_to_learn[0]}
  Start: [{level}] {where_to_start[0].title} — {source}
         {link}
```

Show only `what_to_learn[0]` and `where_to_start[0]`. Do NOT show items 2-3.

---

#### Section 4: DO

```
━━━ DO — try it now ━━━
```

Source: `boards.json` -> `boards.do`

Per action, show first step + link:

```
◆ {title, max 65 chars}
  1. {how_to_start[0]}
  {article.link}
```

Show only `how_to_start[0]`. Do NOT show steps 2-3 or `expected_outcome`.

---

### Step 4: Render footer

```
--------------------------------------------------------------------
distilled.makinote.cn · 130+ sources · by makino
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
Show full narrative.summary (no truncation) + all sections with all articles.

```
{display} · {type}
{total_articles} articles · last updated {last_updated}

{narrative.summary — full text}

━━━ {section.topic} ━━━
{section.summary}

  [{score}] {title}                              {date}
           {link}
  ...
```

Show all articles in each section, with links.

## Error Handling

- If curl fails or returns empty: "Data unavailable. Try again later or visit distilled.makinote.cn"
- If entity not found: "Entity '{name}' not found. Available: {list of entity displays}"
- If a section has no data: skip that section silently, do NOT show empty headers.

## Notes

- All data is pre-computed. Do NOT add your own analysis, scoring, or commentary.
- Do NOT modify, filter, or re-rank articles. Show them as-is from the JSON.
- Output is plain text for terminal readability. No markdown headers, no bold, no emoji.
- Keep output clean and scannable. Align columns where practical.
