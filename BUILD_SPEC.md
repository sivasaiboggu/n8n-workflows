# Overnight build spec — 27 n8n workflows for Siva

Owner: Siva (Sivasai Boggu), BTech CS IIIT Kottayam 2023-2027, CGPA 7.73,
sivasaiboggu@gmail.com, +91 9573719115, Gooty, Whispry intern ₹20,000/month,
available immediately, paid roles only.
n8n instance: sivasai-boggu.app.n8n.cloud (Cloud trial).
Tools available: Gmail, Google Sheets, Telegram, Gemini free tier (Google Gemini
API via @n8n/n8n-nodes-langchain.lmChatGoogleGemini), Google Drive, Slack, WhatsApp
(notify only via approved channels — use Telegram where a chat channel is needed
unless the idea explicitly says WhatsApp/Slack).
NEVER use OpenAI or Claude. NEVER Internshala anywhere. No invented data,
no fake credentials, no invented API keys, emails or prices.

## Output layout (exact)

For each workflow, directory `~/workspace/n8n/workflows/<slug>/` containing:
- `workflow.json` — importable n8n workflow JSON
- `SETUP.md` — import steps, credentials to connect, sheet columns / resources needed

## workflow.json format (follow exactly)

```json
{
  "name": "Job Hunter Autopilot — Siva",
  "nodes": [
    {
      "parameters": {"rule": {"interval": [{"field": "hours", "hoursInterval": 6}]}},
      "id": "a1b2c3d4-0001-4000-8000-000000000001",
      "name": "Every 6 hours",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [0, 0]
    },
    {
      "parameters": {"model": "gemini-2.0-flash", "options": {"temperature": 0.3}},
      "id": "a1b2c3d4-0002-4000-8000-000000000002",
      "name": "Gemini Chat Model",
      "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
      "typeVersion": 1,
      "position": [400, 200],
      "credentials": {"googlePalmApi": {"id": null, "name": "Google Gemini(PaLM) Api"}}
    }
  ],
  "connections": {
    "Every 6 hours": {"main": [[{"node": "Next Node Name", "type": "main", "index": 0}]]}
  },
  "active": false,
  "settings": {"executionOrder": "v1"},
  "pinData": {}
}
```

Rules:
- Every node: unique UUID `id`, `name`, real n8n `type`, `typeVersion`, `position`
  ([x, y] numbers, laid out left→right, ~240px apart).
- Use real node types only, e.g. n8n-nodes-base.scheduleTrigger,
  n8n-nodes-base.webhook, n8n-nodes-base.manualTrigger,
  n8n-nodes-base.gmail, n8n-nodes-base.googleSheets,
  n8n-nodes-base.googleDrive, n8n-nodes-base.telegram,
  n8n-nodes-base.slack, n8n-nodes-base.httpRequest,
  n8n-nodes-base.rssFeedRead, n8n-nodes-base.code,
  n8n-nodes-base.if, n8n-nodes-base.set, n8n-nodes-base.wait,
  n8n-nodes-base.splitInBatches, n8n-nodes-base.errorTrigger,
  n8n-nodes-base.switch, n8n-nodes-base.merge,
  @n8n/n8n-nodes-langchain.agent,
  @n8n/n8n-nodes-langchain.lmChatGoogleGemini.
- AI Agent nodes connect to the Gemini chat model node via an `ai_languageModel`
  connection (see n8n convention: connections key "Gemini Chat Model" with
  type "ai_languageModel").
- Credentials: OMIT the `credentials` property ENTIRELY from every node. Do NOT
  include `{"id": null, "name": "..."}` — n8n treats that as a reference to a
  credential not shared with the account and refuses to save the imported
  workflow (this exact failure hit the daily-brief import). With no credentials
  property, n8n shows "No credential set" and the user picks/creates one on
  import — that is the importable-with-no-edits behaviour. Name the credential
  the node needs in a Sticky Note and in SETUP.md instead. NEVER a real
  id/key/secret. Secrets only in credentials, never in node parameters. Mark
  any ID the user must fill as `YOUR_<THING>` in parameters.
- Every `connections` entry must reference a node `name` that exists in `nodes`,
  with matching output/input types (`main` or `ai_languageModel`).
- Every workflow MUST include: an Error Trigger node wired to a "Log Error to
  Sheet" Google Sheets node + a notify step; retry on every external call
  (HTTP Request nodes: `"options": {"response": {"response": {"responseFormat": "json"}}, "timeout": 10000}` and note retry in SETUP where the node type lacks built-in retry — use the workflow Settings retry or a Code-node retry loop where applicable); a human-approval step (Telegram/Slack/Gmail Approve step or an IF node on an approval sheet value) before ANY irreversible action (sending emails, spending money, posting publicly, deleting); and a Google Sheets "Log Execution" step.
- Add a Sticky Note node describing the flow in 2-3 lines, and a memory-layer
  note where the idea calls for learning from past runs (Postgres/vector DB —
  as a Sticky Note + a Code/Set node marking what would be stored; do NOT
  invent a live DB connection).
- Job Hunter Autopilot: use Siva's real profile (above), score roles against
  his CV, tailor resume/cover letter per job, save to Drive, Telegram
  Apply/Skip buttons, NEVER Internshala boards (Wellfound, Built In, Shine,
  Jobfound, Himalayas, Cutshort, HiringCafe, LinkedIn Jobs only).
- No node may claim a live result it cannot produce (no invented leads,
  prices, emails, or transcripts).

## Validation (mandatory before reporting done)

1. `python3 -m json.tool <slug>/workflow.json > /dev/null` — must pass.
2. Run a connection check: every node name in `connections` (both keys and
   targets) must exist in `nodes`. Write a tiny python check or do it by hand
   and report the result.
3. SETUP.md must list: how to import, each credential to create/connect,
   every sheet/tab or resource to create with columns, and how to test with
   Manual Trigger before activating.

## Report back

Node count, files written, validation results, and anything you could not
verify (e.g. not imported into live n8n — that is expected).
