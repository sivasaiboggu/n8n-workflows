# Digital Twin Slack Bot — SETUP

## What it does
Slack trigger watches a channel. When a teammate asks a question, the workflow
looks up Siva's past emails/docs/messages (vector-store memory layer), and a
Gemini AI Agent answers in Siva's voice with sources. Low confidence (< 0.70)
or any reply that commits Siva to something is escalated to Siva on Slack
instead — nothing committing is ever sent without his approval.

## Import
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → Import from File → pick `workflow.json`.
2. The workflow imports with **"No credential set"** on every node that needs one
   (no credential IDs are stored in the file — this is deliberate). Click each node
   and select/create the credential listed below. Then save.

## Credentials to connect (pick on import — nothing is pre-filled)
| Node(s) | Credential needed |
|---|---|
| Slack Trigger, Escalate to Siva (Slack), Post Answer to Slack | Slack account (OAuth2) — needs access to the channel + DM to Siva |
| Log Pending Approval, Log Error to Sheet, Log Execution | Google Sheets account (OAuth2) |
| Gemini Chat Model | Google Gemini API key (free tier) |
| Notify Siva (Error) | Telegram account (bot token + chat ID) |

## Resources to create
- **Google Sheet** (one sheet, copy its ID into `YOUR_GOOGLE_SHEET_ID` in the Sheets nodes):
  - `Twin Memory Index` — columns: Source, Title, Snippet, Date. (Index of what is embedded in the vector store; fill as you embed material.)
  - `Pending Approvals` — columns: Asked At, Channel, Question, Draft Answer, Confidence, Reason, Status.
  - `Execution Log` — columns: At, Workflow, Status, Detail.
  - `Errors` — columns: At, Error, Node, Workflow.
- **Slack**: a channel for the bot (`YOUR_SLACK_CHANNEL_ID`) and Siva's Slack user ID (`YOUR_SIVA_SLACK_USER_ID`). Create a Slack app/bot and invite it to the channel.
- **Vector store (memory layer)**: NOT connected yet. The `Memory Lookup (Placeholder)` Code node returns an empty retrieval list on purpose. When Siva connects a vector store (e.g. Pinecone/Supabase pgvector/Qdrant), replace that node with a real retriever returning `[{source, snippet, score}]`. Until then the agent says it has no stored source instead of inventing one.

## Values to fill in (marked `YOUR_*` in the workflow)
- `YOUR_GOOGLE_SHEET_ID`, `YOUR_SLACK_CHANNEL_ID`, `YOUR_SIVA_SLACK_USER_ID`, `YOUR_TELEGRAM_CHAT_ID`.

## Test before activating
1. Click **Execute Workflow** with the Manual Trigger (use a test question via the Slack Trigger input or run the flow manually).
2. Check: a high-confidence, non-committal question posts an answer to Slack; a low-confidence or committing question lands in Slack DM to Siva + the Pending Approvals sheet.
3. Check the Execution Log sheet has a row. Then activate the Slack Trigger.

## Notes
- Retry: external-call nodes (Slack, Sheets, Telegram) have retry-on-fail (3 tries, 5 s apart). The Gemini agent node is retried via the workflow run on failure.
- Errors go to the Errors sheet + a Telegram alert to Siva.
