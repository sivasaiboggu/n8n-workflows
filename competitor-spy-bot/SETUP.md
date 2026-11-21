# Competitor Spy Bot — Siva — SETUP

## What it does
Every Monday 09:00 IST: reads the competitor list from Google Sheets, fetches each
competitor's pricing, blog and jobs pages (public HTTP), diffs the page hash against
the last snapshot, Gemini summarises what changed and why it matters, then posts a
Slack digest — only after human approval in Slack. Nothing is posted publicly
without approval. No invented prices: Gemini only reports what is in the fetched text.

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
| Get Competitor List, Save Snapshot, Log Error to Sheet, Log Execution | Google Sheets (OAuth2) — Siva's Google account |
| Send Approval Digest, Send Monday Digest, Notify on Error (Slack) | Slack API |
| Gemini Chat Model | Google Gemini (PaLM) API (Gemini free tier) |

## Sheets / resources to create
Create one Google Sheet (any name, e.g. `Competitor Spy — Siva`) and replace
`YOUR_COMPETITOR_SHEET_ID` in the Sheet nodes with its ID, and
`YOUR_EXECUTION_LOG_SHEET_ID` / `YOUR_ERROR_LOG_SHEET_ID` with the IDs of your
execution-log and error-log sheets (they may be tabs in one sheet or separate sheets).
- **Competitors** tab — columns: `competitor_name`, `pricing_url`, `blog_url`, `jobs_url`, `last_hash`
  (fill one row per competitor; `last_hash` may start blank).
- **Snapshots** tab — columns: `week`, `competitor`, `new_hash`, `changed`, `summary`
  (written by Save Snapshot; this tab IS the memory layer — the Diff node compares
  the new hash with `last_hash`).
- **Log** tab (execution-log sheet) — columns: `timestamp`, `workflow`, `status`, `detail`.
- **Errors** tab (error-log sheet) — columns: `timestamp`, `workflow`, `node`, `error`, `execution_id`.
- Slack: a channel — replace `YOUR_SLACK_CHANNEL_ID` with its channel ID.

## Test (before activating)
1. Add one test competitor row (a site you control / a public page) to Competitors.
2. Use **Execute Workflow** (manual run) — the Schedule Trigger fires on manual
   execution. Check: Fetch nodes return 200, Diff reports `changed`, Gemini writes
   a summary, Save Snapshot appends a row.
3. Reply APPROVE to the Slack approval message, wait for the Wait node window,
   and confirm the Monday digest posts. Then reply REJECT on a second run and
   confirm nothing is posted and Log Skipped is recorded.
4. Run it a second time with no site change and confirm the "no change" branch
   logs instead of posting.
5. Only then Activate (Mondays 09:00, cron `0 9 * * 1`).

## Notes / limits
- HTTP Request nodes use timeout 10,000 ms and JSON response format; n8n workflow
  Settings retry covers retries where the node type has no built-in retry.
- Split Competitors processes one competitor per batch (batch size 1); add a
  loop-back if you want it to iterate automatically within one run.
- The Wait node waits up to 4 hours for approval; in production swap it for a
  Form Trigger / Slack interactive approval if you prefer a click instead of a reply.
- Error handling: Error Trigger → Log Error to Sheet → Notify on Error (Slack).
