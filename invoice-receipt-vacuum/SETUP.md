# Invoice and Receipt Vacuum — Setup Guide

Watches Gmail for receipts with attachments and accepts photo uploads via webhook. OCR + Gemini extract vendor, date, amount and tax, categorise the spend, flag duplicates and missing receipts, then wait for Siva's approval before anything is posted to accounting (Google Sheet + QuickBooks).

**File:** `workflow.json` — 26 nodes.

## 1. Import the workflow

1. Open your n8n instance (sivasai-boggu.app.n8n.cloud).
2. Go to **Workflows → ⋯ → Import from File** and choose `workflow.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5 is done.
4. Nodes that need a credential show "No credential set". Open each such node and pick/create the credential listed in step 2 — the credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect

| # | Credential | Used by | How to create |
|---|---|---|---|
| 1 | **Gmail OAuth2** | Gmail Trigger | Credentials → New → Gmail OAuth2 → sign in as sivasaiboggu@gmail.com |
| 2 | **Google Sheets OAuth2** | All Google Sheets nodes | Credentials → New → Google Sheets OAuth2 → same Google account |
| 3 | **Google Gemini API** | Google Gemini Chat Model (free tier) | Free key at https://aistudio.google.com/apikey → Credentials → New → Google Gemini(PaLM) API |
| 4 | **Telegram account** | Request Approval, Notify Error via Telegram | Talk to @BotFather, create a bot, paste the token; get your chat ID from @userinfobot |
| 5 | **OCR.space API Key (HTTP Header Auth)** | OCR Receipt | Free key at https://ocr.space/ocrapi/freekey → Credentials → New → Header Auth → header `apikey`, value = your key |
| 6 | **QuickBooks OAuth2 (HTTP Header Auth)** | Post to QuickBooks | Create a QuickBooks developer app, use its OAuth2 bearer token as a Header Auth credential (`Authorization: Bearer <token>`); replace YOUR_COMPANY_ID in the node URL |

No credentials are stored in `workflow.json` at all (no `credentials` property on any node) — n8n shows "No credential set" and you pick/create each one on import. Secrets go only in n8n credentials, never in node parameters.

## 3. Google Sheet — create this first

Create one Google Sheet and add these tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Receipts** | Vendor, Date, Amount, Currency, Tax, Category, Description, Extracted At |
| **Duplicates** | Vendor, Date, Amount, Currency, Flagged At, Reason |
| **Approvals** | Vendor, Date, Amount, Currency, Tax, Category, Status, Queued At — set Status to APPROVED or REJECTED |
| **Accounting Postings** | Vendor, Date, Amount, Currency, Tax, Category, Posted At, Status |
| **Declined** | Vendor, Date, Amount, Status, Logged At |
| **Execution Log** | At, Workflow, Result, Detail |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID (the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

## 4. Placeholders to fill in

Search the workflow for `YOUR_` — every one is a value only you can supply (Sheet ID, chat/channel IDs, company/shop IDs, phone number ID). Nothing is pre-filled with invented data.

## 5. Test before turning on

1. Click **Execute Workflow** on Manual Trigger with a test receipt image attached via the webhook URL (`POST https://<your-n8n>/webhook/invoice-upload`). 2. Check the Receipts/Approvals tabs — the row must be PENDING and nothing in Accounting Postings. 3. Set Status to APPROVED in the Approvals sheet, run again (or wait out the 1h Wait node on a fresh run) and confirm the posting appears. 4. Send the same receipt twice to confirm the Duplicates tab flags it.

## 6. Notes

OCR.space free tier is rate-limited; the HTTP Request node has a 10s timeout and the workflow's error path logs failures to the Errors sheet. The Gmail Trigger polls every minute. QuickBooks posting only happens after APPROVED — never edit the Approvals sheet Status to APPROVED unless you mean it.

## 7. Error handling & logging

An **Error Trigger** node catches any node failure and writes it to the **Errors** tab, then notifies Siva on Telegram. Every run also writes a row to **Execution Log**. HTTP Request nodes use a 10-second timeout and expect JSON responses; where a node type has no built-in retry, re-run from the failed node after fixing the cause rather than editing history.

## 8. Human approval

Nothing irreversible happens without Siva: every posting / publishing / accounting step sits behind an approval sheet row that must read `APPROVED`. The workflow only ever drafts and queues until then.
