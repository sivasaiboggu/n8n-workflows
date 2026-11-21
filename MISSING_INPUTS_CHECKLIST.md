# n8n — Missing-Inputs Checklist (definitive, 21 Nov 2026 ~22:40 IST)

Instance: https://sivasai-boggu.app.n8n.cloud — account sivasaiboggu@gmail.com, trial ends ~4 Dec 2026.
Sources: workspace JSON scan of all 30 workflows (682 nodes), every workflow's SETUP.md, and `WORKABILITY_AUDIT.md` (tab lists cross-checked against SETUP.md and confirmed).

## Status snapshot

| | Count |
|---|---|
| Workflows in the instance | 30 |
| Live and Published | 1 — Multi-agent Daily Brief (both logging gaps fixed in place 21 Nov ~22:13 IST; dedupe verified; 0 failed executions) |
| Inactive | 29 |
| Fully runnable right now | **0 of 29** — every one needs at least a Google Sheet that does not exist yet |
| Workflows completable with what already exists | **None.** Nothing below can be fixed without Siva supplying something. |

**What already exists and is wired in the live instance:** Gmail ("Gmail account"), Google Sheets
("Google Sheets account"), Gemini ("Google Gemini(PaLM) Api account") — 193 nodes wired and saved
clean on 21 Nov. **Unknown, cannot verify from here:** whether Google Drive, Google Docs and
Google Calendar credentials exist — check on the n8n Credentials page before doing step 4.

**Rules for this whole file:** never invent a sheet ID, token, key, chat ID or URL. Never
re-import a workspace JSON over a live workflow (it has corrupted workflows before; the live
Daily Brief is already ahead of its workspace copy — it has the real sheet ID and the
"Expand Brief Stories for Log" node the workspace copy lacks). Never activate a workflow until
its row below is fully supplied.

---

## Step 1 — Google Sheets: one per workflow (needed by 29/29 — do this first)

For each workflow below, create **one Google Sheet** in the Google account
sivasaiboggu@gmail.com (the account the Sheets credential signs in as), add the tabs listed
for that workflow, and put the exact column headers in row 1 of each tab (the headers are in
that workflow's SETUP.md — most tabs only need the headers; "Execution Log" and "Error Log"
are the same everywhere: `At | Workflow | Result/Status | Detail` and `At | Workflow | Error | Node`).

How to get a sheet's ID: open the sheet — the ID is the long string in the URL between
`/d/` and `/edit`. Then, in n8n, open the workflow and paste that ID into every Google
Sheets node where it says `YOUR_GOOGLE_SHEET_ID` (or the named variant). One sheet per
workflow — do **not** reuse the Daily Brief's sheet (`1Jc7sKto0HXQGoZNc8xCx-OkzCGyx6-nrIhlH5dDR-Yg`);
its rows feed the brief's dedupe and other workflows writing into it would corrupt it.

Fastest way to do all 29: create the sheets in one sitting, then paste the 29 IDs into a
single message/table (workflow name → sheet ID) and the IDs can be pasted into n8n in one
browser session. Tab lists per workflow are in the table in "Per-workflow checklist" below.

## Step 2 — Telegram bot + your chat ID (needed by 26/29 — do this second)

1. In Telegram, message **@BotFather**, send `/newbot`, follow the prompts, give it a name.
   BotFather replies with a bot token (looks like `123456789:AAE...`).
2. In n8n: **Credentials → New → Telegram → Access Token** → paste the token. Name it
   "Telegram bot".
3. Get your chat ID: message **@userinfobot** (or @getmyid_bot) in Telegram — it replies
   with your numeric chat ID.
4. In n8n, open each workflow below and paste that chat ID into every Telegram node where
   it says `YOUR_TELEGRAM_CHAT_ID`. (Telegram nodes are also how 26 workflows ask Siva for
   approvals and send error alerts — without the bot, approvals can never be answered.)

## Step 3 — Slack workspace (needed by 6 workflows)

In n8n: **Credentials → New → Slack** → connect the workspace (OAuth, sign in as Siva) and
note the channel ID of each channel the workflows post to (right-click a channel in Slack →
Copy Link → the ID is the last part, or use "View channel details"). Paste the channel ID
where nodes say `YOUR_SLACK_CHANNEL_ID`. Workflows: Competitor Spy Bot, Digital Twin Slack
Bot, Personal Finance Autopilot, Self-Healing Workflow Monitor, Smart Expense Policing,
Auto-Documentation Bot. Digital Twin also needs Siva's own Slack user ID
(`YOUR_SIVA_SLACK_USER_ID` — in Slack, click your profile → Copy member ID).

## Step 4 — Google Drive / Docs / Calendar credentials (needed by 7 workflows)

In n8n: **Credentials → New → Google Drive OAuth2 / Google Docs OAuth2 / Google Calendar
OAuth2** → sign in as sivasaiboggu@gmail.com for each (same Google account, three separate
credentials — they are NOT covered by the existing Sheets/Gmail credentials). Workflows:
AI Podcast Factory + Job Hunter Autopilot (Drive), Autonomous Research Agent + Dream-Team
(Docs), Meeting Autopilot + Smart Home + Travel Deal Hunter + Voice-Note-to-Everything
(Calendar). Meeting Autopilot and Smart Home also need a calendar ID
(`YOUR_CALENDAR_ID` — in Google Calendar → Settings → the calendar's "Calendar ID").

## Step 5 — n8n API key (needed by 2 workflows)

In n8n Cloud: **Settings → n8n API → Create API key**. Copy the key, then in n8n create a
credential for it, and set the host to `https://sivasai-boggu.app.n8n.cloud` where nodes say
`YOUR_N8N_HOST`. Workflows: Daily Automation Self-Report (reads executions) and
Self-Healing Workflow Monitor (retries failed executions). This key also lets agents
verify/edit workflows via the API without the browser in future.

## Step 6 — One-off keys and services (one workflow each)

| Item | How Siva supplies it | Workflows |
|---|---|---|
| Apollo API key (free tier) | apollo.io → Settings → API keys → copy key. For Lead Sniper: create an **httpHeaderAuth** credential in n8n with it (the Enrich Lead node uses header auth). For Job Search Autopilot: paste it into the Apollo Email Lookup node (`YOUR_APOLLO_API_KEY`). | Lead Sniper, Job Search Autopilot |
| ElevenLabs API key + voice ID | elevenlabs.io → Profile → API key; pick a voice → its voice ID. Paste key as the `xi-api-key` header and voice ID into the TTS node URL. | AI Podcast Factory |
| Podcast host upload URL + RSS feed URL + social post API URL | The URL of the host Siva publishes to, the show's RSS feed URL, and the API endpoint that posts new episodes. | AI Podcast Factory |
| GitHub personal access token | github.com → Settings → Developer settings → Personal access tokens → token with `repo` scope. Create a GitHub credential in n8n with it; also give the username + repo name (`YOUR_GITHUB_USERNAME`, `YOUR_REPO`). | Auto-Documentation Bot |
| Notion integration + parent page / database | notion.so → Settings → Integrations → New integration → copy its token; share the parent page (Auto-Documentation) or database (Voice-Note) with the integration; paste the page/database ID (`YOUR_NOTION_PARENT_PAGE_ID`, `YOUR_NOTION_DATABASE_ID`). | Auto-Documentation Bot, Voice-Note-to-Everything |
| Shopify store + eBay token + Etsy key | Shopify: store URL + access token (`YOUR_SHOP_ID`); eBay: developer program token; Etsy: API key. | Image-to-Product-Listing |
| Linear API token | linear.app → Settings → API → personal API key. The Create Ticket node currently has **no Authorization header** — the key must be added as an `Authorization: <key>` header on that node. | AI Customer Support Triage |
| Fireflies API key | fireflies.ai → Settings → API key. | Meeting Autopilot |
| OCR.space API key + QuickBooks OAuth + company ID | ocr.space → free API key; QuickBooks: connect OAuth in n8n and note the company/realm ID (`YOUR_COMPANY_ID`). | Invoice and Receipt Vacuum |
| Beehiiv API key + publication ID | beehiiv.com → Settings → API → key; publication ID from the dashboard (`YOUR_PUBLICATION_ID`). | News-to-Newsletter Pipeline |
| Buffer API token + profile ID | buffer.com → API token (create an httpHeaderAuth credential in n8n); profile ID from the Buffer dashboard (`YOUR_BUFFER_PROFILE_ID`). Transcript + image-gen API URLs also needed (whichever services Siva uses). | Content Repurposing Machine |
| Kiwi Tequila API key | tequila.kiwi.com → sign up → API key. Plus home airport, destination and dates (`YOUR_HOME_AIRPORT`, `YOUR_DESTINATION`, `YOUR_DATE_FROM`, `YOUR_DATE_TO`). | Travel Deal Hunter |
| Home Assistant URL + token | Home Assistant → Profile → Long-Lived Access Tokens → create one; paste the instance URL (`YOUR_HOME_ASSISTANT_URL`) and the token (httpHeaderAuth credential). | Smart Home + Context Agent |
| SMS provider API + customer phone | Whichever SMS provider Siva uses (API base URL + key as httpHeaderAuth) and the customer phone number format the node expects. | Event-Based Customer Win-Back |
| WhatsApp Business number | Meta WhatsApp Business: phone number ID (`YOUR_WHATSAPP_PHONE_NUMBER_ID`) + access token (WhatsApp credential in n8n). | Habit Coach via WhatsApp |
| Speech-to-text endpoint + Todoist API token | The STT service URL (`YOUR_STT_ENDPOINT`) and a Todoist API token (todoist.com → Settings → Integrations). | Voice-Note-to-Everything |

---

## Per-workflow checklist (29 rows)

Order within each row is the order to supply things. "Sheet" = Step 1, "Telegram" = Step 2,
"Slack" = Step 3, "Drive/Docs/Cal" = Step 4, "n8n API" = Step 5, the rest = Step 6.

| # | Workflow | Sheet + tabs | Missing inputs, in order | Notes |
|---|---|---|---|---|
| 1 | AI Customer Support Triage | 1 sheet: Approvals, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → Linear API token (add as `Authorization` header on Create Ticket node) | Webhook path `support-triage` is already set. RAG/vector store is a placeholder by design (returns empty context) — a future upgrade, not a blocker. |
| 2 | AI Inbox Chief of Staff | 1 sheet: Inbox Log, Delegate Tasks, Approval, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → delegate name (`YOUR_DELEGATE_NAME` — who mail gets delegated to) | Gmail trigger + 4 Gmail nodes already wired. |
| 3 | AI Podcast Factory | 1 sheet: Pending Episodes, Episodes, Execution Log, Errors | Sheet ID → Telegram bot + chat ID → Drive credential + Drive folder ID → ElevenLabs key + voice ID → podcast host upload URL → RSS feed URL → social post API URL | |
| 4 | Auto-Documentation Bot | 1 sheet: Doc Approvals, Declined, Execution Log, Errors | Sheet ID → Telegram bot + chat ID → Slack workspace + channel → GitHub PAT + username + repo → Notion integration + parent page ID | GitHub trigger fires on pushes to the chosen repo. |
| 5 | Autonomous Research Agent | 1 sheet: Approvals, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → Google Docs credential | Webhook path `research-agent` is already set. |
| 6 | Competitor Spy Bot | 3 sheet IDs (or 1 sheet, 3 IDs): Competitors, Snapshots, Log, Errors | Competitor sheet ID + execution/error log sheet IDs → Slack workspace + channel ID | No Telegram in this one — Slack only. |
| 7 | Content Repurposing Machine | 3 sheet IDs (or 1 sheet): Drafts, Log, Errors | Content/execution/error sheet IDs → Telegram bot + chat ID → transcript API URL (+ its header-auth credential) → image-gen API URL (+ its header-auth credential) → Buffer token (header-auth credential) + profile ID | Webhook path `repurpose` is already set. |
| 8 | Crypto/Stock Whale Watcher | 1 sheet: Execution Log, Error Log | Sheet ID → Telegram bot + chat ID | Simplest after the sheet + bot: only 2 inputs. |
| 9 | Daily Automation Self-Report | 1 sheet: Daily Log, Weekly Trends, Control, Error Log | Automation-log sheet ID → Telegram bot + chat ID → n8n API key + host (`YOUR_N8N_HOST` = `https://sivasai-boggu.app.n8n.cloud`) | |
| 10 | Digital Twin Slack Bot | 1 sheet: Pending Approvals, Execution Log, Errors | Sheet ID → Slack workspace + channel ID + Siva's Slack user ID → Telegram bot + chat ID | "Memory Lookup (Placeholder)" code node is a functional placeholder — the bot answers without long-term memory until a memory layer is wired in (future upgrade, not a blocker). |
| 11 | Dream-Team Multi-Agent Pipeline | 1 sheet: Pipeline Drafts, Approval, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → Google Docs credential → webhook path value (choose one, e.g. `dream-team`, and put it where it says `YOUR_DREAMTEAM_WEBHOOK_PATH`) | |
| 12 | Event-Based Customer Win-Back | 1 sheet: Customer Signals, Win-Back Drafts, Approval, Replies, Win-Back Outcomes, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → SMS provider API (+ header-auth credential) + customer phone → webhook path value (choose one, e.g. `win-back`, for `YOUR_WINBACK_WEBHOOK_PATH`) | |
| 13 | Habit Coach via WhatsApp | 1 sheet: Habits, Check-ins, Execution Log, Errors | Sheet ID → WhatsApp Business number ID + token → Telegram bot + chat ID | |
| 14 | Image-to-Product-Listing | 1 sheet: Listing Approvals, Declined, Execution Log, Errors | Sheet ID → Telegram bot + chat ID → Shopify store + token → eBay token → Etsy key | Webhook path `product-photo` is already set. |
| 15 | Invoice and Receipt Vacuum | 1 sheet: Receipts, Duplicates, Approvals, Accounting Postings, Declined, Execution Log, Errors | Sheet ID → Telegram bot + chat ID → OCR.space key → QuickBooks OAuth + company ID | Webhook path `invoice-upload` is already set. |
| 16 | Job Hunter Autopilot | 1 sheet: Applied Log, Tailored Applications, Decisions, Execution Log, Errors | Sheet ID → Telegram bot + chat ID → Drive credential + Drive folder ID | |
| 17 | Lead Sniper | 3 sheet IDs (or 1 sheet): Sequence, Log, Errors | Lead-sequence/execution/error sheet IDs → Telegram bot + chat ID → Apollo key as an httpHeaderAuth credential | Webhook path `lead-sniper` is already set. |
| 18 | Meeting Autopilot | 1 sheet: Approvals, Tasks, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → Calendar credential + calendar ID → Fireflies API key | |
| 19 | News-to-Newsletter Pipeline | 1 sheet: Newsletter Log, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → Beehiiv key + publication ID | |
| 20 | Personal Finance Autopilot | 3 sheet IDs (or 1 sheet): Transactions, Weekly Summaries, Log, Errors | Finance/execution/error sheet IDs → Slack workspace + channel ID → Telegram bot + chat ID | Webhook path `finance-transaction` is already set. |
| 21 | Price Drop Sniper | 1 sheet: Watchlist, Price Log, Alerts, Execution Log, Errors | Sheet ID → Telegram bot + chat ID | Simplest after the sheet + bot: only 2 inputs. |
| 22 | Reputation Radar | 1 sheet: Mentions, Reply Queue, Execution Log, Errors, Weekly Report | Sheet ID → Telegram bot + chat ID → Google Reviews URL → Trustpilot company URL → keywords → X search API URL | |
| 23 | Self-Healing Workflow Monitor | 2 sheet IDs (or 1 sheet): Log, Diagnoses, Errors | Execution/error sheet IDs → Slack workspace + channel ID → n8n API key + host | |
| 24 | Smart Expense Policing | 1 sheet: Policy, Approved, Justifications, Reviewed, Escalations, Execution Log, Errors | Sheet ID → Slack workspace + channel ID → Telegram bot + chat ID | Webhook path `card-transaction` is already set. |
| 25 | Smart Home + Context Agent | 1 sheet: Approvals, Home Log, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → Calendar credential + calendar ID → Home Assistant URL + token | |
| 26 | Social Listening Opportunity Finder | 1 sheet: Skipped, Approval Queue, Approved Replies, Declined, Execution Log, Errors | Sheet ID → Telegram bot + chat ID | Simplest after the sheet + bot: only 2 inputs. |
| 27 | Travel Deal Hunter | 1 sheet: Travel Watchlist, Draft Itineraries, Approval, Execution Log, Error Log | Sheet ID → Telegram bot + chat ID → Calendar credential → Kiwi Tequila key → home airport, destination, dates | |
| 28 | Voice-Note-to-Everything | 3 sheet IDs (or 1 sheet): Log, Errors | Voice-notes/execution/error sheet IDs → Telegram bot + chat ID → Calendar credential → STT endpoint (+ header-auth credential if it needs one) → Todoist token → Notion API key + database ID | "Transcribe Voice (STT — placeholder)" is a stub node: it cannot transcribe until a real STT endpoint is supplied. |
| 29 | Job Search Autopilot | 1 sheet: Applied Log, Sends, Needs Verification, Replies, Follow-ups, Errors | Sheet ID → Drive credential → resume file ID → Apollo API key | **Closest to done.** See below. |

## Closest to done: Job Search Autopilot

Needs only 4 things, in order:
1. **Google Sheet** with the 6 tabs above (headers in its SETUP.md) + paste the sheet ID
   into the 7 Sheets nodes. Seed the **Applied Log** tab with the companies Siva already
   applied to so the dedupe never re-applies (iion, Discover Dollar, Alchemyst, Drivetrain,
   Meraki Labs, PinSec.Ai, Honeywell, Fast Code AI, Offshore Minds, Web Neural, insiderOne,
   Priberry, DBHelm, SkillsCapital, Cehpoint, Stackular, and the rest).
2. **Google Drive credential** in n8n (Step 4) — only used to download the resume PDF.
3. **Resume file ID** — upload `Sivasai_Boggu_Resume.pdf` to Google Drive, open it, paste
   the file ID from its URL into the Download Resume PDF node (`YOUR_RESUME_FILE_ID`).
4. **Apollo API key** — paste into the Apollo Email Lookup node (`YOUR_APOLLO_API_KEY`).
   Without it the workflow still runs safely: every contact goes to Needs Verification and
   nothing is mailed.

Then test with a manual run before turning on the schedule (SETUP.md step 5). Note: no
Apollo key exists in this environment and no Drive credential has been confirmed — both
still need Siva.

## What "done" looks like per workflow

A workflow counts as complete only when: its sheet exists with the right tabs, every
credential it needs is connected in n8n, every `YOUR_…` placeholder in its JSON is replaced
with a real value, it saves cleanly, and a manual test run succeeds. Do not activate
anything on a schedule until its test run passes.

## What was verified for this file (21 Nov ~22:40 IST)

- All 30 workspace JSONs re-scanned: 682 nodes. Live instance has 683 (the extra node is
  "Expand Brief Stories for Log", added live to the Daily Brief at ~22:13 IST and not in
  the workspace copy — the workspace copy of the Daily Brief is therefore stale: it still
  shows `YOUR_GOOGLE_SHEET_ID` and lacks the new node. Do not "fix" this by re-importing.)
- Every one of the 29 inactive workflows still carries at least one placeholder sheet ID
  in its JSON — confirmed by scan, not assumed. No workflow is runnable as-is.
- The audit's per-workflow tab lists were cross-checked against the SETUP.md files
  (table-format tab lists match exactly where present).
- GitHub repo `sivasaiboggu/n8n-workflows` HEAD is still `3fe3307c` (verified via the
  GitHub connector) — nothing new pushed since 21 Nov 09:18 IST.
- Instance reachable (HTTP 200 at root); REST API returns 401 without a key — no n8n API
  key exists yet, so no live API verification was possible from here.
- **Not verified from here** (needs the parent agent / browser): the live Workflows list
  (30 total, Daily Brief Published, other 29 inactive, 0 failed executions — last verified
  by a browser task 21 Nov ~22:13 IST) and whether Drive/Docs/Calendar credentials exist
  in the instance.
