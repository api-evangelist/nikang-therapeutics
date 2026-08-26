---
name: nikang-therapeutics-extract-pipeline
description: >-
  Retrieve NiKang Therapeutics' drug development pipeline — programmes, targets, modalities and
  clinical stages — from the company's own site rather than a third-party database, and cross-check it
  against the dated news archive. Use for competitive landscape or target-class research.
api: nikang-therapeutics:nikang-therapeutics-pages-api
operations:
- listPages
- getPage
- searchContent
- listPosts
---

# Extract the NiKang Therapeutics pipeline

The authoritative pipeline statement is the `science-and-pipeline` page. Everything else is
corroboration.

## Steps

1. **Fetch the pipeline page by slug, not by id.** `GET /wp/v2/pages?slug=science-and-pipeline`
   (`listPages`). Slugs are stable across content edits; ids are not guaranteed to be. The page was
   id 638-adjacent at profiling time with `modified` 2026-02-12 — always read `modified_gmt` and
   report it alongside whatever you extract, because a pipeline is a point-in-time claim.

2. **Read `content.rendered`.** It is HTML. It carries, per programme: the compound code (NKT2152,
   NKT3964, NKT5097), the target (HIF2alpha, CDK2, CDK2/4, KRAS G12D), the modality (allosteric
   inhibitor vs targeted protein degrader), the indication, and the clinical stage.

3. **Corroborate each programme against dated news.** `GET /wp/v2/search?search=NKT3964&subtype=any`
   (`searchContent`), then `GET /wp/v2/posts?search=<compound>&orderby=date&order=desc`
   (`listPosts`). A press release with a date is stronger evidence of current stage than a page whose
   graphic may lag. Where they disagree, report both and say which is more recent.

4. **Get the strategy context.** `GET /wp/v2/pages?slug=about-us` for the discovery approach,
   founding, funding history and the named partnerships (Erasca on SHP2; Pfizer, Hansoh and Roche on
   HIF2).

## Rules

- **Report the retrieval date and `modified_gmt` with every claim.** A clinical stage read today can
  be stale tomorrow, and the page was last modified months before profiling.
- **Do not infer a stage the page does not state.** "Phase 1" and "Phase 1/2" are different claims,
  and discovery/IND-enabling is not a clinical stage at all. If the source is ambiguous, say so.
- **Do not treat this as regulatory or clinical-trial data.** It is a company marketing page. For
  registrational fact, go to the trial registries — NiKang publishes no FHIR, HL7, CDISC or
  clinical-data API of any kind, and nothing on this surface is a substitute for one.
- **`search` is the only cross-type route.** Scope it with `type` (post | term | post-format) and
  `subtype` (post, page, dt_team, category, dt_team_category, any). Everything else is per-collection.
- **Max `per_page` is 100.** Sending more returns HTTP 400 `rest_invalid_param`; the value is
  rejected, not clamped.
