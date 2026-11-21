# AI Inbox Chief of Staff — Siva — SETUP

## What it does
A Gmail trigger fires on each new email. Gemini classifies it as **urgent /
reply / archive / delegate** and logs the classification to Sheets. Urgent mail
is sent to Telegram with Approve/Edit instructions. For urgent + reply mail,
Gemini drafts a reply in Siva's voice (direct, polite, concise) and saves it
as a Gmail draft — **nothing is sent without approval**: only when you write
`APPROVE` in the Approval tab is the reply sent. Archive mail is archived;
delegate mail is logged as a task for a human. Everything is logged to Sheets.

Memory layer: the Inbox Memory sheet (see below) records sender, category and
whether Siva approved/edited the draft, so future drafts learn his voice and
which senders he treats as urgent. No live Postgres/vector DB is claimed.

## Import
**Credentials rule (corrected 20 Nov 2026): this file sets NO credential on any
node** — n8n refuses to save an import that references a null-id credential
(this exact failure hit the daily-brief import). On import each node shows
"No credential set": click it and pick/create the credential listed below.
The same list is in the "Sticky Note — Credentials" node in the workflow.
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → ⋯ → Import from File →
   select `workflow.json` in this folder. Leave inactive until tested.
2. Replace every `YOUR_...` placeholder: `YOUR_GOOGLE_SHEET_ID`,
   `YOUR_TELEGRAM_CHAT_ID`, `YOUR_DELEGATE_NAME` (who delegated mail goes to).

## Credentials to connect (pick/create on import — none are in the file)

No credential is set in `workflow.json` (see Import above) — the table below is what to
pick/create for each node; no real key or secret is stored anywhere in this repo.
| Credential | Where |
|---|---|
| Gmail account (OAuth2, sivasaiboggu@gmail.com) | Gmail Trigger — New Email, Archive Email, Save Draft Reply to Gmail, Send Approved Reply |
| Google Gemini(PaLM) Api | Gemini Chat Model (shared by Classifier + Draft Reply agents) |
| Google Sheets account | Log Email to Sheet, Check Approval Sheet, Log Delegate Task, Log Execution, Log Error to Sheet |
| Telegram account | Send Urgent to Telegram — Approve/Edit, Notify Error via Telegram |

Note: the brief mentions "logs everything to Notion/Sheets" — this build logs
to Google Sheets (the tool in the spec's available set); add a Notion node
later if you connect Notion, using the same Inbox Log columns.

## Google Sheet to create (id = YOUR_GOOGLE_SHEET_ID)
- Tab `Inbox Log` — columns: `timestamp | sender | subject | category | reason`
- Tab `Inbox Memory` — columns: `timestamp | sender | category | draft_approved | siva_correction` (fill `siva_correction` when you edit a draft, so future drafts improve)
- Tab `Delegate Tasks` — columns: `timestamp | sender | subject | delegate_to | status`
- Tab `Approval` — columns: `sender | subject | approval` (write `APPROVE` to send a drafted reply; anything else = draft stays unsent)
- Tab `Execution Log` — columns: `timestamp | workflow | status | summary`
- Tab `Error Log` — columns: `timestamp | workflow | error | node`

## How to test before activating
1. Send yourself a test email from another account (e.g. a job-board alert =
   archive, a recruiter mail = reply, a time-sensitive interview mail =
   urgent). With the workflow inactive, click **Execute Workflow** and check
   the Gmail Trigger picks the test mail up — or use the trigger node's
   "Fetch event" / listen function in n8n.
2. Confirm: Inbox Log has a row with the right category, a Gmail draft exists
   for reply/urgent mail, a Telegram urgent notice arrives for urgent mail,
   and **no reply email was actually sent**.
3. Write `APPROVE` in the Approval tab for that sender/subject, run again,
   and confirm the reply is now sent. Then remove the APPROVE row.
4. Only then toggle **Active**. Start with the trigger polling a label/folder
   if you want to limit which mail it processes — set that in the Gmail
   Trigger node's filters on import.

## Notes / limits
- The classifier is an AI judgement, not a guarantee — check the Inbox Log
  daily at first and correct misclassifications in the Inbox Memory sheet.
- The Approve/Edit step is via the Approval sheet (per the spec's approved
  pattern: an IF node on an approval sheet value). True inline Telegram
  Approve/Edit buttons need a Telegram callback-query webhook; add that as a
  v2 upgrade if wanted — the current flow is explicit and auditable.
- Retry: Gmail/Sheets failures are caught by the Error Trigger → Error Log →
  Telegram path; a failed classification can be re-run by re-executing the
  workflow for that email.
