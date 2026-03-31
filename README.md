# Distilled — Daily AI, triple-distilled.

130+ AI sources scanned daily. Scored, structured, and served as JSON.
Read it in your terminal — same data as [distilled.makinote.cn](https://distilled.makinote.cn).

No API keys. No dependencies. Just `curl`.

## What You Get

Four views matching the website:

| Command | View | What it shows |
|---------|------|--------------|
| `/makino-distilled` | Watching | Entity narratives, topic pills, and top articles |
| `/makino-distilled learn` | Learn | Topics worth studying — why and where to start |
| `/makino-distilled read` | Read | Articles worth reading with core insights |
| `/makino-distilled do` | Do | Actionable items with first steps |

Deep dive into a specific entity:

```
/makino-distilled claude
/makino-distilled openai
/makino-distilled agent
```

## Quick Start

### Claude Code

```bash
# Copy the skill into your project
cp -r makino-distilled /path/to/your/project/.claude/skills/

# Or into global skills
cp -r makino-distilled ~/.claude/skills/
```

Then type `/makino-distilled` in Claude Code.

### OpenClaw

Copy the `makino-distilled` folder into your agent's skills directory, then invoke `/makino-distilled`.

## Data

All data is fetched from public JSON endpoints at `feed.makinote.cn`.
Updated twice daily (09:25 + 20:25 Beijing time).

| Endpoint | Content |
|----------|---------|
| `lists/watchlist.json` | Entity narratives + articles (30-day window) |
| `lists/boards.json` | Learn / Read / Do boards (7-day window) |
| `lists/pulse.json` | Temperature, trends, headlines |

## How It Works

```
130+ sources --> Pipeline (VPS) --> JSON API (CDN) --> This skill (curl + render)
```

The pipeline fetches, scores, summarizes, and structures articles twice daily.
This skill is a read-only terminal client — it fetches the JSON and formats it.
No local processing, no LLM calls, no state.

## For Developers

The JSON endpoints are public. Build your own client:

```bash
# Entity narrative
curl -s https://feed.makinote.cn/lists/watchlist.json | \
  jq '.entities[] | select(.entity_id == "claude") | .narrative.summary'

# Trending keywords
curl -s https://feed.makinote.cn/indexes/trending.json | jq '.trending[:10]'
```

## License

MIT

## Author

[makino](https://github.com/makinotes)
