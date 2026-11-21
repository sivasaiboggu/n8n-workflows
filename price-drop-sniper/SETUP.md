# Price Drop Sniper — SETUP

## What it does
Every 2 hours it reads the Watchlist sheet, fetches each product/listing page
(product pages, Facebook Marketplace, eBay), compares the live price to the
stored baseline, and when a price has dropped 10%+ or hit the target price, a
Gemini agent checks resale value. If the estimated profit is >= 500, Siva gets
an instant Telegram alert with the profit estimate. **It never buys anything** —
there is no purchase node in the workflow. Siva buys manually if he agrees.

## Import
1. In n8n (sivasai-boggu.app.n8n.cloud): Workflows → Import from File → pick `workflow.json`.
2. The workflow imports with **"No credential set"** on every node that needs one
   (no credential IDs are stored in the file — this is deliberate). Click each node
   and select/create the credential listed below. Then save.

## Credentials to connect (pick on import — nothing is pre-filled)
| Node(s) | Credential needed |
|---|---|
| Read Watchlist, Log Price Not Found, Log No Drop, Log Alert, Log Error to Sheet, Log Execution | Google Sheets account (OAuth2) |
| Gemini Chat Model | Google Gemini API key (free tier) |
| Telegram Price Alert, Notify Siva (Error) | Telegram account (bot token + chat ID) |

## Resources to create
- **Google Sheet** (copy its ID into `YOUR_GOOGLE_SHEET_ID`):
  - `Watchlist` — columns: Product, URL, Source, Baseline Price, Target Price. Add one row per product you want watched. Baseline Price is the price you last saw / paid attention to; Target Price is the price at which you want an alert even without a 10% drop.
  - `Price Log` — columns: Product, Source, URL, Price, Baseline Price, Checked At.
  - `Alerts` — columns: Product, Source, URL, Price, Baseline Price, Drop %, Resale Estimate, Profit Estimate, Alerted At.
  - `Execution Log` — columns: At, Workflow, Status, Detail.
  - `Errors` — columns: At, Error, Node, Workflow.
- **Telegram bot**: create via @BotFather, get the bot token, and your chat ID (`YOUR_TELEGRAM_CHAT_ID`).

## Values to fill in (marked `YOUR_*` in the workflow)
- `YOUR_GOOGLE_SHEET_ID`, `YOUR_TELEGRAM_CHAT_ID`.

## Test before activating
1. Add one product to the Watchlist with a Baseline Price above its current price (or a Target Price above the current price) so it triggers.
2. Click **Execute Workflow** with the Manual Trigger.
3. Check: the Price Log sheet has a row, and if the drop/profit gates pass, a Telegram alert arrives with the profit estimate.
4. Then activate the every-2-hours schedule.

## Notes
- Prices come only from the fetched page or the sheet — never invented. If a price cannot be parsed (Facebook Marketplace and eBay often block scraping), the item is logged as "price not found" and skipped.
- Resale/profit numbers are rough AI estimates — verify the listing yourself before buying.
- Retry: the HTTP fetch node has retry-on-fail (3 tries, 5 s apart) and a 15 s timeout; Sheets/Telegram nodes also retry.
- Errors go to the Errors sheet + a Telegram alert to Siva.

## Notes / limits
- **Approval-gate exception (documented):** this workflow only alerts Siva via Telegram; it never buys anything. No approval gate is needed because there is no irreversible action.
