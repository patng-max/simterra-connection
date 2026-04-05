---
name: simterra-luanti-connect
description: "Use when a user pastes a Simterra Luanti connect message or token. Connect an AI agent to a Simterra-Luanti voxel game server. Only for Luanti/Simterra-Luanti — do NOT use this for Godot/Simterra (godot uses different API)."
---

# Simterra-Luanti Connect — Join a Simterra Game as an AI Agent

## When This Skill Is Triggered

- User pastes a Simterra Luanti connect message from the game client
- User sends a registration token for Simterra-Luanti
- User asks to "connect to Simterra" or "join the game" with a Luanti token

## Important — Which Game?

- **Luanti/Simterra-Luanti** (voxel, this skill): Uses `/agent/claim`, `/agent/observe`, `/agent/decision`
- **Godot/Simterra** (3D cozy town): Uses different SSE stream API — DO NOT mix these up

---

## Connection Flow

### Step 1 — Parse the message

The player pastes a message from the Luanti game client. Extract:
- `registration_token` — the token string
- `actor_id` — found in the message or from the connection guide
- `server_url` — usually `http://localhost:3000`

Example raw token from game:
```
/simterra-connect abc123DEF456...
```

### Step 2 — Claim the registration

```
POST {server_url}/agent/claim
Content-Type: application/json
Body: {
  "registration_token": "TOKEN_FROM_PLAYER",
  "agent_id": "my-agent-001"
}
```

Expected response:
```json
{
  "success": true,
  "data": {
    "agent_credentials": "tok_LONG_CREDENTIALS_STRING",
    "expires_in_seconds": 86400
  }
}
```

Save `agent_credentials` as your bearer token.

### Step 3 — Observe world state

```
GET {server_url}/agent/observe?actor_id=ACTOR_ID
Authorization: Bearer AGENT_CREDENTIALS
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
    "balance": "1000.00",
    "currency": "TWD"
  }
}
```

### Step 4 — Make a decision

**Only one action type supported:**

`business/update-strategy` — update prices and stock targets:

```
POST {server_url}/agent/decision
Authorization: Bearer AGENT_CREDENTIALS
Content-Type: application/json

{
  "actor_id": "ACTOR_ID",
  "action_type": "business/update-strategy",
  "parameters": {
    "business_id": "biz-xxx",
    "prices": {"tea": 5.50, "coffee": 7.00},
    "stock_targets": {"tea": 100, "coffee": 50}
  }
}
```

**Committed:**
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

**Rejected:**
```json
{
  "success": false,
  "data": {
    "status": "rejected",
    "message": "Price rejected: tea $0.50 below template minimum $1.00"
  }
}
```

---

## API Reference

Base URL: `http://localhost:3000` (development)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/agent/register` | None | Register, get registration_token |
| POST | `/agent/claim` | None | Exchange registration_token → agent_credentials |
| GET | `/agent/observe` | Bearer | Get world state snapshot |
| POST | `/agent/decision` | Bearer | Submit business action |

---

## Token Expiry

- `registration_token`: 5 minutes TTL (from `/agent/register`)
- `agent_credentials`: 24 hours TTL (from `/agent/claim`)

---

## Error Codes

| Status | Meaning |
|--------|---------|
| 401 | Missing or invalid bearer token |
| 403 | Actor ID mismatch |
| 404 | Business/lot not found |
| 400 | Invalid input |
| 409 | CAS version conflict |

---

## Game Rules

- Prices must be ≥ template minimum cost
- Stock targets must be ≥ 0
- Business must be `status: active`
- Lot must be `status: leased` or `owned`

---

## Connection Guide

Full reference: https://github.com/patng-max/simterra-connection
