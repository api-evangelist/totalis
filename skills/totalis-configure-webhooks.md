---
name: Configure and operate Totalis webhooks
description: >-
  Set up HMAC-signed webhook delivery for settlement, buyback, and funds
  events; verify signatures; monitor deliveries; and replay failures using the
  Totalis RFQ API webhook endpoints.
api: openapi/totalis-openapi-original.json
operations: [getWebhookConfig, setWebhookConfig, rotateWebhookSecret, listWebhookDeliveries, redeliverWebhookDelivery]
generated: '2026-07-21'
method: generated
---

# Configure and operate Totalis webhooks

Webhooks are the durable event channel — at-least-once, signed, retried with
backoff, replayable. The endpoint belongs to the account, not the key that
created it. Managing it needs `account:read` to view and `account:write` to
change; each delivery is additionally gated on the scope its event requires
(`position.*` → `positions:read`, `funds.*` → `balances:read`), re-checked at
delivery time.

## Steps

1. **Set the endpoint** — `setWebhookConfig` (`PUT /v1/webhooks`) with an HTTPS
   `url` (must not resolve to private infrastructure) and an `events` subset of:
   `position.settled`, `position.cancelled`, `position.bought_back`,
   `parlay.status_changed`, `funds.deposited`, `funds.withdrawn`. An empty
   `events` array parks the endpoint without deleting it. Market makers manage a
   separate endpoint with `?owner_kind=mm` (gated on `mm:quote`).
2. **Mint the signing secret** — `rotateWebhookSecret`
   (`POST /v1/webhooks/rotate-secret`). Deliveries only start once a secret
   exists. The `whsec_…` value is shown exactly once; rotation invalidates the
   old secret immediately (no overlap window).
3. **Verify every delivery** — parse `X-Totalis-Signature`
   (`t=<unix-seconds>,v1=<hmac-sha256-hex>`), reject stale timestamps (±5 min),
   recompute HMAC-SHA256 over `"<t>.<raw-body-bytes>"` (never re-serialized
   JSON), and compare constant-time. Respond `2xx` fast; do heavy work async.
4. **Deduplicate** — on `X-Totalis-Event-Id`. Delivery is at-least-once and
   replays reuse the same event id. Don't assume ordering; use
   `data.occurred_at` to reason about sequence.
5. **Monitor** — `getWebhookConfig` (`GET /v1/webhooks`) for the current config +
   subscribable catalog; `listWebhookDeliveries` (`GET /v1/webhooks/deliveries`)
   for recent attempts with status (`delivered` / `failed` / `dead_letter`),
   attempt count, and last response code.
6. **Replay after an outage** — `redeliverWebhookDelivery`
   (`POST /v1/webhooks/deliveries/{id}/redeliver`) re-queues a `delivered` or
   `dead_letter` delivery (a `failed` one mid-retry can't be replayed manually).

## Rules

- Retried: `5xx`, `408`, `429`, timeouts, network errors. Not retried: other
  `4xx` → immediate `dead_letter`.
- `position.bought_back` is webhook-only (no WebSocket equivalent) — any
  integration that needs buyback awareness must run webhooks or poll RFQ status.
- A cancellation fires both `parlay.status_changed` (`status: "cancelled"`) and
  `position.cancelled` — dedupe across the pair.
