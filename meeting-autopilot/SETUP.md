# Meeting Autopilot — Siva — SETUP

## What it does
Calendar trigger → Fireflies/Recall transcript → Gemini extracts decisions
and action items (owner + deadline) → **human approval** → summary email to
attendees → tasks created in Google Sheets (Todoist optional, see §4).
No summary email is sent without approval.

## 1. Import
1. In n8n (`sivasai-boggu.app.n8n.cloud`): Workflows → Import from File →
   select `workflow.json`.
2. Imports **inactive** (`active: false`). Do not activate until the test
   in §5 passes.

## 2. Credentials to connect (NO credential is set in workflow.json — see note below)
| Credential (as named in the file) | What to create |
|---|---|
| Google Calendar account | Google Calendar OAuth2. Used by "Calendar Trigger" and "Check Calendar" (here: the trigger). |
| Google Gemini API | Google Gemini (PaLM) API — Gemini API key, free tier. |
| Telegram account | Telegram Bot API — bot token + `YOUR_TELEGRAM_CHAT_ID`. Approval requests + error alerts. |
| Gmail account | Gmail OAuth2 for `sivasaiboggu@gmail.com`. Sends the summary email. |
| Google Sheets account | Google Sheets OAuth2. Tasks / Execution Log / Error Log / Approval Sheet. |
| Fireflies API Key | HTTP Header Auth — Fireflies API key. Used by "Fetch Transcript" (GraphQL `https://api.fireflies.ai/graphql`). **Recall alternative**: replace this node with a Recall.ai call — the transcript shape differs, so update "Extract Actions Agent" input mapping too. |

CORRECTED RULE: the `credentials` property is omitted entirely from every node in workflow.json. n8n treats a null-id credential placeholder as a reference to a credential not shared with the account and refuses to save the import (this exact failure hit the Daily Brief import). With no credentials property, n8n shows "No credential set" on each node — pick an existing credential or create a new one on import. The per-node credential list is also in the "Sticky Note — Credentials" inside the workflow. No real key/secret is stored in the JSON.

## 3. Google Sheet — create one sheet, id = `YOUR_GOOGLE_SHEET_ID`
Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node. Tabs/columns:
- **Execution Log**: `timestamp | workflow | status | summary`
- **Error Log**: `timestamp | workflow | error | node`
- **Approvals**: `key | value | updated_at` — one row: `key=latest`,
  `value=pending`. Approve by setting `value=approved`, reject with
  `value=rejected`. The workflow waits 60 minutes after the Telegram
  request, then reads this row. If rejected, the run goes to "Merge
  Outcomes" → Log Execution with no email and no tasks.
- **Tasks**: `task | owner | deadline | meeting` — one row per action item
  (this version writes the actions as semicolon-joined values in a single
  row; split into one row per action if you prefer, by adding a Split node
  before "Create Tasks in Sheets").
- **Meetings** (optional, future): `meeting | attendees | decisions | actions | done` — for tracking open actions from past meetings.

## 4. Resources / placeholders to fill
- **Calendar**: replace `YOUR_CALENDAR_ID` in "Calendar Trigger" with your
  calendar ID (usually `sivasaiboggu@gmail.com` / `primary`). The trigger
  fires on event start — the bot must already be recording the meeting in
  Fireflies/Recall for a transcript to exist.
- **Transcript**: "Fetch Transcript" queries Fireflies by `meeting_id` from
  the calendar event. Fireflies meeting IDs do not always equal calendar
  event IDs — on first run, verify the query returns sentences; if it
  returns nothing, the agent is instructed to extract nothing and say the
  transcript is missing (check the run output, fix the ID mapping, re-test).
- **Todoist alternative**: to create tasks in Todoist instead of Sheets,
  replace "Create Tasks in Sheets" with an HTTP Request to
  `https://api.todoist.com/rest/v2/tasks` (Header Auth: Todoist API token),
  one call per action item. The Sheets version is the default in this file.

## 5. Test (Manual Trigger first, before activating)
1. Click **Execute Workflow** (Manual Trigger). A manual run has no calendar
   event, so "Get Meeting Details" produces an empty meeting and the
   transcript fetch will return nothing — expect the agent to report no
   transcript / no actions. That is the correct empty-input behaviour, not a
   failure. For a real test, run it against a real past meeting: use the
   calendar trigger's test event, or pin a real event's JSON on
   "Get Meeting Details" and execute.
2. With a real transcript, verify: decisions + actions extracted, owners are
   actual attendees (or `unassigned`), no invented deadlines.
3. Verify the Telegram approval request arrives; set Approvals
   `value=approved`; wait out (or temporarily shorten) the 60-min Wait node;
   confirm the Gmail summary reaches the attendees and the Tasks row is written.
4. Check Execution Log / Error Log, restore the Wait duration if shortened,
   then activate.

## 6. Notes / limits
- Retry: HTTP Request nodes use `timeout: 10000` and JSON response format;
  enable workflow Settings retry for automatic retries.
- Error handling: Error Trigger → Log Error to Sheet → Telegram alert to Siva.
- The summary email goes only to attendees listed on the calendar event —
  never to invented addresses.
