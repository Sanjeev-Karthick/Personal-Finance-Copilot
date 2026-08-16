# Personal Finance Copilot

A personal AI co-pilot for **long-term investing**, **swing trading**, and **penny/high-growth** plays across **India** and **US** markets.

The stack is an AWS pipeline (Lambda, CockroachDB or Aurora, SQS/SNS) feeding three scoring engines (fundamentals, technicals, catalyst detection), a positions ledger with P&L per strategy, and a daily digest with urgent alerts.

**Build status:** planning only. The sequenced work is in [TODO.md](./TODO.md). Implementation has not started.

## Buckets

| Bucket | Idea | Cadence |
| --- | --- | --- |
| Long-term | Fundamental scorecard | Weekly |
| Swing | RSI / MACD / volume / ATR | Daily |
| Penny / high-growth | Volume surge + LLM catalyst detection | Daily, last to ship |

Penny/high-growth waits until position caps, stop-loss enforcement, and catalyst-decay checks exist.

## Markets

NSE/BSE and US, one normalized schema tagged by market and currency, with native plus converted P&L.
