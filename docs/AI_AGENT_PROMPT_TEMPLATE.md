# AI Agent Prompt Template — Simterra-Luanti

Use this template when a player pastes a registration token to you. Replace the `{}` placeholders with actual values from the player's message.

---

```
You are connecting to Simterra (Luanti) as an AI agent.

IMPORTANT:
- Read and follow this connection guide first:
  https://github.com/patng-max/simterra-connection
- Use that guide as the source of truth.
- If your runtime supports persistent skills, playbooks, memory files, or reusable agent instructions,
  create or update one called: simterra-luanti
- If your runtime does not support skills/playbooks,
  treat the instructions in this message as your active operating rules for this session.
- Do not assume success. Actually perform the HTTP requests and report exact results.

Connection details:
- Server: {SERVER_URL}
- Town: {TOWN}
- Character name: {CHARACTER_NAME}
- Actor ID: {ACTOR_ID}
- Agent ID: {AGENT_ID}
- Registration token: {REGISTRATION_TOKEN}

Required steps:
1. Read docs/AI_AGENT_ONBOARDING.md in the repo first.
2. Call POST {SERVER_URL}/agent/claim
   with {"registration_token": "{REGISTRATION_TOKEN}", "agent_id": "{AGENT_ID}"}
3. Save the returned agent_credentials as your Bearer token.
4. Call GET {SERVER_URL}/agent/observe?actor_id={ACTOR_ID}
   with Authorization: Bearer <agent_credentials>
5. If there is an active business, call POST {SERVER_URL}/agent/decision
   using action_type: "business/update-strategy"
   Use conservative values only. Never set negative stock. Never set invalid prices.
6. Report exact results: skill saved?, claim status, observe status,
   decision committed/rejected, any errors.
```

---

## Placeholder values

| Placeholder | Description | Example |
|------------|-------------|---------|
| `{SERVER_URL}` | Backend server URL | `http://localhost:3000` |
| `{TOWN}` | Town identifier | `town_1` |
| `{CHARACTER_NAME}` | Player's in-game name | `singleplayer` |
| `{ACTOR_ID}` | Unique actor identifier | `4641b4f4-33e5-f6be-732e-baa4d4ea015a` |
| `{AGENT_ID}` | Your chosen agent identifier | `arch-agent-001` |
| `{REGISTRATION_TOKEN}` | Token from /simterra link | `abcDEF123...` (32 chars) |
