# Travel Deal Hunter — Siva — SETUP

## What it does
Every 6 hours it reads a trip watchlist from Google Sheets, checks Google
Calendar for free gaps, fetches flight + hotel prices, alerts on Telegram when a
price drops below your target, and Gemini drafts a day-by-day itinerary from
your preferences + calendar gaps. **Human approval before any booking — this
workflow never books anything.** If you write `APPROVE` in the Approval tab,
it only sends a "ready to book (manual)" Telegram note; you book yourself.

## Import
**Credentials rule (corrected 20 Nov 2026): this file sets NO credential on any
node** — n8n refuses to save an import that references a null-id credential
(this exact failure hit the daily-brief import). On import each node shows
"No credential set": click it and pick/create the credential listed below.
The same list is in the "Sticky Note — Credentials" node in the workflow.
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → ⋯ → Import from File →
   select `workflow.json` in this folder. Leave inactive until tested.
2. Replace every `YOUR_...` placeholder: `YOUR_GOOGLE_SHEET_ID`,
   `YOUR_TELEGRAM_CHAT_ID`, `YOUR_HOME_AIRPORT` (e.g. HYD), `YOUR_DESTINATION`,
   `YOUR_DATE_FROM` / `YOUR_DATE_TO` (DD/MM/YYYY for the Kiwi API), and the
   Kiwi Tequila API credential (header auth).

## Credentials to connect (pick/create on import — none are in the file)

No credential is set in `workflow.json` (see Import above) — the table below is what to
pick/create for each node; no real key or secret is stored anywhere in this repo.
| Credential | Where |
|---|---|
| Google Gemini(PaLM) Api | Gemini Chat Model |
| Google Sheets account | Read Travel Watchlist, Save Draft Itinerary, Check Approval Sheet, Log Execution, Log Error to Sheet |
| Google Calendar account | Read Calendar Gaps |
| Telegram account | Send Deal Alert via Telegram, Notify — Ready to Book, Notify Error via Telegram |
| Kiwi Tequila API (HTTP Header Auth) | Fetch Flight Prices, Fetch Hotel Prices — get a free key at tequila.kiwi.com and store it here, never in a node parameter |

## Google Sheet to create (id = YOUR_GOOGLE_SHEET_ID)
- Tab `Travel Watchlist` — columns: `destination | date_from | date_to | home_airport | target_price | last_seen_price | budget_inr | pace | interests`
- Tab `Draft Itineraries` — columns: `timestamp | destination | itinerary | status`
- Tab `Approval` — columns: `destination | approval` (write `APPROVE` here to allow the manual-booking notice; anything else = awaiting approval)
- Tab `Execution Log` — columns: `timestamp | workflow | status | summary`
- Tab `Error Log` — columns: `timestamp | workflow | error | node`

## How to test before activating
1. Add one watchlist row with a deliberately high `target_price` so a drop is
   detected, then click **Execute Workflow**.
2. Check Telegram for the deal alert, the Draft Itineraries tab for the draft,
   and the Execution Log tab for a row.
3. Leave Approval blank → the run should log "awaiting approval" and book
   nothing. Then write `APPROVE` for that destination, run again, and confirm
   you only get the "Ready to Book (Manual)" Telegram note — still no booking.
4. Only then toggle **Active**.

## Notes / limits
- Flight + hotel prices both use the Kiwi Tequila search endpoint as a
  placeholder provider — swap the hotel node URL for your preferred hotel API
  if you use a different one; the price-drop logic only needs a numeric
  `price`/`total` field in the response.
- Retry: HTTP Request nodes use a 10s timeout and JSON response format; the
  6-hour schedule re-runs automatically, and failures are logged to Error Log.
