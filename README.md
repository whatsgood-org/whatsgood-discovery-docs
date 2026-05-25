# whatsgood-discovery-docs

Public documentation for the **Whats Good Events Discovery API** — a read-only,
cross-city API for trip-planning tools and AI agents to look up upcoming local
events.

This is **separate** from the customer-facing API docs
(`whatsgood-org/whatsgood-api-docs`), which cover the tenant/source-management
surface. Different audience, different surface.

- **Live site:** https://whatsgood-org.github.io/whatsgood-discovery-docs/
- **API base URL:** `https://whatsgoodapi.up.railway.app`
- **Guide:** [`DISCOVERY_API.md`](./DISCOVERY_API.md) (rendered by `index.html` via zero-md, served on GitHub Pages)

Endpoints documented: `GET /discover/cities`, `GET /discover/events`,
`GET /discover/events/{id}`.
