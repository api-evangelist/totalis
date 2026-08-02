---
name: Browse Totalis prediction markets
description: >-
  Discover what is tradable on Totalis — active prediction markets across
  Kalshi and Polymarket, grouped by event or as a flat sync feed — the building
  blocks of a parlay's legs.
api: openapi/totalis-openapi-original.json
operations: [listMarkets, listMarketsFlat, getMarket]
generated: '2026-07-21'
method: generated
---

# Browse Totalis prediction markets

Market endpoints are public (no auth required; `security: []` in the spec), but
authenticated calls share the higher 300 req/min limit. Base URL
`https://api.totalis.trade`.

## Steps

1. **Browse grouped markets** — `listMarkets` (`GET /markets`) lists active markets
   grouped by event. Filter with `category` (`politics`, `sports`, `crypto`,
   `finance`, `economics`, `entertainment`, `weather`, `tech`, or `all`),
   `venue` (`kalshi` / `polymarket`), `subcategory`, `frequency`
   (`daily`/`weekly`/`monthly`/`hourly`), `date_filter`, and full-text `search`.
   The response includes `available_subcategories` and `available_frequencies`
   for building filter UIs. Cursor-paginate at the event level via `meta.cursor`.
2. **Sync the full universe** — `listMarketsFlat` (`GET /v1/markets/list`) is the
   flat, never-truncated per-market feed built for market makers and partners.
3. **Inspect one market** — `getMarket` (`GET /markets/{ticker}`) by Kalshi ticker
   or Polymarket condition ID; pass `venue` when you already know the source venue.

## Rules

- Each market's `ticker` and `venue` are what a parlay leg references
  (`market_ticker`, `side` of `yes`/`no`, optional `venue`, default `kalshi`).
- Parlays take 2–5 legs, $1–$100 USDC bet amounts, and 1.0001x–1000x payout odds.
- Totalis lists recurring markets with reliable liquidity that settle within one
  week — treat the feed as dynamic and re-sync rather than caching long-term.
- On `400 VALIDATION_ERROR` check `error.details.issues`; on `429` honor
  `error.retry_after`.
