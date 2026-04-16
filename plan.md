# Daily Stock Market Briefing — gh-aw Implementation Plan

## Overview

Build a daily morning stock market briefing using **GitHub Agentic Workflows (gh-aw)** that:
- Runs on a daily schedule via GitHub Actions
- Uses **agent-browser** skill to scrape financial websites for market data
- Uses **Claude Sonnet 4.6 via Copilot** (`engine: copilot, model: claude-sonnet-4.6`) to synthesize the briefing
- Follows instructions defined in `daily-summary.md` (the prompt file — already written)
- Outputs the briefing via safe-outputs (e.g., GitHub Issue or committed report file)

### ✅ Feasibility: Confirmed

---

## Architecture

```
[Daily Schedule Trigger — weekdays before market open]
        │
        ▼
[GitHub Actions Runner]
  ├── copilot-setup-steps.yml installs agent-browser + Chrome
        │
        ▼
[gh-aw workflow — engine: copilot, model: claude-sonnet-4.6]
  │  reads daily-summary.md as the instruction/prompt
  │
  ├──► [agent-browser CLI] → Scrape financial websites:
  │       • wallstreetcn.com homepage — overnight US stocks, headlines, flash news
  │       • wallstreetcn.com calendar — key data & events for today/tonight
  │       • Tiger Broker market page — 24/7 news feed
  │       • Read 3-4 articles max (as instructed)
  │
  ▼
[Claude Sonnet 4.6 (Copilot) synthesizes briefing]
  │  Following daily-summary.md format:
  │  隔夜美股 / 全球资产 / 今日关注 / AI与科技产业动态
  │  美股财报 / 宏观指标与市场数据 / 投资者的建议
  │
  ▼
[safe-output: post as GitHub Issue or commit report file]
```

### Key Design: `daily-summary.md` is the Prompt

`daily-summary.md` already exists and contains detailed Chinese-language instructions:
- **Role**: Financial expert creating morning briefings for Malaysian US stock investors
- **Data sources**: wallstreetcn (homepage + calendar), Tiger Broker (24/7 news)
- **Scope**: 3-4 articles max
- **Analysis**: Explain data changes → market reaction (not just reposting news)
- **Sections**: Fixed format with 7 sections
- **Style**: Short sentences, bullet points, dividers, numerical changes, 3 actionable items at end

The gh-aw workflow will reference this file as its instructions.

---

## Component Details

### 1. Engine: Copilot with Claude Sonnet 4.6 ✅

Using the **Copilot engine** with Claude Sonnet 4.6 model — no separate Anthropic API key needed:

```yaml
engine:
  id: copilot
  model: "claude-sonnet-4.6"
```

Benefits:
- Uses existing GitHub Copilot subscription
- No extra API key secrets to manage
- Best-supported engine in gh-aw

### 2. agent-browser in GitHub Actions ✅

agent-browser is a Rust-based browser automation CLI that controls Chrome via CDP.
It needs Chrome installed in the runner — handled by `copilot-setup-steps.yml`:

```yaml
# .github/copilot-setup-steps.yml
steps:
  - name: Install agent-browser and Chrome
    run: |
      npm install -g agent-browser
      agent-browser install
```

The agent-browser skill gives the AI agent these commands:
- `agent-browser open <url>` — navigate to pages
- `agent-browser snapshot -i --json` — get accessibility tree with element refs
- `agent-browser get text <selector>` — extract text content
- `agent-browser click @e1` / `agent-browser fill @e2 "text"` — interact with pages
- `agent-browser screenshot` — capture visual state

### 3. Scheduling ✅

```yaml
schedule: daily on weekdays   # Fuzzy schedule, Monday–Friday
# OR precise:
schedule:
  - cron: "0 0 * * 1-5"      # 8 AM MYT (UTC 00:00) on weekdays
```

### 4. Output via Safe Outputs ✅

gh-aw uses safe-outputs to write back to the repo:
- Commit updated `daily-summary.md` with the day's briefing
- Optionally create a GitHub Issue for each briefing

---

## Implementation Todos

### Todo 1: Initialize Repository
- `git init` the market-briefing project
- Create GitHub repo and push
- Run `gh aw init` to set up gh-aw structure

### Todo 2: Create copilot-setup-steps.yml
- Install Node.js, agent-browser, and Chrome in the Actions runner
- Path: `.github/copilot-setup-steps.yml`

### Todo 3: Write the gh-aw Workflow
- Path: `.github/aw/morning-briefing.md`
- Schedule: daily on weekdays (before market open, ~8 AM MYT / UTC 00:00)
- Engine: copilot with claude-sonnet-4.6
- Reference `daily-summary.md` as the agent's instructions/prompt
- Agent uses agent-browser to scrape wallstreetcn + Tiger Broker
- Output via safe-outputs (GitHub Issue or committed report)

### Todo 4: Test and Iterate
- Run `gh aw run` locally to test the workflow
- Verify agent-browser can scrape wallstreetcn and Tiger Broker from Actions runner
- Verify Claude Sonnet 4.6 model works via Copilot engine
- Tune output quality against the daily-summary.md requirements

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| gh-aw is early-stage | Bugs, breaking changes | Pin versions; have manual fallback |
| Financial sites block scraping | Missing data | Use multiple sources; graceful degradation |
| Rate limits on financial sites | Incomplete data | Stagger requests; cache where possible |
| Network firewall in Actions | Can't reach sites | Verify financial sites are accessible from runners |
| Claude Sonnet 4.6 not available via Copilot | Workflow fails | Fall back to default Copilot model |

---

## Data Sources (from daily-summary.md)

| Source | Data | URL |
|--------|------|-----|
| wallstreetcn 首页 | Overnight US stocks, headlines, flash news | wallstreetcn.com |
| wallstreetcn 日历 | Key data & events for today/tonight | wallstreetcn.com (calendar) |
| Tiger Broker 市场 | 24/7 news feed | tigertrade.com or similar |
