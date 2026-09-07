---
name: addis-energy-harvest-press-assets
description: Enumerate the Addis Energy media library — photography, technology diagrams, team headshots and investor logos — with alt text, MIME types and original source URLs, for assembling a press kit from a company that publishes no press-kit page.
api: addis-energy:addis-energy-media-api
operations:
- listMedia
- getMediaItem
- listPosts
generated: '2026-09-07'
method: generated
source: openapi/addis-energy-media-api-openapi.yml, data-model/addis-energy-data-model.yml
---

# Harvest Addis Energy press and brand assets

Addis Energy has no press-kit page and no asset CDN. Its entire image library is nonetheless
enumerable through the WordPress media endpoint, unauthenticated.

**Base URL:** `https://addisenergy.com/wp-json`
**Auth:** none.

## 1. Enumerate the library

```
GET /wp/v2/media?per_page=100&_fields=id,slug,alt_text,caption,media_type,mime_type,source_url,media_details
```

Verified live at 73 attachments on 2026-09-07 (`X-WP-Total: 73`). One request covers it.

## 2. What you get back

- `source_url` — the original file. This is what you download.
- `mime_type` — `image/png`, `image/jpeg`, `image/svg+xml`. Filter on this rather than on the
  filename extension.
- `alt_text` — the accessibility text the site author wrote. Often the best available caption.
- `media_details.sizes` — the generated variants (thumbnail, medium, large). Use one of these
  rather than the original when you only need a preview.
- `slug` — descriptive and stable; observed examples include team headshots and investor logos.

## 3. Tie an asset back to its story

Attachments carry a `post` field naming the post or page they were uploaded to, and posts carry
`featured_media`. To find the hero image for a news item:

```
GET /wp/v2/posts/{id}?_fields=id,title,featured_media
GET /wp/v2/media/{featured_media}?_fields=id,source_url,alt_text
```

`featured_media: 0` means the post has none — check before you dereference it. Attachments uploaded
outside a post context return `post: null`.

## 4. Rights

An asset being fetchable is not a licence. This library is a company website's media library, not a
released press kit: nothing on the site grants reuse rights. Check
`https://addisenergy.com/terms-conditions/` and ask `contact@addisenergy.com` before publishing any
of it. Team headshots are photographs of identifiable people — treat them as personal data.

## Error handling

| code | status | what to do |
|---|---|---|
| `rest_post_invalid_id` | 404 | The attachment id is gone. Re-list. |
| `rest_invalid_param` | 400 | `per_page` is bounded 1..100 and is rejected, not clamped. |
