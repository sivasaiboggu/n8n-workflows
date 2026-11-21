# Smart Home + Context Agent — Siva — SETUP

## What it does
Every 30 min → weather (Open-Meteo, Gooty) + upcoming calendar events →
Gemini decides safe home actions (lights, thermostat, "leave now, traffic is
bad" alert) → **Safety Check**: safe actions execute via a Home Assistant
webhook; anything flagged (or money/door-related) goes to **human approval**
first. Every decision is logged to Sheets.

## 1. Import
1. In n8n (`sivasai-boggu.app.n8n.cloud`): Workflows → Import from File →
   select `workflow.json`.
2. Imports **inactive** (`active: false`). Do not activate until the test
   in §5 passes.

## 2. Credentials to connect (NO credential is set in workflow.json — see note below)
| Credential (as named in the file) | What to create |
|---|---|
| Google Gemini API | Google Gemini (PaLM) API — Gemini API key, free tier. |
| Google Calendar account | Google Calendar OAuth2. "Check Calendar" reads the next 5 events. |
| Telegram account | Telegram Bot API — bot token + `YOUR_TELEGRAM_CHAT_ID`. Approval requests, action alerts, error alerts. |
| Google Sheets account | Google Sheets OAuth2. Home Log / Execution Log / Error Log / Approval Sheet. |
| Home Assistant Long-Lived Token | HTTP Header Auth — Home Assistant long-lived access token (`Authorization: Bearer <token>`). Used by "Execute via Home Assistant". |

CORRECTED RULE: the `credentials` property is omitted entirely from every node in workflow.json. n8n treats a null-id credential placeholder as a reference to a credential not shared with the account and refuses to save the import (this exact failure hit the Daily Brief import). With no credentials property, n8n shows "No credential set" on each node — pick an existing credential or create a new one on import. The per-node credential list is also in the "Sticky Note — Credentials" inside the workflow. No real key/secret is stored in the JSON. Open-Meteo needs no key.

## 3. Google Sheet — create one sheet, id = `YOUR_GOOGLE_SHEET_ID`
Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node. Tabs/columns:
- **Execution Log**: `timestamp | workflow | status | summary`
- **Error Log**: `timestamp | workflow | error | node`
- **Approvals**: `key | value | updated_at` — one row: `key=latest`,
  `value=pending`. Approve with `value=approved`, reject with
  `value=rejected`. The workflow waits 60 minutes after the Telegram
  request, then reads this row. Rejected → run goes to Merge → Log
  Execution with no action executed.
- **Home Log**: `timestamp | actions | alert | reason` — every run logs here,
  including runs where the agent decided "no action".

## 4. Resources / placeholders to fill — READ THIS BEFORE ACTIVATING
- **`YOUR_HOME_ASSISTANT_URL`**: replace with your real Home Assistant URL
  (e.g. `http://homeassistant.local:8123` — must be reachable from n8n Cloud;
  a local-only URL will **not** work from the cloud instance; use Nabu Casa
  or another externally reachable URL).
- **Entities / services — per-action configuration required.** The
  "Execute via Home Assistant" node is a starting template: it calls
  `/api/services/light/turn_on` and passes the agent's actions array in the
  body. Home Assistant expects one service call per action
  (`light.turn_on`, `climate.set_temperature`, …) with a real `entity_id`.
  Before activating, replace this node with one HTTP call per service you
  actually use, or add a Code/Switch node that routes each action to the
  right service + entity. **Do not activate with the template URL as-is —
  it will fail or do nothing.** List your real entity IDs (e.g.
  `light.living_room`, `climate.thermostat`) in the agent prompt / routing
  node so the agent can only pick entities that exist.
- **Calendar**: replace `YOUR_CALENDAR_ID` (usually
  `sivasaiboggu@gmail.com` / `primary`).
- **Weather**: Open-Meteo coordinates in the URL are Gooty
  (15.33, 77.63), timezone Asia/Kolkata. "Leave now" alerts are based on
  calendar timing, not live traffic — there is no traffic API wired in;
  the agent infers urgency from event start time. Add a traffic source
  later if you want real traffic data; do not treat the alert as live
  traffic until then.

## 5. Test (Manual Trigger first, before activating)
1. Click **Execute Workflow** (Manual Trigger) with Home Assistant execution
   **disabled first**: disconnect or disable "Execute via Home Assistant"
   for the first run, and verify the agent's proposed actions in the run
   output are safe (lights/thermostat only, real entity IDs, no money/door).
2. Verify the "Send Alert" Telegram message and the Home Log row.
3. Force the approval path: temporarily set the agent to propose an action
   with `needs_approval=true` (or edit "Safety Check") and verify the
   Telegram approval request arrives, Approvals `value=approved` lets it
   execute, and `value=rejected` executes nothing.
4. Re-enable execution only after step 1 passes, restore the Wait duration
   if shortened, then activate.

## 6. Safety (hard rule, also in the workflow's sticky note)
- **No action that spends money or unlocks doors, ever, without human
  approval** — and the agent is instructed never to propose those at all.
  The approval path is the only route for flagged actions.
- Retry: HTTP Request nodes use `timeout: 10000` and JSON response format;
  enable workflow Settings retry for automatic retries.
- Error handling: Error Trigger → Log Error to Sheet → Telegram alert to Siva.
