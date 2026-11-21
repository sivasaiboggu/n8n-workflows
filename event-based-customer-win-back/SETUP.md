# Event-Based Customer Win-Back — Siva — SETUP

## What it does
Triggered by a webhook (e.g. from your CRM / billing system) or a daily check,
it reads customer signals from Sheets, detects churn signals — inactivity over
30 days, a failed payment, or a support complaint — and Gemini writes a
personalised win-back message. The draft is saved and sent to you on Telegram
for approval. **Nothing is sent until you write `APPROVE` in the Approval tab.**
Then it sends via email (Gmail) or SMS (your SMS provider), waits 3 days,
checks for a reply, logs the outcome, and **escalates to you on Telegram if
there is no reply**.

## Import
**Credentials rule (corrected 20 Nov 2026): this file sets NO credential on any
node** — n8n refuses to save an import that references a null-id credential
(this exact failure hit the daily-brief import). On import each node shows
"No credential set": click it and pick/create the credential listed below.
The same list is in the "Sticky Note — Credentials" node in the workflow.
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → ⋯ → Import from File →
   select `workflow.json` in this folder. Leave inactive until tested.
2. Replace every `YOUR_...` placeholder: `YOUR_GOOGLE_SHEET_ID`,
   `YOUR_TELEGRAM_CHAT_ID`, `YOUR_WINBACK_WEBHOOK_PATH`, and the SMS node URL
   (`YOUR_SMS_PROVIDER_API`, `YOUR_CUSTOMER_PHONE`, `YOUR_MESSAGE_TEXT` — these
   are placeholders for your real SMS provider's endpoint; do not paste a live
   API key into the URL, use the SMS Provider API credential).

## Credentials to connect (pick/create on import — none are in the file)

No credential is set in `workflow.json` (see Import above) — the table below is what to
pick/create for each node; no real key or secret is stored anywhere in this repo.
| Credential | Where |
|---|---|
| Google Gemini(PaLM) Api | Gemini Chat Model |
| Google Sheets account | Read Customer Signals, Save Draft Win-Back Message, Check Approval Sheet, Check for Reply, Log Win-Back Outcome, Log Execution, Log Error to Sheet |
| Telegram account | Send Approval Request via Telegram, Escalate to Human — No Reply, Notify Error via Telegram |
| Gmail account | Send Win-Back Email |
| SMS Provider API (HTTP Header Auth) | Send Win-Back SMS |

## Google Sheet to create (id = YOUR_GOOGLE_SHEET_ID)
- Tab `Customer Signals` — columns: `customer_name | email | phone | channel | product | days_inactive | payment_status | support_complaint | last_contact`
- Tab `Win-Back Drafts` — columns: `timestamp | customer_name | email | message | status`
- Tab `Approval` — columns: `customer_name | email | approval` (write `APPROVE` to send; anything else = nothing sent)
- Tab `Replies` — columns: `customer_name | email | replied` (`yes`/`no` — fill from your inbox/CRM; see test notes)
- Tab `Win-Back Outcomes` — columns: `timestamp | customer_name | outcome | status`
- Tab `Execution Log` — columns: `timestamp | workflow | status | summary`
- Tab `Error Log` — columns: `timestamp | workflow | error | node`

## How to test before activating
1. Add a test customer row (your own email/phone) with `days_inactive` = 45 and
   `channel` = `email`. Trigger the webhook with a test POST, or click
   **Execute Workflow** (the Daily Churn Check schedule only fires daily).
2. Confirm: a draft appears in Win-Back Drafts, a Telegram approval request
   arrives, and **no email is sent yet**.
3. Write `APPROVE` in the Approval tab, run again, confirm the email arrives.
   Wait — do not wait 3 real days to test the escalation: temporarily set the
   Wait node to 1 minute, run, mark `replied` = `no` in Replies, and confirm
   the Telegram escalation arrives. Then restore Wait to 3 days.
4. Only then toggle **Active** and point your CRM/billing webhook at the
   production webhook URL shown on the Churn Signal Webhook node.

## Notes / limits
- The SMS node URL is a placeholder for your provider (e.g. Twilio / MSG91) —
  replace the URL and auth to match that provider's docs; the email path works
  as-is with Gmail.
- Retry: HTTP Request nodes use a 10s timeout and JSON response format; Gmail /
  Sheets failures are caught by the Error Trigger → Error Log → Telegram path.
