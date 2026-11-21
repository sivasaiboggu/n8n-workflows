# Lead Sniper — Siva — SETUP

## What it does
Lead in (webhook `POST /lead-sniper` or Manual Trigger) → validates it has at
least an email or a company → enriches with company data (Apollo-style API) →
Gemini scores fit 0–100 against Siva's offering (AI SDR agent, LangGraph /
LangChain / RAG, cold-mail automation) and writes a personalised first line
from the lead's recent posts (only supplied data — never invented) → adds the
lead to the cold-email sequence sheet as a **DRAFT** (`approved_to_send = No`).
Hot leads (score ≥ 80) also get a Telegram ping. **This workflow never sends
email** — sending happens only after separate human approval.

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
| Enrich Lead (Apollo-style) | Apollo API key |
| Telegram Ping — Hot Lead, Notify on Error (Telegram) | Telegram API (chat ID = `YOUR_TELEGRAM_CHAT_ID`) |
| Add Hot Lead to Sequence Sheet, Add Lead to Sequence Sheet, Log Error to Sheet, Log Execution | Google Sheets (OAuth2) |
| Gemini Chat Model | Google Gemini (PaLM) API (Gemini free tier) |

## Sheets / resources to create
- **Sequence** tab (sheet `YOUR_LEAD_SEQUENCE_SHEET_ID`) — columns: `timestamp`,
  `name`, `email`, `company`, `score`, `first_line`, `status`
  (this tab IS the memory layer — check it before re-adding a lead; a lead
  already mailed must never be re-added).
- **Log** tab (execution-log sheet `YOUR_EXECUTION_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `status`, `detail`.
- **Errors** tab (error-log sheet `YOUR_ERROR_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `node`, `error`, `execution_id`.
- Webhook: after import, n8n shows the production URL for `POST /lead-sniper`
  with JSON body e.g. `{"name": "...", "email": "...", "company": "...",
  "linkedin_url": "...", "recent_posts": "..."}`.

## Test (before activating)
1. Use Manual Trigger with a test lead you know (your own test data — do not
   use a real person's data for testing).
2. Check: enrichment returns, Gemini returns `{score, reason, first_line}`,
   the row lands in Sequence with status "DRAFT — needs approval" and
   `approved_to_send = No`, and no email is sent anywhere (there is no email
   node in this workflow).
3. Test a hot lead (score ≥ 80) and confirm the Telegram ping arrives; test a
   colder lead and confirm it is logged without a ping.
4. Only then Activate.

## Notes / limits
- HTTP Request nodes use timeout 10,000 ms and JSON response format; workflow
  Settings retry covers retries where the node type has no built-in retry.
- Never send to a lead from this workflow: the sequence sheet is the handoff
  to Siva's verified cold-mail process (verified emails only, 5-minute gap, PDF
  attached, per his standing rules).
- Error handling: Error Trigger → Log Error to Sheet → Notify on Error (Telegram).
