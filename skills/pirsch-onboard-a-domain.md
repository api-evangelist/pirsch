---
name: Onboard a new domain end to end
description: Create a Pirsch domain, mint the API client credentials for it, invite team members, and issue a shareable read-only access link for an embedded dashboard.
api: openapi/pirsch-domains-api-openapi.yml
operations:
  - createDomain
  - listDomains
  - updateDomainSettings
  - createAlternativeDomain
  - createClient
  - inviteMembers
  - listMembers
  - updateMember
  - createAccessLink
generated: '2026-08-13'
method: generated
source: openapi/_original/pirsch-pirsch-api-openapi.yml + https://docs.pirsch.io/api-sdks/api-v1
---

# Onboard a new Pirsch domain

Base URL: `https://api.pirsch.io/api/v1`. Requires an **OAuth client** token with rights on the account. Everything here is in the **configuration** rate-limit tier: 60 requests/minute — pace bulk onboarding accordingly.

## 1. Create the domain — `createDomain`

`POST /domain` with `CreateDomainRequest`. Required: `hostname`, `subdomain`, `timezone`.

- `hostname` — the site being tracked, e.g. `example.com`
- `subdomain` — the Pirsch dashboard slug for it; must be unique across Pirsch
- `timezone` — an IANA name, e.g. `Europe/Berlin`. This sets the day boundary for every statistic that follows, so get it right before data accumulates.

Optional: `organization_id`, `theme_id`, `public`, `group_by_title`, `disable_scripts`, `display_name`, `settings`.

The response is a `Domain`. **Keep the `id`** — it is the value every other call in this flow needs.

## 2. Add extra hostnames — `createAlternativeDomain`

If the site answers on more than one hostname, `POST /domain/alternative` with `{domain_id, hostname}` for each. They report into the same domain rather than splitting the numbers.

## 3. Mint credentials — `createClient`

`POST /client` with `CreateClientRequest`: `domain_id`, `type`, `description`. `type` is an enum:

- `oauth` — read/write client credentials for the API
- `token` — a write-only `pa_` access key for server-side tracking

**The `client_secret` is returned only on creation.** There is no way to read it back and no rotation endpoint — to rotate you `DELETE /client?id=` and create a new one. Store it at the moment you receive it.

Mint two if the integration both sends data and reads statistics: a `token` client for the tracking path, an `oauth` client for reporting. Do not use one high-privilege credential for both.

## 4. Invite the team — `inviteMembers`

`POST /member` with `InviteMembersRequest`: `{id: <domain_id>, emails: [...]}` (or `organization_id` for org-level access).

`GET /member?id=<domain_id>` lists current members. `PUT /member` with `{id: <member_id>, role}` changes a role — **`role` accepts only `Viewer` or `Admin`**; `Owner` cannot be granted through the API. `DELETE /member?id=<member_id>` removes one.

Pending invitations are a separate resource: `listInvitations`, `acceptInvitation`, `deleteInvitation`.

## 5. Issue a read-only share link — `createAccessLink`

`POST /domain/link` with `CreateAccessLinkRequest`: `domain_id` required, plus optional `description` and `valid_until`.

The returned `code` is what you put in a public dashboard URL or an embedded iframe. Set `valid_until` on anything you hand outside the company — an access link with no expiry is a permanent unauthenticated read of the domain's analytics.

Embedding details are in `components/pirsch-components.yml`.

## Verify

`GET /domain?domain=<hostname>` should now return the domain. Then install the tracking script or the server-side integration — see `skills/pirsch-track-server-side.md`.

## Errors and cautions

- `400` returns `{"validation": {...}, "error": [...], "context": {}}`. A taken `subdomain` shows up as a `validation` entry.
- No idempotency key: a retried `POST /domain` creates a **second** domain. Check with `listDomains` before retrying.
- `DELETE /domain?id=` permanently deletes the domain **and all of its data**. There is no soft delete and no undo.
