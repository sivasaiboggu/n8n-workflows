# Crypto/Stock Whale Watcher — Siva — SETUP

## What it does
Every 15 minutes it polls public market APIs (CoinGecko for crypto, Yahoo Finance
screener for stocks) for large movers, filters to moves over ~5% in 24h, Gemini
adds a short news-context note, and Telegram sends an alert with a "why it matters"
summary. **No financial advice. No auto-trading. There is no trade action in this
workflow** — the Enforce No-Trade Guard code node explicitly strips any trade
instruction before the alert is sent.

## Import
**Credentials rule (corrected 20 Nov 2026): this file sets NO credential on any
node** — n8n refuses to save an import that references a null-id credential
(this exact failure hit the daily-brief import). On import each node shows
"No credential set": click it and pick/create the credential listed below.
The same list is in the "Sticky Note — Credentials" node in the workflow.
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → ⋯ → Import from File →
   select `workflow.json` in this folder.
2. Leave the workflow **inactive** until you have tested it (below).
3. Replace every `YOUR_...` placeholder: `YOUR_TELEGRAM_CHAT_ID` (Telegram node
   parameters) and `YOUR_GOOGLE_SHEET_ID` (Google Sheets nodes).

## Credentials to connect (pick/create on import — none are in the file)

No credential is set in `workflow.json` (see Import above) — the table below is what to
pick/create for each node; no real key or secret is stored anywhere in this repo.
| Credential | Where |
|---|---|
| Google Gemini(PaLM) Api — Gemini API key | Gemini Chat Model node |
| Telegram account — bot token from @BotFather | Send Telegram Alert, Notify Error via Telegram |
| Google Sheets account — OAuth2 | Log Execution, Log Error to Sheet |

## Google Sheet to create (one sheet, id = YOUR_GOOGLE_SHEET_ID)
- Tab `Execution Log` — columns: `timestamp | workflow | status | summary`
- Tab `Error Log` — columns: `timestamp | workflow | error | node`

## How to test before activating
1. Click **Execute Workflow** (Manual Trigger equivalent — use the Execute button
   on the canvas; the Schedule Trigger fires on its own schedule only).
2. Check Telegram for the alert (or an empty run if no mover passed 5% — that is
   a valid result; lower the threshold in Filter Whale Threshold temporarily to
   force a pass, then restore it to 5).
3. Check the Execution Log tab has a new row; force an error (e.g. bad CoinGecko
   URL temporarily) and confirm a row appears in Error Log and a Telegram error
   notice arrives, then restore the URL.
4. Only then toggle the workflow **Active**.

## Notes / limits
- **Approval-gate exception (documented):** this workflow only notifies Siva himself via Telegram and never trades, spends or sends to anyone else — the Enforce No-Trade Guard code node blocks any trade action. No approval gate is needed because there is no irreversible action.
- CoinGecko free tier is rate-limited; 15-min polling is well within it.
- Yahoo Finance screener endpoint is unofficial and may change — if Fetch Stock
  Large Moves starts failing, replace its URL with your preferred public
  screener endpoint; the error-logging path will surface failures.
- Retry: HTTP Request nodes use a 10s timeout and JSON response format; enable
  workflow Settings → Retry on failure if your n8n plan supports it, or rely on
  the next 15-min run (the workflow is idempotent — it only sends alerts).
