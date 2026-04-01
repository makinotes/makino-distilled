# Distilled — Daily AI, triple-distilled.

Proactively manage AI information. Keep up with developments. Reduce information anxiety.

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

Pipeline runs twice daily on VPS via crontab (Beijing time):
- **09:25** — morning update
- **20:25** — evening update

If you run before 09:25, you get yesterday's evening data. The skill shows a freshness note when data is stale.

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

## FAQ

**Q: `/makino-distilled` no response or error?**
Make sure you cloned into the correct directory (`~/.claude-internal/skills/makino-distilled`). The folder name must be exactly `makino-distilled`. If you see curl errors, check if your network can reach `feed.makinote.cn`.

**Q: What are the 130+ sources?**
Chinese and English AI media, tech blogs, research labs, newsletters, and developer communities. The full source list is curated and maintained on the backend. You can browse entity coverage at [distilled.makinote.cn](https://distilled.makinote.cn).

**Q: Who picks the tracked entities? Can I add my own?**
The 24 entities (Claude, Agent, OpenAI, etc.) are curated by the pipeline maintainer. Custom entity tracking is not supported yet — this is a read-only client.

**Q: What does the score (e.g. [83]) mean?**
A relevance score from 0-100 assigned by the pipeline. Higher = more relevant to the entity topic. It is NOT a quality rating of the article itself.

**Q: What is the "30-day window"?**
The pipeline tracks articles from the past 30 days. Older articles roll off automatically. This keeps the digest focused on recent developments.

**Q: I see `[NOTE] Data is from yesterday` — is something broken?**
No. The pipeline updates at 09:25 and 20:25 Beijing time. If you run before the morning update, you get last night's data. This is normal.

**Q: Morning run vs evening run — what's different?**
Morning data includes overnight articles. Evening data adds the day's articles. Entity narratives are regenerated each run, so summaries may shift.

**Q: Output is very long. Can I see just one entity?**
Yes. Use `/makino-distilled claude` or `/makino-distilled agent` to deep dive into a single entity with full article lists.

**Q: The .md files keep piling up. Should I clean them?**
The skill saves one file per day (`distilled-YYYY-MM-DD.md`) in your working directory. Delete old ones whenever you want — they are just local snapshots.

**Q: How do I update the skill?**
Run `cd ~/.claude-internal/skills/makino-distilled && git pull`. The skill also auto-checks for updates on each run and shows a notice if a new version is available.

**Q: Will `git pull` overwrite my changes?**
If you edited SKILL.md locally, git pull may conflict. Recommendation: don't modify SKILL.md — open an issue on GitHub instead.

**Q: Is this the same as the website?**
Same data source, same JSON. The website has visual design and interactive features. The skill is a terminal-native reader optimized for Claude Code / OpenClaw workflows.

**Q: Will this project be maintained?**
Yes. The pipeline runs automatically. The skill tracks the upstream API schema. Breaking changes follow a deprecation policy (see `api-schema.md` in the pipeline repo).

**Q: Are article links guaranteed to work?**
Links are scraped from original sources. Some may expire or get paywalled over time. The pipeline does not archive article content.

## License

MIT

## Author

[makino](https://github.com/makinotes)
