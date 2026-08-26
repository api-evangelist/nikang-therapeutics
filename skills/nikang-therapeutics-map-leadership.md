---
name: nikang-therapeutics-map-leadership
description: >-
  Build a structured roster of NiKang Therapeutics' board members and officers, with biographies and
  headshots, from the company's own site. Use for diligence, partnership research or contact mapping.
api: nikang-therapeutics:nikang-therapeutics-team-api
operations:
- listTeamMembers
- getTeamMember
- listTeamCategories
- listMedia
---

# Map NiKang Therapeutics leadership and board

NiKang's named people are published through the The7 theme's `dt_team` custom post type — not through
the WordPress users collection, which is 401-gated. Ten people were published at profiling time.

## Steps

1. **Get the two team categories.** `GET /wp/v2/dt_team_category` (`listTeamCategories`). Two terms
   exist: id 10 `board` (count 7) and id 9 `officers` (count 2). The `count` field tells you how many
   people to expect before you fetch them.

2. **List the people.** `GET /wp/v2/dt_team?per_page=100&orderby=menu_order&order=asc`
   (`listTeamMembers`). Filter one group with `dt_team_category=10` (board) or `dt_team_category=9`
   (officers). Add `_embed` to pull the headshot and the category terms in the same round trip.

3. **Read each person.** In each object:
   - `title.rendered` — the person's name
   - `content.rendered` — the biography, as HTML
   - `dt_team_category` — array of term ids, mapping back to step 1
   - `featured_media` — attachment id of the headshot
   - `link` — the public page for that person

4. **Resolve headshots.** Either read `_embedded['wp:featuredmedia'][0].source_url` from step 2, or
   call `GET /wp/v2/media/{id}` (`listMedia` / `getMediaItem`) and take `source_url`, `alt_text` and
   `media_details` for the size variants.

5. **Fill gaps from the corporate page.** `GET /wp/v2/pages?slug=about-us` returns the about-us page,
   whose `content.rendered` carries founding and strategy narrative that the per-person biographies do
   not repeat.

## Rules

- **`acf` is empty.** An Advanced Custom Fields object is exposed on every `dt_team` record but was
  empty on every object sampled. Do not build a schema expectation on it; parse `content.rendered`.
- **Ten people is not the whole company.** This is the roster NiKang chose to publish, not a
  headcount. Do not present it as one.
- **Do not scrape the HTML pages for this.** The API returns the same content structured, and the
  HTML site is Cloudflare-fronted and will challenge a non-browser User-Agent sooner than
  `/wp-json` will.
- **No PII beyond what is published.** These records are the company's own public bios. Do not
  enrich them against third-party sources inside this skill.
