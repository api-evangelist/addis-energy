---
name: addis-energy-map-the-api-surface
description: Use the self-describing WordPress route index at addisenergy.com to discover every callable route, content type and taxonomy without any documentation — and to tell apart what is anonymously readable from what is administrator-only.
api: addis-energy:addis-energy-discovery-api
operations:
- getRouteIndex
- getWpV2Namespace
- listTypes
- listTaxonomies
- listStatuses
generated: '2026-09-07'
method: generated
source: openapi/addis-energy-discovery-api-openapi.yml, authentication/addis-energy-authentication.yml
---

# Map the Addis Energy API surface from scratch

Addis Energy publishes zero API documentation. It does not need to for this surface to be usable:
the server describes itself. This skill is how you find out what is really there instead of guessing
paths.

**Base URL:** `https://addisenergy.com/wp-json`
**Auth:** none.

## 1. Read the root

```
GET /
```

Returns the discovery document: site identity (`name`, `home`), `namespaces`, `authentication`, and
`routes` — every registered route keyed by pattern, each with its methods and full `args` schema.
Captured 2026-09-07: **218 routes across 9 namespaces**.

This `args` block is the important part. It is the equivalent of OpenAPI parameters — type, default,
enum, and bounds — published by the server itself. Every parameter in this repository's `openapi/`
files was read out of it.

## 2. Separate the public API from the site's plumbing

Nine namespaces are registered; only two are anonymously usable.

**Usable anonymously:** `wp/v2` (read half), `oembed/1.0`.

**Administrator-only — do not attempt:** `contact-form-7/v1` (403 `wpcf7_forbidden`),
`code-snippets/v1`, `siteground-optimizer/v1`, `duplicator/v1`, `wp-site-health/v1`,
`wp-block-editor/v1`, `wp-abilities/v1` (401 `rest_forbidden`). These are the site operator's own
control plane. They are registered because plugins registered them, not because Addis Energy offers
them. `/wp/v2/settings` is likewise 401.

A route appearing in the index is **not** a promise it is callable by you. Probe before you plan.

## 3. Learn the content model

```
GET /wp/v2/types
GET /wp/v2/taxonomies
GET /wp/v2/statuses
```

Verified 2026-09-07: WordPress core only. Types are `post`, `page`, `attachment`, `nav_menu_item`,
`wp_block`, `wp_template`, `wp_template_part`, `wp_global_styles`, `wp_navigation`, `wp_font_family`,
`wp_font_face`. Taxonomies are `category`, `post_tag`, `nav_menu`, `wp_pattern_category`.

**Addis Energy registers no custom post type and no custom taxonomy.** There is no `project`, `well`,
`formation` or `funding` entity to find. The Advanced Custom Fields plugin exposes an `acf` member
on every object, but it was observed empty across every sampled object — no domain data flows
through it either. If you are looking for structured geologic or ammonia-production data, it is not
here; the machine-readable surface carries marketing content only.

## 4. Read the auth block, then stop looking for keys

The root document's `authentication` member advertises exactly one method: WordPress application
passwords, with an authorization endpoint at `/wp-admin/authorize-application.php`. That is a
credential a signed-in site administrator mints for themselves. **There is no developer signup, no
API key programme, and no OAuth server** — `/.well-known/oauth-authorization-server` and
`/.well-known/openid-configuration` both return 404. Do not build an onboarding flow; there is
nothing to onboard to.

## Error handling

| code | status | meaning |
|---|---|---|
| `rest_no_route` | 404 | The path is not registered. The root index is authoritative — re-read it. |
| `rest_forbidden` | 401 | Administrator-only route. Not remediable by a third party. |
| `wpcf7_forbidden` | 403 | Contact Form 7 definitions are private. |
| `rest_comment_disabled` | 403 | Comments are off site-wide. |
