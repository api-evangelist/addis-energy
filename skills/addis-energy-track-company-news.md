---
name: addis-energy-track-company-news
description: Pull the Addis Energy news and press-coverage archive from addisenergy.com, filter it by category or date window, and resolve each item to its full text — for tracking a pre-revenue deep-tech energy company that publishes no press API.
api: addis-energy:addis-energy-posts-api
operations:
- listPosts
- getPost
- listCategories
generated: '2026-09-07'
method: generated
source: openapi/addis-energy-posts-api-openapi.yml, openapi/addis-energy-taxonomy-api-openapi.yml, conventions/addis-energy-conventions.yml
---

# Track Addis Energy company news

Addis Energy publishes no press API, no newsroom feed product and no developer program. Its news
archive is nonetheless fully readable through the WordPress core REST API behind its website, with
no credential. This skill walks it correctly.

**Base URL:** `https://addisenergy.com/wp-json`
**Auth:** none. Do not send an Authorization header; there is no credential to send.

## 1. Learn the categories first

```
GET /wp/v2/categories?_fields=id,name,slug,count
```

Verified live on 2026-09-07: `7` Press Coverage (11 posts), `8` Blog (2 posts), `1` Uncategorized
(0 posts). Do not hardcode these ids without re-reading — they are site-local and can change.

## 2. Pull the archive

```
GET /wp/v2/posts?per_page=100&_fields=id,date,slug,title,link,categories
```

`_fields` is not optional in practice: without it every post carries fully rendered HTML in
`content.rendered` and the response is an order of magnitude larger for no gain.

Read `X-WP-Total` and `X-WP-TotalPages` from the response headers to know when to stop. The archive
was 13 posts at capture, so one request at `per_page=100` is a complete crawl. **Do not probe for
an empty page** — requesting a `page` beyond `X-WP-TotalPages` returns `400
rest_post_invalid_page_number`, not an empty array.

## 3. Narrow it

- Press coverage only: add `&categories=7`
- Company-authored posts only: add `&categories=8`
- Since a date: add `&after=2025-06-01T00:00:00` (ISO 8601, site timezone). Pair with `before` to
  window a range.
- Free-text: use `search` across posts and pages instead — `GET /wp/v2/search?search=ammonia`.

## 4. Resolve one item in full

```
GET /wp/v2/posts/{id}?_embed
```

`_embed` inlines the author, featured media and terms into an `_embedded` object, collapsing four
round trips into one. The body text is `content.rendered` — HTML, already rendered; strip or parse
it, do not expect markdown.

## 5. Poll politely

There is no rate-limit header of any kind on this surface, which means you get no back-off signal —
only a silent cut-off if the origin decides you are abusive. Prefer the RSS feed at
`https://addisenergy.com/feed/` as the change signal and only hit the REST API when the feed shows
something new. Every response carries `X-Robots-Tag: noindex`: read it, but do not republish it into
a search index.

## Error handling

Branch on the `code` field, never on `message` (it is localized).

| code | status | what to do |
|---|---|---|
| `rest_post_invalid_page_number` | 400 | You paged past the end. Stop; you already have everything. |
| `rest_invalid_param` | 400 | Check bounds. `per_page` is 1..100 and is rejected, not clamped. |
| `rest_post_invalid_id` | 404 | The id is gone or was never a post. Re-list. |
| `rest_no_route` | 404 | Re-read `https://addisenergy.com/wp-json/` — it enumerates every real route. |

There is no request-id header, so there is nothing to quote to support — and no API support channel
exists in any case. Fail closed and log the URL you called.
