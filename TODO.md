# Personal Finance Copilot — build TODO

A personal AI co-pilot for **long-term investing**, **swing trading**, and **penny/high-growth** plays across **India (NSE/BSE)** and **US** markets.

This document is the sequenced build plan. **Do not start implementation from this file until a later task explicitly kicks off a phase.** Checkboxes are the source of truth for what is in vs. out of scope.

**Sequence rule:** ship long-term + swing end-to-end first (shared indicators, lower operational risk). Add the penny/high-growth bucket last, after position caps, stop-loss enforcement, and catalyst-decay checks are solid.

---

## Phase 0 — Repo and project scaffolding

Required so the GitHub description matches a real, reproducible codebase.

- [ ] README: purpose, three strategy buckets, India + US markets, high-level architecture (ingestion → SQS → DB → scoring → **stock analysis** → digest/alerts → dashboard)
- [ ] Sample output doc (`docs/sample-output.md`) kept in sync with the product shape
- [ ] Indian long-term analyser spec (`docs/indian-stock-fundamental-analyser.md`) + HTML widget template
- [ ] Portfolio analysis page spec (`docs/portfolio-analysis.md`) — separate route, Kite + Groww MCP read-only
- [ ] `.gitignore` for Terraform/CDK, Python, Node, env files, and local secrets
- [ ] Choose IaC (Terraform **or** CDK) and app language(s); document the choice
- [ ] Repo layout: `infra/`, `services/` (ingest, score, portfolio, digest, api), `dashboard/`, `shared/` (schema, types)
- [ ] Local-dev notes: how to synth/plan infra without applying, how to run Lambdas locally
- [ ] Cost guardrails: AWS budget + CloudWatch billing alarm (personal use, cheap to tear down)
- [ ] CI: lint + unit tests on PR; never store API keys in git

---

## Phase 1 — Set up AWS foundation

Create a dedicated AWS account or use a separate IAM boundary from work. Set up Secrets Manager for API keys (Kite Connect, Alpha Vantage/Polygon, NSEpy creds), an S3 bucket for raw data dumps and logs, and CloudWatch for monitoring. Use Terraform or CDK from day one so the whole stack is reproducible and cheap to tear down/rebuild.

- [ ] Dedicated AWS account **or** hard IAM boundary (separate from work)
- [ ] IaC bootstrap (state backend, lock, environments: `dev` / optional `prod`)
- [ ] Secrets Manager: Kite Connect, Groww (if not solely OAuth via hosted MCP), Alpha Vantage and/or Polygon, NSEpy/NSE-related creds, Anthropic or Bedrock, Telegram bot token (if used), SES/SMTP as needed
- [ ] S3: raw OHLCV/fundamentals dumps, job logs, digest archives
- [ ] CloudWatch: log groups, metric filters, alarms for failed schedules and Lambda errors
- [ ] Least-privilege IAM roles per Lambda/Fargate task (no shared admin role)

---

## Phase 2 — Build the data ingestion pipeline

Use EventBridge Scheduler to trigger a Lambda (or ECS Fargate scheduled task if a job needs to run longer than Lambda's 15-min limit — likely for bulk historical backfills) once daily after market close for both NSE/BSE and US close. Pull OHLCV + fundamentals, normalize into one schema tagged by market/currency, and write to your database. Use SQS as a buffer between fetch and write so a slow API doesn't block the whole run.

- [ ] EventBridge schedules: after NSE/BSE close and after US close (timezone + DST aware)
- [ ] Market calendar / holiday skip so jobs do not treat holidays as missing data
- [ ] Fetch OHLCV + fundamentals (India + US providers)
- [ ] India long-term pack: shareholding (promoter/FII/DII/pledge), FCF, quarterly EPS, peer comps, with **source URL per metric**
- [ ] Normalize to one schema tagged by `market` and `currency`
- [ ] SQS buffer between fetch and write
- [ ] Idempotent daily writes (re-runs do not duplicate bars)
- [ ] Fargate (or equivalent) path for historical backfill beyond Lambda’s 15-minute limit
- [ ] Raw payload dump to S3 before normalize (replay / debug)
- [ ] FX rates (INR/USD) so later P&L can be shown native + converted

---

## Phase 3 — Stand up the database

Run CockroachDB Serverless (free tier is enough for personal use) or Aurora Serverless v2 Postgres if you'd rather stay fully in AWS. Core tables: tickers (market, sector, currency), price_history, fundamentals, scores (per bucket: long-term/swing/penny-growth), positions (your actual holdings tagged by bucket), and alerts_log.

- [ ] Choose CockroachDB Serverless **or** Aurora Serverless v2 Postgres; document tradeoff
- [ ] Migrations from day one (schema as code)
- [ ] Tables:
  - [ ] `tickers` — market, sector, currency, exchange, universe/watchlist flags
  - [ ] `price_history` — OHLCV, adjusted vs unadjusted if needed
  - [ ] `fundamentals`
  - [ ] `scores` — per bucket: long-term / swing / penny-growth
  - [ ] `positions` — actual holdings tagged by bucket + market
  - [ ] `alerts_log`
  - [ ] `analyses` — per-ticker write-up (as-of date, buckets covered, thesis, risks, implied action)
  - [ ] `fundamental_reports` — Indian analyser JSON (ticker, horizon, citations, confidence, HTML or fragment id)
- [ ] Supporting tables as needed: `fx_rates`, `trades` (buy/sell ledger), `digests`, `news_filings` (for catalysts)
- [ ] Connection via Secrets Manager; no public DB if avoidable

---

## Phase 4 — Build the three scoring engines

Three Lambda functions (or one Fargate task with three modes), each reading the same price/fundamentals tables but computing different scores: long-term (fundamental scorecard, runs weekly), swing (technical indicators — RSI/MACD/volume/ATR, runs daily), penny-growth (volume-surge + catalyst detection via an LLM call to Bedrock or the Anthropic API on fresh news/filings, runs daily with tighter thresholds). Write scores back to the scores table.

**Ship long-term + swing first. Defer penny-growth until Phase 5 guardrails exist.**

- [ ] Shared indicator / scorecard library (used by long-term + swing)
- [ ] Long-term engine: fundamental scorecard, **weekly**, using the Indian analyser classifications where the name is NSE/BSE
- [ ] **Indian stock fundamental analyser** (on-demand + weekly refresh): ticker + investment horizon → 8-tab widget
  - [ ] Spec: `docs/indian-stock-fundamental-analyser.md`
  - [ ] Fill `docs/templates/indian-fundamental-report.html` (View tab default)
  - [ ] Both inputs required; no buy/sell/target; cite every metric or `DATA UNAVAILABLE`
  - [ ] Live sources in order: NSE → BSE → Screener.in → Tickertape → Moneycontrol → annual reports / transcripts
  - [ ] Steps 3–11: valuation, growth, health, returns, horizon CAGR scenarios, peers, ownership, View, confidence
  - [ ] API: `{ ticker, horizon_years }` → structured JSON + HTML fragment
- [ ] Swing engine: RSI / MACD / volume / ATR, **daily**
- [ ] Write scores to `scores` with bucket, as-of date, and explanation fields
- [ ] Per-ticker **stock analysis** after scoring
  - [ ] **NSE/BSE long-term:** the fundamental analyser widget (not a free-form essay)
  - [ ] **Swing / US / penny:** shorter note (technicals, catalyst, risks) until a US template exists
  - [ ] Persist to `analyses` / `fundamental_reports`; never invent numbers
  - [ ] First slice: Indian long-term analyser for watchlist or held NSE/BSE names + swing scores
- [ ] Penny-growth engine (**last**): volume-surge + catalyst detection (Bedrock or Anthropic) on fresh news/filings, **daily**, tighter thresholds
- [ ] News/filings ingest for penny bucket (e.g. NSE announcements, SEC EDGAR / 8-K style filings)
- [ ] Catalyst-decay checks (stale catalysts stop scoring as “fresh”)

---

## Phase 5 — Add the portfolio and risk layer

A small Lambda or API endpoint (API Gateway + Lambda) that lets you log buys/sells against the positions table, tagged by strategy bucket and market. Compute per-bucket P&L (native currency + converted total) and enforce your own risk rules in code — e.g. flag if penny/growth exposure exceeds your cap, or if a position is missing a stop-loss.

- [ ] API Gateway + Lambda: log buys/sells → `positions` / `trades`, tagged by bucket + market
- [ ] Per-bucket P&L: native currency + converted total (INR/USD)
- [ ] Risk rules in code:
  - [ ] Flag penny/growth exposure above cap
  - [ ] Flag position missing stop-loss
  - [ ] Other personal caps (per-name, per-market) as you define them
- [ ] Optional later: sync fills from Kite Connect vs manual ledger (do not block the first E2E)
- [ ] **`/portfolio` page** (separate from `/analyse`): live books via **Kite MCP** + **Groww MCP** — spec `docs/portfolio-analysis.md`
  - [ ] Hosted read endpoints: `https://mcp.kite.trade/mcp`, `https://mcp.groww.in/mcp` (or equivalent Kite Connect / Groww APIs server-side)
  - [ ] Read-only: holdings, positions, margins, MF holdings (Kite), LTP; **never** place/modify/cancel orders from this page
  - [ ] Combined ISIN view, broker split, overlap, allocation, bucket P&L, cap / missing-stop flags, ledger reconcile
  - [ ] VIEW paragraph grounded in broker payloads only; DATA UNAVAILABLE if a broker is down
  - [ ] Snapshot table `portfolio_snapshots` for digest vs yesterday

---

## Phase 6 — Wire up the daily digest and alerts

After the scoring jobs run, a Lambda calls the Anthropic API (or Bedrock) to generate a short daily brief across all three buckets, then sends it via SES (email) or a Telegram bot webhook. Use SNS for anything urgent — stop-loss breach, big overnight move on a held penny stock — separate from the daily digest so time-sensitive alerts don't wait for the nightly summary.

- [ ] Orchestrate: scoring complete → digest Lambda (EventBridge / Step Functions)
- [ ] LLM daily brief (Anthropic or Bedrock) across buckets that are live
- [ ] Digest includes the long-term **View** one-liner for Indian names (plus links to the full widget), not scores alone
- [ ] Delivery: SES **or** Telegram bot webhook
- [ ] Persist digest text (S3 and/or `digests` table) for the dashboard
- [ ] SNS urgent path, **not** gated on the nightly digest:
  - [ ] Stop-loss breach
  - [ ] Large overnight move on a held penny name
- [ ] Write every alert to `alerts_log`

**First E2E:** one alert channel (email or Telegram) for long-term + swing only.

---

## Phase 7 — Build a minimal dashboard

A single-page app (React, hosted on S3 + CloudFront, or just Amplify Hosting for simplicity) that reads from a small API Gateway + Lambda layer over your database. Show current scores per bucket, open positions with P&L, and the latest digest. Skip auth complexity since it's personal — just keep the CloudFront distribution private via a signed URL or IP allowlist.

- [ ] Read API (API Gateway + Lambda) over the database
- [ ] React SPA **two pages**:
  - [ ] `/analyse` — Indian fundamental analyser (ticker + horizon → 8-tab widget, View default)
  - [ ] `/portfolio` — Kite + Groww combined books (MCP/API), P&L, overlap, risk flags, VIEW
- [ ] Shared chrome: scores / digest links; do not dump both UIs on one screen
- [ ] Host: S3 + CloudFront **or** Amplify Hosting
- [ ] Personal access only: signed URL **or** IP allowlist (no full auth stack)

---

## Phase 8 — Sequence the build (execution order)

Ship long-term + swing buckets first since they share indicator logic and carry lower operational risk — get the data pipeline, DB, scoring, and one alert channel working end-to-end for those two before adding anything else. Add the penny/high-growth bucket last, once the guardrail logic (position caps, stop-loss enforcement, catalyst-decay checks) is solid, since that bucket does the most damage if it's half-built.

Recommended order:

1. Phase 0 scaffolding (docs + layout only when implementation starts)
2. Phase 1 AWS foundation (IaC)
3. Phase 3 database (schema + migrations; can land with Phase 1)
4. Phase 2 ingestion for NSE/BSE + US, long-term + swing universe only
5. Phase 4 long-term Indian analyser (ticker + horizon widget) + swing (daily) scorers
6. Phase 5 positions + P&L + risk flags (caps + stop-loss) — **required before penny**
7. Phase 6 one digest/alert channel for those two buckets
8. Phase 7 dashboard: `/analyse` widget + **`/portfolio` (Kite + Groww MCP)** + digest
9. **Then** penny/high-growth: ingest catalysts, scorer, tighter thresholds, SNS urgent alerts, exposure caps, catalyst-decay

---

## Definition of first vertical slice (long-term + swing)

Done when all of the following are true:

- [ ] Daily (and weekly long-term) jobs run from EventBridge without manual invoke
- [ ] Normalized prices/fundamentals land in the DB, tagged by market/currency
- [ ] Long-term and swing scores are written for the watchlist
- [ ] At least one NSE/BSE name produces the 8-tab fundamental report from ticker + horizon (View tab default, citations or DATA UNAVAILABLE)
- [ ] At least one position can be logged and P&L shown (native + converted)
- [ ] One digest or alert channel fires after scoring
- [ ] `/portfolio` can show Kite and/or Groww holdings (read-only MCP/API) without mixing that UI into `/analyse`
