# AI Podcast Factory — SETUP

## What it does
Daily at 06:00 IST it reads a news RSS feed, Gemini writes a two-host script
(Siva + co-host) from the real headlines only, text-to-speech turns it into
audio, the audio is saved to Google Drive, and Siva gets a Telegram message with
**Publish / Decline** buttons. Nothing is published or posted on social until
Siva taps Publish — then the episode goes to the podcast host and the link is
posted on social.

## Import
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → Import from File → pick `workflow.json`.
2. The workflow imports with **"No credential set"** on every node that needs one
   (no credential IDs are stored in the file — this is deliberate). Click each node
   and select/create the credential listed below. Then save.

## Credentials to connect (pick on import — nothing is pre-filled)
| Node(s) | Credential needed |
|---|---|
| Queue Episode for Approval, Log Published Episode, Log Declined Episode, Log Error to Sheet, Log Execution | Google Sheets account (OAuth2) |
| Gemini Chat Model | Google Gemini API key (free tier) |
| Save Episode Audio to Drive | Google Drive account (OAuth2) |
| Telegram: Approve Episode?, Telegram: Episode Live, Telegram Trigger (Buttons), Notify Siva (Error) | Telegram account (bot token + chat ID) |
| Text-to-Speech (ElevenLabs) | **No n8n credential — uses the `xi-api-key` HTTP header.** Put Siva's own ElevenLabs API key in `YOUR_ELEVENLABS_API_KEY`. The placeholder key is not real. |

## Resources to create
- **Google Sheet** (copy its ID into `YOUR_GOOGLE_SHEET_ID`):
  - `Episodes` — columns: Date, Title, Status, Published At.
  - `Pending Episodes` — columns: Date, Title, Topics, Drive Audio, Status, Queued At.
  - `Execution Log` — columns: At, Workflow, Status, Detail.
  - `Errors` — columns: At, Error, Node, Workflow.
- **RSS feed** for the daily digest — put a real feed URL in `YOUR_RSS_FEED_URL` (e.g. an AI/tech news RSS feed).
- **ElevenLabs**: Siva's own API key (`YOUR_ELEVENLABS_API_KEY`) and voice ID (`YOUR_VOICE_ID`) from elevenlabs.io. TTS is a placeholder credential — the workflow cannot speak without Siva's own key.
- **Google Drive folder** for episode audio — copy its ID into `YOUR_DRIVE_FOLDER_ID`.
- **Podcast host**: replace `YOUR_PODCAST_HOST_UPLOAD_URL` with the real upload endpoint of Siva's host (e.g. Spotify for Podcasters / Anchor, Buzzsprout). The Drive audio file is the episode file to upload.
- **Social posting**: replace `YOUR_SOCIAL_POST_API_URL` with the real posting endpoint (e.g. Buffer or X API) once Siva connects it.
- **Telegram bot**: create via @BotFather, get the bot token, and your chat ID (`YOUR_TELEGRAM_CHAT_ID`).

## Values to fill in (marked `YOUR_*` in the workflow)
- `YOUR_GOOGLE_SHEET_ID`, `YOUR_RSS_FEED_URL`, `YOUR_ELEVENLABS_API_KEY`, `YOUR_VOICE_ID`, `YOUR_DRIVE_FOLDER_ID`, `YOUR_PODCAST_HOST_UPLOAD_URL`, `YOUR_SOCIAL_POST_API_URL`, `YOUR_TELEGRAM_CHAT_ID`.

## Test before activating
1. Fill in the RSS feed URL, ElevenLabs key + voice ID, Sheet ID and Drive folder ID. Leave the podcast-host and social URLs as placeholders for the first test (they only run after you tap Publish).
2. Click **Execute Workflow** with the Manual Trigger.
3. Check: a script is written from real headlines, audio lands in Drive, the episode is queued in Pending Episodes, and a Telegram approval message arrives.
4. Listen to the audio in Drive. Tap Decline first to check the decline path logs to the Episodes sheet. Re-run and tap Publish only when the host/social URLs are real.
5. Then activate the 06:00 schedule.

## Notes
- A broken or empty script never reaches TTS or publishing (Script OK? gate).
- Long scripts may exceed TTS limits — split the script into parts first if the TTS call fails.
- Retry: HTTP, RSS, Sheets, Drive and Telegram nodes have retry-on-fail (3 tries, 5 s apart); the TTS call has a 120 s timeout.
- Errors go to the Errors sheet + a Telegram alert to Siva.
