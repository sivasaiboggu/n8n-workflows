# n8n Workability Audit — 21 Nov 2026

Instance: https://sivasai-boggu.app.n8n.cloud (sivasaiboggu@gmail.com, trial ends ~4 Dec 2026).
30 workflows in the instance: 29 imported (all INACTIVE) + the live Multi-agent Daily Brief — Siva (Published).
**Never re-import, open for editing, or touch the Daily Brief.**

## Re-verification — 21 Nov 2026 ~22:40 IST (Siva asked to "complete all the workflows")

- **Verdict unchanged: 0 of 29 runnable.** All 30 workspace JSONs re-scanned (682 nodes);
  every one of the 29 inactive workflows still carries at least one placeholder sheet ID.
  Nothing can be completed without Siva supplying inputs — no credential, sheet, bot, key
  or URL was invented, and no accounts/bots/sheets/keys were created on his behalf.
- **Daily Brief is the only live workflow and it is complete:** both logging gaps fixed in
  place 21 Nov ~22:13 IST (Log Brief reconnected via "Expand Brief Stories for Log", Read
  Briefed Log restored, dedupe verified suppressing the 5 logged stories, 0 failed
  executions). The workspace copy of the Daily Brief is now **stale** vs live: it still
  shows `YOUR_GOOGLE_SHEET_ID` and lacks the Expand node (live has 683 nodes, workspace
  has 682). Do not "fix" this by re-importing.
- **Definitive per-workflow checklist:** `MISSING_INPUTS_CHECKLIST.md` — one row per
  workflow with its sheet + tabs, its missing inputs in supply order, and the exact steps
  for Siva to supply each item. Closest to done remains **Job Search Autopilot** (sheet +
  Drive credential + resume file ID + Apollo key).
- **GitHub repo** `sivasaiboggu/n8n-workflows` HEAD still `3fe3307c` (verified via connector).
- **Not verified from this session:** the live Workflows list (last verified by a browser
  task 21 Nov ~22:13 IST) and whether Drive/Docs/Calendar credentials exist in the
  instance. Instance reachable (HTTP 200); REST API returns 401 without a key — no n8n
  API key exists yet.

## Honest verdict up front

**0 of 29 imported workflows are fully workable right now, and wiring the existing
credentials alone cannot change that.** Every single workflow (29/29) references a
Google Sheet by a placeholder ID (`YOUR_GOOGLE_SHEET_ID` or a named variant) that does
not exist yet, and 26/29 also need a Telegram bot + chat ID that does not exist yet.
No values may be invented — the sheets, the bot, and the other accounts below are all
things only Siva can supply or create.

What the browser wiring step CAN do: clear the credential warnings on 193 nodes
(145 Google Sheets + 30 Gemini + 18 Gmail) by selecting the three credentials that
already exist. After that, each workflow will **save cleanly** and its Google/Gmail/AI
parts will be ready — but it will still not RUN until its sheet exists and its
Telegram/Slack/etc. account is connected. Say so plainly to Siva; do not call a
workflow "workable" until its blockers are cleared.

## Credential needs across all 29 workflows (node counts)

| Credential type | Nodes needing it | Exists in instance? |
|---|---|---|
| googleSheetsOAuth2 (Google Sheets) | 145 | YES — "Google Sheets account" |
| googlePalmApi (Gemini) | 30 | YES — "Google Gemini(PaLM) Api account" (auto-selected on many imports already) |
| gmailOAuth2 (Gmail) | 18 | YES — "Gmail account" |
| telegramApi (Telegram bot) | 58 | NO — needs Siva: create bot via @BotFather, token + his chat ID |
| slackApi (Slack) | 13 | NO — needs Siva: Slack workspace + channel |
| googleDriveOAuth2 (Drive) | 3 | UNKNOWN — verify on Credentials page; if missing, Siva connects his Google account in n8n (same Google account, separate credential) |
| googleCalendarOAuth2 (Calendar) | 4 | UNKNOWN — same as Drive |
| googleDocsOAuth2 (Docs) | 2 | UNKNOWN — same as Drive |
| whatsappApi (WhatsApp Business) | 3 | NO — needs Siva: WhatsApp Business number / phone number ID |
| githubApi (GitHub) | 2 | NO — needs Siva: GitHub personal access token |
| notionApi (Notion) | 1 | NO — needs Siva: Notion integration + parent page |
| shopifyApi (Shopify) | 1 | NO — needs Siva: Shopify store access |

All Gemini nodes have a model set (gemini-2.0-flash, via `model` or `modelName`).
Note: the daily brief found Gemini 2.5 Flash 404s in this instance and uses 3.5 Flash —
whether `gemini-2.0-flash` still resolves at runtime is UNPROVEN; if a Gemini node
errors on the model at test time, flag it (fix would be switching the model, not the key).

## Per-workflow status

Legend: WIRE = nodes the browser task can wire with existing creds (Sheets/Gemini/Gmail,
+ Drive/Docs/Calendar IF they exist). BLOCKED = needs something only Siva can supply.
Every workflow needs a Google Sheet created (or an existing sheet's ID pasted in) —
the tabs each sheet needs are listed; one sheet per workflow unless noted.

| # | Workflow (name in instance) | WIRE (nodes) | BLOCKED BY (needs Siva) | Sheet tabs needed |
|---|---|---|---|---|
| 1 | AI Customer Support Triage — Siva | Sheets 3, Gemini 1, Gmail 1 | Telegram bot+chat (3 nodes); sheet ID; Linear API token (Create Ticket HTTP node) | Approvals, Execution Log, Error Log |
| 2 | AI Inbox Chief of Staff — Siva | Sheets 6, Gemini 1, Gmail 4 | Telegram (2); sheet ID; delegate name | Inbox Log, Delegate Tasks, Approval, Execution Log, Error Log |
| 3 | AI Podcast Factory — Siva | Sheets 5, Gemini 1 | Telegram (4); Drive cred; sheet ID; ElevenLabs API key + voice ID; podcast host upload URL; social post API URL; RSS feed URL | Pending Episodes, Episodes, Execution Log, Errors |
| 4 | Auto-Documentation Bot — Siva | Sheets 5, Gemini 1 | Telegram (2); Slack (1); GitHub PAT + username/repo (2); Notion integration + parent page (1); sheet ID | Doc Approvals, Declined, Execution Log, Errors |
| 5 | Autonomous Research Agent — Siva | Sheets 3, Gemini 1, Gmail 1 | Telegram (2); Docs cred; sheet ID | Approvals, Execution Log, Error Log |
| 6 | Competitor Spy Bot — Siva | Sheets 4, Gemini 1 | Slack (3) + channel ID; competitor sheet ID; execution/error log sheet IDs | Competitors, Snapshots, Log, Errors |
| 7 | Content Repurposing Machine — Siva | Sheets 3, Gemini 1 | Telegram (2); sheet IDs; transcript API URL; image-gen API URL; Buffer API token + profile ID | Drafts, Log, Errors |
| 8 | Crypto/Stock Whale Watcher — Siva | Sheets 2, Gemini 1 | Telegram (2); sheet ID | Execution Log, Error Log |
| 9 | Daily Automation Self-Report — Siva | Sheets 5, Gemini 1, Gmail 1 | Telegram (1); automation-log sheet ID; n8n host URL + n8n API credential | Daily Log, Weekly Trends, Control, Error Log |
| 10 | Digital Twin Slack Bot — Siva | Sheets 3, Gemini 1 | Slack (3) + channel ID + Siva's Slack user ID; Telegram (1); sheet ID | Pending Approvals, Execution Log, Errors |
| 11 | Dream-Team Multi-Agent Pipeline — Siva | Sheets 5, Gemini 1 | Telegram (3); Docs cred; sheet ID; webhook path value | Pipeline Drafts, Approval, Execution Log, Error Log |
| 12 | Event-Based Customer Win-Back — Siva | Sheets 8, Gemini 1, Gmail 1 | Telegram (3); sheet ID; SMS provider API + customer phone; webhook path value | Customer Signals, Win-Back Drafts, Approval, Replies, Win-Back Outcomes, Execution Log, Error Log |
| 13 | Habit Coach via WhatsApp — Siva | Sheets 5, Gemini 1, Gmail 1 | WhatsApp Business (3) + phone number ID; Telegram (1); sheet ID | Habits, Check-ins, Execution Log, Errors |
| 14 | Image-to-Product-Listing — Siva | Sheets 5, Gemini 1 | Telegram (2); Shopify (1) + shop ID; eBay API token; Etsy API key | Listing Approvals, Declined, Execution Log, Errors |
| 15 | Invoice and Receipt Vacuum — Siva | Sheets 8, Gemini 1, Gmail 1 | Telegram (2); sheet ID; OCR.space API key; QuickBooks OAuth + company ID | Receipts, Duplicates, Approvals, Accounting Postings, Declined, Execution Log, Errors |
| 16 | Job Hunter Autopilot — Siva | Sheets 7, Gemini 1 | Telegram (4); Drive cred + Drive folder ID; sheet ID | Applied Log, Tailored Applications, Decisions, Execution Log, Errors |
| 17 | Lead Sniper — Siva | Sheets 4, Gemini 1 | Telegram (2); sheet IDs; Apollo API key (Enrich Lead HTTP node) | Sequence, Log, Errors |
| 18 | Meeting Autopilot — Siva | Sheets 4, Gemini 1, Gmail 1 | Telegram (2); Calendar cred + calendar ID; sheet ID; Fireflies API key | Approvals, Tasks, Execution Log, Error Log |
| 19 | News-to-Newsletter Pipeline — Siva | Sheets 3, Gemini 1 | Telegram (2); sheet ID; Beehiiv API key + publication ID | Newsletter Log, Execution Log, Error Log |
| 20 | Personal Finance Autopilot — Siva | Sheets 7, Gemini 2, Gmail 1 | Slack (2) + channel ID; Telegram (1); finance/execution/error sheet IDs | Transactions, Weekly Summaries, Log, Errors |
| 21 | Price Drop Sniper — Siva | Sheets 6, Gemini 1 | Telegram (2); sheet ID | Watchlist, Price Log, Alerts, Execution Log, Errors |
| 22 | Reputation Radar — Siva | Sheets 6, Gemini 1, Gmail 1 | Telegram (2); sheet ID; Google Reviews URL, Trustpilot company URL, keywords, X search API URL | Mentions, Reply Queue, Execution Log, Errors, Weekly Report |
| 23 | Self-Healing Workflow Monitor — Siva | Sheets 3, Gemini 1 | Slack (1) + channel ID; execution/error sheet IDs; n8n host URL + n8n API credential | Log, Diagnoses, Errors |
| 24 | Smart Expense Policing — Siva | Sheets 8, Gemini 1 | Slack (3) + channel ID; Telegram (1); sheet ID | Policy, Approved, Justifications, Reviewed, Escalations, Execution Log, Errors |
| 25 | Smart Home + Context Agent — Siva | Sheets 4, Gemini 1 | Telegram (3); Calendar cred + calendar ID; sheet ID; Home Assistant URL + token | Approvals, Home Log, Execution Log, Error Log |
| 26 | Social Listening Opportunity Finder — Siva | Sheets 7, Gemini 1 | Telegram (2); sheet ID | Skipped, Approval Queue, Approved Replies, Declined, Execution Log, Errors |
| 27 | Travel Deal Hunter — Siva | Sheets 6, Gemini 1 | Telegram (3); Calendar cred; sheet ID; Kiwi Tequila API key; home airport, destination, dates | Travel Watchlist, Draft Itineraries, Approval, Execution Log, Error Log |
| 28 | Voice-Note-to-Everything — Siva | Sheets 3, Gemini 1 | Telegram (4); Calendar cred; sheet IDs; STT endpoint; Todoist API token; Notion API key + database ID | Log, Errors |
| 29 | Job Search Autopilot — Siva | Sheets 7, Gemini 1, Gmail 5 | Drive cred + resume file ID; sheet ID; Apollo API key (Apollo lookup HTTP node) | Applied Log, Sends, Needs Verification, Replies, Follow-ups, Errors |

## What Siva must supply (grouped — one line per item)

**Needed by nearly every workflow — do these first:**
- **Google Sheets (one per workflow, 29 sheets)** — create a sheet per workflow with the tabs listed above, then paste each sheet's ID into its workflow's Sheets nodes. This is the single biggest blocker (29/29 workflows).
- **Telegram bot** — create a bot via @BotFather, connect it in n8n (telegramApi credential), and give the bot his chat ID; used for approval requests + error alerts in 26/29 workflows.

**Needed by several workflows:**
- **Slack workspace + channels** — connect Slack in n8n; 6 workflows (Competitor Spy, Digital Twin, Personal Finance, Self-Healing Monitor, Smart Expense Policing, Auto-Documentation).
- **Google Drive / Docs / Calendar credentials** — connect his Google account for these three types in n8n (same Google account as Gmail/Sheets, separate credentials); needed by 7 workflows. Verify first whether they already exist.

**Needed by one workflow each:**
- **WhatsApp Business number** — Habit Coach (WhatsApp trigger + 2 send nodes).
- **GitHub personal access token** — Auto-Documentation Bot (trigger + README update).
- **Notion integration + parent page / database** — Auto-Documentation Bot, Voice-Note-to-Everything.
- **Shopify store access + eBay token + Etsy key** — Image-to-Product-Listing.
- **ElevenLabs API key + voice ID, podcast host upload URL** — AI Podcast Factory.
- **Apollo API key** — Lead Sniper + Job Search Autopilot (email lookup).
- **Linear API token** — AI Customer Support Triage (ticket creation).
- **Fireflies API key** — Meeting Autopilot (transcripts).
- **OCR.space API key + QuickBooks OAuth + company ID** — Invoice and Receipt Vacuum.
- **Beehiiv API key + publication ID** — News-to-Newsletter Pipeline.
- **Buffer API token + transcript/image-gen API URLs** — Content Repurposing Machine.
- **Kiwi Tequila API key** — Travel Deal Hunter (flight/hotel prices).
- **Home Assistant URL + token** — Smart Home Context Agent.
- **SMS provider API + customer phone** — Event-Based Win-Back.
- **n8n API credential + host URL** — Daily Self-Report + Self-Healing Monitor (they call n8n's own API).
- **Speech-to-text endpoint + Todoist API token** — Voice-Note-to-Everything.

## Browser batch plan (for the parent agent — browser tasks)

The wiring step (selecting the 3 existing credentials on 193 nodes) needs live
browser tasks; it could NOT be done from this subagent (browser delegation is not
available here). Run these batches in order; verify each batch's report before the next.

**Batch 1 — also do the credential inventory first:** open Credentials, list every
credential with exact name + type, and confirm yes/no for Google Drive, Google Docs,
Google Calendar credentials. Then wire: AI Customer Support Triage, AI Inbox Chief of
Staff, AI Podcast Factory, Auto-Documentation Bot, Autonomous Research Agent,
Competitor Spy Bot.
**Batch 2:** Content Repurposing Machine, Crypto/Stock Whale Watcher, Daily Automation
Self-Report, Digital Twin Slack Bot, Dream-Team Multi-Agent Pipeline, Event-Based
Customer Win-Back.
**Batch 3:** Habit Coach via WhatsApp, Image-to-Product-Listing, Invoice and Receipt
Vacuum, Job Hunter Autopilot, Lead Sniper, Meeting Autopilot.
**Batch 4:** News-to-Newsletter Pipeline, Personal Finance Autopilot, Price Drop Sniper,
Reputation Radar, Self-Healing Workflow Monitor, Smart Expense Policing.
**Batch 5:** Smart Home + Context Agent, Social Listening Opportunity Finder, Travel
Deal Hunter, Voice-Note-to-Everything, Job Search Autopilot.
**Final verify:** open the Workflows list — count must still be 30, Daily Brief still
Published and untouched, every other workflow still inactive; spot-check a few editors
for zero Gmail/Sheets/Gemini credential warnings.

**Instructions for every batch task (copy into each browser task):**
- Instance https://sivasai-boggu.app.n8n.cloud, session already signed in as Sivasai.
- Open each workflow from the Workflows list by clicking its name. NEVER use Import.
  NEVER open "Multi-agent Daily Brief — Siva".
- In each editor: on every Gmail / Google Sheets / Google Gemini node showing a
  missing-credential warning, select the existing credential (Gmail account /
  Google Sheets account / Google Gemini(PaLM) Api account). Wire Google Drive /
  Google Docs / Google Calendar nodes ONLY if that credential exists (per Batch 1
  inventory); otherwise leave and report.
- DO NOT touch Telegram, Slack, WhatsApp, GitHub, Notion, Shopify nodes.
- DO NOT change any parameter values (no sheet IDs, chat IDs, URLs, keys). Never invent.
- DO NOT activate/publish anything. SAVE each workflow; confirm no save error.
- SAFETY: if a workflow appears merged into another, STOP, restore the affected
  workflow from Version History, verify it is intact, and report it.
- Report per workflow: nodes wired, nodes still lacking credentials (by type), save OK?

## Sources

- Workflow JSONs: `~/workspace/n8n/workflows/<slug>/workflow.json` (28) +
  `~/workspace/n8n/job-search-autopilot/job-search-autopilot-clean.json`.
- Per-workflow SETUP.md files (placeholder values + API keys) in the same folders.
- Prior import reports: 28 imported 21 Nov ~12:59 IST, Job Search Autopilot re-imported
  ~13:08 IST from the cleaned copy; Daily Brief never re-imported, still Published.
