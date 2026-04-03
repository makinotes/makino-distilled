---
name: makino-distilled
invocation: user
description: "Distilled — Daily AI digest in your terminal. 130+ sources scored and structured into JSON. No API keys, no dependencies — just curl."
version: "4.0"
last_updated: "2026-04-03"
---

# Distilled — Don't scroll. Distill.

You are a terminal-based reader for the Distilled AI daily feed.
You do NOT generate, score, or process any content — you fetch pre-rendered output and display it.
The VPS pipeline pre-renders the terminal digest. Your job is to fetch and present.

Core value: help users proactively manage AI information, keep up with developments, and reduce information anxiety.

Data updates twice daily (09:25 + 20:25 Beijing time) via VPS crontab.
All times in this skill are Beijing time (UTC+8).
Base URL: `https://feed.makinote.cn`

**Data freshness**: Pipeline runs at 09:25 and 20:25 Beijing time.
If you run before 09:25, you get yesterday's evening data.
If you run between 09:25-20:25, you get today's morning data.
If you run after 20:25, you get today's evening data.

When fetching data, always append `?c=skill` to the URL for analytics.

## Commands

| Command | What it shows |
|---------|--------------|
| `/makino-distilled` | Full digest — pre-rendered, all curated entities |
| `/makino-distilled <entity>` | Single entity deep dive (e.g. `/makino-distilled claude`) |

## Execution

### Full digest (`/makino-distilled`)

**Step 1: Fetch pre-rendered digest**

```bash
curl -s "https://feed.makinote.cn/distilled-latest.md?c=skill"
```

This file is pre-rendered by the VPS pipeline. It contains the complete terminal digest with header, all curated entities, sections, articles, and footer. No parsing or rendering needed.

**Step 2: Check for skill updates**

```bash
curl -s https://raw.githubusercontent.com/makinotes/makino-distilled/main/SKILL.md | head -6
```

Extract the `version:` line from remote, compare with local version `4.0`.
If remote version > local version, prepend this notice before the output:

```
[UPDATE] Distilled v{remote} available (you have v4.0). Run: cd ~/.claude/skills/makino-distilled && git pull
```

If versions match or curl fails: show nothing, skip silently.

**Step 3: Display**

Print the fetched content directly to terminal. Do NOT modify, re-format, or add commentary.

**Step 4: Auto-save to local file**

Save the same content to `./distilled-{YYYY-MM-DD}.md` (use `date "+%Y-%m-%d"`, do NOT hardcode).

After saving, print:

```
Saved to ./distilled-{YYYY-MM-DD}.md
```

### Entity Detail (`/makino-distilled <entity>`)

Entity detail is NOT pre-rendered — it requires filtering a single entity from the full dataset.

**Step 1: Fetch watchlist.json**

```bash
curl -s "https://feed.makinote.cn/lists/watchlist.json?c=skill"
```

**Step 2: Find and render entity**

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
`article_count` = sum of articles across all `narrative.sections`.

**Step 3: Auto-save**

Save to `./distilled-{YYYY-MM-DD}-{entity_id}.md`.

## Error Handling

- If curl fails or returns empty: "Data unavailable. Try again later or visit distilled.makinote.cn"
- If entity not found: "Entity '{name}' not found. Available: {list of entity displays}"
- If distilled-latest.md is empty or missing: fall back to fetching watchlist.json and rendering manually (legacy mode)

## Notes

- Full digest is pre-rendered on the server. Do NOT parse JSON for full digest — just fetch the .md file.
- Entity detail still requires JSON parsing (only for single-entity queries).
- All data is pre-computed. Do NOT add your own analysis, scoring, or commentary.
- Do NOT modify, filter, or re-rank articles. Show them as-is.
- Output is plain text for terminal readability. No markdown headers, no bold, no emoji.

## Architecture

```
VPS pipeline (09:25 + 20:25)
  → watchlist.json (870KB, entity narratives + articles)
  → distilled-latest.md (40KB, pre-rendered terminal digest)  ← NEW
  → Vercel CDN (5-min cache)

/makino-distilled (full)     → curl distilled-latest.md → display    (~2s, ~500 tokens)
/makino-distilled <entity>   → curl watchlist.json → filter → render  (~30s, ~20K tokens)
```

## Consumed Endpoints

| Endpoint | Used by | Size |
|----------|---------|------|
| `distilled-latest.md?c=skill` | Full digest | ~40KB |
| `lists/watchlist.json?c=skill` | Entity detail only | ~870KB |

### watchlist.json fields (entity detail only)

```
watchlist.json
├── generated_at              (string, ISO 8601)
├── curated_ids               (string[])
├── meta
│   ├── article_total         (int)
│   └── entity_curated        (int)
└── entities[]
    ├── entity_id             (string)
    ├── display               (string)
    ├── type                  (string)
    ├── last_updated          (string)
    └── narrative
        ├── summary           (string)
        └── sections[]
            ├── topic         (string)
            └── articles[]
                ├── title     (string)
                ├── score     (int)
                ├── date      (string, YYYY-MM-DD)
                └── link      (string, URL)
```

Upstream: VPS pipeline at `/opt/feed-pipeline/` → pre-rendered + JSON published to `feed.makinote.cn` via Vercel CDN.

## Gotchas

| 日期 | 问题 | 根因 | 修复 |
|------|------|------|------|
| 2026-04-02 | PM pipeline 清空 pinned_sources，信源 tab 数据全丢 | refactor 把 `outputs.py` 拆成 `outputs/sources_tab.py`（多了一层目录），但 `config_dir` 的 `dirname` 层数没跟着加 1，导致读不到 `config/watchlist.json` | `dirname` 改为 3 层。**教训：拆模块移文件时必须 grep 所有 `__file__` 相对路径计算，逐个验证** |
| 2026-04-02 | 改前端 CSS/JS 后 pipeline 跑了一次，数据被覆盖但没感知 | `run.sh` 用 `git add api/ indexes/ lists/` 提交数据文件，如果 pipeline 输出了有 bug 的 JSON 会静默覆盖好数据 | 恢复数据后修复 pipeline。**教训：改前端前先检查 pipeline 是否即将跑，改完后验证数据完整性（pinned_sources count > 0）** |
| 2026-04-03 | boards.json 和 watchlist.json 没更新，网站日期停在昨天 | 同一次 refactor（f7dcb94）还断了 `data_loader.py` 的 `_date_range` 引用和 `watchlist_builder.py` 的 `import json`。publish.py 标记为 non-fatal 所以 pipeline 成功但数据缺失 | 补全 import。**教训：refactor 拆模块后必须跑完整 pipeline 验证，不能只看 exit code；non-fatal error 也要检查输出文件是否完整** |
| 2026-04-03 | 用户跑 skill 耗时 7.5 分钟 + 16 万 token | 用户 LLM 解析 870KB JSON 并渲染，每次都重复做相同工作 | VPS 预渲染 distilled-latest.md，skill 只做 curl + display。**教训：渲染逻辑放生产端不放消费端** |
