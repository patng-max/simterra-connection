# Simterra-Luanti — Agent Connection Guide

This document explains how an AI agent connects to a Simterra-Luanti game server.

## Overview

Simterra-Luanti is a voxel world economy simulation built on the Luanti (formerly Minetest) engine. Human players own lots and run businesses in the world. AI agents can observe the world state and make decisions on behalf of the player.

## Connection Flow

### Step 1: Player registers an agent

The player is in the Luanti game and types:
```
/simterra link
```

A registration token is displayed in the chat. The player copies this token and sends it to their AI agent.

### Step 2: Agent claims the link

The AI agent exchanges the registration token for agent credentials:

```
POST http://localhost:3000/agent/claim
Content-Type: application/json

{
  "registration_token": "TOKEN_FROM_PLAYER",
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

Save `agent_credentials` — this is your bearer token.

### Step 3: Observe world state

```
GET http://localhost:3000/agent/observe?actor_id=ACTOR_ID
Authorization: Bearer AGENT_CREDENTIALS
```

Response:
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

### Step 4: Make a decision

Only one action type is supported in Phase 2B:

**`business/update-strategy`** — update prices and stock targets:

```
POST http://localhost:3000/agent/decision
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

**Committed** (valid decision, CAS succeeded):
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

**Rejected** (invalid price, below minimum, etc.):
```json
{
  "success": false,
  "data": {
    "status": "rejected",
    "message": "Price rejected: tea $0.50 below template minimum $1.00"
  }
}
```

## Backend API

Base URL: `http://localhost:3000` (development)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/agent/register` | None | Register agent endpoint, get registration_token |
| POST | `/agent/claim` | None | Exchange registration_token for agent_credentials |
| GET | `/agent/observe` | Bearer | Get current world state snapshot |
| POST | `/agent/decision` | Bearer | Submit one business action |

## Token Expiry

- `registration_token`: 5 minutes TTL (from `/agent/register`)
- `agent_credentials`: 24 hours TTL (from `/agent/claim`)

Refresh by having the player run `/simterra link` again.

## Error Codes

| Status | Meaning |
|--------|---------|
| 401 | Missing or invalid bearer token |
| 403 | Actor ID mismatch (token not for this actor) |
| 404 | Business/lot not found |
| 400 | Invalid input (bad price, unknown item) |
| 409 | CAS version conflict (concurrent update) |

## Game Rules

- Prices must be ≥ template minimum cost
- Stock targets must be ≥ 0
- Business must be `status: active`
- Lot must be `status: leased` or `owned`

## Reference Implementation

See the Godot Simterra reference for full patterns:
https://github.com/patng-max/simterra (server/src/agent/ExternalHttpAgentAdapter.ts)
