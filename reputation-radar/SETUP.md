# Reputation Radar — SETUP

## What it does
Daily at 08:00 IST it monitors Reddit, X, Google Reviews and Trustpilot for
mentions of Siva / his projects, Gemini analyses sentiment, and every mention is
logged to Sheets. Negative mentions get a **draft reply queued for approval**
(Reply Queue sheet + Telegram alert) — replies are **never auto-posted**. Every
Monday at 09:00 IST a weekly sentiment trend report is written to Sheets and
emailed to Siva.

## Import
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → Import from File → pick `workflow.json`.
2. The workflow imports with **"No credential set"** on every node that needs one
   (no credential IDs are stored in the file — this is deliberate). Click each node
   and select/create the credential listed below. Then save.

## Credentials to connect (pick on import — nothing is pre-filled)
| Node(s) | Credential needed |
|---|---|
| Log Mention, Queue Draft Reply, Read Mentions (Week), Write Weekly Report, Log Error to Sheet, Log Execution | Google Sheets account (OAuth2) |
| Gemini Chat Model | Google Gemini API key (free tier) |
| Telegram: Negative Mention, Notify Siva (Error) | Telegram account (bot token + chat ID) |
| Email Weekly Report | Gmail account (OAuth2) |

## Resources to create
- **Google Sheet** (copy its ID into `YOUR_GOOGLE_SHEET_ID`):
  - `Mentions` — columns: Source, Title, URL, Sentiment, Score, Topic, Summary, Found At.
  - `Reply Queue` — columns: Source, Title, URL, Mention Date, Draft Reply, Status, Queued At. (Set Status to "approved" when Siva approves a draft; he posts it himself.)
  - `Weekly Report` — columns: Week Start, Week End, Total Mentions, Positive, Neutral, Negative, By Source, Report.
  - `Execution Log` — columns: At, Workflow, Status, Detail.
  - `Errors` — columns: At, Error, Node, Workflow.
- **Monitored keywords**: put the names/terms to watch in `YOUR_KEYWORDS` in the Monitor Sources node (e.g. `Sivasai Boggu, AI SDR Agent, HireMind AI`).
- **Sources**: Reddit search RSS works as-is (public feed). Replace the placeholders with real URLs/endpoints:
  - `YOUR_TRUSTPILOT_COMPANY_URL` — the company's Trustpilot page.
  - `YOUR_GOOGLE_REVIEWS_URL` — Google Reviews/Places page (needs a Google Places API key for reliable access — see the node's note).
  - `YOUR_X_SEARCH_API_URL` — X has no free public feed; replace with Siva's own X API search endpoint/access when available. Until then that source yields no mentions (by design — never invented).
- **Telegram bot**: create via @BotFather, get the bot token, and your chat ID (`YOUR_TELEGRAM_CHAT_ID`).

## Values to fill in (marked `YOUR_*` in the workflow)
- `YOUR_GOOGLE_SHEET_ID`, `YOUR_KEYWORDS`, `YOUR_TRUSTPILOT_COMPANY_URL`, `YOUR_GOOGLE_REVIEWS_URL`, `YOUR_X_SEARCH_API_URL`, `YOUR_TELEGRAM_CHAT_ID`.

## Test before activating
1. Fill in keywords and the Sheet ID. Leave the X/Trustpilot/Google placeholders if not ready — Reddit alone is enough for a first test.
2. Click **Execute Workflow** with the Manual Trigger.
3. Check: mentions land in the Mentions sheet with sentiment; a negative mention (if any) creates a Reply Queue row + Telegram alert and nothing is posted anywhere.
4. To test the weekly report, run the Weekly schedule branch manually (execute from the "Read Mentions (Week)" node) and check the Weekly Report sheet + email.
5. Then activate both schedules.

## Notes
- Sentiment is judged only from the actual mention text. A source that fails yields zero mentions, never fake ones.
- Retry: HTTP, Sheets, Gmail and Telegram nodes have retry-on-fail (3 tries, 5 s apart); HTTP nodes use a 15 s timeout.
- Errors go to the Errors sheet + a Telegram alert to Siva.

## Notes / limits
- **Approval-gate exception (documented):** this workflow only queues draft replies in a sheet and alerts Siva; the weekly report email goes to Siva himself. No reply is ever sent automatically. No approval gate is needed because nothing is sent to anyone else.
