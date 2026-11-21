# Image-to-Product-Listing — Setup Guide

A photo upload webhook sends the image to Gemini Vision, which identifies the item and writes a title, description, tags and a suggested price. Siva approves in the Listing Approvals sheet; only then does the listing go live on Shopify, eBay and Etsy. Prices are suggestions, never promises.

**File:** `workflow.json` — 19 nodes.

## 1. Import the workflow

1. Open your n8n instance (sivasai-boggu.app.n8n.cloud).
2. Go to **Workflows → ⋯ → Import from File** and choose `workflow.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5 is done.
4. Nodes that need a credential show "No credential set". Open each such node and pick/create the credential listed in step 2 — the credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect

| # | Credential | Used by | How to create |
|---|---|---|---|
| 1 | **Google Gemini API** | Google Gemini Chat Model (free tier) | Free key at https://aistudio.google.com/apikey → Credentials → New → Google Gemini(PaLM) API |
| 2 | **Google Sheets OAuth2** | Queue Listing for Approval, Read Approval Status, Log Declined, Log Execution, Log Error | Credentials → New → Google Sheets OAuth2 → sivasaiboggu@gmail.com |
| 3 | **Telegram account** | Request Approval, Notify Error via Telegram | Talk to @BotFather, create a bot, paste the token; get your chat ID from @userinfobot |
| 4 | **Shopify account** | Create Shopify Product | Credentials → New → Shopify → store URL + Admin API access token |
| 5 | **eBay API Token (HTTP Header Auth)** | Cross-post to eBay | eBay Developers → app → OAuth2 token → Header Auth → header `Authorization`, value `Bearer <token>` |
| 6 | **Etsy API Key (HTTP Header Auth)** | Cross-post to Etsy | Etsy Developers → app → keystring → Header Auth → header `x-api-key`, value = your key; replace YOUR_SHOP_ID in the URL |

No credentials are stored in `workflow.json` at all (no `credentials` property on any node) — n8n shows "No credential set" and you pick/create each one on import. Secrets go only in n8n credentials, never in node parameters.

## 3. Google Sheet — create this first

Create one Google Sheet and add these tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Listing Approvals** | Title, Description, Tags, Suggested Price, Currency, Condition, Category, Status, Queued At — set Status to APPROVED or REJECTED |
| **Declined** | Title, Suggested Price, Status, Logged At |
| **Execution Log** | At, Workflow, Result, Detail |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID (the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

## 4. Placeholders to fill in

Search the workflow for `YOUR_` — every one is a value only you can supply (Sheet ID, chat/channel IDs, company/shop IDs, phone number ID). Nothing is pre-filled with invented data.

## 5. Test before turning on

1. POST a product photo to the webhook (`POST https://<your-n8n>/webhook/product-photo`, multipart with the image). 2. Confirm a PENDING draft appears in Listing Approvals and NOTHING is listed on any marketplace. 3. Mark it APPROVED, wait out the 2h Wait node (or re-run the approval branch) and confirm the Shopify product exists, then eBay and Etsy. 4. Test a REJECTED row to confirm nothing publishes. 5. Test an unclear photo — suggestedPrice should come back null and the description should say so.

## 6. Notes

Gemini Vision describes only what it can see — it never invents brand, model or condition. The suggested price is a starting point: check comparable listings before approving. Each marketplace call has a 10s timeout; a failure on eBay/Etsy is logged to the Errors sheet and stops that chain (Shopify listing already created is NOT rolled back — check and remove it manually if needed).

## 7. Error handling & logging

An **Error Trigger** node catches any node failure and writes it to the **Errors** tab, then notifies Siva on Telegram. Every run also writes a row to **Execution Log**. HTTP Request nodes use a 10-second timeout and expect JSON responses; where a node type has no built-in retry, re-run from the failed node after fixing the cause rather than editing history.

## 8. Human approval

Nothing irreversible happens without Siva: every posting / publishing / accounting step sits behind an approval sheet row that must read `APPROVED`. The workflow only ever drafts and queues until then.
