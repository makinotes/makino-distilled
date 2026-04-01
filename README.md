# Distilled — Daily AI, triple-distilled.

130+ AI sources scanned daily. Scored, structured, and served as JSON.
Read it in your terminal — same data as [distilled.makinote.cn](https://distilled.makinote.cn).

No API keys. No dependencies. Just `curl`.

## What You Get

One command, all tracked entities at a glance:

```
/makino-distilled
```

Deep dive into a specific entity:

```
/makino-distilled claude
/makino-distilled openai
/makino-distilled agent
```

Output auto-saves to `./distilled-{date}.md`.

## Install

### Claude Code

```bash
cd ~/.claude-internal/skills/
git clone https://github.com/makinotes/makino-distilled.git
```

Then type `/makino-distilled` in Claude Code.

### OpenClaw

Copy the `makino-distilled` folder into your agent's skills directory, then invoke `/makino-distilled`.

## Update

```bash
cd ~/.claude-internal/skills/makino-distilled && git pull
```

The skill also checks for updates automatically on each run. If a new version is available, you'll see a notice in the output.

## Data

All data is fetched from public JSON endpoints at `feed.makinote.cn`.
Updated twice daily (09:25 + 20:25 Beijing time).

| Endpoint | Content |
|----------|---------|
| `lists/watchlist.json` | Entity narratives + articles (30-day window) |

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

# All curated entity IDs
curl -s https://feed.makinote.cn/lists/watchlist.json | jq '.curated_ids'
```

## License

MIT

## Author

[makino](https://github.com/makinotes)
