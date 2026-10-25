# Semantic Search Field Test — full content

> Transcribed from 10 screenshots in `D:\screenshots\semantic results` (1.png – 10.png).
> This is the full text of the document, in order. Nothing was added or interpreted.
> Where a screenshot cut text off, it is marked **[cut off in screenshot]**.

---

## Semantic Search Field Test

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

### Headline numbers

| Number | What it means |
|---|---|
| **32 → 78** | Grade points among listings *without* the words (meaning at 3). 21 searches: 15 better, 1 worse, 5 the same. |
| **55 → 56** | Grade points among listings that *contain* the words (meaning at 0.5). 4 searches: 1 better, 1 worse, 2 the same. |
| **24 → 32** | Clear matches in the first 10, all 25 searches; part matches 39 → 70. |
| **475 ms** | Median time to read a new search's meaning (301–893 ms); 1.0 ms when repeated. |

---

## What changed since run 3

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

### The fix, on each run's own results

| Listings that contain the words, first 10 | Keywords only | Meaning at 3 (before) | Meaning at 0.5 (now) |
|---|---|---|---|
| Run 2's results (4 searches) | 42 | 31 | 41 |
| Run 3's results (4 searches) | 42 | 39 | 42 |
| Run 4's results (4 searches, served at 0.5) | 55 | 52 | 56 |

Grade points: 2 per clear match, 1 per part match. Each row re-sorts one run's fetched
results at each weight with the gate tool's own sort, so the searches, words and listings
are identical across a row. At 3, meaning cost these listings points in every row; at 0.5
it costs at most 1 and gains 1 in run 4.

### The effect of meaning, runs 1 to 4

| Group meaning reorders | Run 1 | Run 2 | Run 3 | Run 4 |
|---|---|---|---|---|
| Listings without the words (weight 3) | 27 → 63 (20) | 33 → 76 (23) | 32 → 69 (20) | 32 → 78 (21) |
| Listings that contain the words (weight 3, then 0.5 in run 4) | 86 → 74 (7) | 42 → 31 (4) | 42 → 39 (4) | 55 → 56 (4) |

Keywords only → with meaning, grade points in each search's first 10 of that group
(searches in brackets). Run 1 scored listings without a vector 0; later runs give them the
average. Which searches fall in each group changes with AI search's parse, so read across
a row with care; the table above compares like with like.

---

## Each search, meaning order, run 3 and run 4

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

## How a position is decided

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

## All compared searches

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

## Four findings

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

## Better with meaning — worked example

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

### Before: keywords only

| | Listing | Position | Score | Note |
|---|---|---|---|---|
| ✗ | Listing 1545237 · 1 bd · $950 | #512 | `F 1×.89 + V 1×.97 + R 0.5×.99 + K 3×0 = 2.35` | No park or green space mentioned |
| ✗ | 24-001, 146 rue Pilon, Saint-Eustache, QC · 3 bd · $1,400 | #513 | `F 1×.93 + V 1×.97 + R 0.5×.85 + K 3×0 = 2.32` | empty text |
| ✗ | 1256 Saint-André · 3 bd · $795–3,089 | #514 | `F 1×.83 + V 1×.99 + R 0.5×.80 + K 3×0 = 2.22` | No usable text |
| ✗ | Listing 1544085 · 2 bd · $1,250 | #515 | `F 1×.89 + V 1×.97 + R 0.5×.65 + K 3×0 = 2.19` | No park or green space mentioned |
| ✗ | Listing 1555360 · 2 bd · $1,499 | #516 | `F 1×.95 + V 1×.80 + R 0.5×.86 + K 3×0 = 2.18` | no park or green space mentioned |
| ✗ | Listing 1556553 · 1 bd · $950 | #517 | `F 1×.96 + V 1×.97 + R 0.5×.50 + K 3×0 = 2.18` | No park or green space mentioned |

### Now: keywords + meaning

| | Listing | Position | Score | What meaning read |
|---|---|---|---|---|
| ✓ | Listing 1562241 · Côte-des-Neiges · 2 bd · $1,700 | #553 → #512 | `F 1×.99 + V 1×.51 + R 0.5×.78 + K 3×0 + M 3×.97 = 4.82` | "face à un beau et grand Parc" |
| ✓ | Listing 1545689 · 1 bd · $1,195 | #543 → #513 | `F 1×.90 + V 1×.85 + R 0.5×.41 + K 3×0 + M 3×.93 = 4.74` | "Jardin botanique, parc Maisonneuve" nearby |
| ✗ | Listing 1561847 · 1 bd · $1,800 | #669 → #514 | `F 1×.99 + V 1×.12 + R 0.5×.78 + K 3×0 + M 3×1.00 = 4.47` | almost no text |
| ✓ | Listing 1527201 · Hochelaga-Maisonneuve · 1 bd · $1,193 | #554 → #515 | `F 1×.82 + V 1×.86 + R 0.5×.44 + K 3×0 + M 3×.85 = 4.45` | "pistes cyclables du parc Maisonneuve" |
| ✗ | Listing 1545704 · Mile End · 3 bd · $1,850 | #544 → #516 | `F 1×.90 + V 1×.63 + R 0.5×.83 + K 3×0 + M 3×.77 = 4.25` | no park or green space mentioned |
