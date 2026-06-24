---
name: followin-stock-automation
description: Create or update Followin MCP powered stock watchlist automations for daily premarket monitoring, unusual-move/news alerts, portfolio-aware trade plans, and recurring stock reports. Use when the user asks Codex to track a watchlist, follow self-selected stocks, monitor holdings, produce premarket analysis, sync major news/research/social activity, or generate operation/trading suggestions using Followin MCP.
---

# Followin Stock Automation

## Purpose

Set up a recurring market report that uses Followin MCP as the primary data source for a user's watchlist and holdings. The user only needs to provide the watchlist and current position state; if a schedule is not specified, choose a practical US-market premarket default.

## Prerequisites

Verify Followin MCP before creating automation:

```bash
codex mcp get followin
```

Expected configuration:

```toml
[mcp_servers.followin]
url = "https://mcp.followin.io/v2/mcp"
env_http_headers = { "x-api-key" = "FOLLOWIN_MCP_TOKEN" }
```

Alternative simple configuration, if the user accepts storing the key in config:

```toml
[mcp_servers.followin]
url = "https://mcp.followin.io/v2/mcp"
http_headers = { "x-api-key" = "YOUR_API_KEY_HERE" }
```

If Followin is not installed or authenticated, tell the user exactly what is missing. Do not create a report that pretends Followin data was available.

## Inputs

Collect or infer:

- Watchlist tickers, e.g. `DRAM, SNDK, MU, NOK, MRVL`.
- Holdings state for each ticker: empty/long/short/options, quantity, average cost, and any existing stop or target. If the user only says "empty" or "空仓", treat every ticker as no position.
- Schedule: default to US market weekdays about one hour before regular-session open. For Asia/Shanghai users during US daylight saving time, use weekdays 20:30. Mention DST if schedule precision matters.
- Report destination: default to the current thread unless the user asks for a detached workspace automation.

Avoid asking for details that are not needed to create the first automation. Missing cost basis or quantity is fine when the user is empty.

## Workflow

1. Check for existing automations matching the same watchlist or task name. Prefer updating an existing automation over creating a duplicate.
2. Verify Followin MCP config. If native Followin tools are available, use them for a quick smoke test when useful; otherwise rely on the automation prompt to use Followin at run time.
3. Create or update a Codex automation with a self-contained prompt. Use the app automation tool when available; do not write raw automation directives by hand.
4. Include the watchlist, holdings state, schedule, expected output sections, and failure behavior in the automation prompt.
5. Confirm the created/updated automation id, schedule, tickers, and holdings assumption.

## Automation Prompt Template

Adapt this template:

```text
Use Followin MCP as the primary data source to monitor this US-stock watchlist: {WATCHLIST}.

Current holdings: {POSITIONS}. If a ticker is marked empty/no position, treat it as watchlist-only and give entry plans rather than hold/trim advice.

Each run should produce a concise Chinese premarket report:
1. Market context: index futures, sector/theme backdrop, and risk-on/risk-off tone when available.
2. Per ticker: premarket/last price, daily move, volume or unusual activity, key technical levels, near-term catalysts, major news, research/analyst changes, company filings, and Followin social/Twitter heat.
3. Position-aware plan: for empty positions, give wait/try-entry/breakout/reversal plans with trigger price, invalidation/stop, suggested starter size, and priority. For existing long/short/options positions, give hold/add/trim/stop/hedge guidance tied to price levels and news.
4. Portfolio view: rank the best 1-2 opportunities for today, list what to avoid, and flag correlated risk across the watchlist.
5. Source discipline: clearly label which facts came from Followin MCP. If Followin MCP is unavailable, say so and do not fabricate Followin data.

End with a brief note that this is not personalized investment advice.
```

## Followin Tool Usage

When running an immediate report, use Followin tools in this order:

- `metrics`: current price, movers, recent trend, valuation, fundamentals, analyst targets.
- `news`: last 24h to 7d market news, media, research, and commentary.
- `twitter`: latest and top posts for social heat and narrative shifts.
- `signal`: insider/political/institutional data for tradfi when relevant.

For research-heavy requests, query `news` with `sources=["research"]`. For intraday heat, combine `twitter` `Latest` with `news` recent media.

## Recommendation Rules

Make recommendations executable but conditional:

- Use explicit triggers, e.g. "only consider starter entry above X on volume" or "wait for pullback to Y-Z".
- Always include invalidation/stop logic.
- Scale position sizes conservatively for empty portfolios unless the user specifies risk budget.
- Separate "trade setup" from "investment thesis".
- Do not promise returns, guarantee targets, or imply certainty from social heat.
