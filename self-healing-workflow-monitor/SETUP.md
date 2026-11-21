# Self-Healing Workflow Monitor — Siva — SETUP

## What it does
The Error Trigger fires on any workflow failure → the error is logged to Sheets
→ Gemini reads the error and the failing node and **suggests** a fix (JSON:
`{cause, suggested_fix, confidence, safe_to_retry}`) → the diagnosis is logged
and posted to Slack with a retry option → a human must approve the retry; the
retry re-runs the failed execution via the n8n API. **Fixes are NEVER
auto-applied** — Gemini only suggests, and the retry only re-runs, it does not
edit any workflow.

## Import
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → Import from File → select
   `workflow.json` in this folder.
2. **Credentials rule (corrected 20 Nov 2026): this file sets NO credential on any
   node** — n8n refuses to save an import that references a null-id credential
   (this exact failure hit the daily-brief import). On import each node shows
   "No credential set": click it and pick/create the credential listed below.
   The same list is in the "Sticky Note — Credentials" node in the workflow.
3. Leave the workflow **inactive** until the test below passes, then Activate.
4. In n8n, attach this workflow as the **Error Workflow** (Settings → Error
   workflow, or per-workflow error-workflow setting) for the workflows you want
   monitored — otherwise the Error Trigger never fires.

## Credentials to connect (pick/create on import — none are in the file)
| Node(s) | Credential to connect |
|---|---|
| Log Error to Sheet, Log Diagnosis to Sheet, Log Execution | Google Sheets (OAuth2) |
| Post Slack Alert with Retry Option | Slack API (channel = `YOUR_SLACK_CHANNEL_ID`) |
| Retry Execution via n8n API | n8n API key — also set `YOUR_N8N_HOST` in the node URL (e.g. `https://sivasai-boggu.app.n8n.cloud`) |
| Gemini Chat Model | Google Gemini (PaLM) API (Gemini free tier) |

## Sheets / resources to create
- **Errors** tab (error-log sheet `YOUR_ERROR_LOG_SHEET_ID`) — columns:
  `timestamp`, `workflow`, `node`, `error`, `execution_id`.
- **Diagnoses** tab (same sheet) — columns: `timestamp`, `workflow`, `node`,
  `error`, `suggested_fix`, `status` (this tab IS the memory layer — past
  failures help Gemini recognise repeat failures).
- **Log** tab (execution-log sheet `YOUR_EXECUTION_LOG_SHEET_ID`) — columns: `timestamp`, `workflow`, `status`, `detail`.

## Test (before activating)
1. Use the **Manual Trigger (Test)** node with test data shaped like an error
   payload, e.g. `{"workflow": {"name": "Test Workflow"}, "node": "Test Node",
   "error": "Test error", "execution": {"id": "123", "lastNodeExecuted": "Test Node"}}`.
   (Manual Trigger feeds Log Error to Sheet directly so you can test without a
   real failure.)
2. Check: error + diagnosis rows are logged, the Slack alert posts with the
   suggested fix marked NOT applied, and nothing is retried yet.
3. Reply RETRY / approve and confirm the n8n API retry call is attempted for
   the test execution; repeat with IGNORE and confirm no retry and
   "Log Decision — No Retry" is recorded.
4. Trigger a real failure in a small test workflow with this workflow set as
   its error workflow, and confirm the Error Trigger fires end-to-end.
5. Only then Activate (and set it as error workflow on the workflows to monitor).

## Notes / limits
- This workflow's own error handling: its trigger IS the Error Trigger, wired
  to Log Error to Sheet → diagnosis → Slack alert. Its own failures surface in
  n8n's execution log / Log Execution.
- HTTP Request nodes use timeout 10,000 ms and JSON response format; workflow
  Settings retry covers retries where the node type has no built-in retry.
- The Wait node waits up to 4 hours for a human decision; swap for a Form
  Trigger if you prefer a click approval.
- Retrying re-runs the failed execution as-is; it does not fix the underlying
  cause — apply Gemini's suggested fix by hand, in n8n, after reviewing it.
