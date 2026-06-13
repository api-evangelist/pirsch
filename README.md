# Pirsch

Pirsch is a privacy-first website analytics platform built and hosted in Germany. It provides GDPR, CCPA, PECR, and Schrems II compliant analytics without cookies or personal data storage. Developers can query page views, sessions, custom events, conversion goals, funnels, and traffic sources via a RESTful API.

## Links

- **Website:** https://pirsch.io
- **Documentation:** https://docs.pirsch.io
- **API Reference:** https://docs.pirsch.io/api-sdks/api
- **API Guide:** https://docs.pirsch.io/api-sdks/api-guide
- **Pricing:** https://pirsch.io/pricing
- **Blog:** https://pirsch.io/blog
- **GitHub Org:** https://github.com/pirsch-analytics
- **X:** https://x.com/PirschAnalytics
- **LinkedIn:** https://www.linkedin.com/products/emvi-software-gmbh-pirsch-analytics/

## API

Base URL: `https://api.pirsch.io/api/v1`

Authentication: OAuth client credentials (Bearer token) or Access Key (`pa_*` prefix) for write-only operations.

Key capability groups:
- **Data collection** — page views, events, sessions (single and batch)
- **Statistics** — visitors, pages, referrers, UTM, geo, OS, browser, device, funnels, goals, real-time active visitors, CSV export
- **Domain management** — CRUD for domains, hostnames, subdomains, themes, custom domains
- **Team management** — members, invitations, roles
- **Webhooks & email reports**
- **Traffic filters, conversion goals, funnels, short links, saved views**

## SDKs

| Language | Repository |
|----------|-----------|
| Go | https://github.com/pirsch-analytics/pirsch-go-sdk |
| JavaScript / TypeScript | https://github.com/pirsch-analytics/pirsch-js-sdk |
| PHP / Laravel | https://github.com/pirsch-analytics/laravel-pirsch |
| WordPress | https://github.com/pirsch-analytics/pirsch-wordpress |

## Plans

| Plan | Price | Page Views/mo | Sites |
|------|-------|---------------|-------|
| Standard | $6/mo | 10,000 | 50 |
| Plus | $12/mo | 10,000 | Unlimited |
| Enterprise | Custom | Custom | Unlimited |

30-day free trial, no credit card required.

## Rate Limits

| Endpoint Category | Limit |
|------------------|-------|
| Security (login, password reset) | 10 req/min |
| Configuration (domains, members, etc.) | 60 req/min |
| Statistics & data collection | Unlimited (subject to change) |

Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `Retry-After`

---

*APIs.json profile maintained by [Kin Lane](mailto:kin@apievangelist.com)*
