# Habit Coach via WhatsApp — Setup Guide

Sends Siva a daily 7 AM WhatsApp check-in, parses his plain-text replies (mood, sleep, workouts) with Gemini, stores trends in Sheets, nudges him when patterns slip, and emails a weekly insights summary on Sundays. Supportive coaching only — no medical claims, diagnosis or treatment advice.

**File:** `workflow.json` — 21 nodes.

## 1. Import the workflow

1. Open your n8n instance (sivasai-boggu.app.n8n.cloud).
2. Go to **Workflows → ⋯ → Import from File** and choose `workflow.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5 is done.
4. Nodes that need a credential show "No credential set". Open each such node and pick/create the credential listed in step 2 — the credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect

| # | Credential | Used by | How to create |
|---|---|---|---|
| 1 | **Google Sheets OAuth2** | Read Habits Sheet, Store Check-in, Read Weekly Trends, Log Execution, Log Error | Credentials → New → Google Sheets OAuth2 → sivasaiboggu@gmail.com |
| 2 | **Google Gemini API** | Google Gemini Chat Model (free tier) | Free key at https://aistudio.google.com/apikey → Credentials → New → Google Gemini(PaLM) API |
| 3 | **WhatsApp Business account** | Send WhatsApp Check-in, WhatsApp Trigger, Send WhatsApp Nudge | Meta Developers → WhatsApp Business app → connect your number; paste YOUR_WHATSAPP_PHONE_NUMBER_ID in the send nodes. The recipient is Siva (+91 9573719115) — change it in the nodes if coaching someone else |
| 4 | **Gmail OAuth2** | Send Weekly Insights Email | Credentials → New → Gmail OAuth2 → sivasaiboggu@gmail.com |
| 5 | **Telegram account** | Notify Error via Telegram | Talk to @BotFather, create a bot, paste the token; get your chat ID from @userinfobot |

No credentials are stored in `workflow.json` at all (no `credentials` property on any node) — n8n shows "No credential set" and you pick/create each one on import. Secrets go only in n8n credentials, never in node parameters.

## 3. Google Sheet — create this first

Create one Google Sheet and add these tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Habits** | Habit, Target, Streak, Last Check-in — one row per habit Siva tracks |
| **Check-ins** | Date, Mood, Sleep Hours, Worked Out, Note |
| **Execution Log** | At, Workflow, Result, Detail |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID (the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

## 4. Placeholders to fill in

Search the workflow for `YOUR_` — every one is a value only you can supply (Sheet ID, chat/channel IDs, company/shop IDs, phone number ID). Nothing is pre-filled with invented data.

## 5. Test before turning on

1. Run Manual Trigger — a WhatsApp check-in should arrive on Siva's phone. 2. Reply in plain text, e.g. 'slept 7 hours, mood 8, did a workout' — the Check-ins tab should gain a row with Mood 8, Sleep 7, Worked Out true. 3. To test a nudge, add 3 rows with Worked Out = FALSE and run the reply path again. 4. Weekly email: run the Sunday schedule node manually and check Gmail.

## 6. Notes

WhatsApp Business Cloud requires a Meta app and a verified sender number — the workflow will not send until that credential is connected. This coach gives general wellness nudges only; it never diagnoses, treats, or makes medical claims. If Siva reports a health concern, the right move is a clinician, not this workflow.

## 7. Error handling & logging

An **Error Trigger** node catches any node failure and writes it to the **Errors** tab, then notifies Siva on Telegram. Every run also writes a row to **Execution Log**. HTTP Request nodes use a 10-second timeout and expect JSON responses; where a node type has no built-in retry, re-run from the failed node after fixing the cause rather than editing history.

## 8. Human approval

Nothing irreversible happens without Siva: every posting / publishing / accounting step sits behind an approval sheet row that must read `APPROVED`. The workflow only ever drafts and queues until then.

## Notes / limits
- **Approval-gate exception (documented):** this workflow only messages Siva himself (WhatsApp to his own number +919573719115, weekly email to his own inbox). No approval gate is needed because nothing goes to anyone else.
