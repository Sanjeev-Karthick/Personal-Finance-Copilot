# Personal Finance Copilot

A personal AI co-pilot for **long-term investing**, **swing trading**, and **penny/high-growth** plays across **India** and **US** markets.

**Interactive core:** Claude (or Cursor) + **Kite MCP** + **Groww MCP** + the Indian fundamental-analyser prompt. That is enough for ticker VIEWs and live books.

**Optional later:** an AWS pipeline (scheduler, DB, scoring engines, digest, `/analyse` + `/portfolio` pages) for work that must happen when you are not in a chat.

**Build status:** planning only. **Default is Claude + MCP, not AWS.** When a standalone platform is actually required: [docs/standalone-vs-mcp.md](./docs/standalone-vs-mcp.md). AWS checklist if you outgrow chat: [TODO.md](./TODO.md).

## Buckets

| Bucket | Idea | Cadence |
| --- | --- | --- |
| Long-term | Indian analyser: ticker + horizon → 8-tab VIEW | Weekly + on demand |
| Swing | RSI / MACD / volume / ATR | Daily |
| Penny / high-growth | Volume surge + LLM catalyst detection | Daily, last to ship |

Penny/high-growth waits until position caps, stop-loss enforcement, and catalyst-decay checks exist.

## Markets

NSE/BSE and US, one normalized schema tagged by market and currency, with native plus converted P&L.

## Two ways to run this

**Claude + MCP (enough for analysis and books).** Connect `https://mcp.kite.trade/mcp` and `https://mcp.groww.in/mcp`, use the Indian analyser spec as the prompt, ask for a combined portfolio VIEW. No platform required.

**Standalone (only for unattended work).** Scheduler, score history, push alerts, US/news ingest, deterministic RSI/scorecard, `/analyse` and `/portfolio` URLs. Do not build this until chat is a habit and those gaps hurt. Details: [docs/standalone-vs-mcp.md](./docs/standalone-vs-mcp.md).

Other docs: [TODO.md](./TODO.md) · [docs/sample-output.md](./docs/sample-output.md) · [docs/indian-stock-fundamental-analyser.md](./docs/indian-stock-fundamental-analyser.md) · [docs/portfolio-analysis.md](./docs/portfolio-analysis.md).

## What you see

In **Claude:** 8-tab analyser artefact + a portfolio VIEW from MCP.

On a **platform (later):** `/analyse` and a **separate** `/portfolio` page, digest, urgent alerts.

No buy/sell/target. Hosted Kite MCP does not place orders; neither should chat nor a future page.
