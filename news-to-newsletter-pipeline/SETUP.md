# News-to-Newsletter Pipeline — Siva — SETUP

## What it does
Daily 07:00 → RSS (Hacker News) + Hacker News API + Reddit r/technology →
dedupe → Gemini ranks and summarises the top 5 in Siva's tone (direct, fast,
evidence-led) → **DRAFT saved to Beehiiv**. It **never auto-sends** — Siva
reviews and sends with one click. Telegram notifies when the draft is ready.

## 1. Import
1. In n8n (`sivasai-boggu.app.n8n.cloud`): Workflows → Import from File →
   select `workflow.json`.
2. Imports **inactive** (`active: false`). Do not activate until the test
   in §5 passes.

## 2. Credentials to connect (NO credential is set in workflow.json — see note below)
| Credential (as named in the file) | What to create |
|---|---|
| Google Gemini API | Google Gemini (PaLM) API — Gemini API key, free tier. Used by "Gemini Chat Model". |
| Google Sheets account | Google Sheets OAuth2. Used by Read Past Newsletter Log / Log Execution / Log Error. |
| Telegram account | Telegram Bot API — bot token + `YOUR_TELEGRAM_CHAT_ID`. Used for the draft-ready notice and error alerts. |
| Beehiiv API Key | HTTP Header Auth — Beehiiv API key. Used by "Save Draft to Beehiiv". |

CORRECTED RULE: the `credentials` property is omitted entirely from every node in workflow.json. n8n treats a null-id credential placeholder as a reference to a credential not shared with the account and refuses to save the import (this exact failure hit the Daily Brief import). With no credentials property, n8n shows "No credential set" on each node — pick an existing credential or create a new one on import. The per-node credential list is also in the "Sticky Note — Credentials" inside the workflow. No real key/secret is stored in the JSON. RSS, Hacker News (Algolia API) and
Reddit need no credentials.

## 3. Google Sheet — create one sheet, id = `YOUR_GOOGLE_SHEET_ID`
Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node. Tabs/columns:
- **Execution Log**: `timestamp | workflow | status | summary`
- **Error Log**: `timestamp | workflow | error | node`
- **Newsletter Log**: `date | subject | stories | opens | clicks` — the
  workflow reads this tab (lookup `key=latest` is not used here; the node
  reads the tab) so topics are not repeated. Fill `opens`/`clicks` manually
  from Beehiiv after each send — future ranking should weight what readers
  actually opened.

## 4. Resources / placeholders to fill
- **Beehiiv**: replace `YOUR_PUBLICATION_ID` in the "Save Draft to Beehiiv"
  URL (`https://api.beehiiv.com/v2/publications/YOUR_PUBLICATION_ID/posts`)
  with your real publication ID, and confirm the `status: "draft"` field —
  the post must be created as a draft, never published, by this node.
- **Substack alternative**: Substack has no public write API for creating
  drafts, so the Beehiiv HTTP node cannot just be repointed at Substack. If
  you prefer Substack, keep this workflow as the draft generator and paste
  the draft (from the "Draft Newsletter" node output / Execution Log) into
  Substack manually — or use Beehiiv as the sender.
- **Sources**: RSS feed is `https://news.ycombinator.com/rss`; Hacker News
  via `hn.algolia.com/api/v1/search?tags=front_page`; Reddit via
  `reddit.com/r/technology/top/.json`. Edit these URLs to change sources.
  Reddit's public JSON endpoint can rate-limit — if it fails, the Error
  Trigger logs it and the run stops; the other sources are not sent alone.

## 5. Test (Manual Trigger first, before activating)
1. Click **Execute Workflow** (Manual Trigger). The three source nodes fan
   out from the trigger and merge in "Merge Sources".
2. Verify: "Dedupe & Normalise" outputs ≤ 40 items with real URLs;
   "Rank & Summarise Agent" returns 5 stories, each with a URL from the
   input (check no invented URL); "Draft Newsletter" produces a subject +
   body in Siva's tone.
3. Verify a **draft** (not published) post appears in Beehiiv, and the
   Telegram "draft ready" message arrives.
4. Check the Execution Log row. Confirm nothing was sent to subscribers.
5. Only then activate — the Schedule node runs daily at 07:00 IST (cron
   `0 7 * * *`; check the n8n instance timezone is Asia/Kolkata).

## 6. Notes / limits
- Retry: HTTP Request nodes use `timeout: 10000` and JSON response format;
  enable workflow Settings retry if you want automatic retries.
- Error handling: Error Trigger → Log Error to Sheet → Telegram alert to Siva.
- The agent is instructed to use only URLs from the fetched input and to
  avoid repeating topics from the Newsletter Log — verify the first few
  drafts by eye before trusting this fully.

## Notes / limits
- **Approval-gate exception (documented):** this workflow saves a DRAFT to Beehiiv only and notifies Siva; nothing is published or sent until he sends it himself from Beehiiv. No approval gate is needed because the draft is not an irreversible action.
