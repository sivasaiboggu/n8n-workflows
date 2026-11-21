# Auto-Documentation Bot — Setup Guide

On GitHub push / merged PR, fetches the diff, Gemini drafts README + changelog + release notes, then waits for Siva's approval before updating docs on GitHub, posting release notes to Slack and creating a Notion page. No public post happens without APPROVED in the Doc Approvals sheet.

**File:** `workflow.json` — 20 nodes.

## 1. Import the workflow

1. Open your n8n instance (sivasai-boggu.app.n8n.cloud).
2. Go to **Workflows → ⋯ → Import from File** and choose `workflow.json`.
3. The workflow opens **inactive** — that is intentional. Do not activate it until step 5 is done.
4. Nodes that need a credential show "No credential set". Open each such node and pick/create the credential listed in step 2 — the credential names in the file are just labels; pick your real credentials when prompted.

## 2. Credentials to connect

| # | Credential | Used by | How to create |
|---|---|---|---|
| 1 | **GitHub account** | GitHub Trigger, Update README / Changelog | Credentials → New → GitHub → personal access token with `repo` scope |
| 2 | **GitHub Personal Access Token (HTTP Header Auth)** | Get Commit Diff | Same token as above, Header Auth → header `Authorization`, value `token <PAT>` |
| 3 | **Google Gemini API** | Google Gemini Chat Model (free tier) | Free key at https://aistudio.google.com/apikey → Credentials → New → Google Gemini(PaLM) API |
| 4 | **Google Sheets OAuth2** | Queue Docs for Approval, Read Approval Status, Log Declined, Log Execution, Log Error | Credentials → New → Google Sheets OAuth2 → sivasaiboggu@gmail.com |
| 5 | **Telegram account** | Request Approval, Notify Error via Telegram | Talk to @BotFather, create a bot, paste the token; get your chat ID from @userinfobot |
| 6 | **Slack account** | Post Release Notes to Slack | Credentials → New → Slack → OAuth2 with `chat:write` scope |
| 7 | **Notion account** | Create Notion Release Page | Notion → Settings → Integrations → new internal integration → share the parent page with it |

No credentials are stored in `workflow.json` at all (no `credentials` property on any node) — n8n shows "No credential set" and you pick/create each one on import. Secrets go only in n8n credentials, never in node parameters.

## 3. Google Sheet — create this first

Create one Google Sheet and add these tabs (exact names matter — the nodes look them up):

| Tab | Columns (row 1, in order) |
|---|---|
| **Doc Approvals** | Repo, Commit, Release Notes, Status, Queued At — set Status to APPROVED or REJECTED |
| **Declined** | Repo, Commit, Status, Logged At |
| **Execution Log** | At, Workflow, Result, Detail |
| **Errors** | At, Node, Error, Workflow |

Then open every Google Sheets node in the workflow and paste the Sheet ID (the long string in the sheet URL) where it says `YOUR_GOOGLE_SHEET_ID`.

## 4. Placeholders to fill in

Search the workflow for `YOUR_` — every one is a value only you can supply (Sheet ID, chat/channel IDs, company/shop IDs, phone number ID). Nothing is pre-filled with invented data.

## 5. Test before turning on

1. Set YOUR_GITHUB_USERNAME / YOUR_REPO in the Get Commit Diff and Update README nodes, and YOUR_NOTION_PARENT_PAGE_ID in the Notion node. 2. Run Manual Trigger — a draft should appear in Doc Approvals as PENDING and nothing should change on GitHub/Slack/Notion. 3. Mark it APPROVED and re-run the approval branch (or wait out the 2h Wait node) — CHANGELOG.md updates, Slack gets the notes, Notion gets a page. 4. Test a REJECTED row to confirm nothing publishes.

## 6. Notes

The GitHub Trigger needs the webhook registered on the repo — n8n does this automatically when the workflow is active and the credential is connected. The bot only ever edits CHANGELOG.md via the GitHub node; change `filePath` if a repo keeps its changelog elsewhere. Drafts never invent features — Gemini is instructed to stick to the diff.

## 7. Error handling & logging

An **Error Trigger** node catches any node failure and writes it to the **Errors** tab, then notifies Siva on Telegram. Every run also writes a row to **Execution Log**. HTTP Request nodes use a 10-second timeout and expect JSON responses; where a node type has no built-in retry, re-run from the failed node after fixing the cause rather than editing history.

## 8. Human approval

Nothing irreversible happens without Siva: every posting / publishing / accounting step sits behind an approval sheet row that must read `APPROVED`. The workflow only ever drafts and queues until then.
