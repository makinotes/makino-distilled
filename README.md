<div align="center">

# makino-distilled — Don't scroll. Distill.

> 130+ AI 信源按主题聚合评分，只看高质量信息，追踪长期趋势不追热点，降低信息焦虑。

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-5A67D8.svg)](https://docs.claude.com/en/docs/claude-code/skills)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-gray.svg)](https://github.com/anthropics/claude-code)

[Features](#features) · [Install](#install) · [Usage](#usage) · [Data](#data) · [FAQ](#faq)

</div>

---

## Install

```bash
cd ~/.claude/skills/
git clone https://github.com/makinotes/makino-distilled.git
```

Then type `/makino-distilled` in Claude Code.

## Usage

```
/makino-distilled
```

Full digest — every curated entity at a glance (per-section top 3 articles).

```
/makino-distilled claude
/makino-distilled openai
/makino-distilled agent
```

Single entity deep dive — full narrative + all articles.

Output auto-saves to `./distilled-{date}.md`.

## Example Output

Full output from a real run (752 lines, 24 entities, 969 articles):

> **[demo/distilled-2026-04-03.md](demo/distilled-2026-04-03.md)**

Preview (first 30 lines):

```
DISTILLED · 04-03
Don't scroll. Distill.
Proactively manage AI info · Keep up with developments · Reduce anxiety

969 articles · 24 entities · data from 04-03 16:57

━━━ WATCHING ━━━

◆ Claude  [Product]  33 articles
  最近最值得关注的是 Claude Code 源码泄露和从中暴露的 Harness Engineering 工程实践...

  ── Claude Code 源码泄露 (6) ──
  [77] 突发！Claude Code 源码泄露，扒出这些隐藏功能
       https://mp.weixin.qq.com/s/...  (04-01)
  [72] Claude Code 512,000 行源码泄露，有3点启发
       https://mp.weixin.qq.com/s/...  (03-31)

  ── Harness (6) ──
  [70] 一夜之间，全世界的 Agent 能力提高了一个档次
       https://mp.weixin.qq.com/s/...  (04-03)

  ...24 entities, ~750 lines total
```

## Configuration

There is nothing to configure. Every user sees the same curated ranking; scoring and entity tracking live in the backend. If a dimension matters to you, open an issue and say which one.

## Features

- Zero dependencies, zero API keys (just curl)
- Data refreshes once a day, around 06:45 Beijing time (22:45 UTC)
- Auto-detect data freshness (stale data warning)
- Auto-check for skill updates on each run
- Same data as [ai.makinote.cn](https://ai.makinote.cn), terminal-native

## What Problem Does This Solve

Keeping up with AI developments means scrolling through dozens of sources daily. Most of it is noise. This skill distills 130+ sources into structured, scored summaries — you read the terminal output in 2 minutes instead of scrolling feeds for an hour. Know what matters, skip what doesn't.

## Data

All data is fetched from public JSON endpoints at `ai.makinote.cn`.

The pipeline runs once a day on a VPS via crontab, at about 06:45 Beijing time (22:45 UTC).

If you run before the daily update, you get the previous day's data. The skill shows a freshness note when data is stale.

| Endpoint | Content |
|----------|---------|
| `lists/watchlist.json` | Entity narratives + articles (30-day window) |

## How It Works

```
130+ sources --> Pipeline (VPS) --> JSON API (CDN) --> This skill (curl + render)
```

The pipeline fetches, scores, summarizes, and structures articles once a day.
This skill is a read-only terminal client — it fetches the JSON and formats it.
No local processing, no LLM calls, no state.

## For Developers

The JSON endpoints are public. Build your own client:

```bash
# Entity narrative
curl -s https://ai.makinote.cn/lists/watchlist.json | \
  jq '.entities[] | select(.entity_id == "claude") | .narrative.summary'

# All curated entity IDs
curl -s https://ai.makinote.cn/lists/watchlist.json | jq '.curated_ids'
```

## Website

[ai.makinote.cn](https://ai.makinote.cn) — same data, visual interface.

## Update

```bash
cd ~/.claude/skills/makino-distilled && git pull
```

The skill also checks for updates automatically on each run. If a new version is available, you'll see a notice in the output.

## FAQ

**Q: `/makino-distilled` no response or error?**
Make sure you cloned into the correct directory (`~/.claude/skills/makino-distilled`). The folder name must be exactly `makino-distilled`. If you see curl errors, check if your network can reach `ai.makinote.cn`.

**Q: What are the 130+ sources?**
Chinese and English AI media, tech blogs, research labs, newsletters, and developer communities. The full source list is curated and maintained on the backend. You can browse entity coverage at [ai.makinote.cn](https://ai.makinote.cn).

**Q: Who picks the tracked entities? Can I add my own?**
The curated entities (Claude, Agent, OpenAI, etc.) are picked by the pipeline maintainer. Custom entity tracking is not supported yet — this is a read-only client.

**Q: What does the score (e.g. [83]) mean?**
A relevance score from 0-100 assigned by the pipeline. Higher = more relevant to the entity topic. It is NOT a quality rating of the article itself.

**Q: What is the "30-day window"?**
The pipeline tracks articles from the past 30 days. Older articles roll off automatically. This keeps the digest focused on recent developments.

**Q: I see `[NOTE] Data is from yesterday` — is something broken?**
No. The pipeline updates once a day at about 06:45 Beijing time. If you run before that, you get the previous day's data. This is normal.

**Q: Output is very long. Can I see just one entity?**
Yes. Use `/makino-distilled claude` or `/makino-distilled agent` to deep dive into a single entity with full article lists.

**Q: The .md files keep piling up. Should I clean them?**
The skill saves one file per day (`distilled-YYYY-MM-DD.md`) in your working directory. Delete old ones whenever you want — they are just local snapshots.

**Q: How do I update the skill?**
Run `cd ~/.claude/skills/makino-distilled && git pull`. The skill also auto-checks for updates on each run and shows a notice if a new version is available.

**Q: Will `git pull` overwrite my changes?**
If you edited SKILL.md locally, git pull may conflict. Recommendation: don't modify SKILL.md — open an issue on GitHub instead.

**Q: Is this the same as the website?**
Same data source, same JSON. The website has visual design and interactive features. The skill is a terminal-native reader optimized for agentic workflows.

**Q: Will this project be maintained?**
Yes. The pipeline runs automatically, and this skill is updated whenever the upstream JSON changes.

**Q: Are article links guaranteed to work?**
Links are scraped from original sources. Some may expire or get paywalled over time. The pipeline does not archive article content.

## License

Apache 2.0 — see [LICENSE](LICENSE) and [NOTICE](NOTICE)

## Community & Contact

Bugs, questions and ideas: open an issue. 也可以在公众号「马奇诺」后台留言。

<img src="assets/wechat-qr-makino.jpg" alt="公众号 马奇诺" width="200">

## Author

[makino](https://github.com/makinotes)
