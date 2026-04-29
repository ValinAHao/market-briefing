# Market Briefing 📈

A fully automated daily stock market briefing for **Malaysian investors focused on US stocks**, powered by [GitHub Agentic Workflows (gh-aw)](https://github.github.com/gh-aw/).

Twice a weekday, an AI agent scrapes live financial data, synthesizes the key moves, and publishes a polished HTML briefing to **GitHub Pages** — no manual effort required.

## What It Does

The workflow runs on two cron schedules (UTC):

- **Morning recap** — `00:00 UTC` Tue–Sat (08:00 MYT), recapping the US session that just closed and looking ahead to tonight's open.
- **Night preview** — `12:30 UTC` Mon–Fri (20:30 MYT), a tactical pre-open preview ~1 hour before US markets open.

An AI agent (Claude Sonnet 4.6 via GitHub Copilot) performs the entire pipeline end-to-end:

1. **Scrape** — Uses Playwright browser tools to gather real-time data from financial sources (wallstreetcn, Tiger Brokers)
2. **Analyze** — Synthesizes market data into a structured briefing covering 7 sections, written in Chinese
3. **Publish** — Outputs `docs/index.html` via an auto-merging pull request, instantly live on GitHub Pages

### Briefing Sections

| # | Section | Content |
|---|---------|---------|
| 1 | 隔夜美股 | Overnight US stock market recap |
| 2 | 全球资产 | Global asset overview (bonds, commodities, FX, crypto) |
| 3 | 今日关注 | Key events and data releases for today |
| 4 | AI与科技产业动态 | AI & tech industry developments |
| 5 | 美股财报 | US earnings highlights |
| 6 | 宏观指标与市场数据 | Macro indicators and market data |
| 7 | 投资者的建议 | 3 actionable watch-list items (morning run targets tonight's open; night run targets the open in ~1 hour) |

## How It Works

```
┌─────────────────────────────────┐
│  Cron Triggers (UTC)            │
│  • 00:00 Tue–Sat → morning      │
│  • 12:30 Mon–Fri → night        │
└──────────────┬──────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│         GitHub Actions Runner                │
│  copilot-setup-steps.yml installs Playwright │
└──────────────┬───────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│  gh-aw Workflow (Claude Sonnet 4.6)          │
│  Reads: morning- or night-briefing-          │
│         template.html (per run)              │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │  Playwright Browser Scraping           │  │
│  │  • wallstreetcn.com      → Headlines,  │  │
│  │                            market data  │  │
│  │  • wallstreetcn.com/cal  → Events &    │  │
│  │                            data today   │  │
│  │  • tigerbrokers.com/news → 24/7 news   │  │
│  │  (3-4 articles max)                    │  │
│  └────────────────┬───────────────────────┘  │
│                   │                          │
│                   ▼                          │
│  ┌────────────────────────────────────────┐  │
│  │  AI Analysis & HTML Generation         │  │
│  │  • Explains data → market reaction     │  │
│  │  • Populates chosen template with data │  │
│  │  • Outputs docs/index.html             │  │
│  └────────────────┬───────────────────────┘  │
└───────────────────┼──────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────┐
│  Auto-Merge Pull Request                     │
│  docs/index.html → main branch               │
│                                              │
│            ┌──────────┐                      │
│            │  GitHub   │                     │
│            │  Pages    │ ← Latest briefing   │
│            └──────────┘   always live        │
└──────────────────────────────────────────────┘
```

## Repository Structure

```
market-briefing/
├── daily-summary.md                  # Original prompt notes (reference)
├── morning-briefing-template.html    # HTML/CSS layout for the morning recap
├── night-briefing-template.html      # HTML/CSS layout for the night preview
├── docs/
│   └── index.html                    # Latest briefing (auto-updated, served by Pages)
└── .github/
    ├── workflows/
    │   ├── market-briefing.md          # gh-aw workflow definition
    │   ├── market-briefing.lock.yml    # Compiled GitHub Actions workflow
    │   ├── auto-merge-briefing.yml     # Auto-merges briefing PRs after success
    │   └── copilot-setup-steps.yml     # Runner setup (Playwright install)
    ├── aw/
    │   └── actions-lock.json         # Pinned action SHAs for reproducibility
    └── agents/
        └── agentic-workflows.agent.md
```

## Usage

The workflow runs automatically on weekdays. To trigger it manually:

```bash
gh workflow run market-briefing.lock.yml
```

To recompile after editing `market-briefing.md`:

```bash
gh aw compile
```
