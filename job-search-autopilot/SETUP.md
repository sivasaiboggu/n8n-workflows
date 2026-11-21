> **Import note (fixed 21 Nov 2026):** no credential is set in this workflow's JSON — the placeholder credential references that made n8n refuse to save on import were removed. On import n8n shows "No credential set"; connect your own credentials there. See the "Sticky Note — Credentials" node in the workflow for the exact list.

# Job Search Autopilot — Setup Guide

An all-in-one n8n workflow for Siva's job search: it watches 8 job boards twice a day,
researches each new company with Gemini, writes a personalised cold mail, sends it **only**
to a verified email with a 5-minute gap, logs everything to Google Sheets, and tracks replies
and Day 4–5 follow-ups.

**File:** `job-search-autopilot.json` — 33 nodes.

## 1. Import the workflow

1. Open your n8n instance (Cloud or self-hosted).
2. Go to **Workflows → ⋯ → Import from File** and choose `job-search-autopilot.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5 is done.
4. n8n will show the credentials below as "not connected". Connect each one (step 2) — the
   credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect (3 core + 1 for the resume)

| # | Credential type in n8n | Used by | How to create |
|---|---|---|---|
| 1 | **Gmail OAuth2** | Send Cold Mail, Gmail Trigger, Notify Siva, Draft Follow-up | n8n Credentials → New → Gmail OAuth2 → sign in as **sivasaiboggu@gmail.com** |
| 2 | **Google Sheets OAuth2** | All 7 Google Sheets nodes | Credentials → New → Google Sheets OAuth2 → same Google account |
| 3 | **Google Gemini API** | Google Gemini Chat Model (free tier) | Get a free key at [Google AI Studio](https://aistudio.google.com/apikey) → Credentials → New → Google Gemini(PaLM) API → paste the key |
| 4 | **Google Drive OAuth2** | Download Resume PDF | Credentials → New → Google Drive OAuth2 → same Google account (only needed for the resume attachment) |

Optional, not a credential: an **Apollo API key** (free tier) — paste it into the
**Apollo Email Lookup** node where it says `YOUR_APOLLO_API_KEY`. Without it, every contact
goes to the Needs Verification sheet and nothing is mailed (safe default).

## 3. Google Sheet — create this first

Create one Google Sheet and add these 6 tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Applied Log** | Company, Role, Board, Applied At, Source |
| **Sends** | Company, Person, Email, Role, Board, Subject, Sent At, Gmail Message ID |
| **Needs Verification** | Company, Role, Board, Listing URL, Reason, Found At |
| **Replies** | Thread ID, From, Subject, Snippet, Received At |
| **Follow-ups** | Company, Email, Role, Drafted At, Status |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID
(the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

Seed **Applied Log** with the companies Siva already applied to (iion, Discover Dollar,
Alchemyst, Drivetrain, Meraki Labs, PinSec.Ai, Honeywell, Fast Code AI, Offshore Minds,
Web Neural, insiderOne, Priberry, DBHelm, SkillsCapital, Cehpoint, Stackular, and the rest)
so the dedupe step never re-applies.

## 4. Two placeholders to fill in

1. **Download Resume PDF node** — upload `Sivasai_Boggu_Resume.pdf` to Google Drive, open it,
   and paste the file ID from its URL into the node (`YOUR_RESUME_FILE_ID`).
2. **Apollo Email Lookup node** — paste your Apollo key (`YOUR_APOLLO_API_KEY`).

## 5. Test before turning on the schedule

1. Click **Execute Workflow** (or the **Manual Trigger** node's Execute) — this runs one full
   pass: boards → filter → dedupe → research → gate → send → log.
2. Check the **Sends** sheet: one row per mail, with the Gmail message ID.
3. Check Gmail Sent: the mail should be there with the PDF attached.
4. Check **Needs Verification**: any company without a verified email must appear here and
   must **not** have been mailed.
5. Only when the manual run is clean: toggle the workflow **Active**. The Schedule Trigger
   then runs it at **09:15 and 19:00 IST** automatically. The Gmail Trigger watches for
   replies continuously, and the follow-up branch drafts (never sends) Day 4–5 follow-ups.

## How each part works

- **Board watcher** — `Board Sources` lists Wellfound, Built In, Shine, Jobfound, Himalayas,
  Cutshort, HiringCafe and LinkedIn Jobs. **Internshala is deliberately not in the list.**
  `Fetch Board Pages` → `Parse & Filter Roles` keeps only fitting, paid roles.
- **Dedupe** — `Dedupe New Roles` drops anything already in Applied Log.
- **Company research** — the LangChain AI Agent (Gemini 2.0 Flash, free tier) researches each
  company and returns JSON: company, founder name, a real hook, subject and mail body.
  It is instructed never to invent a person, email or company claim.
- **Verified-email gate** — `Apollo Email Lookup` → `Normalise Contact` → `Verified Email?`
  (IF node). Only a verified email passes, and the bounce list is hard-coded in the gate:
  `jobs@iion.io`, `rthornton@eteaminc.com`, `careers@hackshade.in`, `careers@techsima.com`.
  Anything else goes to Needs Verification and is never mailed.
- **Send** — one company at a time (`Split In Batches`, batch size 1), Gmail send with the
  PDF resume attached, `Log Send` writes company/person/time/message ID, then `Wait 5 Minutes`
  before the next company.
- **Reply tracker** — `Gmail Trigger` → `Is a Reply?` → reply logged to the Replies sheet and
  Siva gets a notification mail. `Read Sent Log` → `Find No-Reply (Day 4-5)` → `Draft Follow-up`
  creates a **draft only** — Siva reviews and sends it himself.
- **Error handling** — the `Error Trigger` catches failures from any node, logs them to the
  Errors sheet and notifies Siva, so one bad board or one bad mail never stops the run.

## Things I could not verify (read before first run)

- **Not imported into a live n8n instance** — I built and validated the JSON, but I have no
  access to Siva's n8n (not signed in; it asks for the instance name). Import it yourself
  and run the manual test in step 5.
- **Board scraping is best-effort** — job boards change markup and LinkedIn blocks scraping.
  If a board returns nothing, the parse node yields no roles for it and the run continues;
  the error is logged. For reliable results, replace a board's HTTP node with that board's
  official API/RSS feed if one exists.
- **Node type versions** — built against current n8n node types (Schedule Trigger 1.2, Gmail
  2.1, Google Sheets 4.2, AI Agent 1.8, Gemini chat model 1). If your n8n is older, it may
  ask you to update a node on import — accept the update.
- **Gmail attachment** — the send node attaches binary property `data` from the Drive
  download node. If your n8n version names the binary property differently, re-select it
  in the Send Cold Mail node.
- **Apollo free tier** — email verification needs Apollo credits; when they run out, contacts
  route to Needs Verification instead of being mailed.
