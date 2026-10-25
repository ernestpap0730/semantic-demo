# Semantic Search — slide deck, full content

> Transcribed from 13 screenshots in `D:\screenshots\pptx` (1.png – 13.png).
> One section per slide, in order. Nothing was added or interpreted.

---

## Slide 1 — Title

# Semantic search

**Ranking rental results by what renters mean, not only by the words they type**

```
RN-5121 · RN-5124 · RN-5389 · rentals.ca · October 2026
```

---

## Slide 2 — Some wishes can't become filters

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

## Slide 3 — How semantic search works

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

## Slide 4 — How a position is decided

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

## Slide 5 — Green spaces, asked in French

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

## Slide 6 — A quiet place to work from home

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

## Slide 7 — A renovated kitchen near the metro

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

## Slide 8 — The quality test in numbers

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

## Slide 9 — "No carpets" now ranks the right way

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

## Slide 10 — bge-m3 chosen; OpenAI tested and dropped

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

## Slide 11 — A renter's words stay private

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

## Slide 12 — Three switches, all off in production

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

## Slide 13 — What's done, and what's left

| Status | Item | Detail |
|---|---|---|
| **Done** | Full meaning index on develop | 26,117 listings, approved Oct 20, refreshed hourly |
| **Done** | Quality test passed | +6.0% English, +8.4% French over keywords alone |
| **Before production** | A person checks the grades | a spot-check of the AI model's grades, then the production index |
| **Next ticket** | Commute-aware ranking | closer to the metro or park asked for ranks higher |

```
try it: RN-5124 test site · RN-5122 uses RN-5114's stored distances
```
