# Simterra-Luanti — Agent Connection Guide

Connect an AI agent to your Simterra game so it can manage your business while you play.

## How it works

1. In the Luanti game, type `/simterra link`
2. Click **OPEN TOKEN PAGE** → **Copy Full Message for AI**
3. Paste the message to your AI agent

That's it — your AI agent handles the rest.

## Prompt to send to your AI agent

After running `/simterra link`, copy the message from the token page. It looks like this:

```
You are connecting to Simterra (Luanti) as an AI agent.

Read this first and use it as the source of truth:
https://github.com/patng-max/simterra-connection/blob/main/docs/AI_AGENT_ONBOARDING.md

If your runtime supports persistent skills, playbooks, or memory, save these Simterra rules for future use. Otherwise use them for this session.

Connection details:
- Server: {SERVER_URL}
- Town: {TOWN}
- Character name: {CHARACTER_NAME}
- Actor ID: {ACTOR_ID}
- Agent ID: {AGENT_ID} (if missing, generate a stable one and reuse it)
- Registration token: {REGISTRATION_TOKEN}

Required steps:
1. Call POST {SERVER_URL}/agent/claim with registration_token and agent_id
2. Save returned agent_credentials as Bearer token
3. Call GET {SERVER_URL}/agent/observe?actor_id={ACTOR_ID}
4. If there is an active business, call POST {SERVER_URL}/agent/decision using action_type business/update-strategy
5. Use conservative values only
6. Report exact claim / observe / decision results and any errors
```

Replace the `{}` placeholders with the values from the token page.

## For AI agents

Full technical details, error codes, request shapes, and operating rules:

📄 **[docs/AI_AGENT_ONBOARDING.md](./docs/AI_AGENT_ONBOARDING.md)**

📋 **[docs/AI_AGENT_PROMPT_TEMPLATE.md](./docs/AI_AGENT_PROMPT_TEMPLATE.md)**

## Game rules (brief)

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
| 400 | Bad input (price, stock) |
