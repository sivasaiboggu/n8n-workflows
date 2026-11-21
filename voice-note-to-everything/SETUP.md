# Voice-Note-to-Everything — Siva — SETUP

## What it does
Telegram voice note → transcription → Gemini splits it into tasks, calendar
events and ideas (JSON only, using only what was said) → after Siva approves the
split in Telegram, it creates Todoist tasks, Google Calendar events and Notion
notes, then logs everything to Sheets. Nothing is created until approved.

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
| Telegram Trigger, Get Voice File, Send Approval Summary, Notify on Error (Telegram) | Telegram API (bot token; chat ID = `YOUR_TELEGRAM_CHAT_ID`) |
| Transcribe Voice (STT — placeholder) | Point it at Gemini transcription or any speech-to-text service of your choice — the node URL is currently the placeholder `https://YOUR_STT_ENDPOINT/transcribe` (returns JSON with a `text` field). Add that service's auth as a credential on the node; no key or secret is in the file. No OpenAI/Claude is used anywhere — Gemini only. |
| Create Todoist Tasks | Todoist API token |
| Create Calendar Events | Google Calendar (OAuth2) |
| Save Ideas to Notion | Notion API key — also set `YOUR_NOTION_DATABASE_ID` in the node body |
| Log Created Items, Log Error to Sheet, Log Execution | Google Sheets (OAuth2) |
| Gemini Chat Model | Google Gemini (PaLM) API (Gemini free tier) |

## Sheets / resources to create
- **Log** tab (sheet `YOUR_VOICE_NOTES_SHEET_ID`) — columns: `timestamp`, `transcript`, `tasks`, `status`
  (this tab IS the memory layer — one row per voice note).
- **Log** tab (execution-log sheet `YOUR_EXECUTION_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `status`, `detail`.
- **Errors** tab (error-log sheet `YOUR_ERROR_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `node`, `error`, `execution_id`.
- Telegram bot: create via @BotFather, send it a voice note first so the chat is open.

## Test (before activating)
1. Send a short test voice note to the Telegram bot, e.g. "Remind me to email
   the Drivetrain founder tomorrow at 10am and note the idea of a LangGraph demo".
2. Use **Execute Workflow** / trigger via Telegram and check: transcription text
   appears, Gemini returns JSON with tasks/events/ideas, and the approval summary
   arrives in Telegram — and nothing is created yet.
3. Reply APPROVE and confirm: a Todoist task, a Calendar event and a Notion note
   appear, and Log Created Items appends a row.
4. Repeat with REJECT and confirm nothing is created and Log Skipped is recorded.
5. Only then Activate.

## Notes / limits
- HTTP Request nodes use timeout 10,000 ms and JSON response format; workflow
  Settings retry covers retries where the node type has no built-in retry.
- The Transcribe node posts the voice file to the STT endpoint — check that
  endpoint's privacy/retention before sending sensitive notes.
- The Wait node waits up to 4 hours for approval; swap for a Form Trigger if
  you prefer a click approval.
- Error handling: Error Trigger → Log Error to Sheet → Notify on Error (Telegram).
