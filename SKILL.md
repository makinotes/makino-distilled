---
name: makino-distilled
invocation: user
description: "Distilled — Daily AI digest in your terminal. 130+ sources scored and structured into JSON. No API keys, no dependencies — just curl."
version: "3.1"
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
- Extract the `version:` line from remote, compare with local version `3.1`
- If remote version > local version, prepend this notice before the header:

```
[UPDATE] Distilled v{remote} available (you have v3.1). Run: cd ~/.claude-internal/skills/makino-distilled && git pull
```

- If versions match or curl fails: show nothing, skip silently.

### Step 2: Render header

```
DISTILLED · {MM-DD}
Daily AI, triple-distilled.

{meta.article_total} articles · {meta.entity_curated} entities · updated {generated_at}

{headline}
```

Where:
- `{generated_at}` = from `watchlist.json` field `generated_at`, format as "HH:MM UTC+8"
- `{meta.*}` = from `watchlist.json` -> `meta` object
- If `headline` is empty or unavailable, skip the headline line

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
