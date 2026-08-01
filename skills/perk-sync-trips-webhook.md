---
name: perk-sync-trips-webhook
description: Keep an external system in sync with Perk trips by subscribing to the trip.created/updated webhook and pulling fresh trip detail.
api: Perk Travel & Spend API
base_url: https://api.perk.com
operations:
  - subscribe-to-event         # POST webhook subscription
  - update-subscription        # PATCH webhook subscription
  - list-all-trips             # GET /trips
  - retrieve-a-trip-by-id      # GET /trips/{id}
mcp_tools:
  - trips_list_trips
  - trips_get_trip
events:
  - trip.created/updated
method: generated
source: https://developers.perk.com/docs/subscribing-to-webhooks
---

# Sync Perk trips via webhook

Stay current with trip changes without polling.

## Auth
API key (`Authorization: apikey <key>`, `Api-Version: 1`) for customers, or OAuth 2.0 for partners.

## Steps
1. Build an HTTPS handler that accepts `POST` JSON and returns `2xx`.
2. Subscribe with **subscribe-to-event**, giving a unique target URL, a `secret`, and the `trip.created/updated` event. Target URL must be unique per account (duplicate → `409`).
3. On each delivery, verify the `Tk-webhook-hmac-sha256` header: compute `HMAC-SHA256(secret, body)` and compare. Reject on mismatch.
4. Read the `Tk-webhook-event` header; for `trip.created/updated`, call **retrieve-a-trip-by-id** (`GET /trips/{id}`) to pull the authoritative current trip, then upsert into your system.
5. Use the Users endpoints to refresh travelers, since the common integration pattern is: webhook fires → re-fetch travelers to stay in sync.

## Rules
- Return `2xx` fast; non-2xx (including 3xx/redirects) is treated as failure and retried with exponential backoff.
- Ignore deliveries where `Tk-webhook-test` is set if you don't want test traffic.
- Rotate the webhook `secret` periodically via **update-subscription**.
