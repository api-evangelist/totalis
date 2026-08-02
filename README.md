# Totalis

Totalis (totalis.trade), operated by UCALLEDIT, Inc., is a Y Combinator-backed (Spring 2026) derivatives layer for prediction markets: users combine two to five outcomes across venues like Kalshi and Polymarket into a single parlay, market makers compete to price it through an RFQ system, and positions settle in non-custodial Solana vaults with USDC collateral.

The Totalis RFQ API is a documented REST + SSE surface at `https://api.totalis.trade` with scoped API keys, a WebSocket for post-trade events, and HMAC-signed webhooks — covering markets, quote requests, parlays, portfolio, funds, and market-maker flows. Developer docs live at [docs.totalis.trade](https://docs.totalis.trade), including a published OpenAPI and llms.txt.

Backed by: y-combinator

## Artifacts

- `openapi/` — the published Totalis RFQ API OpenAPI 3.0.3 (25 operations)
- `overlays/` — API Evangelist overlay of enhancements and coverage notes
- `llms/` — the provider-published llms.txt (verbatim)
- `authentication/` — auth profile: scoped API keys (11 scopes) + Privy JWT
- `conventions/` — cross-cutting request/response semantics
- `errors/` — error-code catalog and envelope
- `rate-limits/` — published rate limits and X-RateLimit headers
- `lifecycle/` — versioning + object/credential lifecycles
- `asyncapi/` — webhook and WebSocket event surfaces as AsyncAPI
- `conformance/` — standards conformance assertions
- `data-model/` — entity-relationship graph derived from the spec
- `mcp/` — candidate MCP tool surface derived from the spec
- `skills/` — packaged Agent Skills grounded in real operationIds
- `security/` — domain security probe results
- `well-known/` — /.well-known/ probe record (no real surface exists)
