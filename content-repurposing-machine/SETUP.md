# Content Repurposing Machine — Siva — SETUP

## What it does
YouTube URL in (webhook `POST /repurpose` or Manual Trigger) → validates it is a
YouTube URL → fetches the transcript → Gemini writes a blog post, an 8-post X
thread, a LinkedIn post and 5 short-form hooks (only claims in the transcript;
no invented quotes/stats) → generates a thumbnail image draft → saves drafts to
Sheets → NOTHING is scheduled or published until Siva approves in Telegram,
then it schedules via Buffer.

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
| Fetch Transcript | Transcript API key — also set `YOUR_TRANSCRIPT_API_URL` (any YouTube-transcript service; no key in the file) |
| Generate Thumbnail Image | Image Generation API key — also set `YOUR_IMAGE_GEN_API_URL` |
| Schedule via Buffer | Buffer API token — also set `YOUR_BUFFER_PROFILE_ID` in the node body |
| Send Approval Request, Notify on Error (Telegram) | Telegram API (chat ID = `YOUR_TELEGRAM_CHAT_ID`) |
| Save Drafts to Sheet, Log Error to Sheet, Log Execution | Google Sheets (OAuth2) |
| Gemini Chat Model | Google Gemini (PaLM) API (Gemini free tier) |

## Sheets / resources to create
- **Drafts** tab (sheet `YOUR_CONTENT_SHEET_ID`) — columns: `timestamp`, `url`, `blog`, `status`
  (this tab IS the memory layer — one row per video; re-check it before re-doing a video).
- **Log** tab (execution-log sheet `YOUR_EXECUTION_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `status`, `detail`.
- **Errors** tab (error-log sheet `YOUR_ERROR_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `node`, `error`, `execution_id`.
- Buffer: connect the X / LinkedIn profiles you want scheduled to; note the profile IDs.
- Webhook: after import, n8n shows the production URL for `POST /repurpose`
  with JSON body `{"url": "https://www.youtube.com/watch?v=..."}`.

## Test (before activating)
1. Use Manual Trigger with test data `{"url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ"}`
   (or any video you own / have rights to repurpose — do not repurpose other
   creators' footage for public posting without permission; see note below).
2. Check: transcript is fetched, Gemini returns blog + X thread + LinkedIn post +
   5 hooks, an image draft is generated, and a Drafts row is saved with status
   "draft — awaiting approval" — and nothing is scheduled yet.
3. Reply APPROVE in Telegram and confirm Buffer receives the scheduled update.
4. Repeat with REJECT and confirm nothing is scheduled and Log Skipped is recorded.
5. Only then Activate.

## Notes / limits
- Copyright: this workflow repurposes transcripts for Siva's own drafts. Do not
  publish re-uploads of other creators' footage; Siva's standing rule is to use
  YouTube Remix, Creative Commons footage, or original commentary-style shorts.
- HTTP Request nodes use timeout 10,000 ms and JSON response format; workflow
  Settings retry covers retries where the node type has no built-in retry.
- The Wait node waits up to 4 hours for approval; swap for a Form Trigger if
  you prefer a click approval.
- Error handling: Error Trigger → Log Error to Sheet → Notify on Error (Telegram).
