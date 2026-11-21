# Dream-Team Multi-Agent Pipeline — Siva — SETUP

## What it does
Send a topic (webhook POST or Manual Trigger) and four Gemini AI Agent nodes
run in sequence, each passing its output to the next: **Researcher** (facts +
sources, marks anything unverified) → **Writer** (draft from research only, no
invented facts) → **Fact-checker** (checks every claim, flags/corrects) →
**Editor** (polish, facts intact). The final piece is saved as a **draft** in
Google Docs and recorded in Sheets. **Never auto-published. Human approval
before publishing** — writing `APPROVE` in the Approval tab only sends a
"publish it yourself" Telegram note; you publish manually from Docs.

Memory layer: past topics, sources, fact-check flags and editor notes are kept
in the Pipeline Memory sheet (see below) so later runs can avoid repeats and
past mistakes. No live Postgres/vector DB is claimed — the sticky note on the
canvas marks what would be stored there if you add one later.

## Import
**Credentials rule (corrected 20 Nov 2026): this file sets NO credential on any
node** — n8n refuses to save an import that references a null-id credential
(this exact failure hit the daily-brief import). On import each node shows
"No credential set": click it and pick/create the credential listed below.
The same list is in the "Sticky Note — Credentials" node in the workflow.
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → ⋯ → Import from File →
   select `workflow.json` in this folder. Leave inactive until tested.
2. Replace every `YOUR_...` placeholder: `YOUR_GOOGLE_SHEET_ID`,
   `YOUR_TELEGRAM_CHAT_ID`, `YOUR_DREAMTEAM_WEBHOOK_PATH`, `YOUR_TOPIC`
   (only used if no topic is supplied).

## Credentials to connect (pick/create on import — none are in the file)

No credential is set in `workflow.json` (see Import above) — the table below is what to
pick/create for each node; no real key or secret is stored anywhere in this repo.
| Credential | Where |
|---|---|
| Google Gemini(PaLM) Api | Gemini Chat Model (shared by all four agents) |
| Google Docs account | Save Draft to Google Docs |
| Google Sheets account | Save Draft Record to Sheet, Check Approval Sheet, Log Execution, Log Error to Sheet |
| Telegram account | Send Approval Request via Telegram, Notify — Approved to Publish, Notify Error via Telegram |

## Google Sheet to create (id = YOUR_GOOGLE_SHEET_ID)
- Tab `Pipeline Drafts` — columns: `timestamp | topic | status | doc`
- Tab `Pipeline Memory` — columns: `timestamp | topic | sources_found | fact_check_flags | editor_notes` (fill a row after each run so future runs learn from it)
- Tab `Approval` — columns: `topic | approval` (write `APPROVE` to allow the manual-publish notice; anything else = draft only)
- Tab `Execution Log` — columns: `timestamp | workflow | status | summary`
- Tab `Error Log` — columns: `timestamp | workflow | error | node`

## How to test before activating
1. Click **Execute Workflow** with a Manual Trigger run, or POST
   `{"topic": "How federated learning defends against Sybil attacks", "audience": "CS students"}`
   to the webhook URL. (Use a topic you know well so you can judge the output.)
2. Confirm the Google Doc created is a **draft** (title starts "DRAFT —"),
   the Pipeline Drafts row says "draft — awaiting approval", and a Telegram
   approval request arrives. Confirm nothing was published anywhere.
3. Leave Approval blank → the run logs "draft only — not published". Then
   write `APPROVE`, run again, and confirm you only get the manual-publish
   Telegram note.
4. Read the draft critically — the Fact-checker is an AI check, not a guarantee.
   Only then toggle **Active**.

## Notes / limits
- All four agents share one Gemini Chat Model node (gemini-2.0-flash,
  temperature 0.3) via `ai_languageModel` connections.
- Retry: the workflow Settings retry applies to agent calls; a failed agent
  step is caught by the Error Trigger → Error Log → Telegram path, and the
  run can be re-executed from the failed node.
