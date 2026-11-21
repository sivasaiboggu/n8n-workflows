# Daily Automation Self-Report — Siva — SETUP

## What it does
Every evening at **21:00 IST** the workflow reads today's rows from the
**Daily Log** sheet (written by Siva's other automations) and, if configured,
the n8n API execution list → a Code node normalises today's runs per workflow
→ Gemini summarises what each automation did, what succeeded, what failed and
what needs Siva's attention → the report is logged to the **Weekly Trends**
tab (the memory layer) → the **Control** sheet flag `send_report` is checked
(default **ON**) → if ON, Gmail sends the report to Siva only
(`sivasaiboggu@gmail.com`, subject `Your automations today — <date>`) → the
run itself is logged back to the Daily Log tab.

**No invented data:** if the Daily Log has no rows for today and the n8n API
is not configured, the report says **"No runs recorded today"** and nothing
else about activity. Gemini is instructed (system + user prompt) to use only
the data it is given.

## 1. Import
1. In n8n (`sivasai-boggu.app.n8n.cloud`): Workflows → Import from File →
   select `workflow.json`.
2. The workflow imports **inactive** (`active: false`). Do not activate until
   the test in §5 passes.
3. Set the workflow timezone to **Asia/Kolkata** (workflow Settings →
   Timezone) so the 21:00 cron trigger fires at 21:00 IST.

## 2. Credentials to connect (all placeholders — n8n will prompt on import)
No credential is stored in the JSON (the `credentials` property is omitted
from every node on purpose — n8n refuses imports that reference unshared
credentials). On import, n8n shows "No credential set" on each node below;
pick or create the credential there.

| Credential | What to create | Used by |
|---|---|---|
| Gmail account | Gmail OAuth2 for `sivasaiboggu@gmail.com` | "Send Report Email" |
| Google Sheets account | Google Sheets OAuth2 | "Read Daily Log", "Log Report to Weekly Trends", "Read Send Flag", "Log Execution", "Log Error to Sheet" |
| Google Gemini API | Google Gemini (PaLM) API — Gemini API key, free tier | "Gemini Chat Model" |
| Telegram account | Telegram Bot API — bot token + `YOUR_TELEGRAM_CHAT_ID` (your chat with the bot) | "Send Telegram Alert" (error branch only) |
| n8n API key (optional) | HTTP Header Auth with header `X-N8N-API-KEY` and your n8n API key | "Fetch Today's Executions via n8n API" — **optional placeholder**. If you do not connect it, the node is set to continue on error and the report is built from the Daily Log sheet alone; "Combine Log + API Data" marks `api_available: false`. No real key is in the file. |

Fill `YOUR_TELEGRAM_CHAT_ID` in "Send Telegram Alert" and replace
`YOUR_N8N_HOST` in the API node with your instance host
(`sivasai-boggu.app.n8n.cloud`) if you connect the API key.

## 3. Google Sheet — one sheet, id = `YOUR_AUTOMATION_LOG_SHEET_ID`
Replace `YOUR_AUTOMATION_LOG_SHEET_ID` in every Google Sheets node with the
real sheet ID (or URL). This is the same log sheet Siva's other workflows
write to. Tabs and columns (first row = headers):

- **Daily Log**: `workflow | time | status | items processed | errors` —
  written by Siva's other workflows during the day (one row per run), and by
  this workflow's "Log Execution" step for the report run itself
  (`workflow=Daily Automation Self-Report`).
- **Control**: `key | value | updated_at` — one row: `key=send_report`,
  `value=ON`. This is the human-approval flag: set `value=OFF` to pause the
  email (the report is still generated and logged to Weekly Trends, but
  nothing is sent). If the row is missing entirely, the workflow defaults
  to ON — Siva is the only recipient, so ON is the safe default.
- **Weekly Trends**: `date | report_summary | workflows_succeeded | workflows_failed | needs_attention | status` —
  one row per daily report. This is the **memory layer**: past reports feed
  the weekly trend — compare a week's rows to spot which workflow keeps
  failing. A future version can feed past Weekly Trends rows into the Gemini
  prompt; this version does not invent a live database connection.
- **Error Log**: `timestamp | workflow | node | error | execution_id` —
  written by the Error Trigger branch when this workflow itself fails.

## 4. Resources / placeholders to fill
- **Schedule**: cron `0 21 * * *` in "Every Evening 21:00 IST". With the
  workflow timezone set to Asia/Kolkata (§1 step 3) this is 21:00 IST daily.
- **n8n API node**: placeholder by design (§2). It lists executions
  (`GET /api/v1/executions?limit=100`); it does not filter by date, so the
  Combine node treats the list as supplementary context only — the Daily Log
  sheet is the source of truth for "today".
- **Date handling**: "Normalise Today's Log" computes today in IST
  (UTC+5:30) in code, so rows are filtered by Siva's local date regardless
  of the n8n server timezone. Other workflows must write `time` as an ISO
  timestamp (e.g. `2026-11-20T15:30:00+05:30`) for the filter to match.

## 5. Test (Manual Trigger first, before activating)
1. With the workflow open, click **Execute Workflow** (Manual Trigger).
   With an empty Daily Log, expect: the Gmail step is reached and the email
   body says "No runs recorded today" — verify no numbers are invented.
   (The email goes to Siva himself; to avoid even that during testing, set
   the Control flag to OFF first and confirm the run ends at "Mark Skipped —
   Flag OFF" with a "Log Execution" row `status=skipped — send_report flag OFF`.)
2. Add 2–3 test rows to Daily Log for today (one success, one failure) and
   run again with the flag ON: verify the email arrives with subject
   `Your automations today — <today's date>`, the per-workflow breakdown is
   correct, and the "Needs your attention" section lists only the failing
   workflow.
3. Verify the Weekly Trends tab gained one row and the Daily Log tab gained
   a `Daily Automation Self-Report` row.
4. To test the error branch, temporarily break a node (e.g. a wrong sheet
   ID) and run: verify an Error Log row and a Telegram alert arrive, then
   restore the sheet ID.
5. Restore the Control flag to ON, re-check the cron timezone (§1 step 3),
   and activate the workflow.

## 6. Notes / limits
- Retry: the HTTP Request node uses `timeout: 10000` and JSON response
  format, and is set to continue on error (the API is optional). For other
  nodes, enable retry in workflow Settings if you want automatic retries on
  failure — the Google Sheets / Gmail nodes have no built-in retry parameter.
- Error handling: Error Trigger → Log Error to Sheet (Error Log tab) →
  Telegram alert to Siva. A failed report run never sends a partial email.
- Approval pattern: the email goes only to Siva himself, but the send still
  passes through the Control-sheet flag (IF node "Send Report?") — the same
  approval pattern the other workflows use, so sending can be paused from
  the sheet without deactivating the workflow.
- The Gemini summary is only as good as the Daily Log: workflows that do
  not log their runs will not appear in the report — that is by design
  (missing data is reported as missing, never filled in).

## Notes / limits
- **Approval-gate exception (documented):** this workflow only emails the daily report to Siva himself (sivasaiboggu@gmail.com); the Send Flag sheet tab lets him turn the email off. No approval gate is needed because nothing is sent to anyone else and nothing irreversible happens.
