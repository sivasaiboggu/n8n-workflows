# Job Hunter Autopilot — SETUP

## What it does
Daily at 09:15 IST it checks 8 job boards (Wellfound, Built In, Shine, Jobfound,
Himalayas, Cutshort, HiringCafe, LinkedIn Jobs — **never Internshala**), dedupes
against the Applied Log sheet, scores each new role 0-100 against Siva's real CV
with Gemini, tailors resume bullets + a cover letter for scores >= 60, saves the
pack to Google Drive, and sends Siva a Telegram message with **Apply / Skip**
buttons. Tapping Apply only logs the decision and returns the apply link — the
workflow never submits an application itself.

## Import
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → Import from File → pick `workflow.json`.
2. The workflow imports with **"No credential set"** on every node that needs one
   (no credential IDs are stored in the file — this is deliberate). Click each node
   and select/create the credential listed below. Then save.

## Credentials to connect (pick on import — nothing is pre-filled)
| Node(s) | Credential needed |
|---|---|
| Read Applied Log, Log Tailored Application, Log Skipped Role, Log Apply Decision, Log Skip Decision, Log Error to Sheet, Log Execution | Google Sheets account (OAuth2) |
| Gemini Chat Model | Google Gemini API key (free tier) |
| Save Pack to Drive | Google Drive account (OAuth2) |
| Telegram: Apply or Skip?, Telegram: Apply Link, Telegram Trigger (Buttons), Notify Siva (Error) | Telegram account (bot token + chat ID) |

## Resources to create
- **Google Sheet** (copy its ID into `YOUR_GOOGLE_SHEET_ID`):
  - `Applied Log` — columns: Company, Role, Board, Applied At, Status. (Pre-fill with roles already applied to, so they are deduped.)
  - `Tailored Applications` — columns: Board, Company, Role, Score, Fit Reason, Listing URL, Drive File, Logged At.
  - `Decisions` — columns: Company, Role, Score, Decision, Decided At.
  - `Execution Log` — columns: At, Workflow, Status, Detail.
  - `Errors` — columns: At, Error, Node, Workflow.
- **Google Drive folder** for tailored packs — copy its ID into `YOUR_DRIVE_FOLDER_ID`.
- **Telegram bot**: create via @BotFather, get the bot token, and your chat ID (`YOUR_TELEGRAM_CHAT_ID`).

## Values to fill in (marked `YOUR_*` in the workflow)
- `YOUR_GOOGLE_SHEET_ID`, `YOUR_DRIVE_FOLDER_ID`, `YOUR_TELEGRAM_CHAT_ID`.

## Test before activating
1. Click **Execute Workflow** with the Manual Trigger.
2. Check: new roles appear in Tailored Applications with scores, packs land in Drive, and a Telegram message with Apply/Skip buttons arrives.
3. Tap Skip on one message — check the Decisions sheet logs it. Tap Apply on another — check you get the apply link back and the decision is logged.
4. Then activate the 09:15 schedule.

## Notes
- Siva's real profile is baked into the agent prompt: BTech CS IIIT Kottayam 2023-2027, CGPA 7.73, LangGraph/LangChain/RAG/FastAPI, Whispry ₹20,000/month, projects AI SDR / Voice Receptionist / HireMind / SecureFlow / AuditMind, sivasaiboggu@gmail.com, +91 9573719115, Gooty, available immediately, paid roles only.
- Retry: HTTP and Sheets/Drive/Telegram nodes have retry-on-fail (3 tries, 5 s apart); HTTP nodes also use a 10-20 s timeout. Boards that block scraping error out and are logged — the run continues with the next board.
- Errors go to the Errors sheet + a Telegram alert to Siva.
