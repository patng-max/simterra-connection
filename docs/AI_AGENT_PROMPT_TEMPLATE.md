# AI Agent Prompt Template — Simterra-Luanti

Use this template when a player pastes a registration token. Replace `{}` placeholders with actual values from the player's message.

---

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

---

## Placeholder reference

| Placeholder | Description | Example |
|------------|-------------|---------|
| `{SERVER_URL}` | Backend server URL | `http://localhost:3000` |
| `{TOWN}` | Town identifier | `town_1` |
| `{CHARACTER_NAME}` | Player's in-game name | `singleplayer` |
| `{ACTOR_ID}` | Unique actor identifier | `4641b4f4-33e5-f6be-732e-baa4d4ea015a` |
| `{AGENT_ID}` | Your chosen agent identifier | `arch-agent-001` |
| `{REGISTRATION_TOKEN}` | Token from /simterra link | `abcDEF123...` (32 chars) |
