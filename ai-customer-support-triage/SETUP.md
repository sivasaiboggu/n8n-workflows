# AI Customer Support Triage — Siva — SETUP

## What it does
Webhook/form (or WhatsApp) message → normalised → RAG context over support
docs (placeholder — see below) → Gemini triages. Confident (≥ 0.8) → draft
reply → **human approval** → Gmail reply. Not confident → ticket in
Linear/Jira + Telegram alert. Nothing customer-facing is sent without approval.

## 1. Import
1. In n8n (`sivasai-boggu.app.n8n.cloud`): Workflows → Import from File →
   select `workflow.json`.
2. The workflow imports **inactive** (`active: false`). Do not activate until
   the test in §5 passes.

## 2. Credentials to connect (NO credential is set in workflow.json — see note below)
| Credential (as named in the file) | What to create |
|---|---|
| Google Gemini API | Google Gemini (PaLM) API — Gemini API key, free tier. Used by "Gemini Chat Model". |
| Gmail account | Gmail OAuth2 for `sivasaiboggu@gmail.com`. Used by "Send Customer Reply". |
| Telegram account | Telegram Bot API — bot token + `YOUR_TELEGRAM_CHAT_ID` (your chat with the bot). Used for approval requests, team alerts, error alerts. |
| Google Sheets account | Google Sheets OAuth2. Used by Log Execution / Log Error / Read Approval Sheet. |
| Linear API Token | HTTP Header Auth — Linear personal API token. Used by "Create Ticket" (GraphQL to `https://api.linear.app/graphql`). **If you use Jira instead**, replace this node with a Jira node / Jira HTTP call and update the body — do not just change the URL. |

CORRECTED RULE: the `credentials` property is omitted entirely from every node in workflow.json. n8n treats a null-id credential placeholder as a reference to a credential not shared with the account and refuses to save the import (this exact failure hit the Daily Brief import). With no credentials property, n8n shows "No credential set" on each node — pick an existing credential or create a new one on import. The per-node credential list is also in the "Sticky Note — Credentials" inside the workflow. CORRECTED RULE: the `credentials` property is omitted entirely from every node in workflow.json. n8n treats a null-id credential placeholder as a reference to a credential not shared with the account and refuses to save the import (this exact failure hit the Daily Brief import). With no credentials property, n8n shows "No credential set" on each node — pick an existing credential or create a new one on import. The per-node credential list is also in the "Sticky Note — Credentials" inside the workflow. No real key/secret is stored in the JSON. Fill `YOUR_TELEGRAM_CHAT_ID` in the
Telegram nodes.

## 3. Google Sheet — create one sheet, id = `YOUR_GOOGLE_SHEET_ID`
Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node with the real
sheet ID (or URL). Tabs and columns (first row = headers):
- **Execution Log**: `timestamp | workflow | status | summary`
- **Error Log**: `timestamp | workflow | error | node`
- **Approvals**: `key | value | updated_at` — one row: `key=latest`,
  `value=pending`. To approve a reply, set `value=approved`; to reject,
  `value=rejected`. The workflow waits 60 minutes after the Telegram request,
  then reads this row. A rejected/expired approval sends **nothing** (the run
  ends without a customer reply and without a Log Execution row — check the
  Approvals tab to see why).

## 4. Resources / placeholders to fill
- **Webhook path** `support-triage`: point your form / WhatsApp provider at
  the webhook URL n8n shows (test URL while testing, production URL once
  active). Expected POST body:
  `{"channel": "form", "sender": "customer@example.com", "message": "..."}`
  `sender` must be a real replyable email address for the Gmail reply to work.
- **RAG / vector store — NOT connected (placeholder by design).**
  "Retrieve Docs Context" is a Code node that returns empty context and
  `retrieval: "not_connected"`. Until a vector store is wired in, the agent
  is instructed to mark low confidence and ticket the message rather than
  invent a docs answer. To connect one later: replace that Code node with a
  vector-store retriever over your support docs; the agent prompt already
  consumes `docs_context`. Suggested future tab: **Support Log**
  (`timestamp | sender | channel | summary | confidence | resolved`) — not
  written by this version.
- **Linear**: the ticket body carries sender, message, summary and suggested
  reply as context. Verify the GraphQL mutation against your Linear workspace
  on first run — the issue title/description format is a starting point.

## 5. Test (Manual Trigger first, before activating)
1. With the workflow open, click **Execute Workflow** (Manual Trigger).
   The run will reach "Parse & Normalise Message" with empty input and fail
   at validation — that is expected for a manual run with no webhook body.
   For a real test, use the webhook **test URL**: send the POST body above,
   then watch the run.
2. Send a simple question (e.g. "What are your support hours?"). With no
   docs connected, expect: confidence < 0.8 → "Create Ticket" fires and a
   Telegram alert arrives. Verify the Linear ticket exists.
3. To test the confident path, temporarily lower the threshold in
   "Confidence Check" or connect docs — then verify the Telegram approval
   request arrives, set Approvals `value=approved`, wait for the 60-min Wait
   node (or shorten it for the test), and confirm the Gmail reply is sent
   only after approval.
4. Check Execution Log / Error Log rows, then re-activate the Wait duration
   if you shortened it, and activate the workflow.

## 6. Notes / limits
- Retry: HTTP Request nodes use `timeout: 10000` and JSON response format;
  n8n workflow Settings retry covers the rest (enable in workflow Settings
  if you want automatic retries on failure).
- Error handling: Error Trigger → Log Error to Sheet → Telegram alert to Siva.
- WhatsApp is notify-only in this setup — the inbound channel in the brief
  is form/WhatsApp; the webhook accepts either, but sending a WhatsApp reply
  needs an approved WhatsApp provider — use Gmail/Telegram until that is set up.
