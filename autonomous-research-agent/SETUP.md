# Autonomous Research Agent — Siva — SETUP

## What it does
Chat/webhook topic → web search (DuckDuckGo) → fetch top pages → cross-check
sources → Gemini writes a cited report → **human approval** → saved to
Google Docs → Siva emailed the link. **Never invents sources** — every claim
must trace to a fetched URL; untraceable claims are omitted.

## 1. Import
1. In n8n (`sivasai-boggu.app.n8n.cloud`): Workflows → Import from File →
   select `workflow.json`.
2. Imports **inactive** (`active: false`). Do not activate until the test
   in §5 passes.

## 2. Credentials to connect (NO credential is set in workflow.json — see note below)
| Credential (as named in the file) | What to create |
|---|---|
| Google Gemini API | Google Gemini (PaLM) API — Gemini API key, free tier. |
| Telegram account | Telegram Bot API — bot token + `YOUR_TELEGRAM_CHAT_ID`. Approval requests + error alerts. |
| Google Docs account | Google Docs OAuth2. "Save to Google Docs" creates the report document. |
| Gmail account | Gmail OAuth2 for `sivasaiboggu@gmail.com`. Emails Siva the report link. |
| Google Sheets account | Google Sheets OAuth2. Execution Log / Error Log / Approval Sheet. |

CORRECTED RULE: the `credentials` property is omitted entirely from every node in workflow.json. n8n treats a null-id credential placeholder as a reference to a credential not shared with the account and refuses to save the import (this exact failure hit the Daily Brief import). With no credentials property, n8n shows "No credential set" on each node — pick an existing credential or create a new one on import. The per-node credential list is also in the "Sticky Note — Credentials" inside the workflow. No real key/secret is stored in the JSON. DuckDuckGo HTML search and the
page fetcher need no credentials.

## 3. Google Sheet — create one sheet, id = `YOUR_GOOGLE_SHEET_ID`
Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node. Tabs/columns:
- **Execution Log**: `timestamp | workflow | status | summary`
- **Error Log**: `timestamp | workflow | error | node`
- **Approvals**: `key | value | updated_at` — one row: `key=latest`,
  `value=pending`. Approve with `value=approved`, reject with
  `value=rejected`. The workflow waits 60 minutes after the Telegram
  request, then reads this row. Rejected → nothing is saved or emailed.
- **Research Log** (optional, future): `topic | sources | doc_link | date` —
  not written by this version; the sticky note describes wiring it in so
  repeat topics reuse past reports instead of re-researching.

## 4. Resources / placeholders to fill
- **Webhook path** `research-agent`: point your chat frontend at the webhook
  URL n8n shows (test URL while testing, production URL once active).
  Expected POST body: `{"topic": "Sybil attacks in federated learning"}`.
  A topic is required — "Validate Topic" throws a clear error if it is missing.
- **Page fetching**: "Fetch Top Pages" uses `https://r.jina.ai/<url>` (Jina
  Reader) to turn pages into text. It fetches the URL(s) returned by the
  search step — verify on first run that it returns readable text for your
  topics; some paywalled/JS-heavy pages return little or nothing, in which
  case the report will be thinner (by design — no invented filler).
- **Google Docs**: the created document's link comes from the "Save to
  Google Docs" node output. The email to Siva points at that node output —
  on first run, confirm the link in the email actually opens the doc; if
  the Docs node returns the ID only, edit the email body to build the URL
  as `https://docs.google.com/document/d/<id>/edit`.
- **Email recipient** is fixed to `sivasaiboggu@gmail.com` (Siva himself).

## 5. Test (Manual Trigger first, before activating)
1. Use the webhook **test URL** (a bare Manual Trigger run has no topic and
   correctly fails validation): POST `{"topic": "Sybil attacks in federated learning"}`.
2. Verify: "Search the Web" returns real results; "Fetch Top Pages" returns
   page text; "Cross-Check Sources" keeps only items with a URL; the agent's
   report cites only those URLs (spot-check 2–3 citations by opening them).
3. Verify the Telegram approval request arrives; set Approvals
   `value=approved`; wait out (or temporarily shorten) the 60-min Wait node;
   confirm the Google Doc is created and the Gmail to Siva arrives with a
   working link.
4. Test rejection once: set Approvals `value=rejected` on a second run and
   confirm nothing is saved or emailed.
5. Check Execution Log / Error Log, restore the Wait duration if shortened,
   then activate.

## 6. Notes / limits
- Retry: HTTP Request nodes use `timeout: 10000` (search uses JSON format,
  page fetch uses text format); enable workflow Settings retry for automatic
  retries.
- Error handling: Error Trigger → Log Error to Sheet → Telegram alert to Siva.
- Source discipline is enforced by prompt + the Cross-Check Code node, but
  it is not a formal verifier — spot-check citations on the first few
  reports before trusting them blindly.
