---
name: Track page views and events from a server
description: Send page views, custom events and session keep-alives to Pirsch from server-side code using a write-only access key, including the batch endpoints and their ordering requirement.
api: openapi/pirsch-tracking-api-openapi.yml
operations:
  - getToken
  - sendPageView
  - sendPageViewBatch
  - sendEvent
  - sendEventBatch
  - keepSessionAlive
  - keepSessionAliveBatch
generated: '2026-08-13'
method: generated
source: openapi/_original/pirsch-pirsch-api-openapi.yml + https://docs.pirsch.io/get-started/backend-integration
---

# Track page views and events from a Pirsch server integration

Base URL: `https://api.pirsch.io/api/v1`

## Choose the credential

Use a **`pa_`-prefixed access key** for tracking. It is write-only, needs no token exchange, and is the credential Pirsch designed for stateless server-side sending. Send it as `Authorization: Bearer pa_...`.

Only fall back to `getToken` (OAuth client credentials) if the same process also needs to read statistics — an access key cannot read.

## Send a single page view — `sendPageView`

`POST /hit`

Required: `url`, `ip`, `user_agent`. Everything else is optional but improves the record: `accept_language`, `referrer`, `title`, `screen_width`, `screen_height`, the `sec_ch_ua*` client hints, and a flat string `tags` map.

Forward the **end user's** IP and User-Agent, not your server's. Pirsch derives geo, device and browser from them, and hashes the IP rather than storing it.

## Send a custom event — `sendEvent`

`POST /event`

Everything from the page view, plus a required `event_name` and optional `event_duration`, `event_meta` and `non_interactive`.

`event_meta` must be a **flat map of string to string** — nested objects and numeric values are not accepted. Serialize amounts as strings (`"199.99"`).

An event named here is what a webhook subscribes to. See `asyncapi/pirsch-webhooks.yml`.

## Batch — `sendPageViewBatch`, `sendEventBatch`, `keepSessionAliveBatch`

`POST /hit/batch`, `POST /event/batch`, `POST /session/batch` take a JSON **array**. Each item adds a required `time` field (ISO 8601, UTC).

**Sort the array by `time` ascending before you send it.** Pirsch does not sort for you, and out-of-order batches produce wrong session durations and wrong time-on-page. This is the single most common server-side integration bug.

## Keep sessions alive — `keepSessionAlive`

`POST /session` with `ip` and `user_agent` extends an existing visitor session. Note that session extensions count as 10% of a page view against your plan quota (see `plans/pirsch-plans-pricing.yml`).

## Retries — read this

Pirsch supports **no idempotency key**. There is no `Idempotency-Key` header, no dedupe on batch items, and no replay semantics. A retry after a timeout will double-count.

Practical rule: on a network error where you never saw a response, prefer dropping the hit over retrying it. Analytics tolerates a missing page view far better than an inflated one. If you must retry, retry only on a `429` or `5xx` where the body clearly indicates the request was rejected.

## Rate limits

Data-collection endpoints are currently **unlimited**, but that is stated as subject to change. Watch `X-RateLimit-Limit` / `X-RateLimit-Remaining` / `X-RateLimit-Reset` on responses and honor `Retry-After` on a `429`. See `rate-limits/pirsch-rate-limits.yml`.

## Errors

`400` returns `{"validation": {...}, "error": [...], "context": {}}` — read `validation` for the offending field. There is no error code to switch on. See `errors/pirsch-problem-types.yml`.

## Alternative: the browser script

If you only need client-side tracking, load `https://api.pirsch.io/pa.js` with `id="pianjs"` and `data-code`. Its full attribute surface is in `components/pirsch-components.yml`.
