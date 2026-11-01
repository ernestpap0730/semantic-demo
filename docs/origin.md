# Origin — the full content of every screenshot

> Source: `D:\screenshots` · 57 images across 9 subfolders · read 2026-10-25.
> This file holds what the screenshots **say**. It is a transcript, not an opinion.
> For the explanation of each folder, read [`summary.md`](./summary.md).
>
> Marked **[cut off]** where a screenshot clipped the text.
> Marked **[folder name mismatch]** where a folder name does not match its contents.

## Folder map

| # | Folder | Images | What it is |
|---|---|---|---|
| 1 | `chat/` | 1 | A Slack status message to the team |
| 2 | `jira/rn_5122/` | 5 | The sprint board, plus the **RN-5389** ticket (negated wishes) |
| 3 | `jira/Rn_5124/` | 4 | The **RN-5124** ticket (semantic ranking behind a quality gate) |
| 4 | `project/` | 8 | The RentalsCa repo in an editor: file tree, specs, a terminal |
| 5 | `semantic/github/` | 5 | GitHub pull request #518 on Rentals-API, plus a second repo |
| 6 | `semantic/pptx/` | 13 | The slide deck |
| 7 | `semantic/quality gate/` | 7 | The Semantic Quality Gate page (how to run the gate) |
| 8 | `semantic/search field test/` | 10 | The field test document, run 4 |
| 9 | `semantic/search quality report/` | 4 | The Semantic Search Quality Report, Oct 21 |

---

# 1. `chat/` — Slack status message

One screenshot. Posted by **Chris**, Wednesday 21 October, 3:30 PM.

> Hi, guys! Here's where the ranking engine and semantic search stand.
>
> **What changed**
>
> - **Ranking engine (RN-5121):** in cities where it's switched on, results are ordered by
>   a score (freshness, value, rotation, and how well a listing matches the words AI search
>   couldn't turn into filters) instead of newest first. Paid tiers still come first.
> - **Semantic search (RN-5124):** on top of that, listings are ranked by meaning, not just
>   exact words. "Quiet" also finds "calme" or "peaceful", and a French search can find an
>   English listing.
> - Every listing on develop now has a "meaning" built for it. The approval for sending
>   listing text is in place, and the index rebuilds itself after each weekly refresh.
> - AI search suggestions now keep those leftover words, so pressing Enter ranks by them too.
>
> **What you can try:** the RN-5124 test site, Montréal only, at `rn-5124.cd.rentals-dev.io`.
>
> Type into AI search and press Enter:
>
> - `Quiet apartment for working from home in Montreal`
> - `Apartment near the metro with a modern renovated kitchen in Montreal`
> - `Appartement près d'un parc et des espaces verts à Montréal`
> - `Appartement lumineux avec une grande terrasse à Montréal`
>
> Add `&semantic_ranking=0` to the results address to see the same search without meaning.

---

# 2. `jira/rn_5122/` — sprint board and the RN-5389 ticket

**[folder name mismatch]** The folder is named `rn_5122`, but the screenshots show the
sprint board and the **RN-5389** ticket. RN-5122 appears only as a card on the board.

Jira instance: `rentsync.atlassian.net` · Space: **Rentals Network** · Board:
**Rentals Network - Scrum** · Epic parent: **RN-5120 Ranking and meaning**

## 2.1 The sprint board

| Column | Count | Cards |
|---|---|---|
| TO DO | 2 / 49 | **RN-5341** Rank listings by distance to the nearby places… (Planning, 12) · **RN-5122** Use commute time as a ranking s… (**Blocked**) |
| IN PROGRESS | 1 / 29 | **RN-5389** Semantic search: handle negated wishes ("no…") (1: In Progress) |
| NEEDS PEER REVIEW | 1 / 14 | **RN-5124** Add semantic ranking behind a quality gate (4. Peer Review, 8) · *Needs input* |
| READY FOR QA | 1 / 2 | **RN-5123** Build the listing meaning index (2: Ready for QA, 12) · *Needs input* |
| IN QA | — | empty |
| FINAL REVIEW | — | empty |
| READY FOR RELEASE | — | empty |
| DONE | — | empty in view |

All cards carry the epic label **Ranking and meaning**. Assignee avatar: CS.

## 2.2 RN-5389 — Semantic search: handle negated wishes

Status **1: In Progress** · Priority **Medium** · Assignee **Christopher Da Silva** ·
Reporter **Christopher Da Silva** · Sprint **Oct 22 – Nov 5 (+1)** · Parent **RN-5120** ·
Category (migrated) **Product Improvement** · Applicability Driver (migrated)
**Customer-driven** · Story Points: none · Reviewers: none

### Problem

- "No carpets" ranks carpeted homes **first**. Keyword search ignores "no", and the meaning
  model reads "no carpets" as close to "carpets".
- The AI can turn a negated wish into the opposite filter ("no pets" → pet-friendly).

### Step 1: wrong filters taken back (RentalsCa) — built

- A fixed rule detects negations in English and French; look-alikes like "not too pricey"
  are skipped.
- A filter asking for the negated thing is removed. "No balcony needed" is ignored for ranking.
- Measured: **0 of 48** negated wishes became filters. The quality gate still passes.

### Step 2: push down listings that have the excluded thing (Jev)

1. **Search:** Jev answers "Does the renter want to avoid X?" X is the renter's own words.
2. **Listings:** Jev answers "Does this home have X?" for the top 48 results (pages 1–2), in
   2 calls of 24 at once (about 0.5 s). Answers are saved per phrase, and known features
   (carpet, basement, shared laundry) are checked ahead of time for every listing, together
   with photo tags and amenities.
3. **Ranking (Rentals-API):** listings flagged "yes" move down; all others keep their place.
   Off until measured.
4. No AI-written text ever reaches ranking. Jev only answers yes or no.

### Measured so far

| | Result |
|---|---|
| Jev accuracy (carpet, 440 listings) | 27/27, including "carpet-free" → no |
| Batches of 24 listings per call | 240/240 match single calls, about 0.5 s |
| Ranking, 27 negated searches | 0.37 → 0.39, **2 better, 0 worse** |
| Tried and dropped | AI rewrite (unpredictable), meaning penalty, contrast score, reranker (all worse or even) |

**Limit:** listing text rarely mentions what a home lacks, so text alone helps few searches.
Photo tags cover about **14%** of develop listings against about **99%** in production.

### Next

- Measure Step 2 on production-level photo tags.
- Rocky's service: add the Jev questions and Step 1's filter take-back, and return leftover
  words so ranking runs on that path.

### Acceptance criteria

- A negated wish is never a filter, on both the Django and service paths.
- With Step 2 on, no negated search ranks worse, and the quality gate holds.
- Only fixed codes and yes/no answers reach ranking.

### Comment by Christopher Da Silva (3 days before capture)

> Measurement before building step 2 (12 negated searches, 8 English and 4 French, every
> page-one listing graded blind; ranking quality nDCG@10, 1.0 = perfect first page):

| Version | All | English | French | Better / worse vs today |
|---|---|---|---|---|
| Today (negated words sent as keywords) | 0.447 | 0.570 | 0.202 | — |
| Step 1 only (negated words dropped) | 0.360 | 0.482 | 0.116 | 3 / 6 |
| Step 1 + meaning penalty, weight 1 | 0.347 | 0.480 | 0.083 | 3 / 6 |
| Step 1 + meaning penalty, weight 3 | 0.314 | 0.436 | 0.068 | 3 / 8 |
| Step 1 + meaning penalty, weight 10 | 0.329 | 0.435 | 0.116 | 3 / 9 |
| **Positive rewrite (hand-written)** | **0.572** | 0.564 | **0.587** | 8 / 4 |

> - **Penalty:** listings almost never name what the renter wants to avoid (1 of 28 Toronto
>   listings mentions carpet); they describe the **[cut off]**

---

# 3. `jira/Rn_5124/` — RN-5124, Add semantic ranking behind a quality gate

Status **4. Peer Review** · Priority **High** · Assignee **Christopher Da Silva** ·
Labels **rentalsca** · Sprint **Oct 22 – Nov 5 (+2)** · Story Points **8** ·
Parent **RN-5120** · Fix versions **None** · Category (migrated) **Product Improvement** ·
Applicability Driver (migrated) **Customer-driven**

## Summary

> Semantic ranking should only reach renters if it measurably beats the keyword ranking, in
> both English and French.
>
> This ticket adds the meaning-based signal to the ranking engine, tests it offline on real
> past searches, and releases it to a small percentage of renters with a kill switch.
>
> Result: renters searching for "bright and quiet" get homes that are, without losing any
> filter guarantees.

## Background

> The repo already has an offline promotion gate for the parse (`evaluate_ai_search`:
> labeled cases, per-provider, critical-field mismatches) and a percentage rollout via the
> `ai_search` user-tracking flag (`useAiSearchGate.js`). The meaning index must beat
> RN-5121's FTS-ranked baseline, not `random()`. The words the parser cannot map (the
> `ignored` log) are the progress metric: their share should fall as meaning is acted on.
>
> RN-4613 (Prep) is the existing "Semantic search — let renters search rentals.ca in plain
> language" parent of RN-4614 and RN-5001; this work continues that line. RN-4940 lists the
> French trigger phrases the FR gate must cover. RN-4866's attribution gap must be closed
> (RN-5109) before replayed searches can be labelled by outcome.

## Current state

- Baseline ranking logs per-lane + `search_id` (RN-5121).
- Unparsed-words log exists.
- Rollout precedent now includes the commute gate chain (`useCommuteGate.js`: master switch
  → URL override → budget → rollout-complete → local data → no-tracking default → bucket)
  and `RentalsTracking.ready()` for reading flags only once the SDK has them — reuse it,
  including a rollout-complete escape, because deleting a flag row reads as 0%.

## Scope

**In scope**

- Replay harness: replay logged queries; compare click/favourite rank under baseline vs
  hybrid; EN and FR slices; minimum sample sizes.
- Vector lane in fusion: `SortType.semantic`; rollout flag + kill switch.
- Report unparsed-words share before/after.

**Out of scope**

- Personalization (RN-5129 epic).
- Learned re-ranker.

## Dependencies

| Dependency | Status |
|---|---|
| RN-5121 | Blocked by |
| RN-5123 | Blocked by |
| RN-5109 `search_id` labels | Ready |

## Acceptance criteria

All three unchecked at time of capture:

- [ ] Hybrid beats baseline on both EN and FR replay slices by the agreed margin; otherwise
      no rollout.
- [ ] Rollout % configurable; kill switch tested.
- [ ] Unparsed-words share reported.

## Linked work items

**is blocked by**

| Ticket | Title | Status |
|---|---|---|
| RN-5121 | Build the ranking engine | **Done** |
| RN-5123 | Build the listing meaning index | 2: Ready for… |

**relates to**

| Ticket | Title | Status |
|---|---|---|
| RN-4613 | [RentalsCa] Semantic search — let renters search re… | Prep |
| RN-4940 | Add poi support to semantic search | Done |
| RN-4866 | Track leads and impressions by source for AI sema… | Prep |
| RN-5341 | Rank listings by distance to the nearby places asked… | Planning |

Subtasks: none. Agents panel: "Claude Agent for Jira History unavailable" (last month).

---

# 4. `project/` — the RentalsCa repo in an editor

## 4.1 Repository tree (`RentalsCa`)

```
RentalsCa/
├── rentalsca/
│   ├── providers.py
│   ├── runtime_init.py
│   ├── settings.py
│   ├── testing.py
│   └── threadrunner.py
├── rentalsca_phoenix/
│   ├── __pycache__/  management/  searchalerts_gen2/  static/
│   ├── templates/  templatetags/  tests/  views/
│   ├── __init__.py       aws_utils.py        billing.py
│   ├── caching.py        carousels.py        cityutils.py
│   ├── company_carousel.py                   emailactions.py
│   ├── emailbase.py      endpointapi.py      exceptions.py
│   ├── firstinquiry.py   flatten.py          i18namenities.py
│   ├── i18nkeywords.py   i18nmisc.py         i18nproptypes.py
│   ├── i18nseo.py        image_generation.py
│   ├── listingdraftreminders.md / .py
│   ├── listinghead.py    listingprocess.py   listingssr.py
│   └── listingupgrades.md
├── RentalsCa.egg-info/
├── rentalspiccer/
│   ├── __pycache__/  management/  tests/
│   ├── __init__.py   core.py   utils.py
├── scripts/
│   ├── gen_icons_demo.py   ruff_changed.py
│   ├── snyk_test.sh        strict_pytest.sh
├── specs/
│   ├── rn-3740-upgrade-to-django/
│   └── rn-5120-ranking-and-meaning/
│       ├── scripts/
│       ├── design.md        evaluation.md    README.md
│       ├── research.md      rn-5122-plan.md  spec.md
│       └── tasks.md
├── venv/
├── .bumpversion.cfg  .coveragerc  .dockerignore
├── .env  .env.secrets  .gitignore
├── .markdownlint.yaml  .npmrc
├── AGENTS.md  ARCHITECTURE.md  archived-changelog.rst
├── CHANGELOG.md  CLAUDE.md
```

Open editor tabs: `csv_to_python_array.sh`, `listing-meaning-index.md` (twice),
`rn-5122-plan.md`, `spec.md`, `tasks.md`.

## 4.2 `specs/rn-5120-ranking-and-meaning/rn-5122-plan.md`

### RN-5122 — POI proximity as a ranking signal: implementation plan

> **Status:** draft for review, re-scoped 2026-10-17, checked against the code the same day.
> Replaces the commute travel-time band plan (reset: RentalsCa `78c71ae4a`, Rentals-API
> `a54e8839`, still reachable by hash).
> **Branches:** `dev/chris/rn-5122/commute-ranking` in RentalsCa and Rentals-API, at the
> RN-5124 tips. Rentals-Deploy only for switching on (not in this ticket).

#### 1. Goal

> When a search asks for a place nearby ("near a park", "close to the metro"), the closer
> listings come first within the radius, and a result can say how close ("Park 180 m").
> Today the POI filter is in/out only: a listing 50 m from the park and one 480 m away rank
> alike (semantic field test, run 3).
>
> Built on RN-5114's precomputed distances (`rentalscore_listing_poi_distance`: each
> listing's distance to its nearest POI of a category), on the server, inside the `ranked`
> and `semantic` sorts, so it works together with keywords and meaning (RN-5121, RN-5124):
> "2 bedroom near a park on a quiet street" ranks by the park's distance *and* by "quiet
> street". No provider calls, no credits, no per-search cost beyond one indexed read per
> matched listing.

**Acceptance (re-scoped):**

- Within a search that has a nearby-POI filter, listings closer to the asked-for place type
  rank higher, as a weighted factor alongside keywords and meaning, on both engines alike.
- Each result's ranking states its distance to that place type; server-rendered cards say it
  ("Park 180 m"). In-place pages are ordered the same way but unlabeled (the browser's
  GraphQL library selects no ranking).
- No regression in page time; nothing changes for searches without a nearby-POI filter.

#### 2. What exists (RN-5114, measured on the local copy of develop)

| | |
|---|---|
| **Table** | `rentalscore_listing_poi_distance`: `property_id`, `category_id`, `metres` (rounded up; NULL = nothing within 5 km), unique `(property_id, category_id)`, index `(category_id, metres)` |
| **Categories with rows** | `bus_stop`, `grocery_store`, `gym`, `park`, `subway_station`, `university` (all 26,202 listings each); `streetcar_stop` and `train_station` have no rows (no POIs) |
| **Freshness** | `sync_listing_poi_distances` hourly, `--full` nightly — **but the cron is off in every environment** in Rentals-Deploy |
| **Rentals-API use** | the nearby-POI **filter** only, quick engine, for categories in `nearby_poi_precomputed_categories` — **empty everywhere deployed** |
| **Spread inside the default radius** | park (500 m) p10/p50/p90 = 74/203/391 m; metro (800 m) 124/355/678 m; bus (400 m) 43/124/286 m |

> So the data is real and varied enough to reorder results, but production does not compute
> it yet: switching the feature on there depends on RN-5114's rollout (the cron and the
> setting).

#### 3. How a search gets its places (unchanged)

- AI search turns "near a park" into `nearby_pois=park` (8 categories, `POI_CATEGORIES`,
  default radii: park 500 m, metro 800 m, bus 400 m, grocery 400 m, gym 800 m, university
  1,500 m).
- The Nearby Places picker sets the same filter in the browser.
- Both reach Rentals-API as `filters.nearbyPois: [{category, maxDistanceMeters}]` — from the
  server's search **and** from the browser's own in-place query (`rentalsjs-gql` sends
  `filters`). So a factor derived from the filters needs **no new argument** and applies to
  in-place pages too.
- A named place ("near McGill") is either a commute zone (AI search) or a `name`
  requirement: no precomputed data. Out of scope.

#### 4. Design (Rentals-API)

> Reviewed against the code on 2026-10-17; the review's fixes are folded in (marked *review*).

**A factor, not a lane.** `proximity`, one more weighted factor of the `ranked` score (as
keywords and meaning): a lane would put a listing 50 m from a park above every quieter,
cheaper, better-matching one. As a factor it blends.

**Switched on explicitly** (*review*). A new setting `ranking_proximity_categories` (empty by
default, like `nearby_poi_precomputed_categories`): only the categories listed there count.
"Has rows" is not enough: develop has rows from a one-off backfill (2026-10-05, cron
suspended) and a table's age cannot be read from it. Enabled per site in Rentals-Deploy
together with RN-5114's cron. Weight `ranking_proximity_weight`, proposed **2.0**, chosen on
the golden set (T106).

**Which requirements count.** From the filter sets that filtered (`applied_filter_sets`):
category-only requirements (no `name`, no `subcategory`) whose category is listed, each with
its own clamped radius (`_nearby_poi_radius`), deduplicated by (category, radius) as the
filter does. Named places, subcategories and unlisted categories still filter as today and do
not rank.

**Per-listing value.** For each counted requirement: `closeness = 1 − metres / radius` only
when the stored `metres <= radius`; otherwise (a stale row against the live filter, which is
the deployed path) or with no row (listed since the last sync), **unknown** (*review*). The
factor is the mean of the known closenesses; a listing with none known scores the mean of
the others, neither helped nor sunk, as a listing with no meaning vector does.

**OR'd filter groups** (*review*, quick engine only; RentalsCa never sends groups): each set
scored on its own, the listing taking its best set's score; the factor applies only when
every set that filtered has a counted requirement.

**SQL** (*review*). One LATERAL per search, `OFFSET 0` (as `keyword_relevance`), joining the
listing's rows to an unnest of the requirements' `(category_id, radius)` pairs on the unique
`(property_id, category_id)` index: one probe per listing and counted category. It returns
the factor and each counted category's `metres` (for `places`). Same builder for both
engines: the quick engine filters from this table already, and sqladb, which filters live,
reads it for ranking only.

**Explanation** (*review*). `ranking.factors` lists `proximity`. New
`RankingExplanation.places: [RankingPlace {category: POICategory!, metres: Int}]`, only
categories with a known closeness. Built in `ranking_explanation` for both engines; the
per-category columns get a `ranking_place_` prefix that `pop_ranking_columns` pops, so none
leaks into the listing record. `RankingFactor`'s docstring lists the new name.

**Semantic search.** No change to the meaning path: the factor sits beside it. The keyword
band still leads, then the score (proximity included) orders each band.

#### 5. Design (RentalsCa)

- **First, independently: stop category words ranking as keywords** (*review*). AI search's
  leftover words are all its `ignored` terms, and nothing removes a word whose category
  became a filter. Worse, PostgreSQL's English stemmer reads "parking" as `park` (checked):
  a leftover "park" puts every listing mentioning parking (17,693 of 26,202 on the develop
  copy) in the keyword band. Withdraw `ignored` terms that name an applied category, as
  `withdraw_applied_system_terms` does for transit systems, with golden cases. **Worth
  shipping now: it hurts keyword ranking today.**
- **No new search input.** The server already sends `nearbyPois`; the browser's in-place
  query sends the same categories and radii (same Python table), and `ranked`. So in-place
  pages are ordered alike, and a keyword-less page 2 fetched in place continues page 1.
- **GraphQL document:** `ranking { … places { category metres } }` behind a `with_places`
  switch, off by default (`api.py`'s pattern), so an older Rentals-API never breaks a page.
- **Labels** (*review*): server-rendered cards and their SSR twin say "Park 180 m". Needs a
  Python map of the category labels mirroring JS's (`nearbyPoiMarkers.js`), one distance
  formatter for both (EN "1.2 km", FR "1,2 km"), and a template lookup by listing id from
  `result_rankings`.
- **Limitation, stated in the acceptance:** in-place pages carry no ranking (the browser's
  library selects none), so a Nearby Places search (no keywords) shows the label on its
  server-rendered page 1 only; page 2 and any filter change are ordered by proximity but
  unlabeled. Labelling them needs `ranking { places }` in `rentalsjs-gql` (another repo): a
  follow-up decision.
- **Events:** `listing_impression` already sends the whole ranking (now with `proximity` and
  `places`). `search_results_rendered.ranked_by_proximity`, derived from the rankings (as
  `_ranked_by_meaning`), not the filters.

#### 6. Tasks (review's order)

| Task | Repo | What |
|---|---|---|
| T101 | RentalsCa | Withdraw `ignored` terms naming an applied POI category; golden cases. Independent, shippable now |
| T102 | Rentals-API | Settings (`ranking_proximity_categories`, weight); requirements with radii from the applied filter sets; the LATERAL (factor + metres per category); `metres <= radius` else unknown; best set across OR groups; both engines |
| T103 | Rentals-API | `RankingExplanation.places`, popping, both engines, GraphQL type, contracts, docs |
| T104 | Rentals-API | Tests: unit, integration (stale row, no row, several categories, OR groups), GraphQL (both engines alike; no filter = unchanged) |
| T105 | both | Measure: `EXPLAIN (ANALYZE, BUFFERS)` on Toronto and Montréal, 0/1/3 requirements, live and precomputed filter paths, cold and warm; LATERAL vs a hash join |
| T106 | RentalsCa | Generalise `evaluation.py`'s re-weighting from meaning to any factor; choose the weight on the golden set's POI searches (ranked cities) |
| T107 | RentalsCa | `with_places` (off by default); labels (Vue + SSR, EN/FR); `ranked_by_proximity`; i18n |
| T108 | both | Docs, CHANGELOG; Jira text (draft below); Rentals-Deploy (separate): RN-5114 cron per site, then the categories setting, then `with_places` |

#### 7. Risks

- **Production has no current rows** (RN-5114's cron is off): nothing changes until a site
  enables the cron and lists categories.
- **Pods briefly disagree** after a change of the setting or a backfill (coverage cached per
  process): page 1 and page 2 could differ for a few minutes. Minor.
- **Only where the `ranked` sort applies** (`RANKED_SORT_PLACES`); the field test must use
  those cities.
- **Page time:** one index probe per matched listing and counted category; measured before
  choosing defaults (T105).

#### 8. Open decisions

1. **Default weight** (proposed 2.0), chosen on the golden set.
2. **More categories:** restaurant (32,564 POIs), coffee shop (12,498), school (16,205) have
   POI data but no precomputed distances and are not AI-search filters. Adding them would let
   "near restaurants" filter and rank by distance. A separate step: it changes what AI search
   filters.
3. **Labels on in-place pages:** add `ranking { places }` to `rentalsjs-gql` (another repo),
   or accept server pages only.
4. **Without a POI filter:** proximity ranks only from filters (proposed), never from
   leftover words.
5. **RN-5122 Jira text** re-scoped (draft below); commute travel-time bands move to a later
   ticket or are dropped.

#### 9. Draft Jira text for RN-5122 (for approval)

> **Use proximity to nearby places as a ranking signal.** When a search filters by a nearby
> place type (park, metro, grocery, gym, bus stop, university), rank closer listings higher
> within the radius, as a weighted factor of the `ranked`/`semantic` score, from RN-5114's
> precomputed distances; each result can state its distance ("Park 180 m"). No provider
> calls. Depends on RN-5114's cron being enabled per site. The commute travel-time bands
> previously planned here move to a separate ticket.

## 4.3 `specs/rn-5120-ranking-and-meaning/spec.md`

> **Spec: Ranking and meaning (RN-5120)**
> Branch: `dev/chris/rn-5120/ranking-specs` · Created: 2026-10-06 · Status: **Draft**

Section headings present but **empty** at capture: Problem · Goals · Non-goals · Users and
scenarios · Primary scenario · Acceptance scenarios · Edge cases · Requirements · Functional
requirements · Non-functional requirements · Business constraints.

**Done when** (all unchecked):

- [ ] The default order is deterministic and explainable; page two never repeats page one.
      (RN-5121)
- [ ] Within a search filtered by a nearby place type, closer listings rank higher, with
      their distance stated. (RN-5122)
- [ ] Every active listing has an embedding kept current within one refresh window. (RN-5123)
- [ ] Semantic ranking beats the keyword baseline on the golden set in EN and FR (the replay
      of real searches was retired on 2026-10-16: events no longer record a search's words),
      and is live for a rollout percentage. (RN-5124)

## 4.4 `specs/rn-5120-ranking-and-meaning/tasks.md`

> **Tasks: Ranking and meaning (RN-5120)**

**Prerequisites**

- [x] RN-5109 Instrument search latency and establish a performance baseline
- [ ] RN-5114 Precompute each listing's distance to nearby places (RentalsCa #1592,
      Rentals-API #506, Rentals-Deploy #421)

**RN-5121 — Build the ranking engine** *(no body)*

**RN-5122 — Use proximity to nearby places as a ranking signal**

> Re-scoped 2026-10-17; tasks T101–T108 in `rn-5122-plan.md`. The commute travel-time band
> work was reset (RentalsCa `78c71ae4a`, Rentals-API `a54e8839`) and moves to a later ticket.

**RN-5123 — Build the listing meaning index** *(no body)*

**RN-5124 — Add semantic ranking behind a quality gate** *(no body)*

**Downstream (other epics, blocked by RN-5121 or RN-5123)**

- RN-5128 Use photo tags and captions in search ranking
- RN-5131 Spike: build a taste profile for each renter
- RN-5132 Launch personalized alerts, feeds and similar homes

**Validation checklist**

- [ ] Every functional requirement in `spec.md` maps to at least one task
- [ ] Every quality gate in `evaluation.md` has a task that measures it
- [ ] SSR templates updated where visible order or content changes (CLAUDE.md, SSR parity)

## 4.5 The terminal pane (RN-5389 hand-off notes)

> **How it works:** for a search like "no carpets" on the results page:
>
> 1. Our negation detector reads what the renter wants to avoid from the search words
>    ("carpets"). That needs no AI.
> 2. The page fetches the first two pages' listing IDs (48) in one Rentals-API request.
> 3. Jev is asked, 24 listings per call with all calls at once: "Does Listing *n* say its
>    home has `<carpets>`?"
> 4. Listings Jev says "yes" to go to Rentals-API as `demote` and move to the bottom of their
>    paid tier, on every page.
> 5. Answers are saved for a day per phrase and listing text, and each answer is kept for an
>    hour, so pages 1 and 2 agree.
> 6. If Jev fails or is slow, or Rentals-API doesn't know `demote`, the page ranks as it does
>    today.
>
> **Checks:**
>
> - **End to end** (local, real Rentals-API and real Jev): the "no carpets" Toronto page. Jev
>   flagged one listing ("Tile and carpet flooring"). With the switch off it's #2 on page 1;
>   with it on, it's off pages 1 and 2.
> - **Speed:** about 1 s extra the first time a negated search is seen (typically 1,124 ms
>   against 95 ms), then nothing extra for repeats. The cost comes from the extra Rentals-API
>   request plus Jev. If the 1 s is a problem for the first visitor, we could do the Jev check
>   in the background instead, so the first visitor sees today's ranking and later visitors
>   get the improved one.
> - **Tests:** 3,055 RentalsCa tests pass, and 404 Rentals-API tests on both engines. Lint is
>   clean on my lines.
>
> **RentalsCa** (`dev/chris/rn-5389/jev-exclusions`, uncommitted): the Jev module
> (`rentalsbase/rentler/meaning/negated.py`), the `demote` plumbing, the results-page hook,
> the `NEGATION_DEMOTE_ENABLED` setting (off by default), the feature-flag doc and the
> CHANGELOG. It also includes the earlier AI-rewrite removal. Some files contain both changes,
> so I'd make one commit:
>
> ```
> RN-5389: replace the AI rewrite with Jev exclusions
>
> Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
> ```
>
> Then push and open a PR into `dev/chris/rn-5124/query-meaning`:
>
> ```
> git push -u origin dev/chris/rn-5389/jev-exclusions
> ```
>
> **Rentals-API:** you committed it (`dbc7a84e`). Push it and open its PR into
> `dev/chris/rn-5124/meaning-index-read`:
>
> ```
> git push … origin dev/chris/rn-5389/negated-wishes
> ```
>
> **Before it works on the RN-5124 test site:**
>
> 1. Merge the Rentals-API PR, so the test site's Rentals-API knows `demote`.
> 2. Ask Rocky to add `NEGATION_DEMOTE_ENABLED: "true"` to `cd/rn-5124.yaml` in
>    Rentals-ArgoCD-Apps.
> 3. **Data approval:** this sends listing text and the avoided words to Workers AI's Jev
>    model. The October 20 approval covered bge-m3 embeddings on the same provider. Check
>    whether that approval also covers Jev.

A second terminal screenshot adds:

> One thing to know before merging: the comma change alters the `keywords` in every AI search
> results URL, not just negated ones ("bright, big windows" instead of "bright big windows").
> Ranking reads the same words, so results don't change. But cached pages and shared links
> using the old form are now different URLs, so each gets a fresh cache entry the first time.

Status lines seen: `Crunched for 9m 47s · done 11:46 AM` and `Sautéed for 6m 3s · done 12:58 PM`,
with `auto mode on (shift+tab to cycle)`.

---

# 5. `semantic/github/` — pull request #518 and a second repo

## 5.1 PR #518 — Rentals-API

`github.com/torontorentalsuser/Rentals-API/pull/518`

> **RN-5124: Add the semantic sort (rank by meaning)** #518 · **Open**
> `cdsrentsync` wants to merge **18 commits** into `prerelease` from
> `dev/chris/rn-5124/meaning-index-read`

Conversation 5 · Commits 18 · Checks 10 · Files changed 36 · **+1,946 −84**
Reviewers: **brendandagys**, Copilot (Balanced) · Labels: `auto-deploy` · 2 participants ·
No assignees · No milestone · No projects

### Summary

> RN-5124: a `semantic` sort for `rentalListings`. It is the `ranked` sort (RN-5121) with one
> more factor: how close each listing's meaning is to the search's words, read from the
> listing meaning index RentalsCa builds (RN-5123). The caller sends the query's vector;
> Rentals-API never calls an embedding provider.
>
> Docs: `docs/concepts/search-ranking.md` (the factor, its edge cases, errors) and the new
> `docs/concepts/listing-meaning-index.md` (the tables, the active index, grants).

### What's in it

- `sortType: semantic` with `meaning: {indexId, vector}` (`types/criteria.py`,
  `sqladb/sorting.py`, both engines):
  - Meaning is cosine similarity to the listing's vector, scaled over the matched listings,
    weighted by `ranking_meaning_weight` (**default 3**, as keywords).
  - It only reorders listings within their paid tier and keyword band; it adds no band of its
    own.
  - A listing with no vector scores the mean of the others, so a missing vector neither helps
    nor hurts.
  - Each edge's `ranking` lists the `meaning` factor, as it does the others.
- The active index (`sqladb/meaning_index.py`): read from RentalsCa's
  `rentler_embedding_index`, cached per process for 60 s.
  - A vector for another index or of another size ranks without meaning (an info line, never
    the vector): expected for up to a minute after RentalsCa switches indexes.
  - The lookup checks that both engines' roles may read the vectors; a missing grant turns
    meaning off instead of failing every semantic search.
- **Bound values stay out of errors** outside development, on both engines
  (`hide_parameters`): a search's words and vector never show in a GraphQL error or a
  resolver's warning. This applies to every statement, not only searches, so a failing import
  or lead statement also logs its SQL without values.
- **A malformed vector is the caller's mistake:** empty, over 4,096 numbers, not finite,
  outside 32-bit range, all zeros, or too short to compare. It raises `MeaningVectorError`:
  an error in the response whose message starts `meaning.vector:` (RentalsCa reads this as a
  refusal), logged at INFO without a traceback or the vector, on both engines.
- CI moves to the `15-3-4.1.4-pgvector` Postgres image and checks the `vector` extension is
  available.
- Also in this PR: `contracts/artifacts/schema.graphql` catches up on five POI categories
  from RN-5012 (it was stale), a time-pinning fix in `test_daily_highlighted`, and a
  formatter change in one CI test.
- Contracts regenerated: `contracts/rentals-api-schema.graphql`, `baseline-schema.graphql`,
  `contracts/artifacts/schema.graphql`.

### Before merging

1. The `vector` extension must exist wherever this runs (it already does where RN-5123
   migrated). CI needs the `postgres-server-postgis-15-3-4.1.4-pgvector` image, which is
   published.
2. Rentals-API reads RentalsCa's tables; RN-5123's migration must be applied in that database
   first. Until an index is active, `semantic` ranks exactly as `ranked`.

### Not done in this PR (on purpose)

- No caller turns `semantic` on yet: RentalsCa sends it only behind its switch and rollout
  (RentalsCa PR).
- Weights stay at their starting values until RentalsCa's golden-set gate
  (`manage.py semantic_golden`) has enough graded cases to tune them.
- Sentry's report of an exception still holds the stack's local variables (the vector among
  them); production's Sentry and a Python 3.13 crash in the Sentry SDK are a separate ticket.
- The vector-size checks stay three (non-finite, all zero, too short or too long for 32-bit);
  one simpler check was suggested in review and left for later.

### Testing

- **Unit and integration tests:** full suite **2,738 passed, 25 skipped**, on the pgvector
  image. Both engines rank a meaning query alike; NULL, all-zero and missing vectors, one
  listing and all-equal similarities are covered.
- **Contracts:** all three schema files regenerate byte for byte.
- **Local end-to-end:** RentalsCa's results pages ranked by meaning against this branch on a
  copy of the dev database.

### Related

- RentalsCa PR (`dev/chris/rn-5124/query-meaning`): embeds the words and sends `semantic`.
- Rentals-Deploy PR (`dev/chris/rn-5124/semantic-ranking-settings`).
- RN-5123 (merged): the listing meaning index.

### Commits (18, all by `cdsrentsync`)

| Message | SHA | Checks |
|---|---|---|
| RN-5124: read the listing meaning index and test on pgvector | `c525ea6` | |
| RN-5124: add a meaning factor to the ranked sort | `61c1cea` | |
| RN-5124: add the semantic sort with a meaning query vector | `7d47034` | |
| RN-5124: refuse vectors pgvector cannot compare | `a2ab47d` | |
| RN-5124: keep the meaning lookup a subquery and refresh the baseline | `7abad68` | |
| RN-5124: read the query vector once and pin the daily highlight day | `e90e303` | |
| RN-5124: correct the saturated distance notes and pin ranking days | `2ce1d60` | |
| RN-5124: pin the semantic test day and guard stored vector sizes | `e3d0a49` | |
| RN-5124: hide bound values and score missing vectors at the mean | `c7c7786` | |
| RN-5124: scope hidden values to searches and correct meaning notes | `4c3dd30` | |
| RN-5124: hide values on both engines, probe rw, fix docs | `9b71331` | |
| RN-5124: vector error type, quiet quick engine, review fixes | `af29059` | 3/4 |
| **Oct 17, 2026** — RN-5124: **weigh meaning less among keyword matches** | `82cf525` | ✓ 6/10 |
| **Oct 20, 2026** — Merge branch 'prerelease' into `dev/chris/rn-5124/meaning-index-read` | `5dd5a97` | ✓ 6/10 |
| **Oct 21, 2026** — RN-5124: review fixes: grant check tests, 0-weight factors, Top Slot | `1926edd` | ✓ 8/10 |
| **Oct 21, 2026** — RN-5124: **weigh meaning fully among keyword matches** | `5e86173` | ✗ 7/10 |
| **Oct 21, 2026** — RN-5124: final gate numbers in the meaning band weight docs | `8571c92` | ✗ 3/10 |
| **Oct 21, 2026** — Merge remote-tracking branch 'origin/prerelease' into `dev/chris/rn-5124/meaning-index-read` | `6ef3d78` | ✓ 8/10 |

## 5.2 Second repo: `rentals_ai_search_rs`

`github.com/torontorentalsuser/rentals_ai_search_rs` · **Private**

> **About:** Rust service that turns a plain-language rental search into a filtered
> rentals.ca results URL

`main` · 16 Branches · 0 Tags · Issues 5 · Pull requests 3 · 0 stars · 0 watching · 0 forks ·
No releases published · No packages published

Latest commit: `rocky-rentals` — "Merge pull request #23 from torontorentalsuser/de…"

| Path | Last commit message |
|---|---|
| `.github/` | ci(build-image): |
| `docs/` | ci(build-image): |
| `scripts/` | RN-5218: Releas… |
| `spikes/` | docs: record Re… |
| `src/` | RN-5218: Releas… |
| `static/` | chore: cargo-wa… |
| `.dockerignore` | build(image): add a multi-arch release Dockerfile (RN-…) |
| `.gitignore` | feat(config): load settings from config.toml via configure |
| `AGENTS.md` | RN-5218: Release develop to main (POI v2 parity, neigh…) |
| `ARCHITECTURE.md` | RN-5218: Release develop to main (POI v2 parity, neigh…) |
| `Cargo.lock` | feat(metrics): export OTLP metrics for requests, model … |
| `Cargo.toml` | feat(metrics): export OTLP metrics for requests, model … |
| `Dockerfile` | build(image): add a multi-arch release Dockerfile (RN-…) |
| `README.md` | docs: resolve docs-architecture audit warnings |

**Contributors (2):** `rocky-rentals` (Rocky Burt, Rentals) · `brendandagys` (Brendan Dagys)

---

# 6. `semantic/pptx/` — the slide deck

13 slides. Full transcript below; also kept standalone in [`../semantic/ppt.md`](../semantic/ppt.md).

### Slide 1 — Title

## Semantic search

**Ranking rental results by what renters mean, not only by the words they type**

```
RN-5121 · RN-5124 · RN-5389 · rentals.ca · October 2026
```

---

### Slide 2 — Some wishes can't become filters

> "sunny apartment with a nice view in Montreal under 2000"

| | |
|---|---|
| **Became filters** | `Montréal` · `Apartments` · `Rent under $2,000` |
| **Left over** | `sunny` · `nice view` |

28 of 42 test searches left words over. Keywords only match the exact words: "ensoleillé"
or "lumineux" never match "sunny".

```
AI search · keyword ranking RN-5121 · field test, 42 searches
```

---

### Slide 3 — How semantic search works

**Every hour, ahead of time**

| Step | Name | Detail |
|---|---|---|
| 1 | Listing text | description, captions and photo tags |
| 2 | Details removed | phones, emails, links |
| 3 | **Meaning model** | bge-m3 on Workers AI |
| 4 | **Meaning index** | stored per listing in our database |

↓ *compared: closer meaning ranks higher*

**Each search, live**

| Step | Name | Detail |
|---|---|---|
| 1 | Leftover words | "quiet street", after the filters |
| 2 | Details removed | names, phones, emails, addresses |
| 3 | **Meaning model** | same model; 2 s limit, kept a day |
| 4 | **Ranking** | paid tier, then words, then score |

```
website: Django · ranking: Rentals-API · index: PostgreSQL with pgvector · model: bge-m3
```

---

### Slide 4 — How a position is decided

| Order | Step | Rule |
|---|---|---|
| First | **Paid tier** | higher placements always lead |
| Then | **Has the words** | listings that contain them come first |
| Within each | **Score** | five factors, weighted |

| Factor | A listing scores higher when | Weight |
|---|---|---|
| Freshness | it was listed more recently | 1 |
| Value | its rent is lower than similar units | 1 |
| Rotation | today's turn among equals | 0.5 |
| Keywords | its text has more of the words | 3 |
| **Meaning** | its meaning is closer to the words | **3** |

```
ranked sort RN-5121 · meaning factor RN-5124 · meaning weight set by the quality test
```

---

### Slide 5 — Green spaces, asked in French

> "appartement près d'un parc et des espaces verts"

**Filters:** `Apartments` · `Park nearby` — **Left over:** `espaces verts`

| Filters only | With meaning |
|---|---|
| **0 of 10** | **3 of 10** |
| clear matches · 1 part match | clear matches · 3 part matches |
| 9 never mention a park or green space | • "face à un beau et grand Parc"<br>• "Jardin botanique, parc Maisonneuve"<br>• "pistes cyclables du parc Maisonneuve" |

```
field test run 4 · first 10 of the standard tier · none contain the words, so before = filters alone
```

---

### Slide 6 — A quiet place to work from home

> "appartement calme pour le télétravail"

**Filters:** `Apartments` — **Left over:** `calme pour le télétravail`

| Filters only | With meaning |
|---|---|
| **0 of 10** | **2 of 10** |
| clear matches · 2 part matches | clear matches · 3 part matches |
| 8 say nothing about quiet or remote work | • "Bright, Quiet" (an English listing)<br>• "immeuble tranquille"<br>• "Insonorisation supérieure" |

```
field test run 4 · first 10 of the standard tier · none contain the words, so before = filters alone
```

---

### Slide 7 — A renovated kitchen near the metro

> "walking distance to the subway, renovated kitchen"

**Filters:** `Metro nearby` — **Left over:** `renovated kitchen`

| Filters only | With meaning |
|---|---|
| **0 of 10** | **1 of 10** |
| clear matches · 0 part matches | clear match · 4 part matches |
| none mention a renovated kitchen | • "Cuisine et salle de bain entièrement rénovées"<br>• "nicely updated split level"<br>• "Condo tout neuf" |

```
field test run 4 · first 10 of the standard tier · none contain the words, so before = filters alone
```

---

### Slide 8 — The quality test in numbers

| Number | Label | Detail |
|---|---|---|
| **+68%** | better first page | Score 0.42 → 0.71 out of 1, against AI search before this work, when leftover words were dropped. |
| **93 of 103** | searches improved | A better first page in 93 searches, the same in 6, worse in 4. |
| **+72%** | English | 0.43 → 0.74 over 71 searches; 65 better, 2 worse. |
| **+58%** | French | 0.41 → 0.65 over 32 searches; 28 better, 2 worse. |
| **Passed** | meaning quality gate | Meaning adds +6.0% in English and +8.4% in French over keywords alone; the bar is 5%. |
| **26,117** | listings with a meaning | Every listing on the develop copy, refreshed every hour. |

```
103 searches with leftover words · graded blind by an AI model · 2026-10-21 · develop copy
```

---

### Slide 9 — "No carpets" now ranks the right way

> "apartment with no carpets in Toronto"

**Before:** ranked by "carpets" → **Now:** ranked by "hardwood floors"

| 15 new negated searches | 12 in the quality test | Filters |
|---|---|---|
| **+25%** | **+31%** | **0 of 48** |
| score 0.41 → 0.52 · 8 better, 0 worse | score 0.39 → 0.51 · 8 better, 3 worse | negated wishes applied as a filter |

```
RN-5389 · English and French · "no pets", "sans tapis", "not a condo", "no balcony needed"
```

---

### Slide 10 — bge-m3 chosen; OpenAI tested and dropped

| In use · Cloudflare Workers AI | Tested · OpenAI |
|---|---|
| **bge-m3** | **text-embedding-3-small** |
| • 1,024 numbers per text | • 1,536 numbers per text |
| • $0.0118 per million tokens | • $0.02 per million tokens |
| • 26,117 listings indexed on develop | • no better than bge-m3 on the quality test |

**Quality gate: passed.** Meaning had to add 5% or more over keywords alone, in both
languages. It adds +6.0% in English and +8.4% in French: 58 searches better, 17 worse.

```
manage.py semantic_golden · 71 English, 32 French searches · models switch without a deploy
```

---

### Slide 11 — A renter's words stay private

| | |
|---|---|
| **Personal details removed** | names, phones, emails and addresses, before the words leave |
| **Never recorded** | events say only whether meaning ranked the page |
| **Out of error reports** | reports and traces drop the words and any map location |
| **Never breaks a page** | 2-second limit; any failure ranks by keywords alone |

```
seven review rounds on privacy · only made-up test searches sent to providers
```

---

### Slide 12 — Three switches, all off in production

| # | Switch | Detail |
|---|---|---|
| 1 | **Master switch** | per environment; off in production |
| 2 | **Cities with the new ranking** | turned on city by city |
| 3 | **Share of searches** | a percentage; 0% stops it with no deploy |

The quality test has passed on develop, with every listing in the meaning index. Before
production, a person spot-checks its grades.

```
SEMANTIC_RANKING_ENABLED · RANKED_SORT_PLACES · semantic_ranking rollout · golden set gate
```

---

### Slide 13 — What's done, and what's left

| Status | Item | Detail |
|---|---|---|
| **Done** | Full meaning index on develop | 26,117 listings, approved Oct 20, refreshed hourly |
| **Done** | Quality test passed | +6.0% English, +8.4% French over keywords alone |
| **Before production** | A person checks the grades | a spot-check of the AI model's grades, then the production index |
| **Next ticket** | Commute-aware ranking | closer to the metro or park asked for ranks higher |

```
try it: RN-5124 test site · RN-5122 uses RN-5114's stored distances
```


---

# 7. `semantic/quality gate/` — the Semantic Quality Gate page

> **RN-5124 · SEMANTIC SEARCH**
>
> # Semantic Quality Gate
>
> Before ranking by meaning goes to production, we grade real search results with and without
> it. An AI model grades every listing blind, a person spot-checks a sample, and the gate
> passes only if meaning makes the top 10 clearly better in both English and French.

**Status pills at capture:**

| | |
|---|---|
| 🟢 | Meaning index live on the RN-5124 test site |
| 🟠 | **Gate not run yet** |
| 🟠 | AI grading and 44 test searches built, **not yet merged** |

## How it works

| Step | Name | Detail |
|---|---|---|
| 1 · Collect | **Run every search twice** | Each test search goes through AI search, then its results page is fetched without meaning and with it. |
| 2 · Export | **List what to grade** | Every listing in either top 10 goes into one spreadsheet, with the text the meaning model read. |
| 3 · Grade | **AI grades, a person spot-checks** | The model grades every listing 0, 1 or 2, never told which order it came from. A random 10% goes to a person, graded blind. |
| 4 · Score | **Compare and decide** | Scores both orders per language and weight, says pass or fail, and how often the person agreed with the model. |

## What passing means

| Rule | Detail |
|---|---|
| **5% better** | Average top-10 quality with meaning at least 5% above today's ranking, in English and in French separately. |
| **20+ searches** | At least 20 compared searches in each language, so one lucky search can't carry the result. |
| **90% graded** | Every compared top 10 at least 90% graded, in both orders. |
| **More wins than losses** | Meaning wins more searches than it loses, beyond what chance would give (sign test). |

> Pass mark proposed, to be agreed with product, together with grading by an AI model in
> place of people (with the spot-check). The report states the agreement rate, so a low one
> flags grades not to trust. Four meaning weights are tried in one run (1, 3, 5 and 10; 3 is
> today's default), and the chance made stricter to match, so the best of four can't pass on
> luck.

## Terms in the report

| Term | Definition given |
|---|---|
| `ranked` | Today's order: paid tier, listings with the search's words, then freshness, value and keywords. |
| `semantic w=3` | The same order with meaning added, weighted 3. The report shows each weight tried. |
| `nDCG@10` | One number for the quality of a top 10, from 0 to 1. Good matches near the top count most. |
| `P@5` | The share of the top 5 that people graded relevant. |
| `Wins / losses` | Searches where meaning's top 10 scored higher or lower than today's. |
| `Agreement` | In the spot-check, how often the person gave the model's grade, and how often both called a listing relevant or not. |

## How the report looks

`report.md` — *Layout only · values appear after the first run*

> Cases: — collected, — compared; — skipped (no words left over).

| EN · ARM | NDCG@10 | P@5 | GRADED TOP 10 | WINS / LOSSES VS RANKED |
|---|---|---|---|---|
| ranked | – | – | – | |
| semantic w=1 | – | – | – | – / – |
| semantic w=3 | – | – | – | – / – |
| semantic w=5 | – | – | – | – / – |
| semantic w=10 | – | – | – | – / – |

> The same table follows for FR, then for all searches together.

Verdict pills per weight (w=1, 3, 5, 10): `PASS · FAIL · TOO FEW CASES · UNCONFIRMED`

> "Unconfirmed" means a listing past the fetched pages could still reach a top 10 at that
> weight: collect again with more pages.
>
> **Grading** · Graded by `anthropic:<model>` `semantic-grade-v1`. A person graded — of them
> blind: the same grade on — (— %), the same call on relevant or not on — (— %).

## How to start it

> From the bastion, against the RN-5124 test site's server. It reads the develop database and
> the test site's own Rentals-API. During the run the tool switches meaning on for every city
> and ignores the rollout percentage.

**1 · Collect** — about 10 to 30 minutes.

```bash
kubectl --context rentals-dev-v3 -n rentals-cd exec deploy/rentalsca-rn-5124-rentalsca-app -c app -- \
  ./manage.py semantic_golden collect --out /tmp/runs.json
```

**2 · Export and copy out** — copy the files out straight away: the server's `/tmp` is lost on
a restart.

```bash
POD=$(kubectl --context rentals-dev-v3 -n rentals-cd get pods -o name | grep rn-5124-rentalsca-app | head
kubectl --context rentals-dev-v3 -n rentals-cd exec $POD -c app -- \
  ./manage.py semantic_golden export /tmp/runs.json --out /tmp/to-grade.csv
kubectl --context rentals-dev-v3 -n rentals-cd cp $POD:/tmp/runs.json ./runs.json -c app
kubectl --context rentals-dev-v3 -n rentals-cd cp $POD:/tmp/to-grade.csv ./to-grade.csv -c app
```

**3 · Grade with AI, then spot-check** — a few minutes. The model receives each listing's
text, so `--provider` must be one approved for listing text: `anthropic` or `openai`. It also
writes `grades.spot-check.csv`: a random 10% of the grade left blank.

```bash
kubectl --context rentals-dev-v3 -n rentals-cd exec $POD -c app -- \
  ./manage.py semantic_golden grade /tmp/runs.json --sheet /tmp/to-grade.csv --provider anthropic --out /t
kubectl --context rentals-dev-v3 -n rentals-cd cp $POD:/tmp/grades.csv ./grades.csv -c app
kubectl --context rentals-dev-v3 -n rentals-cd cp $POD:/tmp/grades.spot-check.csv ./spot-check.csv -c app
```

> Then one person opens `spot-check.csv`, gives each row 0, 1 or 2 without looking at the
> model's grades, and saves it. About a quarter of an hour. The rows hold listing text from
> the develop database: **keep the file inside the team.**

**4 · Score** — prints the report above with real values, a verdict per weight and the
agreement.

```bash
kubectl --context rentals-dev-v3 -n rentals-cd cp ./spot-check.csv $POD:/tmp/spot-check.csv -c app
kubectl --context rentals-dev-v3 -n rentals-cd exec $POD -c app -- \
  ./manage.py semantic_golden score /tmp/runs.json --grades /tmp/grades.csv --spot-check /tmp/spot-check.c
  --weights 1,3,5,10 --out /tmp/report.md
kubectl --context rentals-dev-v3 -n rentals-cd exec $POD -c app -- cat /tmp/report.md
```

## Status on the RN-5124 test site

| Status | Item | Detail |
|---|---|---|
| **DONE** | Meaning index built and active | Every searchable listing in the develop database, Workers AI bge-m3, built by the hourly job's first-build switch. |
| **DONE** | Ranking by meaning works on the site | Montréal searches with leftover words rank by meaning, typed into AI search or opened from a link. |
| **DONE** | The gate tool is in the branch | `manage.py semantic_golden` with collect, export and score, in RentalsCa #1636. |
| **BUILT** | 44 test searches | 22 English and 22 French made-up searches with wishes no filter can hold, plus the golden-set searches that leave words over. The gate needs at least 20 compared per language. On the RN-5124 branch, not yet deployed. |
| **BUILT** | AI grading with a spot-check | `semantic_golden grade` and `score --spot-check`. On the RN-5124 branch; tests to run before it is pushed and deployed. |
| **TO DO** | Approval for the grader | Grading sends listing text to the grading provider. Confirm Anthropic, or OpenAI, is covered by the listing-text approval. |
| **TO DO** | Agree the pass mark and AI grading | The four rules above, and grading by an AI model with a person's spot-check, are proposed. Product signs them off before the first run. |
| **NEXT** | Run, grade and score | Steps 1 to 4 above: about half an hour of running, and a quarter of an hour of one person's spot-check. |
| **NEXT** | CI on RentalsCa #1636 | Two test jobs failed on test-only problems; the fix is ready and waiting to be pushed. |

## Early signal, not the gate

> **32 → 78**
>
> Grade points in the top 10 among listings without the search's words, 21 searches, without
> and with meaning (15 better, 1 worse). Field test run 4, 2026-10-17, on a local copy with
> **282 listings embedded**, graded by Claude in a conversation, with **no blind setup or
> spot-check**. The gate replaces this with blind grades on every listing of the full index.

Source files named at the foot of the page:
`RentalsCa rentalsca_phoenix/management/commands/semantic_golden.py` ·
`rentalsbase/rentler/meaning/evaluation.py` · `docs/listing-meaning-index.md`

---

# 8. `semantic/search field test/` — the field test document, run 4

10 screenshots. Full transcript below; also kept standalone in [`../semantic/semantic_results.md`](../semantic/semantic_results.md).

> Note: this folder was called `semantic results` when first captured.

### Semantic Search Field Test

The same AI searches ranked two ways: the old way (keywords only) and the new way
(keywords plus meaning). Each search shows what the renter typed, what AI search made
of it, both result lists, and the numbers behind every position. This is the fourth run,
with the same searches as the first three. Meaning now weighs 0.5 instead of 3 among
listings that already contain the words.

```
RN-5124   Run 4, 2026-10-17   42 searches typed into AI search   25 compared
Montréal, local copy of develop
```

Page links: Since run 3 · How a position is decided · All searches · Better · Worse ·
No change · Weight check · Location and POIs · Not compared

#### Headline numbers

| Number | What it means |
|---|---|
| **32 → 78** | Grade points among listings *without* the words (meaning at 3). 21 searches: 15 better, 1 worse, 5 the same. |
| **55 → 56** | Grade points among listings that *contain* the words (meaning at 0.5). 4 searches: 1 better, 1 worse, 2 the same. |
| **24 → 32** | Clear matches in the first 10, all 25 searches; part matches 39 → 70. |
| **475 ms** | Median time to read a new search's meaning (301–893 ms); 1.0 ms when repeated. |

---

### What changed since run 3

- **Meaning weighs 0.5 among listings that contain the words.** Rentals-API has a second
  meaning weight for them (`ranking_meaning_keyword_band_weight`, default 0.5); listings
  without the words keep 3. Their text already says what was asked, and one word matches
  each of them to nearly the same degree, so at 3 a listing that only resembled the search
  passed one that named it. At 0.5 meaning only breaks ties there.
- **AI search's parse varies between runs.** It left different words in 13 of the 42
  searches. "Near restaurants" now leaves "near restaurants", not "restaurants";
  "près d'un parc et des espaces verts" and "rénové avec planchers de bois franc" leave
  words again; "big balcony or terrace" leaves none.
- **65 listings shown had no grade yet** for their search and were graded for this run,
  on the same scale.

#### The fix, on each run's own results

| Listings that contain the words, first 10 | Keywords only | Meaning at 3 (before) | Meaning at 0.5 (now) |
|---|---|---|---|
| Run 2's results (4 searches) | 42 | 31 | 41 |
| Run 3's results (4 searches) | 42 | 39 | 42 |
| Run 4's results (4 searches, served at 0.5) | 55 | 52 | 56 |

Grade points: 2 per clear match, 1 per part match. Each row re-sorts one run's fetched
results at each weight with the gate tool's own sort, so the searches, words and listings
are identical across a row. At 3, meaning cost these listings points in every row; at 0.5
it costs at most 1 and gains 1 in run 4.

#### The effect of meaning, runs 1 to 4

| Group meaning reorders | Run 1 | Run 2 | Run 3 | Run 4 |
|---|---|---|---|---|
| Listings without the words (weight 3) | 27 → 63 (20) | 33 → 76 (23) | 32 → 69 (20) | 32 → 78 (21) |
| Listings that contain the words (weight 3, then 0.5 in run 4) | 86 → 74 (7) | 42 → 31 (4) | 42 → 39 (4) | 55 → 56 (4) |

Keywords only → with meaning, grade points in each search's first 10 of that group
(searches in brackets). Run 1 scored listings without a vector 0; later runs give them the
average. Which searches fall in each group changes with AI search's parse, so read across
a row with care; the table above compares like with like.

---

### Each search, meaning order, run 3 and run 4

**[cut off in screenshot]** — the final column "WHY THEY DIFFER" was clipped on the right
edge of screenshots 3, 4 and 5. The readable part is kept below.

| What the renter typed | Words left, run 3 | Words left, run 4 | Points run 3 | Points run 4 | Why they differ |
|---|---|---|---|---|---|
| appartement près d'un dépanneur à Montréal | dépanneur | dépanneur | 3 | 3 | 10 of 10 listings in common |
| near restaurants in montreal | restaurants | near restaurants | 7 | **14** | AI search left other words; 4 of 10 listings in common |
| bright apartment with big windows and lots of natural light in Montreal | big windows / lots of natural light | bright apartment with big windows and lots of natural light | 6 | **16** | AI search left other words; another group compared; 0 of 10 listings in common |
| heritage building with character and high ceilings in Montreal | heritage building with character and high ceilings | heritage building with character and high ceilings | 7 | 7 | 9 of 10 listings in common |
| appartement lumineux avec grandes fenêtres à Montréal | appartement lumineux avec grandes fenêtres | lumineux avec grandes fenêtres | 13 | **3** | AI search left other words; another group compared; 0 of 10 listings in common |
| cuisine rénovée avec électroménagers neufs à Montréal | cuisine rénovée électroménagers neufs | cuisine rénovée avec électroménagers neufs | 7 | 8 | AI search left other words; 9 of 10 listings in common |
| immeuble patrimonial avec plafonds hauts à Montréal | immeuble patrimonial avec plafonds hauts | immeuble patrimonial avec plafonds hauts | 1 | 1 | 10 of 10 listings in common |
| proche des cafés et restaurants appartement Montréal | cafés et restaurants | proche des cafés et restaurants | 0 | 1 | AI search left other words; 5 of 10 listings in common |
| quiet apartment for working from home in Montreal | quiet (for working from home) | quiet working from home | 4 | 5 | AI search left other words; 7 of 10 listings in common |
| sunny apartment with a nice view in Montreal under 2000 | sunny nice view | sunny nice view | 7 | 7 | 10 of 10 listings in common |
| recently renovated apartment with hardwood floors in Montreal | recently renovated | recently renovated | 3 | 3 | 10 of 10 listings in common |
| family friendly neighbourhood near good schools in Montreal | good schools | near good schools | 0 | 0 | AI search left other words; 9 of 10 listings in common |
| modern condo with floor to ceiling windows in Montreal | modern floor to ceiling windows | floor to ceiling windows | 12 | **4** | AI search left other words; another group compared; 0 of 10 listings in common |
| apartment with a backyard for my dog in Montreal | backyard for my dog | backyard for my dog | 1 | 1 | 10 of 10 listings in common |
| apartment with lots of storage space in Montreal | lots of storage space | lots of storage space | 1 | 1 | 10 of 10 listings in common |
| appartement calme pour le télétravail à Montréal | calme pour le télétravail | calme pour le télétravail | 7 | 7 | 10 of 10 listings in common |
| logement avec grande terrasse à Montréal | grande terrasse | grande terrasse | 7 | 7 | 10 of 10 listings in common |
| appartement près d'un parc et des espaces verts à Montréal | – | espaces verts | – | 9 | Not compared last run |
| appartement rénové avec planchers de bois franc à Montréal | – | appartement rénové | – | 19 | Not compared last run |
| vue sur le fleuve à Montréal | vue sur le fleuve | vue sur le fleuve | 2 | 2 | 10 of 10 listings in common |
| appartement ensoleillé avec cour arrière à Montréal | ensoleillé cour arrière | appartement ensoleillé cour arrière | 4 | **1** | AI search left other words; 5 of 10 listings in common |
| quartier tranquille pour une famille à Montréal | quartier tranquille | quartier tranquille | 6 | 6 | 10 of 10 listings in common |
| 2 bedroom near a park on a quiet street in Montreal | quiet street | quiet street | 2 | 2 | 10 of 10 listings in common |
| apartment near restaurants and cafés in Montreal | near restaurants and cafés | near restaurants and cafés | 1 | 1 | 10 of 10 listings in common |
| walking distance to the subway, renovated kitchen, Montreal | renovated kitchen | renovated kitchen | 6 | 6 | 10 of 10 listings in common |

The two runs' first 10s are not always the same group: when AI search leaves other words,
or a listing changes group, the lists compare different things.

---

### How a position is decided

| Step | Name | Rule |
|---|---|---|
| 1 | **Tier** — placement level | Higher tiers always come first. 0 is the standard tier. |
| 2 | **Keyword group** — contains the leftover words or not | Within a tier, listings that contain the words come first. |
| 3 | **Score** — sum of five factors | Meaning is the new factor. It only reorders listings inside one tier and keyword group, and weighs less in the group that contains the words. |

Every factor is a value from 0 to 1, multiplied by its weight. The score is the sum:

```
score = F 1×freshness + V 1×value + R 0.5×rotation + K 3×keywords + M 3×meaning
listings that contain the words: … + K 3×keywords + M 0.5×meaning
```

- **Freshness**: how recently the listing was created.
- **Value**: rent compared with similar floor plans nearby.
- **Rotation**: a daily shuffle so the same listings do not always lead.
- **Keywords**: how many of the leftover words the tag line and description contain.
- **Meaning**: how close the listing's text is in meaning to the leftover words, scaled
  across the search's results so the closest is 1 and the farthest 0. A listing with no
  vector gets the average of the others. Its weight is 3 for listings without the words
  and 0.5 for listings that contain them, where it only breaks ties.

Each listing below shows its own sum, for example `F 1×.85 + V 1×.68 + R 0.5×.17 + K 3×0 + M 3×.99 = 4.59`.
Hover a term for its exact value. "#612 → #455" is the listing's position in all results
before and now.

Page 1 changed in 15 of 25 searches. The local copy has a vector for 282 listings, and the
higher tiers that fill the first pages mostly have none. So each search card compares the
group where meaning acts most: the tier and keyword group holding the most listings with a
vector.

**Grading key**

| Mark | Meaning |
|---|---|
| ✓ | **Clear match**: the text says what was asked |
| ◐ | **Part match**: half of it, something close, or only a photo tag |
| ✗ | **No match** |

Grades were given by Claude from each listing's text, for the leftover words in the
context of the full search. Grade points count 2 per clear and 1 per part match in the
first 10. This is not a substitute for the human-graded release gate.

---

### All compared searches

| What the renter typed | Leftover words | Results | With a vector | Page 1 changed | Clear | Part | Grade points |
|---|---|---|---|---|---|---|---|
| appartement près d'un parc et des espaces verts à Montréal | espaces verts | 881 | 129 | yes | 0 → 3 | 1 → 3 | +8 |
| cuisine rénovée avec électroménagers neufs à Montréal | cuisine rénovée avec électroménagers neufs | 960 | 197 | no | 0 → 0 | 2 → 8 | +6 |
| logement avec grande terrasse à Montréal | grande terrasse | 960 | 197 | yes | 0 → 1 | 1 → 5 | +6 |
| walking distance to the subway, renovated kitchen, Montreal | renovated kitchen | 872 | 187 | no | 0 → 1 | 0 → 4 | +6 |
| appartement calme pour le télétravail à Montréal | calme pour le télétravail | 916 | 136 | yes | 0 → 2 | 2 → 3 | +5 |
| sunny apartment with a nice view in Montreal under 2000 | sunny nice view | 780 | 111 | no | 0 → 0 | 3 → 7 | +4 |
| near restaurants in montreal | near restaurants | 960 | 196 | yes | 6 → 7 | 0 → 0 | +2 |
| appartement lumineux avec grandes fenêtres à Montréal | lumineux avec grandes fenêtres | 916 | 136 | yes | 0 → 0 | 1 → 3 | +2 |
| quiet apartment for working from home in Montreal | quiet working from home | 916 | 136 | no | 1 → 1 | 1 → 3 | +2 |
| modern condo with floor to ceiling windows in Montreal | floor to ceiling windows | 116 | 28 | yes | 0 → 0 | 2 → 4 | +2 |
| appartement près d'un dépanneur à Montréal | dépanneur | 916 | 136 | yes | 0 → 0 | 2 → 3 | +1 |
| immeuble patrimonial avec plafonds hauts à Montréal | immeuble patrimonial avec plafonds hauts | 960 | 183 | yes | 0 → 0 | 0 → 1 | +1 |
| proche des cafés et restaurants appartement Montréal | proche des cafés et restaurants | 916 | 136 | yes | 0 → 0 | 0 → 1 | +1 |
| recently renovated apartment with hardwood floors in Montreal | recently renovated | 220 | 27 | no | 1 → 1 | 0 → 1 | +1 |
| apartment with lots of storage space in Montreal | lots of storage space | 916 | 136 | no | 0 → 0 | 0 → 1 | +1 |
| apartment near restaurants and cafés in Montreal | near restaurants and cafés | 916 | 136 | yes | 0 → 0 | 0 → 1 | +1 |
| heritage building with character and high ceilings in Montreal | heritage building with character and high ceilings | 960 | 167 | yes | 0 → 0 | 8 → 7 | **−1** |
| quartier tranquille pour une famille à Montréal | quartier tranquille | 168 | 14 | no | 1 → 1 | 5 → 4 | **−1** |
| bright apartment with big windows and lots of natural light in Montreal | bright apartment with big windows and lots of natural light | 916 | 136 | yes | 6 → 6 | 4 → 4 | +0 |
| family friendly neighbourhood near good schools in Montreal | near good schools | 168 | 14 | no | 0 → 0 | 0 → 0 | +0 |
| apartment with a backyard for my dog in Montreal | backyard for my dog | 916 | 136 | yes | 0 → 0 | 1 → 1 | +0 |
| appartement rénové avec planchers de bois franc à Montréal | appartement rénové | 220 | 27 | no | 9 → 9 | 1 → 1 | +0 |
| vue sur le fleuve à Montréal | vue sur le fleuve | 960 | 205 | yes | 0 → 0 | 2 → 2 | +0 |
| appartement ensoleillé avec cour arrière à Montréal | appartement ensoleillé cour arrière | 916 | 136 | yes | 0 → 0 | 1 → 1 | +0 |
| 2 bedroom near a park on a quiet street in Montreal | quiet street | 593 | 81 | no | 0 → 0 | 2 → 2 | +0 |

---

### Four findings

**Meaning helps where keywords find nothing**
Among listings without the words, grade points rose from 32 to 78: 15 searches better,
1 worse. It finds wording keywords miss, such as "immeuble tranquille" for "calme pour le
télétravail", "huge terrazzo" for a large terrace and "Cuisine et salle de bain
entièrement rénovée" for a renovated kitchen. This group keeps weight 3.

**Where keywords already match, meaning now only breaks ties**
Among listings that contain the words, points went from 55 to 56 and clear matches from
21 to 22, at weight 0.5. "Near restaurants", which lost the most at weight 3 (12 → 7 in
run 3), now gains: 12 → 14, with listings that name nearby restaurants kept at the top.
At 3 the same results would score 52.

**Very short listings still float up**
#1561847, whose whole text is "Additional amenities: Central Heat", has the highest
meaning score in 3 of the 25 searches.

**Gains are mostly part matches**
Clear matches in the first 10 went from 24 to 32 across all searches; part matches rose
from 39 to 70. Light, view and renovation searches gain from listings that say "lumineux"
or "rénové" without the exact words.

---

### Better with meaning — worked example

**FR** — "appartement près d'un parc et des espaces verts à Montréal"

AI search made it: `types: all-apartments` · `nearby pois: park` ·
`leftover words: "espaces verts"`

881 results · 129 with a vector · page 1 changed · meaning read in 617 ms

**0 → 3** clear matches in the first 10 of the group meaning reorders
(part matches 1 → 3 · grade points 1 → 9)

First 10 results overall: the same in both orders. They sit in higher tiers, which meaning
does not reorder across.

**Where meaning acts:** Tier 0 (standard), listings without the leftover words:
370 listings, 68 with a vector, starting at result #512.

#### Before: keywords only

| | Listing | Position | Score | Note |
|---|---|---|---|---|
| ✗ | Listing 1545237 · 1 bd · $950 | #512 | `F 1×.89 + V 1×.97 + R 0.5×.99 + K 3×0 = 2.35` | No park or green space mentioned |
| ✗ | 24-001, 146 rue Pilon, Saint-Eustache, QC · 3 bd · $1,400 | #513 | `F 1×.93 + V 1×.97 + R 0.5×.85 + K 3×0 = 2.32` | empty text |
| ✗ | 1256 Saint-André · 3 bd · $795–3,089 | #514 | `F 1×.83 + V 1×.99 + R 0.5×.80 + K 3×0 = 2.22` | No usable text |
| ✗ | Listing 1544085 · 2 bd · $1,250 | #515 | `F 1×.89 + V 1×.97 + R 0.5×.65 + K 3×0 = 2.19` | No park or green space mentioned |
| ✗ | Listing 1555360 · 2 bd · $1,499 | #516 | `F 1×.95 + V 1×.80 + R 0.5×.86 + K 3×0 = 2.18` | no park or green space mentioned |
| ✗ | Listing 1556553 · 1 bd · $950 | #517 | `F 1×.96 + V 1×.97 + R 0.5×.50 + K 3×0 = 2.18` | No park or green space mentioned |

#### Now: keywords + meaning

| | Listing | Position | Score | What meaning read |
|---|---|---|---|---|
| ✓ | Listing 1562241 · Côte-des-Neiges · 2 bd · $1,700 | #553 → #512 | `F 1×.99 + V 1×.51 + R 0.5×.78 + K 3×0 + M 3×.97 = 4.82` | "face à un beau et grand Parc" |
| ✓ | Listing 1545689 · 1 bd · $1,195 | #543 → #513 | `F 1×.90 + V 1×.85 + R 0.5×.41 + K 3×0 + M 3×.93 = 4.74` | "Jardin botanique, parc Maisonneuve" nearby |
| ✗ | Listing 1561847 · 1 bd · $1,800 | #669 → #514 | `F 1×.99 + V 1×.12 + R 0.5×.78 + K 3×0 + M 3×1.00 = 4.47` | almost no text |
| ✓ | Listing 1527201 · Hochelaga-Maisonneuve · 1 bd · $1,193 | #554 → #515 | `F 1×.82 + V 1×.86 + R 0.5×.44 + K 3×0 + M 3×.85 = 4.45` | "pistes cyclables du parc Maisonneuve" |
| ✗ | Listing 1545704 · Mile End · 3 bd · $1,850 | #544 → #516 | `F 1×.90 + V 1×.63 + R 0.5×.83 + K 3×0 + M 3×.77 = 4.25` | no park or green space mentioned |


---

# 9. `semantic/search quality report/` — Semantic Search Quality Report

Published as a Claude artifact. Address seen in the browser:
`claude.ai/artifact/6owb9QDRwK2Fp2UcnrQoRD`

> **AI SEARCH · RANKING AND MEANING (RN-5120) · OCTOBER 21, 2026**
>
> # Semantic Search Quality Report
>
> Semantic search, keyword ranking and the meaning index together, rank AI search results
> 68% better than before the epic: 93 of 103 test searches now get a better first page, 4 a
> worse one.

## Headline cards

| Number | Label |
|---|---|
| **+68%** | ranking quality over AI search before the epic |
| **93 / 103** | searches with a better first page (4 worse) |
| **+72% · +58%** | English · French |
| **Passed** | meaning index quality gate, in both languages |

## Progress at a glance

**Semantic search lifts ranking quality 68% over AI search before the epic**

| Slice | Before | After | Change |
|---|---|---|---|
| English (71 searches) | 0.427 | 0.735 | +72% |
| French (32 searches) | 0.407 | 0.645 | +58% |
| All searches (103 searches) | 0.421 | 0.707 | +68% |

**A better first page in 93 of 103 searches, a worse one in 4**

| Language | Better | Unchanged | Worse |
|---|---|---|---|
| English (71 searches) | 65 (92%) | 4 | 2 |
| French (32 searches) | 28 (88%) | 2 | 2 |

## How semantic search works

> Filters still pick the listings. Semantic search then ranks them by the words no filter
> could hold, in two ways: listings that use the same words (**keyword ranking**) and listings
> that mean the same thing in other words or another language (**the meaning index**).

**Live path**

```
Renter's search  "quiet apartment with a big terrace in Montréal"
        ↓
AI search reads it  — splits the words into filters and leftover words
        ↓
Filters                          Leftover words
city, home type, beds,           "quiet", "big terrace"
baths, price, nearby place       wishes no filter can hold
        ↓                                ↓
Matching listings                Meaning score
every listing that fits          how close a listing's text
all the filters                  is to the leftover words
```

**Ahead of time**

```
Listing text  (tag line, photo features, photo captions, description)
        ↓
bge-m3 model  on Cloudflare Workers AI; turns text into meaning
        ↓
Meaning index  one entry per listing, refreshed every hour
```

**Ranking order**

1. Paid tier first
2. Then listings whose text has the exact leftover words (keyword ranking, RN-5121)
3. Then a weighted score: freshness 1, value 1, rotation 0.5, keyword 3, **meaning 3**
   (RN-5124)

## How it was measured

> Each test search was run three ways on the same listings: AI search before the epic
> (filters only), keyword ranking only, and full semantic search. Every listing on page one
> was graded blind (2 = clearly offers what was asked, 1 = partly, 0 = not at all), and each
> order was scored from 0 to 1 on how well it puts the best listings first (1.0 = a perfect
> first page). Of **355 test searches, 103 had words left over** to rank by (71 English, 32
> French).

## The meaning index quality gate: passed

> Before it ships, the meaning index has to earn its place: on top of keyword ranking alone,
> it must lift ranking quality by at least 5% in English and in French, with clearly more
> searches better than worse. It now adds **+6.0% in English and +8.4% in French**, and
> improves 58 searches for 17 worse.

**Both languages now clear the 5% gate**

| Language | Before (meaning weighed 0.5 among keyword matches) | Now (meaning weighed fully, 3.0) |
|---|---|---|
| English | +2.4% | **+6.0%** |
| French | +4.8% | **+8.4%** |

*Gain over keyword ranking alone, meaning weight 3 · 71 English, 32 French searches*

### What changed

- **What made it pass:** among listings that already contain the renter's words, meaning used
  to count at one sixth of its weight. At full weight it now picks the best fit among them,
  which lifted it past the 5% bar in both languages.
- **Tested and dropped:** OpenAI's embedding model (no better than the bge-m3 model in use)
  and embedding a full sentence instead of the bare words (worse).
- **Weak spot:** street-address searches, where meaning can push the exact building down the
  page.

## Try it yourself

> Run these on the RN-5124 test site (company sign-in). Type a search into the AI search box
> and press Enter for semantic search; add `&semantic_ranking=0` to the address for keyword
> ranking only. Scores on the test site will differ a little from these.

| Search to type | Before the epic | Keyword ranking only | Semantic search |
|---|---|---|---|
| apartments near old port | 0.13 | 1.00 | 0.99 |
| apartment close to the beach and the lake in Toronto | 0.03 | 0.98 | 0.98 |
| apartment in a quiet neighbourhood good for families in Calgary | 0.27 | 0.83 | **1.00** |
| appartement près d'une pharmacie et d'un parc pour chiens à Edmonton | 0.23 | 0.56 | **0.84** |
| apartment with a rooftop terrace and a BBQ area in Toronto | 0.27 | 0.44 | **0.73** |
| appartement avec beaucoup de rangement à Montréal | 0.37 | 0.64 | **0.84** |

> Scores are ranking quality from 0 to 1 (1.0 = a perfect first page).
>
> Evaluation run on October 21, 2026 against a local copy of the develop database (26,117
> listings in the meaning index). Grades by one AI model with a written rubric; a person's
> spot-check is pending.
