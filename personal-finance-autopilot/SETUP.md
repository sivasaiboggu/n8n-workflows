# Personal Finance Autopilot — Siva — SETUP

## What it does
Two branches, one workflow:
- **Transactions:** bank/Stripe webhook (`POST /finance-transaction`) sends a
  transaction payload → Gemini categorises it (Food / Transport / Subscriptions /
  Shopping / Bills / Income / Other) and flags unusual spend → the Sheet
  dashboard is updated; unusual spend triggers a Slack alert only (no money moves).
- **Weekly summary:** every Sunday 18:00 → reads the week's transactions →
  Gemini writes a summary with **generic budgeting tips only** → draft saved to
  Sheets → Siva approves in Telegram → weekly summary email is sent to himself.
  No personal financial advice, no investment advice, no recommendations about
  specific products.

## Import
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → Import from File → select
   `workflow.json` in this folder.
2. **Credentials rule (corrected 20 Nov 2026): this file sets NO credential on any
   node** — n8n refuses to save an import that references a null-id credential
   (this exact failure hit the daily-brief import). On import each node shows
   "No credential set": click it and pick/create the credential listed below.
   The same list is in the "Sticky Note — Credentials" node in the workflow.
3. Leave the workflow **inactive** until the test below passes, then Activate.

## Credentials to connect (pick/create on import — none are in the file)
| Node(s) | Credential to connect |
|---|---|
| Update Sheet Dashboard, Read This Week's Transactions, Save Weekly Draft, Log Error to Sheet, Log Execution, Log Weekly Execution | Google Sheets (OAuth2) |
| Flag Unusual Spend (Slack), Notify on Error (Slack) | Slack API (channel = `YOUR_SLACK_CHANNEL_ID`) |
| Send Approval Request | Telegram API (chat ID = `YOUR_TELEGRAM_CHAT_ID`) |
| Send Weekly Summary Email | Gmail (OAuth2) — sends to sivasaiboggu@gmail.com |
| Gemini Chat Model, Gemini Chat Model (Weekly) | Google Gemini (PaLM) API (Gemini free tier) |
| (none) | **No bank or Stripe credential exists in this workflow — transactions arrive via webhook payload only. Do not add real bank credentials.** |

## Sheets / resources to create
Create one Google Sheet (e.g. `Finance — Siva`, ID = `YOUR_FINANCE_SHEET_ID`):
- **Transactions** tab — columns: `date`, `merchant`, `amount`, `currency`, `category`, `unusual`
  (this tab + Dashboard IS the memory layer — weekly summaries read past rows for averages).
- **Weekly Summaries** tab — columns: `week`, `summary`, `status`.
- Add a **Dashboard** tab yourself (or a pivot/chart) summarising monthly totals
  by category from Transactions — the workflow appends the raw rows; the
  dashboard view is a Sheet pivot/chart, not a node.
- **Log** tab (execution-log sheet `YOUR_EXECUTION_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `status`, `detail`.
- **Errors** tab (error-log sheet `YOUR_ERROR_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `node`, `error`, `execution_id`.
- Webhook: after import, n8n shows the production URL for
  `POST /finance-transaction` with JSON body e.g.
  `{"amount": 499, "currency": "INR", "merchant": "Test Store", "description": "test", "date": "2026-11-20"}`.
  Point your bank/Stripe webhook (or a manual sender) at it — never share bank login details.

## Test (before activating)
1. Use Manual Trigger / webhook test with a small test transaction
   (`amount: 499, merchant: "Test Store"`). Check: Gemini categorises it, the
   Transactions row is appended, and no email is sent.
2. Send an unusually large test amount for a category and confirm the Slack
   "Unusual spend" alert posts.
3. Test the weekly branch with Manual Trigger on the Sunday schedule node
   (or Execute Workflow on the weekly chain): confirm a draft is saved, the
   Telegram approval arrives, APPROVE sends the email to sivasaiboggu@gmail.com,
   and REJECT keeps it as a draft only.
4. Only then Activate.

## Notes / limits
- Generic budgeting tips only — this workflow must never give personal financial
  or investment advice; the Gemini prompts say so explicitly.
- HTTP Request nodes use timeout 10,000 ms and JSON response format; workflow
  Settings retry covers retries where the node type has no built-in retry.
- The weekly Wait node waits up to 24 hours for approval; swap for a Form
  Trigger if you prefer a click approval.
- Error handling: Error Trigger → Log Error to Sheet → Notify on Error (Slack).
