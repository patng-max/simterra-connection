# AI Agent Onboarding — Simterra-Luanti

Canonical reference for AI agents connecting to Simterra-Luanti. Read this before making any API calls.

## Connection summary

1. Call `POST /agent/claim` with `registration_token` + `agent_id`
2. Save `agent_credentials` from the response — this is your **Bearer token**
3. Call `GET /agent/observe?actor_id=` with Bearer auth
4. Call `POST /agent/decision` to act

---

## Step 1 — Claim the registration

```
POST {server_url}/agent/claim
Content-Type: application/json

{
  "registration_token": "REGISTRATION_TOKEN",
  "agent_id": "my-agent-001"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "agent_credentials": "tok_LONG_CREDENTIALS_STRING",
    "expires_in_seconds": 86400
  }
}
```

**Save `agent_credentials` securely.** It expires in 24 hours.

**Choosing `agent_id`:** Use a stable identifier — the same agent should use the same `agent_id` across sessions. If the player did not provide one, generate one (e.g. `myagent-001`) and reuse it. The `agent_id` is not a secret.

---

## Step 2 — Observe world state

```
GET {server_url}/agent/observe?actor_id={actor_id}
Authorization: Bearer {agent_credentials}
```

Example response:
```json
{
  "lot": {
    "lot_id": "lot-1",
    "district_id": "town-center",
    "x": 0,
    "z": 0,
    "status": "leased",
    "owner_actor_id": "..."
  },
  "business": {
    "business_id": "biz-xxx",
    "lot_id": "lot-1",
    "template_revision_id": "rev-cafe-v1",
    "status": "active",
    "stock_state": {"tea": 50, "coffee": 30},
    "price_state": {"tea": 5.00, "coffee": 6.50},
    "version": 3
  },
  "account": {
    "account_id": "acc-xxx",
    "balance": "1000.00",
    "currency": "TWD",
    "version": 1
  }
}
```

**Always observe before making a decision.** The server state is authoritative.

---

## Step 3 — Make a decision

### Supported action: `business/update-strategy`

Update prices and/or stock targets.

```
POST {server_url}/agent/decision
Authorization: Bearer {agent_credentials}
Content-Type: application/json

{
  "actor_id": "{actor_id}",
  "action_type": "business/update-strategy",
  "parameters": {
    "business_id": "{business_id}",
    "prices": {"tea": 5.50, "coffee": 7.00},
    "stock_targets": {"tea": 100, "coffee": 50}
  }
}
```

**Response — committed:**
```json
{
  "success": true,
  "data": {
    "status": "committed",
    "message": "Price updated: tea $5.00 → $5.50",
    "version": 4
  }
}
```

**Response — rejected:**
```json
{
  "success": false,
  "data": {
    "status": "rejected",
    "message": "Price rejected: tea $0.50 below minimum $1.00"
  }
}
```

---

## Error handling

| HTTP status | Code | Meaning | Action |
|-------------|------|---------|--------|
| 401 | `token_invalid` | Registration token wrong or expired | Stop. Request fresh token from player. |
| 401 | `unauthorized` | Bearer token missing or invalid | Stop. Request fresh registration. |
| 403 | — | Actor ID mismatch | Do not retry without new registration. |
| 404 | — | Business or lot not found | Report clearly — player may not have a business yet. |
| 400 | — | Bad input (price, stock) | Fix values. See conservative rules below. |
| 409 | — | CAS version conflict | Re-observe to get latest version, then retry. |

**Never pretend a failed call succeeded.** Report exact errors to the player.

---

## Conservative decision rules

1. **Always observe before acting.** Never decide without current state.
2. **Use small price increments.** Move prices in steps of $0.50–$1.00. If the server rejects a price, never go lower than the rejected value — use it as the new floor.
3. **Never set stock targets below 0.**
4. **Watch your balance.** A low balance means the business is losing money.
5. **Prefer gradual adjustments.** A series of small changes is safer than one large move.
6. **If rejected, re-observe before retrying.** The business state may have changed.

---

## Persistence

**If your runtime supports persistent skills, playbooks, or memory files:**

Save these rules as `simterra-luanti` (or your preferred name). Reload them on each session.

**If your runtime does not support persistence:**

Treat these rules as active operating rules for this session. Re-read this doc if you reconnect with a new token.

---

## API reference

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/agent/claim` | None | Exchange registration_token → agent_credentials |
| GET | `/agent/observe` | Bearer | Get world state snapshot |
| POST | `/agent/decision` | Bearer | Submit one business action |

Base URL: `http://localhost:3000` (development)

---

## Game constraints

- Prices must be ≥ template minimum cost
- Stock targets must be ≥ 0
- Business must be `status: active`
- Lot must be `status: leased` or `owned`
- Only supported action: `business/update-strategy`
