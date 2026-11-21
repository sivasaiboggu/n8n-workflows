# Smart Expense Policing — Setup Guide

A webhook receives company-card transactions. Gemini checks each one against the policy stored in Sheets: normal spend auto-approves (log only — no money moves), odd spend triggers a Slack DM asking the employee for justification, and no reply within 24 hours escalates to the #finance channel.

**File:** `workflow.json` — 21 nodes.

## 1. Import the workflow

1. Open your n8n instance (sivasai-boggu.app.n8n.cloud).
2. Go to **Workflows → ⋯ → Import from File** and choose `workflow.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5 is done.
4. Nodes that need a credential show "No credential set". Open each such node and pick/create the credential listed in step 2 — the credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect

| # | Credential | Used by | How to create |
|---|---|---|---|
| 1 | **Google Sheets OAuth2** | Read Expense Policy, all log nodes | Credentials → New → Google Sheets OAuth2 |
| 2 | **Google Gemini API** | Google Gemini Chat Model (free tier) | Free key at https://aistudio.google.com/apikey → Credentials → New → Google Gemini(PaLM) API |
| 3 | **Slack account** | Ask Employee for Justification, Escalate to Finance, Slack Trigger | Credentials → New → Slack → OAuth2 with `chat:write` and channel-message scopes; set YOUR_SLACK_CHANNEL_ID on the Slack Trigger |
| 4 | **Telegram account** | Notify Error via Telegram | Talk to @BotFather, create a bot, paste the token; get your chat ID from @userinfobot |

No credentials are stored in `workflow.json` at all (no `credentials` property on any node) — n8n shows "No credential set" and you pick/create each one on import. Secrets go only in n8n credentials, never in node parameters.

## 3. Google Sheet — create this first

Create one Google Sheet and add these tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Policy** | Category, Limit, Receipt Required Above, Notes — one row per rule, e.g. Meals | 2000 | 1000 | Receipt required above ₹1,000 |
| **Approved** | Employee, Merchant, Amount, Currency, Policy Rule, Status, Logged At |
| **Justifications** | Employee, Merchant, Justification, Received At — filled by the Slack Trigger when an employee replies |
| **Reviewed** | Employee, Merchant, Amount, Justification, Status, Logged At |
| **Escalations** | Employee, Merchant, Amount, Status, Logged At |
| **Execution Log** | At, Workflow, Result, Detail |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID (the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

## 4. Placeholders to fill in

Search the workflow for `YOUR_` — every one is a value only you can supply (Sheet ID, chat/channel IDs, company/shop IDs, phone number ID). Nothing is pre-filled with invented data.

## 5. Test before turning on

1. Fill the Policy tab with at least one rule. 2. POST a normal transaction to the webhook (`POST https://<your-n8n>/webhook/card-transaction`, JSON: employee, merchant, amount, currency, employeeSlackId) — it should land in Approved. 3. POST an over-limit / banned-category one — the employee should get a Slack DM. 4. Reply in Slack (fills Justifications) and confirm the Reviewed path; or leave it 24h to confirm escalation to #finance. Use Manual Trigger + a pinned test payload to test without the webhook.

## 6. Notes

Auto-approval only ever writes a log row — this workflow never moves money, refunds, or blocks a card. The 24h Wait node resumes automatically; the Slack Trigger records justifications as they arrive. The 'Justification Received?' IF checks whether the Justifications sheet has a value for the transaction (empty = escalate). Tune the policy wording in the Policy tab, not in the prompt.

## 7. Error handling & logging

An **Error Trigger** node catches any node failure and writes it to the **Errors** tab, then notifies Siva on Telegram. Every run also writes a row to **Execution Log**. HTTP Request nodes use a 10-second timeout and expect JSON responses; where a node type has no built-in retry, re-run from the failed node after fixing the cause rather than editing history.

## 8. Human approval

Nothing irreversible happens without Siva: every posting / publishing / accounting step sits behind an approval sheet row that must read `APPROVED`. The workflow only ever drafts and queues until then.
