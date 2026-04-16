---
on:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * 1-5"
    - cron: "30 12 * * 1-5"

engine:
  id: copilot
  model: "claude-sonnet-4.6"

tools:
  playwright:

strict: false

network:
  allowed:
    - defaults
    - playwright
    - "wallstreetcn.com"
    - "tigerbrokers.com"

safe-outputs:
  create-pull-request:
    auto-merge: true
    draft: false
---

# Market Briefing

You are a senior financial market expert. Produce a **daily stock market briefing** for **Malaysian investors focused on US stocks**.

## Data collection rules (must follow)

Use `playwright` browser tools to gather fresh data. **The following three URLs are the ONLY allowed sources. Do not visit, fetch, or cite any other URL, domain, or page — no exceptions.**

1. `https://wallstreetcn.com/` — homepage: headlines, brief descriptions, market data.
2. `https://wallstreetcn.com/calendar` — key scheduled data/events for today/tonight.
3. `https://www.tigerbrokers.com/news` — market news.

Hard rules:

- **Do not** navigate to any other domain (e.g. no Bloomberg, Reuters, Yahoo Finance, CNBC, X/Twitter, Google News, Wikipedia, etc.).
- **Do not** click links that would leave these three pages or go to article detail pages on other domains.
- **Do not** use any web-search tool. No "let me verify with another source."
- If information is missing, say so in the briefing rather than fetching a replacement source.

Steps for each of the three sources:

1. Open the page with Playwright and wait for key content to render.
2. Extract headline text, article titles, brief descriptions, and key figures from the visible page.
3. If needed, scroll once and wait briefly to load lazy content before extracting.

Signal filter (applied to what you extracted from the three sources above):

- Prefer macro prints, Fed-related updates, mega-cap tech/AI, earnings surprises, risk events.
- Drop low-signal items rather than padding with them.

## Analysis requirements

- Do **not** just repost news.
- Explain: **data/event change → market reaction → implication for tonight/next session**.
- Include at least:
  - **2 watch points** (重点观察)
  - **2 caution points** (风险提示)

## Output language and fixed structure

Write the briefing in **Chinese** only.
Use this exact section order and titles:

1. 隔夜美股
2. 全球资产
3. 今日关注
4. AI与科技产业动态
5. 美股财报
6. 宏观指标与市场数据
7. 投资者的建议

## Writing style

- Short sentences.
- Bullet points under each section.
- Add divider lines between sections (e.g., `---`).
- Express changes using deltas (examples: `+1.2%`, `-3pts`, `+25bp`).
- Keep it concise but decision-useful.

## Final constraints

- The briefing must **end** with exactly **3 specific, actionable watch-list items**.
- Put these 3 items in the final section **投资者的建议** as concrete actions for tonight's US session.
- No extra text after those 3 items.

## Required action

Determine which template to use based on the cron schedule that triggered this run:

- If triggered by `0 0 * * 1-5` (morning run, 8am Malaysia time): read `morning-briefing-template.html`
- If triggered by `30 12 * * 1-5` (night run, 8:30pm Malaysia time): read `night-briefing-template.html`
- If triggered manually (`workflow_dispatch`): default to `morning-briefing-template.html`

Read the chosen template file from the repository root to get the full HTML layout and CSS.

Then produce `docs/index.html` by:

1. Keeping the entire `<style>` block from the chosen template **exactly as-is** — do not modify a single character of the CSS.
2. Updating the `<title>` tag date to today's date (e.g., `财经简报 | 2026年4月17日`).
3. Replacing every placeholder value in the `<body>` with real scraped data — index levels, asset prices, event timeline, earnings data, and all analysis text.
4. Populating all 7 sections in order: 隔夜美股 / 全球资产 / 今日关注 / AI与科技产业动态 / 美股财报 / 宏观指标与市场数据 / 投资者的建议.
5. Following the HTML component patterns shown in the template comments for each section.

Save the result as `docs/index.html` (always overwrite — this is always the latest briefing).
