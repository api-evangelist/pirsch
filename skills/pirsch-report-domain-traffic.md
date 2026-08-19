---
name: Report traffic for a Pirsch domain
description: Authenticate against Pirsch, resolve a domain ID, and pull a complete traffic report — totals, the visitor time series, top pages, referrers and channels — for a date range.
api: openapi/pirsch-statistics-api-openapi.yml
operations:
  - getToken
  - listDomains
  - getTotalStatistics
  - getVisitorStatistics
  - getPageStatistics
  - getReferrerStatistics
  - getChannelStatistics
generated: '2026-08-13'
method: generated
source: openapi/_original/pirsch-pirsch-api-openapi.yml + https://docs.pirsch.io/api-sdks/api-v1
---

# Report traffic for a Pirsch domain

Base URL: `https://api.pirsch.io/api/v1`

## Before you start

- You need an **OAuth client**, not an access key. Access keys are `pa_`-prefixed and **write-only** — they cannot read statistics. See `authentication/pirsch-authentication.yml`.
- Statistics reads are currently unlimited, but the token exchange counts against the 10 requests/minute security tier. Cache the token.

## 1. Get a token — `getToken`

`POST /token` with `{"client_id": "...", "client_secret": "..."}`. Security is explicitly disabled on this operation, so send no Authorization header.

The response carries `access_token` and `expires_at`. Send `Authorization: Bearer <access_token>` on every call below. There is no refresh token: on a `401`, re-run this step once and retry.

## 2. Resolve the domain ID — `listDomains`

`GET /domain` returns every domain the client can see. Filter with `domain=` or `search=` if you know the hostname. Take `id` from the match — this is the value every statistics call needs.

**Watch the parameter names.** Statistics operations scope by `id` (meaning the *domain* id). Write operations elsewhere in this API use `domain_id`. They are not interchangeable.

## 3. Pull the totals — `getTotalStatistics`

`GET /statistics/total?id=<domain_id>&from=YYYY-MM-DD&to=YYYY-MM-DD`

`from` and `to` are required and must be plain dates. Optional `tz` sets the timezone (defaults to the dashboard setting) and `scale` accepts `day`, `week`, `month` or `year`.

Returns `visitors`, `views`, `sessions`, `bounces`, `bounce_rate` and `cr`.

## 4. Pull the time series — `getVisitorStatistics`

`GET /statistics/visitor?id=…&from=…&to=…&scale=day`

Returns an array of `VisitorStats`, one per bucket. Use the same `scale` you intend to chart; do not resample client-side from a coarser bucket.

## 5. Pull top pages — `getPageStatistics`

`GET /statistics/page?id=…&from=…&to=…&limit=100&offset=0&sort=visitors&direction=desc`

Pagination is offset/limit and **`limit` is hard-capped at 100**. The response is a bare array — there is no `total`, no `has_more` and no next link, so you cannot distinguish a full last page from a truncated one. Page until you get back fewer than `limit` rows.

## 6. Pull acquisition — `getReferrerStatistics` and `getChannelStatistics`

`GET /statistics/referrer?id=…&from=…&to=…&limit=100` and `GET /statistics/channel?id=…&from=…&to=…`.

For campaign detail, the UTM breakdowns are five separate operations: `getUtmSourceStatistics`, `getUtmMediumStatistics`, `getUtmCampaignStatistics`, `getUtmContentStatistics`, `getUtmTermStatistics`.

## Filtering

Every statistics operation accepts the same filter dimensions as query parameters (`path`, `hostname`, `country`, `city`, `browser`, `os`, `platform`, `language`, `utm_*`, `tag`, and more). Operators:

- `!value` — negate
- `~value` — contains
- `^value` — excludes

Repeating a parameter ORs the values: `city=London&city=Berlin`.

## Errors

Pirsch does **not** use RFC 9457. The envelope is `{"validation": {...}, "error": [...], "context": {}}` with no machine-readable code — you have to read the message text. See `errors/pirsch-problem-types.yml`.

- `400` — read `validation` and fix the named parameters
- `401` — token expired, re-run step 1 once
- `429` — honor `Retry-After`; check `X-RateLimit-Reset`

## Do not

- Do not retry a failed write blindly — Pirsch supports no idempotency key.
- Do not assume a status page exists to check during an outage; there isn't one.
