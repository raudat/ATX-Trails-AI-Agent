# Requirements

## Platform

- An n8n instance (Cloud or self-hosted) with LangChain / AI Agent nodes available.
- If using n8n Cloud's free trial, confirm you're within the plan's active-workflow limit — the Active toggle may be blocked otherwise.

## Credentials / Connected Accounts

| Credential | Used By | Notes |
|---|---|---|
| Anthropic API key | Anthropic Chat Model node | Powers the AI Agent's reasoning (Claude) |
| Google account (OAuth2) — Calendar scope | CheckCalendar node | Reads today's events to look for "Trail Run" |
| Google account (OAuth2) — Sheets scope | GetTrailList node | Reads the trail data spreadsheet |
| Google account (OAuth2) — Gmail scope | SendMail node | Sends the recommendation email |
| Weather API credential/key (if required by the weather node used) | GetWeather node | Returns current conditions for Austin, TX |

## Data Requirements

- A Google Sheet with the trail list, containing at minimum these columns:
  - Name
  - Miles
  - Elevation (feet)
  - Estimated Time
  - Shade Level ("Shady", "Some Shade", or "Exposed")
- A Google Calendar event titled exactly **"Trail Run"** on any day you want the agent to produce a recommendation. No event = no run suggested.

## n8n Nodes Used

- Schedule Trigger
- AI Agent
- Anthropic Chat Model (sub-node)
- Simple Memory (sub-node)
- Google Calendar (as CheckCalendar tool)
- Weather node (as GetWeather tool)
- Google Sheets (as GetTrailList tool)
- Gmail (as SendMail tool)

## Text Input Notes

- All text pasted into node fields (system prompt, tool descriptions) must use plain ASCII punctuation. Curly quotes, apostrophes, and em/en dashes from word processors or notes apps will cause a "Cannot convert argument to a ByteString" error when sent to the Anthropic API.

## Optional (for production/scaling)

- Simple Memory stores session data locally in the n8n instance and isn't shared across workers. For production use with scaling (Queue Mode, multiple workers), swap in an external memory store such as Redis or Postgres instead.
