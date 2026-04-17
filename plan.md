# Market Briefing — Implementation Plan

## Overview

Build a daily stock market briefing using **GitHub Agentic Workflows (gh-aw)** that:
- Runs on a daily schedule via GitHub Actions
- Uses **Playwright** browser tools to scrape financial websites for market data
- Uses **Claude Sonnet 4.6 via Copilot** (`engine: copilot, model: claude-sonnet-4.6`) to synthesize the briefing
- Follows instructions defined in `daily-summary.md` (the prompt file — already written)
- Outputs the briefing as an HTML file using `template.html` layout
- Hosts the briefing on GitHub Pages

---

## Phase 2: HTML Output + GitHub Pages Hosting

### Problem
Currently the `morning-briefing.md` gh-aw workflow posts the briefing as a GitHub Issue. We want to:
1. Output an HTML file instead, using `template.html` as the fixed layout
2. Commit the HTML directly to the repo
3. Host it on GitHub Pages
4. Rename all files from "morning-briefing" to "market-briefing"

### Approach

#### File Renaming
- `.github/workflows/morning-briefing.md` → `.github/workflows/market-briefing.md`
- `.github/workflows/morning-briefing.lock.yml` → delete (regenerated as `market-briefing.lock.yml`)
- Workflow name: "Market Briefing"

#### Workflow Changes (`market-briefing.md`)
- Change `safe-outputs` from `create-issue` to `create-pull-request` with `auto_merge: true`
- Update `permissions` to `contents: write` and `pull-requests: write`
- Instruct agent to read `template.html`, keep layout/CSS identical, populate with scraped data
- Output file: `docs/index.html` — always the latest briefing, overwritten each run
- The PR auto-merges so `docs/index.html` updates automatically on main

> **Note:** gh-aw only supports `create-pull-request` for writing files (no direct commit). With `auto_merge: true`, the PR merges immediately, so `docs/index.html` updates automatically without manual intervention.

#### GitHub Pages Setup
- Use `docs/` folder on `main` branch for GitHub Pages
- Create initial `docs/index.html` placeholder
- Enable Pages via `gh` CLI (source: main branch, /docs folder)

### Todos

1. **rename-workflow-files** — Rename `morning-briefing.md` → `market-briefing.md`, delete old lock file.
2. **update-workflow** — Rewrite `market-briefing.md`: updated name, permissions, safe-outputs, and instructions to generate HTML into `docs/`.
3. **setup-github-pages** — Create `docs/index.html` placeholder. Enable GitHub Pages (main branch, /docs folder).
4. **recompile-workflow** — Run `gh aw compile` to generate `market-briefing.lock.yml`.

### Notes
- Template has 7 sections: 隔夜美股, 全球资产, 今日关注, AI与科技产业动态, 美股财报, 宏观指标与市场数据, 投资者的建议
- Template CSS (~710 lines) must not be modified
- HTML comments in template document each section's pattern for the agent

---

## Architecture

```
[Daily Schedule Trigger — weekdays before market open]
        │
        ▼
[GitHub Actions Runner]
  ├── copilot-setup-steps.yml installs Playwright
        │
        ▼
[gh-aw workflow — engine: copilot, model: claude-sonnet-4.6]
  │  reads daily-summary.md + template.html
  │
  ├──► [Playwright] → Scrape financial websites:
  │       • wallstreetcn.com homepage — overnight US stocks, headlines, flash news
  │       • wallstreetcn.com calendar — key data & events for today/tonight
  │       • Tiger Broker market page — 24/7 news feed
  │       • Read 3-4 articles max (as instructed)
  │
  ▼
[Claude Sonnet 4.6 (Copilot) synthesizes briefing]
  │  Following template.html structure:
  │  隔夜美股 / 全球资产 / 今日关注 / AI与科技产业动态
  │  美股财报 / 宏观指标与市场数据 / 投资者的建议
  │
  ▼
[create-pull-request (auto_merge) → docs/index.html → GitHub Pages]
```

## Data Sources (from daily-summary.md)

| Source | Data | URL |
|--------|------|-----|
| wallstreetcn 首页 | Overnight US stocks, headlines, flash news | wallstreetcn.com |
| wallstreetcn 日历 | Key data & events for today/tonight | wallstreetcn.com (calendar) |
| Tiger Broker 市场 | 24/7 news feed | tigertrade.com or similar |

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| gh-aw is early-stage | Bugs, breaking changes | Pin versions; have manual fallback |
| Financial sites block scraping | Missing data | Use multiple sources; graceful degradation |
| Rate limits on financial sites | Incomplete data | Stagger requests; cache where possible |
| Network firewall in Actions | Can't reach sites | Verify financial sites are accessible from runners |
| Claude Sonnet 4.6 not available via Copilot | Workflow fails | Fall back to default Copilot model |
