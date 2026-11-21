> **Import note (fixed 21 Nov 2026):** no credential is set in this workflow's JSON — the placeholder credential references that made n8n refuse to save on import were removed. On import n8n shows "No credential set"; connect your own credentials there. See the "Sticky Note — Credentials" node in the workflow for the exact list.

# Multi-agent Daily Brief — Setup Guide

A three-agent n8n workflow for Siva: a **Researcher** finds the news, a **Writer**
summarises it, a **Reviewer** checks it, and the approved brief is emailed to
**sivasaiboggu@gmail.com** every morning at **07:00 IST**.

**File:** `multi-agent-daily-brief.json` — 29 nodes.

## The three agents

| # | Agent | What it does |
|---|---|---|
| 1 | **Researcher** (RSS + filter + dedupe — no AI) | Pulls fresh items from 8 real feeds (TechCrunch AI, The Verge AI, Hacker News, MIT Technology Review AI, LangChain Blog, OpenAI News, Anthropic News, TechCrunch Startups for funding/hiring signals), keeps AI / AI-agent / hiring-signal items only, and drops anything already in the Briefed Log sheet so nothing repeats. |
| 2 | **Writer** (LangChain AI Agent, free-tier Gemini 2.0 Flash) | Turns the researcher's items into a tight morning brief: top 5 stories with a one-line why-it-matters, 2–3 job leads relevant to an AI agent engineer, and a "one thing to learn today" — every item with its source URL. It may use only the research pack; it never invents news, facts, people or job leads. |
| 3 | **Reviewer** (separate LangChain AI Agent, free-tier Gemini 2.0 Flash) | Fact-checks the draft against the source items, drops anything unverified or invented, corrects URLs, and either approves the cleaned draft or sends it back **once** for a single writer revision (loop max 1). |

After the Reviewer, a mechanical **Final Source Check** (no AI) verifies every URL in
the brief exists in the fetched research items. **An unreviewed draft is never sent** —
if nothing verified survives, the run logs a skip instead of emailing an empty or
invented brief.

## 1. Import the workflow

1. Open your n8n instance — Siva's Cloud trial instance is
   **sivasai-boggu.app.n8n.cloud** (account sivasaiboggu@gmail.com).
2. Go to **Workflows → ⋯ → Import from File** and choose `multi-agent-daily-brief.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5
   is done.
4. n8n will show the credentials below as "not connected". Connect each one (step 2) —
   the credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect (3)

| # | Credential type in n8n | Used by | How to create |
|---|---|---|---|
| 1 | **Gmail OAuth2** | Send Brief, Notify Siva (Error) | n8n Credentials → New → Gmail OAuth2 → sign in as **sivasaiboggu@gmail.com** |
| 2 | **Google Sheets OAuth2** | Read Briefed Log, Log Brief, Log Skipped Brief, Log Error | Credentials → New → Google Sheets OAuth2 → same Google account |
| 3 | **Google Gemini API** | Writer / Reviewer / Revision Gemini models (free tier) | Get a free key at [Google AI Studio](https://aistudio.google.com/apikey) → Credentials → New → Google Gemini(PaLM) API → paste the key |

All three Gemini model nodes use `models/gemini-2.0-flash` on the free tier.

## 3. Google Sheet — create this first

Create one Google Sheet and add these 3 tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Briefed Log** | Brief Date, Sent At, Stories, Title, URL |
| **Skipped** | Brief Date, Reason, Skipped At |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID
(the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

## 4. Test before turning on the schedule

1. Click **Execute Workflow** (or the **Manual Trigger** node's Execute) — this runs one
   full pass: feeds → filter → dedupe → Writer → Reviewer → (revision if asked, max 1) →
   source check → send → log.
2. Check Gmail: the brief should arrive at sivasaiboggu@gmail.com with the subject
   `Your AI morning brief — <date>`, with top stories, job leads and one thing to learn,
   every item carrying a source URL.
3. Check the **Briefed Log** sheet: one row for the sent brief.
4. If the run produced nothing sendable, check the **Skipped** sheet — that means the
   Reviewer + source check found nothing verified, which is the safe behaviour, not a bug.
5. Only when the manual run is clean: toggle the workflow **Active**. The Schedule Trigger
   then runs it at **07:00 IST** every morning automatically.

## How each part works

- **Researcher** — `Feed Sources` lists the 8 feeds. `Fetch Feeds` (RSS) →
  `Parse & Filter Items` keeps AI/agent/hiring items only (max 6 per source) →
  `Read Briefed Log` → `Dedupe New Items` drops already-briefed URLs →
  `Aggregate Research` bundles up to 15 items into one research pack.
- **Writer** — the LangChain AI Agent (Gemini 2.0 Flash) drafts the brief as JSON from
  the research pack only. `Parse Writer Draft` extracts it; unparsable output is treated
  as an empty draft so the Reviewer rejects it.
- **Reviewer** — a second, separate AI Agent with a different instruction fact-checks the
  draft, returns a cleaned draft plus `approved` / `needsRevision` / `issues`. `Needs Revision?`
  sends it back to `Writer Agent — Revision` **once only** (`revisionCount < 1`); after that
  the reviewed version goes out.
- **Final Source Check** — mechanical, no AI: drops any brief item whose URL is not in the
  fetched items. `Has Verified Items?` sends the brief only if at least one verified story
  survived; otherwise it logs to **Skipped** and sends nothing.
- **Send** — Gmail to sivasaiboggu@gmail.com, subject `Your AI morning brief — <date>`,
  then `Log Brief` writes the sent brief to the Briefed Log sheet.
- **Error handling** — the `Error Trigger` catches failures from any node, logs them to
  the Errors sheet and notifies Siva, so one dead feed never kills the brief.

## Things I could not verify (read before first run)

- **Not imported into a live n8n instance yet** — the JSON is built and validated, but
  it has not been imported or test-run. Import it into sivasai-boggu.app.n8n.cloud and
  run the manual test in step 4.
- **RSS feeds are best-effort** — publishers change feed URLs and formats. If a feed
  returns nothing, the parse node yields no items for it and the run continues; the
  error is logged. If a feed is dead, replace its URL in the `Feed Sources` node.
- **Node type versions** — built against current n8n node types (Schedule Trigger 1.2,
  RSS Feed Read 1.2, Gmail 2.1, Google Sheets 4.2, AI Agent 1.8, Gemini chat model 1).
  If your n8n is older, it may ask you to update a node on import — accept the update.
- **Gemini free tier** — three agent calls per brief (Writer, Reviewer, plus a Revision
  call only when the Reviewer asks for one). Well within the free tier for one brief a day.
- **No Internshala anywhere** in this workflow, and no job lead is ever invented — if the
  day's sources contain no real hiring signal, the job-leads section comes back empty.
