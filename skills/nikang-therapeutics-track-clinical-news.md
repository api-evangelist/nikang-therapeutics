---
name: nikang-therapeutics-track-clinical-news
description: >-
  Pull NiKang Therapeutics' news archive from its own site and detect what is new since a prior check —
  press releases, clinical data announcements and scientific presentations for NKT2152, NKT3964 and
  NKT5097. Use when monitoring NiKang for pipeline or corporate developments.
api: nikang-therapeutics:nikang-therapeutics-posts-api
operations:
- listPosts
- getPost
- listCategories
---

# Track NiKang Therapeutics clinical and corporate news

NiKang publishes 16 news posts at `https://www.nikangtx.com/wp-json`. No key is required.

## Steps

1. **Confirm the category you want.** `GET /wp/v2/categories?per_page=100` (`listCategories`).
   Three categories are registered; only `news` (id 13) holds posts. `timeline` and `uncategorized`
   are registered but empty, so filtering by them returns nothing — that is correct, not a failure.

2. **List posts since your last check.** `GET /wp/v2/posts` (`listPosts`) with:
   - `after=<ISO 8601 timestamp of your last run>` — publication-date window
   - `modified_after=<same>` — catches edits to posts you already have
   - `per_page=100` (the hard maximum; 101 returns HTTP 400 `rest_invalid_param`)
   - `orderby=date&order=desc`
   - `_fields=id,slug,date_gmt,modified_gmt,link,title,excerpt` to keep the payload small

3. **Read the totals from the headers, not the body.** `X-WP-Total` and `X-WP-TotalPages` are
   returned on every collection response and are exposed cross-origin. If `X-WP-TotalPages` is
   greater than 1, follow the RFC 8288 `Link` header's `rel="next"` rather than incrementing `page`
   by hand.

4. **Fetch full bodies only for the new ids.** `GET /wp/v2/posts/{id}` (`getPost`). The rendered
   release text is in `content.rendered` as HTML.

5. **Resolve the hero image if you need it.** `featured_media` is an attachment id, or `0` when
   unset. Either call `GET /wp/v2/media/{id}` or re-run step 2 with `_embed` and read
   `_embedded['wp:featuredmedia'][0].source_url`.

## Rules

- **Do not try to resolve `author`.** `/wp/v2/users` returns 401 `rest_user_cannot_view`. The
  integer author id is a dead end for an anonymous caller. NiKang's named people live in
  `/wp/v2/dt_team`, not in the users collection.
- **Branch on `code`, never on `message`.** Errors are `{code, message, data:{status}}` as
  `application/json` — not RFC 9457 problem+json.
- **An HTML body is not an API error.** A non-browser User-Agent can be served a Cloudflare
  challenge page. Send a real browser User-Agent, and if the body is HTML with no `code` field,
  back off and retry rather than parsing it.
- **Rate.** No rate-limit headers exist. Honour `Crawl-delay: 10` from robots.txt and the
  `max-age=600` cache window. The archive changes a few times a year; polling faster than daily
  buys nothing.
- **Read-only.** Write methods are registered but require an authenticated WordPress user. There is
  nothing here to undo, and nothing to make idempotent.
