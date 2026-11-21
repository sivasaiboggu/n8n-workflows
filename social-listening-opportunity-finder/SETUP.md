# Social Listening Opportunity Finder — Setup Guide

Every 6 hours it searches Reddit and X for people asking questions Siva's products solve (AI SDR agent, Voice Receptionist, HireMind resume analyser). Gemini filters for genuine buyers, drafts a helpful non-spammy reply, and queues it for approval. It NEVER auto-posts — approved replies go to a sheet for Siva to post manually.

**File:** `workflow.json` — 23 nodes.

## 1. Import the workflow

1. Open your n8n instance (sivasai-boggu.app.n8n.cloud).
2. Go to **Workflows → ⋯ → Import from File** and choose `workflow.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5 is done.
4. Nodes that need a credential show "No credential set". Open each such node and pick/create the credential listed in step 2 — the credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect

| # | Credential | Used by | How to create |
|---|---|---|---|
| 1 | **Google Sheets OAuth2** | All Google Sheets nodes | Credentials → New → Google Sheets OAuth2 → sivasaiboggu@gmail.com |
| 2 | **Google Gemini API** | Google Gemini Chat Model (free tier) | Free key at https://aistudio.google.com/apikey → Credentials → New → Google Gemini(PaLM) API |
| 3 | **Telegram account** | Notify Siva — Reply Ready, Notify Error via Telegram | Talk to @BotFather, create a bot, paste the token; get your chat ID from @userinfobot |
| 4 | **X API Bearer Token (HTTP Header Auth)** | Search X | X Developer Portal → app → Bearer Token → Credentials → New → Header Auth → header `Authorization`, value `Bearer <token>` |

No credentials are stored in `workflow.json` at all (no `credentials` property on any node) — n8n shows "No credential set" and you pick/create each one on import. Secrets go only in n8n credentials, never in node parameters.

## 3. Google Sheet — create this first

Create one Google Sheet and add these tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Approval Queue** | Source, Author, Original Post, Product, Draft Reply, Status, Queued At — set Status to APPROVED or REJECTED |
| **Approved Replies** | Source, Author, Original Post, Draft Reply, Status, Approved At — post these manually |
| **Skipped** | Source, Author, Text, Reason, Logged At |
| **Declined** | Source, Author, Status, Logged At |
| **Execution Log** | At, Workflow, Result, Detail |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID (the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

## 4. Placeholders to fill in

Search the workflow for `YOUR_` — every one is a value only you can supply (Sheet ID, chat/channel IDs, company/shop IDs, phone number ID). Nothing is pre-filled with invented data.

## 5. Test before turning on

1. Run Manual Trigger — Reddit results should land in Skipped or Approval Queue. 2. Check that no reply is posted anywhere (there is no posting node in this workflow by design). 3. Mark one queued row APPROVED, wait out the 2h Wait node (or re-run the approval branch manually) and confirm it appears in Approved Replies. 4. Edit the Reddit/X search queries in the HTTP nodes to add keywords for new products.

## 6. Notes

Reddit's public search JSON endpoint needs no auth but is rate-limited; X search needs a bearer token (free tier is limited — if it 429s, the error path logs it and Reddit results still flow). Replies are deliberately short, helpful first, product mention only if it fits. Never add an auto-post node — that would break the no-spam rule.

## 7. Error handling & logging

An **Error Trigger** node catches any node failure and writes it to the **Errors** tab, then notifies Siva on Telegram. Every run also writes a row to **Execution Log**. HTTP Request nodes use a 10-second timeout and expect JSON responses; where a node type has no built-in retry, re-run from the failed node after fixing the cause rather than editing history.

## 8. Human approval

Nothing irreversible happens without Siva: every posting / publishing / accounting step sits behind an approval sheet row that must read `APPROVED`. The workflow only ever drafts and queues until then.
