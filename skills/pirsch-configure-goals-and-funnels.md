---
name: Configure conversion goals and funnels
description: Define Pirsch conversion goals from path patterns or custom events, test the regex before saving, build a multi-step funnel, and read the resulting conversion statistics.
api: openapi/pirsch-conversion-goals-api-openapi.yml
operations:
  - listConversionGoals
  - createConversionGoal
  - updateConversionGoal
  - deleteConversionGoal
  - testGoalRegex
  - listFunnels
  - createOrUpdateFunnel
  - deleteFunnel
  - getGoalStatistics
  - getFunnelStatistics
generated: '2026-08-13'
method: generated
source: openapi/_original/pirsch-pirsch-api-openapi.yml + https://docs.pirsch.io/advanced/conversion-goals
---

# Configure Pirsch conversion goals and funnels

Base URL: `https://api.pirsch.io/api/v1`. Requires an **OAuth client** token — this is a read/write management surface, so a `pa_` access key will not work.

Goal and funnel management sits in the **configuration** rate-limit tier: 60 requests/minute.

## 1. Test the pattern first — `testGoalRegex`

`POST /goal/regex` with `{"regex": "...", "sample": "..."}` returns `{"match": true|false}`.

Run this before you save a path-pattern goal. A goal with a wrong pattern silently records zero conversions, and there is no error to tell you.

## 2. Create the goal — `createConversionGoal`

`POST /goal` with `CreateGoalRequest`. Required: `domain_id` and `name`. Then pick **one** of two goal shapes:

- **Path goal** — set `path_pattern` to the regex you just tested.
- **Event goal** — set `event_name`, optionally narrowed by `event_meta_key` + `event_meta_value`.

Optional on either: `custom_metric_key` and `custom_metric_type` to attach a numeric metric (e.g. revenue) pulled from event metadata, plus `delete_reached` and `email_reached` to control notification behavior.

Note the parameter split: this write uses `domain_id`, while `listConversionGoals` scopes by `id`. That inconsistency runs through the whole API.

## 3. List and amend — `listConversionGoals`, `updateConversionGoal`

`GET /goal?id=<domain_id>&search=<optional>` lists goals.

`PUT /goal` takes `UpdateGoalRequest` — the create body plus a required `id`. It is a full replace, not a patch: send every field you want to keep.

`DELETE /goal?id=<goal_id>` removes one.

## 4. Build the funnel — `createOrUpdateFunnel`

`POST /funnel` with `CreateFunnelRequest`. Required: `domain_id`, `name`, and `steps`.

`steps` is an **ordered array**, each item `{name, filter}`. The `filter` object takes the same dimensions as the statistics filters — path, event name, country, UTM parameters, tags. Order matters: Pirsch measures drop-off in the sequence you send.

Include `id` in the body to update an existing funnel; omit it to create a new one. This is one endpoint doing both, so an accidental missing `id` creates a duplicate rather than failing.

## 5. Read the results — `getGoalStatistics`, `getFunnelStatistics`

`GET /statistics/goals?id=<domain_id>&from=YYYY-MM-DD&to=YYYY-MM-DD`

`GET /statistics/funnel?id=<domain_id>&funnel_id=<funnel_id>&from=…&to=…` — note `funnel_id` is required *in addition to* the domain `id`.

Conversion rate comes back as `cr` on the statistics payloads; `custom_metric_avg` and `custom_metric_total` carry the numeric metric if you configured one.

## Errors

`400` returns `{"validation": {...}, "error": [...], "context": {}}`. A goal that references a nonexistent domain returns a general `error` entry, not a field-level one. See `errors/pirsch-problem-types.yml`.

## Gotchas

- Goals are counted forward only. Creating a goal does not backfill conversions from historical data.
- `createOrUpdateFunnel` and `createOrUpdateShortLink` are upserts keyed on an optional `id`; treat a missing `id` as "create".
- There is no idempotency key, so a retried `POST /goal` creates a second goal with the same name.
