# Import ALL — 28 n8n workflows for Siva — master import guide

Instance: **sivasai-boggu.app.n8n.cloud** (n8n Cloud trial).
Verified 20 Nov 2026 (fix-up pass): **28 workflows, 619 nodes, all checks pass** —
every `workflow.json` parses, every connection references a node that exists,
no `credentials` key anywhere, no OpenAI/Claude anywhere, `active: false`
everywhere, every workflow has an Error Trigger, a human-approval gate (or a
documented notify-only exception) and a Log Execution step.
See **§6 Verification** at the bottom for the full checklist.

> **Live import was NOT possible from here** — these are files only. The import
> itself needs Siva (or a browser session) in n8n. Everything below is written
> so the import can be done in one sitting, workflow by workflow.

---

## 0. The one rule that broke the daily-brief import — read first

**No workflow.json in this batch sets a credential on any node.** n8n refuses
to save an import that references a credential with a null id (that exact
failure hit the daily-brief import). So:

- On import, every node that needs a credential shows **"No credential set"**.
  That is expected, not an error.
- Click the node → pick an existing credential or create a new one → save.
- The credential each node needs is named in two places: the
  **"Sticky Note — Credentials"** node on the canvas, and the Credentials
  section of that workflow's `SETUP.md`.
- **No real key, token or secret is stored anywhere in this repo.** Secrets go
  in n8n credentials only, never in a node parameter. Anything Siva must fill
  in is marked `YOUR_...` in the workflow.

## 1. Before importing anything (one-time setup, ~20 min)

1. **Timezone:** in n8n go to Settings and set the instance timezone to
   **Asia/Kolkata**. All cron schedules below (07:00, 09:15, 21:00 …) assume
   IST; if the instance stays on UTC they will fire 5½ hours early.
2. **Telegram bot:** talk to @BotFather, create one bot, paste its token as
   the **Telegram** credential. Message the bot once and get your chat ID
   from @userinfobot — that is `YOUR_TELEGRAM_CHAT_ID` everywhere.
3. **Gemini:** create a Google AI Studio API key (free tier) → **Google
   Gemini (PaLM) API** credential. One credential serves every workflow.
4. **Google:** connect **Gmail** (sivasaiboggu@gmail.com), **Google Sheets**,
   **Google Drive**, **Google Calendar** and **Google Docs** OAuth2
   credentials (same Google account).
5. **Google Sheets:** most workflows log to Sheets. Easiest is **one log
   sheet per workflow** (the `SETUP.md` lists the exact tabs and columns) —
   or a single shared sheet with all tabs; either works as long as the tab
   names match exactly, because the nodes look tabs up by name. Copy each
   Sheet ID into the `YOUR_GOOGLE_SHEET_ID` / `YOUR_*_SHEET_ID` fields.
   The Daily Self-Report (§4, last) reads the other workflows' logs, so the
   tab names in §3 matter — don't rename them.
6. **Other accounts (only for the workflows that need them):** Slack workspace
   + bot, WhatsApp Business (Meta Developers), Todoist, Notion, Linear,
   GitHub PAT, Fireflies, Beehiiv, ElevenLabs, Kiwi Tequila (free), Apollo,
   Shopify/eBay/Etsy, QuickBooks, OCR.space, X API, Home Assistant URL +
   long-lived token, SMS provider, n8n API key (for the Self-Healing Monitor
   and the Self-Report — see §4).

## 2. Import order

Import in this order. Rule of thumb: **receivers before senders, daily
drivers first, monitors last.**

| Phase | Workflows | Why this order |
|---|---|---|
| **A — Daily drivers first** | 1. `job-hunter-autopilot` · 2. `news-to-newsletter-pipeline` · 3. `ai-inbox-chief-of-staff` · 4. `meeting-autopilot` · 5. `autonomous-research-agent` · 6. `dream-team-multi-agent-pipeline` | The autopilot-adjacent set Siva uses every morning; the existing daily-brief workflow sits alongside these. Get the morning loop working before the rest. |
| **B — Webhook receivers** | 7. `lead-sniper` · 8. `ai-customer-support-triage` · 9. `content-repurposing-machine` · 10. `image-to-product-listing` · 11. `invoice-receipt-vacuum` · 12. `personal-finance-autopilot` · 13. `smart-expense-policing` · 14. `event-based-customer-win-back` | Import receivers before anything that calls them, so their production webhook URLs exist when the senders are wired (§5). For the two with placeholder paths (`dream-team-multi-agent-pipeline`, `event-based-customer-win-back`), pick the path on import — e.g. `dream-team-topic`, `win-back` — and use that exact path in §5. |
| **C — Schedulers & notifiers** | 15. `crypto-stock-whale-watcher` · 16. `travel-deal-hunter` · 17. `price-drop-sniper` · 18. `reputation-radar` · 19. `competitor-spy-bot` · 20. `social-listening-opportunity-finder` · 21. `smart-home-context-agent` · 22. `habit-coach-whatsapp` · 23. `voice-note-to-everything` · 24. `ai-podcast-factory` | Pollers and alerters; each is independent, so any order inside the phase works. |
| **D — Money & docs** | 25. `auto-documentation-bot` · 26. `digital-twin-slack-bot` | Needs the most third-party accounts; do after the core loop is proven. |
| **E — Monitors — LAST** | 27. `self-healing-workflow-monitor` · 28. `daily-automation-self-report` | **Self-Healing second-to-last, Self-Report LAST.** Self-Healing must be set as the instance error workflow (§4) after the others exist; Self-Report reads the other workflows' logs, so it is only meaningful once they have run. |

Import mechanics (same for every workflow): Workflows → ⋯ → **Import from
File** → select `<slug>/workflow.json` → replace every `YOUR_...`
placeholder → connect the credentials from the Sticky Note / `SETUP.md` →
create the Sheet tabs → **test with Manual Trigger / Execute Workflow
(§3, right column) → only then Activate**. Leave everything **inactive**
until its own test passes — nothing in this batch is safe to activate blind,
because several workflows send email/Telegram on their first run.

## 3. Per-workflow table

"Test" = what to do **before** activating. Every workflow has a Manual
Trigger node except the ones marked *(no Manual Trigger — use the canvas
**Execute Workflow** button instead)*.

| # | Workflow (`slug`) | Credentials to connect | Sheet tabs to create | Test before activating |
|---|---|---|---|---|
| 1 | **Job Hunter Autopilot** (`job-hunter-autopilot`) | Google Sheets, Gemini, Google Drive, Telegram | 1 sheet: tabs per `SETUP.md` (Applied Log etc.); Drive folder for tailored packs (`YOUR_DRIVE_FOLDER_ID`) | Manual Trigger → check a scored role is saved to Drive + Telegram "Apply or Skip?" arrives; tap Skip and confirm nothing is applied. |
| 2 | **News-to-Newsletter Pipeline** (`news-to-newsletter-pipeline`) | Gemini, Google Sheets, Telegram, Beehiiv API key (Header Auth) | Execution Log, Error Log, Newsletter Log | Manual Trigger → Beehiiv gets a **DRAFT only** + Telegram "draft ready"; confirm nothing is sent. |
| 3 | **AI Inbox Chief of Staff** (`ai-inbox-chief-of-staff`) | Gmail, Gemini, Google Sheets, Telegram | Inbox Log, Inbox Memory, Delegate Tasks, Approval, Execution Log, Error Log | Send yourself a test email → check classification + draft saved, nothing sent; write `APPROVE` in Approval tab and confirm the reply goes out. |
| 4 | **Meeting Autopilot** (`meeting-autopilot`) | Google Calendar, Gemini, Telegram, Gmail, Google Sheets, Fireflies API key (Header Auth) | Execution Log, Error Log, Approvals, Tasks, Meetings (optional) | Manual Trigger with a past meeting → transcript fetched, summary email to yourself, tasks queued; confirm nothing is sent before approval. |
| 5 | **Autonomous Research Agent** (`autonomous-research-agent`) | Gemini, Telegram, Google Docs, Gmail, Google Sheets | Execution Log, Error Log, Approvals, Research Log (optional) | POST a topic to `/webhook/research-agent` (test URL) → report Doc created + email to yourself with the link. |
| 6 | **Dream-Team Multi-Agent Pipeline** (`dream-team-multi-agent-pipeline`) | Gemini, Google Docs, Google Sheets, Telegram | Pipeline Drafts, Pipeline Memory, Approval, Execution Log, Error Log | Manual Trigger with a topic → draft saved to Docs (never published); write `APPROVE` and confirm only the manual-publish notice arrives. |
| 7 | **Lead Sniper** (`lead-sniper`) | Apollo API key, Telegram, Google Sheets, Gemini | Sequence, Log, Errors | POST a test lead to `/webhook/lead-sniper` → enriched + scored + added to Sequence sheet as **DRAFT**; confirm no email is sent. |
| 8 | **AI Customer Support Triage** (`ai-customer-support-triage`) | Gemini, Gmail, Telegram, Google Sheets, Linear API token (Header Auth) | Execution Log, Error Log, Approvals | POST a test message to `/webhook/support-triage` → ticket created in Linear + reply queued for approval; confirm no reply is sent unapproved. |
| 9 | **Content Repurposing Machine** (`content-repurposing-machine`) | Transcript API key, Image Gen API key, Buffer API token, Telegram, Google Sheets, Gemini | Drafts, Log, Errors | POST a YouTube URL to `/webhook/repurpose` → blog/X/LinkedIn drafts saved; confirm nothing is scheduled or published. |
| 10 | **Image-to-Product-Listing** (`image-to-product-listing`) | Gemini, Google Sheets, Telegram, Shopify, eBay token (Header Auth), Etsy key (Header Auth) | Listing Approvals, Declined, Execution Log, Errors | POST a test photo to `/webhook/product-photo` → listing draft queued; set Status `APPROVED` in the sheet and confirm it posts, `REJECTED` and confirm it doesn't. |
| 11 | **Invoice & Receipt Vacuum** (`invoice-receipt-vacuum`) | Gmail, Google Sheets, Gemini, Telegram, OCR.space key (Header Auth), QuickBooks OAuth2 (Header Auth) | Receipts, Duplicates, Approvals, Accounting Postings, Declined, Execution Log, Errors | POST a test receipt image to `/webhook/invoice-upload` → row PENDING in Approvals, nothing in Postings; approve in sheet and confirm posting appears; send the same receipt twice and confirm Duplicates flags it. |
| 12 | **Personal Finance Autopilot** (`personal-finance-autopilot`) | Google Sheets, Slack, Telegram, Gmail, Gemini | Transactions, Weekly Summaries, Dashboard, Log, Errors | POST a test transaction to `/webhook/finance-transaction` → row in Transactions + Slack/Telegram alert for unusual spend; confirm no money moves. |
| 13 | **Smart Expense Policing** (`smart-expense-policing`) | Google Sheets, Gemini, Slack, Telegram | Policy, Approved, Justifications, Reviewed, Escalations, Execution Log, Errors | POST a test card transaction to `/webhook/card-transaction` → flagged/escalated per Policy tab; reply a justification in Slack and confirm it lands in Justifications. |
| 14 | **Event-Based Customer Win-Back** (`event-based-customer-win-back`) | Gemini, Google Sheets, Telegram, Gmail, SMS Provider API (Header Auth) | Customer Signals, Win-Back Drafts, Approval, Replies, Win-Back Outcomes, Execution Log, Error Log | Add a test customer to Customer Signals → draft win-back saved + Telegram approval request; write `APPROVE` and confirm the email/SMS goes out; leave unapproved and confirm nothing is sent. |
| 15 | **Crypto/Stock Whale Watcher** (`crypto-stock-whale-watcher`) | Gemini, Telegram, Google Sheets | Execution Log, Error Log | *(No Manual Trigger — Execute Workflow button.)* Confirm a Telegram alert arrives (temporarily lower the 5% threshold in Filter Whale Threshold to force a pass, then restore it); confirm no trade action exists. |
| 16 | **Travel Deal Hunter** (`travel-deal-hunter`) | Gemini, Google Sheets, Google Calendar, Telegram, Kiwi Tequila API (Header Auth) | Travel Watchlist, Draft Itineraries, Approval, Execution Log, Error Log | *(No Manual Trigger — Execute Workflow button.)* Add a watchlist row with a low target price → deal alert + draft itinerary; write `APPROVE` and confirm only the manual-booking notice arrives — this workflow never books. |
| 17 | **Price Drop Sniper** (`price-drop-sniper`) | Google Sheets, Gemini, Telegram | 1 sheet (`YOUR_GOOGLE_SHEET_ID`) with Watchlist + Log tabs per `SETUP.md` | Manual Trigger → price read + Telegram alert on a drop; confirm nothing is bought. |
| 18 | **Reputation Radar** (`reputation-radar`) | Google Sheets, Gemini, Telegram, Gmail | 1 sheet (`YOUR_GOOGLE_SHEET_ID`) per `SETUP.md`; set `YOUR_KEYWORDS` in Monitor Sources | Manual Trigger → mentions logged, negative one queued as a draft reply + Telegram alert; confirm no reply is sent automatically. |
| 19 | **Competitor Spy Bot** (`competitor-spy-bot`) | Google Sheets, Slack, Gemini | Competitors, Snapshots, Log, Errors | Manual Trigger → snapshot rows written; change a competitor page hash and confirm the diff + Slack digest report the change. |
| 20 | **Social Listening Opportunity Finder** (`social-listening-opportunity-finder`) | Google Sheets, Gemini, Telegram, X API Bearer token (Header Auth) | Approval Queue, Approved Replies, Skipped, Declined, Execution Log, Errors | Manual Trigger → opportunity queued as a draft reply; set Status `APPROVED` and confirm it only marks the row — replies are posted manually. |
| 21 | **Smart Home + Context Agent** (`smart-home-context-agent`) | Gemini, Google Calendar, Telegram, Google Sheets, Home Assistant long-lived token (Header Auth) | Execution Log, Error Log, Approvals, Home Log | Manual Trigger → context read (calendar, weather) + alert; confirm no action that spends money or unlocks doors runs without approval. |
| 22 | **Habit Coach via WhatsApp** (`habit-coach-whatsapp`) | WhatsApp Business (Meta), Telegram, Gmail, Google Sheets, Gemini | Habits, Check-ins, Weekly Trends, Execution Log, Errors (per `SETUP.md`) | Manual Trigger → WhatsApp check-in arrives on Siva's own number (+919573719115); reply and confirm the check-in is stored. |
| 23 | **Voice-Note-to-Everything** (`voice-note-to-everything`) | Telegram, Gemini, Google Sheets, Google Calendar, Todoist token, Notion key; **STT: point the "Transcribe Voice (STT — placeholder)" node at Gemini transcription or any STT service** (`https://YOUR_STT_ENDPOINT/transcribe`, no key in the file) | Voice Notes Log, Execution Log, Errors | Send a test voice note to the Telegram bot → transcription + split summary arrives, nothing created; reply APPROVE and confirm Todoist task + Calendar event + Notion note appear; repeat with REJECT and confirm nothing is created. |
| 24 | **AI Podcast Factory** (`ai-podcast-factory`) | Google Sheets, Gemini, Google Drive, Telegram, ElevenLabs API key (header `xi-api-key`, no n8n credential) | Episode sheet per `SETUP.md`; Drive folder for audio (`YOUR_DRIVE_FOLDER_ID`); RSS feed URL; podcast-host upload URL | Manual Trigger → episode audio generated + queued for approval; approve in Telegram and confirm upload + social post only happen after approval. |
| 25 | **Auto-Documentation Bot** (`auto-documentation-bot`) | GitHub account, GitHub PAT (Header Auth), Gemini, Google Sheets, Telegram, Slack, Notion | Doc Approvals, Declined, Execution Log, Errors | Push a test commit / merge a test PR → release-notes draft queued; set Status `APPROVED` and confirm docs publish, `REJECTED` and confirm they don't. |
| 26 | **Digital Twin Slack Bot** (`digital-twin-slack-bot`) | Slack, Google Sheets, Gemini, Telegram | 1 sheet (`YOUR_GOOGLE_SHEET_ID`) per `SETUP.md`; Slack channel + Siva's Slack user ID | DM the bot in Slack → Gemini replies in Siva's voice from the memory sheet; confirm escalation/approval path triggers for anything it can't answer. |
| 27 | **Self-Healing Workflow Monitor** (`self-healing-workflow-monitor`) | Google Sheets, Slack, n8n API key, Gemini | Errors, Diagnoses, Log | Manual Trigger with a faked failure → Gemini diagnosis logged + Slack alert with retry option; **then set it as the instance error workflow (§4)**. Do not activate its schedule — it has none; it fires on failures. |
| 28 | **Daily Automation Self-Report** (`daily-automation-self-report`) | Gmail, Google Sheets, Gemini, Telegram, n8n API key (optional placeholder) | Daily Log, Control (`send_report` flag), Weekly Trends, Error Log | Manual Trigger → report email arrives at sivasaiboggu@gmail.com summarising the day; set `send_report` OFF in Control and confirm the next run skips sending. **Import and activate LAST.** |

> Sheet-tab note: where the table says "per `SETUP.md`", that workflow's
> `SETUP.md` lists the exact tab names and column headers — copy them
> verbatim. Tab names are looked up by the nodes, so a renamed tab breaks
> the workflow silently (the node errors, the Error Trigger logs it).

## 4. Activation notes — what runs when, and what stays off

**Timezone first:** set Asia/Kolkata in n8n Settings (§1.1) before activating
any schedule, or every time below will be 5½ hours off.

| Schedule (IST, after timezone fix) | Workflows |
|---|---|
| Every 15 min | Crypto/Stock Whale Watcher |
| Every 30 min | Smart Home + Context Agent |
| Every 2 hours | Price Drop Sniper |
| Every 6 hours | Travel Deal Hunter, Social Listening Opportunity Finder |
| Daily 06:00 | AI Podcast Factory |
| Daily 07:00 | News-to-Newsletter Pipeline, Habit Coach (check-in) |
| Daily 08:00 | Reputation Radar (daily scan) |
| Daily 09:00 | Competitor Spy Bot (Mondays only) |
| Daily 09:15 | Job Hunter Autopilot |
| Daily 21:00 | Daily Automation Self-Report |
| Weekly Sun 18:00 | Habit Coach (weekly insights), Personal Finance Autopilot (weekly summary) |
| Weekly Mon 09:00 | Reputation Radar (weekly trend report) |
| Daily churn check (24 h interval) | Event-Based Customer Win-Back (plus its webhook) |
| Event-driven only (no schedule) | AI Inbox Chief of Staff (Gmail trigger), Meeting Autopilot (Calendar trigger), Digital Twin (Slack trigger), Voice-Note (Telegram trigger), Habit Coach replies (WhatsApp trigger), Job Hunter / Podcast Factory (Telegram button triggers), Self-Healing Monitor (Error Trigger — see below) |
| Webhook only | Lead Sniper, Support Triage, Research Agent, Content Repurposing, Image-to-Product-Listing, Invoice Vacuum, Personal Finance, Smart Expense Policing, Dream-Team, Win-Back — production URL is live only **after** activation |

**Stays inactive until tested:** everything, per §2 — no exceptions. In
particular keep inactive until their manual test passes: **Job Hunter**
(Telegram Apply/Skip must be proven before it can act on roles),
**AI Inbox Chief of Staff** (it archives real email), **Smart Expense
Policing** and **Invoice Vacuum** (they touch money records),
**Win-Back** (it emails/SMSes customers), and **Smart Home** (it can act on
the house — approval gate must be proven first).

**Self-Healing Monitor — special setup:** it has no schedule. Its Error
Trigger fires on failures, but only if n8n knows to call it: in n8n go to
**Settings → Error workflows** (or the workflow's Settings → Error
Workflow) and select **Self-Healing Workflow Monitor**. Then any workflow
failure triggers it: Gemini diagnoses the error, posts a Slack alert with a
retry option, waits up to 4 hours for a human decision, and retries via the
n8n API **only if approved**. It also needs `YOUR_N8N_HOST` set
(`https://sivasai-boggu.app.n8n.cloud`) and the n8n API key credential.

**Daily Self-Report — special setup:** it reads the shared **Daily Log**
sheet plus the n8n API executions list, and emails Siva at 21:00. It only
produces a useful report once the other workflows have actually run, which
is why it is imported and activated **last**. The `send_report` flag in the
**Control** tab turns the email off without deactivating the workflow. The
n8n API node is an optional placeholder — the report still works from the
Daily Log sheet alone if no n8n API key is connected.

**Webhook test vs production URLs:** while a workflow is inactive, its
webhook answers on the **test** URL (`/webhook-test/<path>`) only when you
click Execute Workflow / Listen for test event. After activation it answers
on the **production** URL (`/webhook/<path>`). Use the test URL for the §3
tests, and give external services (forms, Stripe, WhatsApp provider) the
production URL only after activation.

## 5. Webhook-chaining map

**Nothing in this batch calls another workflow yet** — no cross-workflow
HTTP calls are wired inside the JSONs. The chains below are the designed
handoffs: each one is either a shared-sheet/Telegram handoff that already
works, or a one-node wiring job (add an HTTP Request node in the source,
POST to the target's production webhook URL).

| From | To | Handoff (how it works / what to wire) |
|---|---|---|
| Job Hunter Autopilot | Lead Sniper (`POST /webhook/lead-sniper`) | Job Hunter surfaces a hiring company with a verified contact → POST `{company, contact, role}` to Lead Sniper to start the outreach sequence. **To wire:** add an HTTP Request node in Job Hunter after the Telegram "Apply" approval. Until wired, paste the contact into Lead Sniper manually. |
| Social Listening Opportunity Finder | Lead Sniper (`POST /webhook/lead-sniper`) | An opportunity post with an identifiable prospect → same Lead Sniper webhook. **To wire:** HTTP Request node after an `APPROVED` row appears in the Approval Queue. |
| Competitor Spy Bot | Dream-Team Multi-Agent Pipeline (`POST /webhook/<your-dream-team-path>`) | Monday spy digest finds a notable competitor change → send it as a Dream-Team topic so the four agents draft the analysis piece. **To wire:** HTTP Request node in the Spy Bot's digest step. |
| Reputation Radar | AI Inbox Chief of Staff | Negative mention needing an email reply → Radar queues the draft reply in its sheet and alerts Siva on Telegram; Siva approves the wording, then the reply goes out through the Inbox workflow's Gmail draft/approve flow. **Already works as a sheet + Telegram handoff — no wiring needed.** |
| News-to-Newsletter Pipeline | AI Podcast Factory | The 07:00 newsletter's top story can become a podcast episode topic — paste it as the Podcast Factory's topic (Manual Trigger). **Manual handoff, no wiring needed.** |
| Invoice & Receipt Vacuum | Smart Expense Policing | An approved receipt posting can be checked against the Policy tab — both workflows read the same Gmail/webhook sources; if Siva wants them linked, point the Vacuum's approval step at the Expense Policing webhook (`POST /webhook/card-transaction`). **Suggested, not wired.** |
| AI Inbox Chief of Staff | Meeting Autopilot | An email that confirms a meeting → the Calendar trigger picks it up and the Autopilot prepares the brief. **Already works via Google Calendar — no wiring needed.** |
| Voice-Note-to-Everything | Habit Coach / Personal Finance | A voice note saying "log this expense" or "I skipped the gym" creates the Todoist/Calendar/Notion item; the Habit Coach and Finance workflows read their own sheets, so keep the sheet names aligned if Siva wants voice notes to feed them. **Suggested, not wired.** |
| **Self-Healing Workflow Monitor** | **all 27 other workflows** | **Wired by configuration, not by nodes:** set it as the instance error workflow (§4). Its Error Trigger then fires on any workflow failure, Gemini diagnoses it, Slack gets the alert + retry option, and it retries via the n8n API only after human approval. |
| **Daily Automation Self-Report** | **all 27 other workflows** | **Reads, doesn't trigger:** every evening at 21:00 it reads the shared Daily Log sheet + the n8n API execution list and emails Siva one summary. It watches the others; nothing watches it except its own Log Execution row. |

## 6. Verification (fix-up pass, 20 Nov 2026)

Checks run on all 28 `workflow.json` files (script: `python3 -m json.tool`
per file + connection/property audit):

- ✅ **28/28 parse** with `python3 -m json.tool`.
- ✅ **619 nodes total** (614 built + 5 new "Sticky Note — Credentials" nodes
  added in this pass).
- ✅ **Connections:** every source and every target name in `connections`
  exists in that workflow's `nodes` — 0 dangling references.
- ✅ **No `credentials` key in any file** — 55 null-id credential references
  stripped from the 5 batch-3 files (AI Inbox Chief of Staff 13, Crypto/Stock
  Whale Watcher 5, Dream-Team Multi-Agent Pipeline 10, Event-Based Customer
  Win-Back 14, Travel Deal Hunter 13), each now carrying a "Sticky Note —
  Credentials" node naming the credential per node; §2 (Import) of each of
  their `SETUP.md` files now states the corrected rule and why.
- ✅ **No OpenAI/Claude anywhere** — the one Whisper node in
  Voice-Note-to-Everything (`https://api.openai.com/v1/audio/transcriptions`,
  model `whisper-1`) was replaced with **"Transcribe Voice (STT —
  placeholder)"** → `https://YOUR_STT_ENDPOINT/transcribe`, no key/secret in
  the node, renamed in `connections`, Sticky Note and `SETUP.md` (point it
  at Gemini transcription or any STT service of Siva's choice). The only
  remaining "OpenAI" text in the batch is the sentence "No OpenAI/Claude is
  used anywhere" in that workflow's `SETUP.md`.
- ✅ **`active: false` on all 28.**
- ✅ **Error Trigger node in all 28**, wired to Log Error to Sheet + a
  Telegram/Slack notify step.
- ✅ **Human-approval gate before irreversible actions in 22/28** (Telegram
  approve step, approval-sheet IF, or Wait-for-approval). The other 6 are
  **documented exceptions — they only notify Siva himself or save drafts**:
  Crypto/Stock Whale Watcher (Telegram alert only, Enforce No-Trade Guard,
  never trades), Daily Automation Self-Report (report email only to
  sivasaiboggu@gmail.com, send flag in sheet), Habit Coach via WhatsApp
  (WhatsApp to Siva's own +919573719115 + email to himself), News-to-
  Newsletter Pipeline (Beehiiv **draft** only, Siva sends it himself),
  Price Drop Sniper (Telegram alert only, never buys), Reputation Radar
  (draft replies queued in a sheet, weekly report to himself, nothing sent
  automatically). Each exception is noted in that workflow's `SETUP.md`
  under Notes / limits. (Job Hunter Autopilot's gate is its Telegram
  "Apply or Skip?" buttons + the "Apply?" IF — it does not apply anywhere
  until Siva taps Apply.)
- ✅ **Log Execution step in all 28.**

### Still unverified (needs a live browser / n8n session — not possible here)

- **Live import itself** — import each file in n8n and confirm it saves
  (this is what the credential strip was for), the nodes show "No credential
  set", and the workflow runs.
- **Credential connections** — picking/creating each credential can only be
  done in Siva's n8n account.
- **Production webhook URLs** — only exist after activation.
- **Real runs** — first scheduled runs, Telegram delivery, Sheet appends,
  and the Self-Healing error-workflow setting (§4) all need the live
  instance. Run the §3 test for each workflow before activating it.
