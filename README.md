# n8n Workflows — Siva

30 importable n8n workflows: 28 built overnight 20–21 Nov 2026 plus the two built earlier the same evening. Every workflow is
a valid n8n JSON file you can import as-is: no credential is set in any file,
so n8n shows "No credential set" on import and you pick/create each one there.

## The workflows

| # | Workflow | Folder |
|---|---|---|
| 1 | Digital Twin Slack Bot | `digital-twin-slack-bot/` |
| 2 | Job Hunter Autopilot | `job-hunter-autopilot/` |
| 3 | Price Drop Sniper | `price-drop-sniper/` |
| 4 | AI Podcast Factory | `ai-podcast-factory/` |
| 5 | Reputation Radar | `reputation-radar/` |
| 6 | Invoice and Receipt Vacuum | `invoice-receipt-vacuum/` |
| 7 | Habit Coach via WhatsApp | `habit-coach-whatsapp/` |
| 8 | Social Listening Opportunity Finder | `social-listening-opportunity-finder/` |
| 9 | Auto-Documentation Bot | `auto-documentation-bot/` |
| 10 | Smart Expense Policing | `smart-expense-policing/` |
| 11 | Image-to-Product-Listing | `image-to-product-listing/` |
| 12 | Crypto/Stock Whale Watcher | `crypto-stock-whale-watcher/` |
| 13 | Travel Deal Hunter | `travel-deal-hunter/` |
| 14 | Event-Based Customer Win-Back | `event-based-customer-win-back/` |
| 15 | Dream-Team Multi-Agent Pipeline | `dream-team-multi-agent-pipeline/` |
| 16 | AI Inbox Chief of Staff | `ai-inbox-chief-of-staff/` |
| 17 | Competitor Spy Bot | `competitor-spy-bot/` |
| 18 | Voice-Note-to-Everything | `voice-note-to-everything/` |
| 19 | Content Repurposing Machine | `content-repurposing-machine/` |
| 20 | Lead Sniper | `lead-sniper/` |
| 21 | Self-Healing Workflow Monitor | `self-healing-workflow-monitor/` |
| 22 | Personal Finance Autopilot | `personal-finance-autopilot/` |
| 23 | AI Customer Support Triage | `ai-customer-support-triage/` |
| 24 | News-to-Newsletter Pipeline | `news-to-newsletter-pipeline/` |
| 25 | Meeting Autopilot | `meeting-autopilot/` |
| 26 | Smart Home + Context Agent | `smart-home-context-agent/` |
| 27 | Autonomous Research Agent | `autonomous-research-agent/` |
| 28 | Daily Automation Self-Report | `daily-automation-self-report/` |
| 29 | Multi-agent Daily Brief (researcher → writer → reviewer, 07:00 IST) | `multi-agent-daily-brief/` |
| 30 | Job Search Autopilot (boards → research → cold mail, 09:15/19:00 IST) | `job-search-autopilot/` |

The two workflows from earlier the same evening (29–30) live in this repo too,
so this is the complete collection. Their originals are also at
`../job-search-autopilot/` and `../multi-agent-daily-brief/`.

## How to use

1. Read [IMPORT_ALL.md](IMPORT_ALL.md) — import order, what to connect per
   workflow, and activation notes.
2. In n8n: Workflows → Import from File → pick a workflow's `workflow.json`.
3. Each workflow's `SETUP.md` lists its credentials, sheet tabs, and how to
   test with Manual Trigger before activating.
4. Set the n8n instance timezone to Asia/Kolkata so the cron schedules fire
   at the right IST times.

## Rules baked into every workflow

- Gemini only (free tier) — no OpenAI, no Claude.
- Placeholder credentials only — secrets live in n8n credentials, never in nodes.
- Error handling + retry on every external call; executions logged to Sheets.
- A human-approval step before anything irreversible (sending, spending,
  posting, publishing).
- Job Hunter Autopilot: paid roles only, never Internshala.
