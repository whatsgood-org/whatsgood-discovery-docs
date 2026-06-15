# Whats Good — Events Discovery API

A simple, **read-only** API for discovering upcoming local events across **every
Whats Good city**. Built for trip-planning tools and AI agents: look up the
city someone is visiting and get back what's happening while they're there.

This is a read-only surface — there's nothing to create, approve, or manage.

- **Base URL:** `https://whatsgoodapi.up.railway.app`

---

## Authentication

You'll be issued a **read-only API key** by Whats Good. Send it in the
**`X-API-Key`** request header on every request:

```bash
curl "https://whatsgoodapi.up.railway.app/discover/cities" \
  -H "X-API-Key: YOUR_KEY"
```

> ⚠️ **Header only.** Discovery keys cannot be passed via a `?api_key=` query
> parameter — keys in URLs leak through server access logs, `Referer` headers,
> and browser history. Sending a discovery key in the query string returns
> **401**. Always use the header.

> Want a key? Contact Whats Good at **dustin@checkwhatsgood.com**.

### Errors

| Status | Meaning |
|--------|---------|
| `401` | Missing or invalid API key (or a discovery key sent via `?api_key=`) |
| `403` | This key isn't a discovery (read-only) key |
| `404` | No such event |
| `422` | Query params failed validation (e.g. missing required `city`) |
| `429` | Rate limit exceeded — back off and retry after `Retry-After` seconds |

---

## How to use it (two-step flow)

1. **Call `GET /discover/cities` once** to learn which cities are supported.
2. **Then call `GET /discover/events?city=…`** with one of those values.

`city` is **required** on `/discover/events` — you'll get `422` if you omit it.
This keeps queries scoped to a destination and avoids accidentally pulling
events from every city in a single call.

Every event the API returns is **browsable**: upcoming (starts in the future),
not a duplicate, and not removed. Results come back soonest first.

---

## Quickstart

```bash
BASE=https://whatsgoodapi.up.railway.app
KEY="YOUR_KEY"

# 1. See which cities have events right now
curl -s "$BASE/discover/cities" -H "X-API-Key: $KEY"

# 2. Pull upcoming events for a destination
curl -s "$BASE/discover/events?city=St.%20Petersburg" -H "X-API-Key: $KEY"

# 3. Narrow to a trip window and a category
curl -s "$BASE/discover/events?city=Boise&category=Music&start_date_to=2026-07-01" \
  -H "X-API-Key: $KEY"
```

---

## `GET /discover/cities`

The list of cities supported by the Discovery API. Call this first to learn
which `city` values are valid for `GET /discover/events`.

```json
{
  "items": [
    { "city": "Boise", "state": "Idaho" },
    { "city": "St. Petersburg", "state": "Florida" }
  ],
  "total": 2
}
```

A city only appears here if it currently has at least one browsable upcoming
event — you can safely query any city in the list and expect results. The
endpoint advertises **coverage**, not inventory size, so it deliberately does
not return event counts.

---

## `GET /discover/events`

List upcoming events for a single city (paginated, soonest first).

| Param | Required | Purpose |
|-------|----------|---------|
| `city` | ✅ | City name, e.g. `St. Petersburg` (case-insensitive, exact). Must be one returned by `GET /discover/cities` |
| `state` | | State/region name (case-insensitive, exact). Useful when two cities share a name |
| `category` | | Category name, e.g. `Music` (case-insensitive, exact) |
| `is_free` | | `true` = only free events, `false` = only paid |
| `start_date_to` | | Upper bound on start date (ISO 8601). The lower bound is always "now" |
| `search` | | Free-text match on title/description |
| `page`, `page_size` | | Pagination (`page_size` max **100**, default 20) |

```json
{
  "items": [
    {
      "id": 50274,
      "title": "Movement Mondays",
      "description": "A weekly community dance session.",
      "start_date": "2026-06-01T19:00:00Z",
      "end_date": "2026-06-01T21:00:00Z",
      "city": "St. Petersburg",
      "state": "Florida",
      "venue_name": "The Studio",
      "venue_address": "123 Central Ave",
      "latitude": 27.77,
      "longitude": -82.64,
      "category": "Wellness & Health",
      "is_free": true,
      "price_min": 0,
      "price_max": 0,
      "is_recurring": false,
      "url": "https://example.com/event",
      "images": ["https://…/photo.jpg"]
    }
  ],
  "total": 520,
  "page": 1,
  "page_size": 20,
  "total_pages": 26
}
```

---

## `GET /discover/events/{id}`

Get a single event by its `id` (same shape as one item above). Returns `404` if
the event doesn't exist or isn't browsable.

```bash
curl -s "$BASE/discover/events/50274" -H "X-API-Key: $KEY"
```

---

## Rate limit

Each discovery key is limited to **120 requests per 60 seconds** across all
`/discover/*` endpoints. Over-budget requests get HTTP **429** with a
`Retry-After` header (seconds until your window resets):

```
HTTP/1.1 429 Too Many Requests
Retry-After: 37
```

The window is fixed, not sliding — once it resets, you have your full budget
again. Cache `/discover/cities` (it changes slowly) and paginate `/discover/events`
rather than hammering. Sustained abuse may lead to a key being revoked.

Usage is tracked per key (request count, timing, route). Bodies, query params,
and user-agents are **not** logged.

---

Questions or need a key? Contact Whats Good at **dustin@checkwhatsgood.com**.
