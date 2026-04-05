# Simterra-Luanti — Agent Connection Guide

Connect an AI agent to your Simterra game so it can manage your business while you play.

## How it works

1. In the Luanti game, type `/simterra link`
2. Click **OPEN TOKEN PAGE** → **Copy Full Message for AI**
3. Paste the message to your AI agent

That's it — your AI agent handles the rest.

## Quick Start

```
/simterra link  →  Luanti opens browser with token page
                →  Click "Copy Full Message for AI"
                →  Paste to your AI agent
                →  AI agent connects and starts managing your business
```

## Prompt to send to your AI agent

Paste this message to your AI agent after running `/simterra link`:

```
You are connecting to Simterra (Luanti) as an AI agent.

IMPORTANT:
- Read and follow this connection guide first:
  https://github.com/patng-max/simterra-connection
- Use that guide as the source of truth.
- If your runtime supports persistent skills, playbooks, or memory files,
  create or update one called: simterra-luanti
- If your runtime does not support skills/playbooks,
  treat the instructions in this message as your active operating rules for this session.
- Do not assume success. Actually perform the HTTP requests and report exact results.

Connection details:
- Server: http://localhost:3000
- Town: town_1
- Character name: YOUR_CHARACTER_NAME
- Actor ID: YOUR_ACTOR_ID
- Agent ID: YOUR_AGENT_ID
- Registration token: YOUR_TOKEN

Required steps:
1. Read docs/AI_AGENT_ONBOARDING.md in the repo first.
2. Call POST /agent/claim with registration_token and agent_id.
3. Save the returned agent_credentials as your Bearer token.
4. Call GET /agent/observe?actor_id=YOUR_ACTOR_ID with Bearer token.
5. If there is an active business, call POST /agent/decision using business/update-strategy.
6. Use conservative values only. Never set negative stock. Never set invalid prices.
7. Report exact results of each step.

Report back: skill saved?, claim status, observe status, decision committed/rejected, any errors.
```

Replace `YOUR_*` values with the actual values from the token page.

## For AI agents

Full technical details, error codes, request shapes, and operating rules:

📄 **[docs/AI_AGENT_ONBOARDING.md](./docs/AI_AGENT_ONBOARDING.md)** — read this first

📋 **[docs/AI_AGENT_PROMPT_TEMPLATE.md](./docs/AI_AGENT_PROMPT_TEMPLATE.md)** — reusable prompt template

## Game Rules (brief)

- Prices must be ≥ template minimum
- Stock targets must be ≥ 0
- Business must be `active`, lot must be `leased` or `owned`
- Only supported action: `business/update-strategy`

## Token expiry

- Registration token: **5 minutes** — refresh by running `/simterra link` again
- Agent credentials: **24 hours**

## Error codes

| Code | Meaning |
|------|---------|
| 401 | Invalid/expired bearer token — request fresh registration |
| 403 | Actor ID mismatch |
| 404 | Business or lot not found |
| 409 | CAS conflict — re-observe before retry |
| 400 | Bad input (price, stock value) |
