# Trail Running Assistant — n8n AI Agent

## Overview

An n8n workflow that automatically decides whether today is a good day for a trail run and, if so, emails a trail recommendation. It uses an AI Agent (powered by Claude) that reasons over your calendar, the weather, and a list of candidate trails.

## How It Works

1. **Trigger** fires (Schedule Trigger — set to run once daily, e.g. each morning).
2. The **AI Agent** node runs, using Claude (Anthropic Chat Model) as its reasoning engine and a system prompt defining its role, task, and constraints.
3. The agent calls **CheckCalendar** to look for an event titled "Trail Run" scheduled for today. If there isn't one, it stops and does nothing.
4. If there's a Trail Run event, the agent calls **GetWeather** for current conditions in Austin, TX.
5. The agent calls **GetTrailList** to pull the list of candidate trails (name, miles, elevation gain, estimated time, shade level).
6. It picks one trail that fits the time available, matches today's weather/shade needs, and hasn't been recommended too recently.
7. The agent calls **SendMail** to email a short recommendation (trail name, distance, elevation, estimated time, and why it fits today) — or an explanation if no trail fits or the weather's no good.

## Workflow Nodes

| Node | Type | Role |
|---|---|---|
| Trigger | Schedule Trigger | Kicks off the workflow on a daily schedule |
| AI Agent | LangChain Agent node | Orchestrates reasoning and tool calls |
| Anthropic Chat Model | Chat Model sub-node | Claude — the LLM behind the agent |
| Simple Memory | Memory sub-node | Keeps session context (n8n's built-in local memory) |
| CheckCalendar | Tool (Google Calendar) | Checks for today's "Trail Run" event |
| GetWeather | Tool (Weather) | Returns current temperature/conditions for Austin, TX |
| GetTrailList | Tool (Google Sheets) | Reads the trail list spreadsheet |
| SendMail | Tool (Gmail) | Sends the final recommendation or status message |

## System Prompt

```
Role: You are a smart, weather-aware trail running assistant. You help me decide whether I should go for a trail run today and which trail would be the best match. Your tone should be friendly, helpful, and concise - like a knowledgeable friend who runs.

Task: Decide if I should go for a trail run today. If so, recommend the best trail based on time, weather, and trail attributes. If it's not a good day to run, explain why and stop.

Input: You have access to the following:
- My calendar via the CheckCalendar tool. You're looking for an event titled "Trail Run" scheduled for today.
- The current weather in Austin, Texas via the GetWeather tool.
- A list of trail runs from the GetTrailList tool. Each trail includes:
  - Name
  - Distance (miles)
  - Elevation gain (feet)
  - Estimated time (minutes)
  - Shade Level: "Shady", "Some Shade", or "Exposed"

Tools: Use only these tools to perform your reasoning and take action:
- CheckCalendar: Returns today's events.
- GetWeather: Returns temperature and conditions in Austin, Texas.
- GetTrailList: Returns a list of trails with the fields described above.
- SendMail: Sends me your final recommendation or status message.

Constraints:
- First, use CheckCalendar. If there is no event titled "Trail Run" today, do nothing.
- Use GetWeather. If it's very hot or sunny, avoid trails marked "Exposed" and prefer "Shady" or "Some Shade".
- Use GetTrailList and choose one trail that:
  - Fits within the estimated time range
  - Matches today's weather and shade needs
  - Is not repeated too frequently
- Don't recommend anything if no trail fits the constraints.

Output: Use the SendMail tool to send a friendly summary like:
- The name of the recommended trail
- Distance, elevation, and estimated time
- A short reason why it's a great fit today based on weather

If no trail is appropriate, explain why clearly and politely. Keep it brief and helpful.
```

## Data Source

The trail list lives in a Google Sheet with columns: **Name, Miles, Elevation, Estimated Time, Shade Level** — matching the Austin-area trails reference sheet (Barton Creek Greenbelt, River Place, Wild Basin, McKinney Falls, etc.).

## Known Issues / Things to Double-Check

- The **CheckCalendar** node's canvas subtitle showed "create: event" rather than a read/lookup operation — worth confirming it's actually set to look up existing events (e.g. "Get Many"), not create one.
- The Anthropic Chat Model rejected requests containing smart/curly punctuation (em dashes "—", curly quotes " " ' ') copy-pasted from other apps — Claude's API requires plain ASCII (Latin-1) characters in request text. Retype rather than paste when editing node text fields.
- **Simple Memory** needs a valid session ID. It auto-populates when the workflow starts from n8n's native Chat Trigger node; with any other trigger (like a Schedule Trigger), set Session ID to "Define Below" and supply your own expression.
