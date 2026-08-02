---
name: Read a Totalis portfolio, parlays, and P&L
description: >-
  Pull the authenticated user's account state on demand — profile, wallet,
  portfolio rollup, parlay (RFQ) history with settlement detail, and realized
  P&L — using the Totalis RFQ API read endpoints.
api: openapi/totalis-openapi-original.json
operations: [getOrCreateUser, getWallet, getUserPortfolio, getUserVault, listRfqsV1, getRfqV1, getPnlV1]
generated: '2026-07-21'
method: generated
---

# Read a Totalis portfolio, parlays, and P&L

Authenticate every request with `X-API-Key: <key>` against `https://api.totalis.trade`.
A read-only key needs the `account:read`, `positions:read`, and `balances:read` scopes.
All responses are snake_case JSON wrapped in `{ "data": ... }`; a `401` means a bad key,
a `403` means the key lacks the endpoint's scope.

## Steps

1. **Confirm identity** — `getOrCreateUser` (`GET /v1/me`) returns `id`, `username`,
   and `wallet_address`. It auto-creates the user record on first call.
2. **Snapshot balances** — `getWallet` (`GET /v1/wallet`) for the wallet address,
   SOL/USDC balances, vault balance, and `locked_amount`; `getUserVault`
   (`GET /v1/vault`) for vault state and active position summaries.
3. **Roll up the portfolio** — `getUserPortfolio` (`GET /v1/portfolio`) returns
   balance, stats, RFQ status counts, and a summary in one call.
4. **Walk parlay history** — `listRfqsV1` (`GET /v1/rfqs`) with optional `status`
   filter. Page with cursors: pass the previous response's `meta.cursor` as
   `?cursor=` until `meta.has_more` is false. Cursors are opaque — never build them.
5. **Read one parlay's settlement** — `getRfqV1` (`GET /v1/rfqs/{id}`). Once the RFQ
   is terminal it carries settlement detail: per-leg outcomes, payout, and the
   settle/buyback transaction. Note the field renames after pricing:
   `user_cost` → `user_stake`, `mm_cost` → `mm_risk`, `total_payout` → `payout`.
6. **Chart realized P&L** — `getPnlV1` (`GET /v1/pnl`) returns realized P&L per day
   over a `1D` / `1W` / `1M` / `ALL` window.

## Rules

- Respect rate limits: 300 req/min authenticated; back off for `error.retry_after`
  seconds on a `429` (`RATE_LIMITED`) and watch the `X-RateLimit-*` headers.
- Reads are the reconciliation source of truth — after a WebSocket disconnect or a
  missed webhook, re-read `GET /v1/rfqs/{id}` and watch `status` (e.g.
  `bought_back`, which has no WebSocket event at all).
- Error envelope is `{ "error": { "code", "message", "details?" } }` — see
  `errors/totalis-problem-types.yml`.
