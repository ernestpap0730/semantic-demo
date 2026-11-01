# `semantic workflow` — summary

**Source:** `D:\screenshots\code\workflow\semantic workflow` · 17 images
**The document:** "Negated Wishes Workflow" · ticket **RN-5389** · as built on **2026-10-25**

---

## What this document is

It follows **one search** from the search box to the finished page:

> "quiet 2 bed in Toronto, **no carpets**"

It shows all 10 steps, what each one runs on, why it was built that way, what else was tried,
and where it is slow.

All numbers come from a local stack on develop data, across **52 negated searches**. Anything
not measured is honestly labelled **"idea"**.

⚠️ **Note the name.** The folder is called `semantic workflow`, but the document is about
**negated wishes** — the "no carpets" problem. Semantic search is only step 5 of 10. This is
a *neighbour* of your work, not your work itself.

---

## The problem in one paragraph

A renter types "no carpets". Two different systems both get it wrong:

1. **Keyword search ignores the word "no"** and matches "carpets".
2. **The meaning model reads "no carpets" as very close to "carpets"** — because it is, as a
   topic.
3. **The language model can flip it into the opposite filter** — "no pets" becomes
   *pet-friendly*.

So the renter asks to avoid something and the site shows it to them first.

---

## The 10 steps

| # | Step | Runs on | What happens |
|---|---|---|---|
| 1 | Search box | Browser | Renter types plain words, English or French |
| 2 | Language model parse | LLM via AI Gateway | → `city=toronto`, `beds=2`, leftover: "quiet", "no carpets" |
| 3 | **Negation guard** | RentalsCa | Fixed rules undo a filter the model turned the wrong way round |
| 4 | Results URL | Browser | `/toronto?beds=2&keywords=quiet, no carpets` |
| 5 | **Meaning vector** | Workers AI bge-m3 | The keywords become 1,024 numbers. ← *this is semantic search* |
| 6 | Find what to avoid | RentalsCa | Same rules read out the phrase to avoid: "carpets" |
| 7 | Fetch first two pages | Rentals-API + Postgres | One extra search for 48 listing ids, then their text |
| 8 | **Jev reads each listing** | Workers AI Jev | "Does this listing say it has carpets?" 24 per call, 2 calls at once |
| 9 | Ranked search with demote | Rentals-API + Postgres | Flagged listings move to the bottom of their tier |
| 10 | Render and cache | Django, Redis, Cloudflare | Page cached up to an hour |

**If step 5 produces no vector** (slow, failed, or the rollout is off), steps 6–8 are skipped
entirely and the page ranks by keywords alone. The avoid-check only runs on pages that
meaning already ranked.

---

## Where the time goes

| Part | Measured | Note |
|---|---|---|
| Whole avoid check, first visit | **+0.85 s** median, +1.3 s slow end | 15 searches, nothing cached |
| Jev alone (2 calls at once) | 0.64 s median, 1.07 s slow end | 47 listings on average |
| Extra first-pages search + text | about 0.2 s | the rest of the 0.85 s |
| **Repeat visit** | **+0 s** | answer kept for the hour |
| Hard limit | 2.5 s | past that, the page ranks without the check |

So the first renter to type a new negated search waits almost a second longer. Everyone after
them waits nothing.

---

## What they tried and threw away

This is the most valuable part of the document. **Five approaches were measured and rejected.**

| Approach | Result | Verdict |
|---|---|---|
| Let the model rewrite "no carpets" → "hardwood floors" | **+25% measured** | ❌ **removed** — changes what the renter asked for, unpredictably |
| Penalise the negated words' meaning | Ranked worse at every weight | ❌ tried |
| Contrast score | Ranked worse | ❌ tried |
| A reranker model | Ranked worse | ❌ tried |
| Drop the negated words from keyword ranking | **Worse: 0.36 vs 0.45** | ❌ tried |
| 40 listings per Jev call | Mixed one listing up with another | ❌ tried |
| Hide flagged listings instead of demoting | — | ❌ not chosen: a wrong flag would delete a good listing |

**Read row one again.** The rewrite scored **+25%** — the best result they got — and they
**removed it anyway**, because it silently changed the renter's request. That is a team
choosing predictability over a score.

**And row five is counter-intuitive.** You would think dropping "no carpets" from keyword
matching would help. It ranked *worse*. The reason given: the negated words still **carry the
topic**. "Carpets" tells the system this is a flooring search, even when the renter wants the
opposite.

---

## What they built instead

Two fixed, boring, predictable pieces.

### The negation guard (step 3)

Plain Python rules. Word lists for English and French: "no", "not", "without", "sans",
"pas de", "anything but", "…-free", "… not allowed". Accents folded.

**Why rules and not AI:** "Predictable: same input, same output, testable." · "No latency, no
cost." · "**Never rewrites the renter's words.**"

It runs *after* any provider, so every model gets the same check.

*Weak points they admit:* only phrasings it knows — "allergic to cats" and "I hate carpet"
are missed. False triggers had to be fixed one at a time, like "near No Frills" (a shop) and
"never lived in".

### Jev (step 8)

Jev is a **yes/no classifier**. It gets a text and a fixed question, and answers with a score
from 0 to 1. **It writes no text of its own.**

The question: *"Does Listing K7Q2 say its home has «carpets»?"* A score of 0.5 or more marks
the listing.

| Fact | Value |
|---|---|
| Accuracy | **130 of 134 flags right** |
| Batching check | 24-per-call agreed with one-per-call on **240 of 240** |
| Speed | about 0.5 s a call |
| Cost | $0.042 per million input tokens |

**Why it was chosen:** "Writes nothing, so it cannot change the renter's search."

That sentence is the design philosophy of this whole ticket. AI is allowed to *answer*
questions. It is never allowed to *write* anything that reaches the ranking.

---

## A clever security detail

Each listing in a Jev call is given a **random four-letter label** (like `K7Q2`), and
label-like text is stripped from the listings themselves.

**Why:** so one listing's text cannot pretend to speak for another. A landlord could
otherwise write "Listing 3 has no carpets" in their own description and hijack the answer.

The measured effect of this guard: **0.72 → 0.04**. That is a real prompt-injection defence,
measured.

---

## What protects the page

| Guard | Setting |
|---|---|
| Jev time limit | 2.0 s for all calls together |
| Whole check limit | 2.5 s, then the page ranks without it |
| Outage pause | 3 failed checks in a row → Jev not asked for 60 s |
| Load cap | 3 checks and 6 Jev calls at once per process |
| Old Rentals-API | Refused `demote` → searched again without it |
| Abuse | Random labels · personal details scrubbed before words leave |
| Off switch | `NEGATION_DEMOTE_ENABLED=false` |

The busy cap was measured: 20 searches at once went from **8.3 s down to 3.3 s**.

---

## The honest weak points

The document does not hide these.

| Problem | Number |
|---|---|
| Listing text rarely says what a home **lacks** | **26 of 52** searches had nothing flagged at all |
| Hints are missed | "22-storey building" not read as "in a tower" — 7 of 38 missed |
| "no" is a stopword in Postgres | Keyword ranking **still rewards "carpets"** |
| All-or-nothing | A 0.51 and a 0.99 are demoted equally |
| "no deposit" | Pushes down 17 listings that mention a deposit — **undecided** whether that is right |
| Commas reorder results | 10 of 12 searches kept 8–10 of the same top 10 — **not graded** |

That first row is the big one. **Half the searches get no benefit**, because landlords write
about what their home *has*, not what it lacks.

---

## The fix list, in their order

Seven improvements. Items 2, 3, 5 and 6 are marked **untested ideas**.

| # | Improvement | Why |
|---|---|---|
| **1** | **Get leftover words from the AI search service** | "**This decides whether the feature works at all.**" If Rocky's Rust service answers the search, no keywords reach the URL and none of this runs |
| 2 | Store Jev's answers ahead of time, during the hourly sync | Removes the wait, and covers every listing instead of only the first 48 |
| 3 | Drop the extra first-pages search | Under load it was the bottleneck |
| 4 | Add photo tags in production | Measured **+14% on 7 searches, none worse**. Develop has tags on ~10% of listings; production has ~99% |
| 5 | Give the language model an "avoid" field | Catches phrasings the rules miss |
| 6 | Shorten the edge cache for unchecked pages | A page served while Jev was down should not last an hour |
| 7 | Grade the comma change and "no deposit" | Both change results today **without a quality grade** |

**Item 1 is the same blocker named in the project workflow document.** Two separate reviews,
written by different people about different things, both land on it. Take it seriously.

**Item 4 is the easy win.** +14% measured, nothing got worse, and the data already exists in
production. It is only weak on develop because that environment has fewer photo tags.

---

## One thing hidden in the first screenshot

Image 1 shows the terminal where this document was generated. The developer caught their own
mistake before publishing:

> "I need to fix an error — step 5 proposed embedding only the wanted words, but our data
> already showed dropping negated words scored worse (0.36 vs 0.45). I'll swap in a different
> approach before publishing."

Worth noticing. The first draft recommended something their own measurements had already
disproved. They checked their draft against their data and fixed it.

**Lesson for you:** when a plan sounds sensible, check it against the numbers you already
have. "Obviously we should drop the negated words" is exactly the kind of idea that feels
right and measures wrong.

---

## How this connects to your job

| | |
|---|---|
| **Is this your ticket?** | No. RN-5389 is a neighbour of RN-5124. |
| **Does it depend on you?** | Yes. The avoid-check only runs on pages **meaning already ranked** (step 5). No vector → steps 6–8 are skipped. |
| **Shared blocker** | Leftover words from the Rust service. Both features die without them. |
| **Shared model host** | Both bge-m3 and Jev run on Cloudflare Workers AI. |
| **Open approval question** | The Oct 20 data approval covered bge-m3. Whether it also covers **Jev** is still unanswered. |

---

## New terms

| Term | Plain meaning |
|---|---|
| **Negated wish** | A wish phrased backwards: "no carpets", "sans tapis". |
| **Negation guard** | Fixed Python rules that undo a filter the model turned the wrong way round. |
| **Jev** | A yes/no classifier. Gets a text and a question, returns a score 0–1. Writes no text. |
| **Classifier** | A model that picks from fixed answers instead of writing free text. |
| **Demote** | Push a listing to the bottom of its tier. Not hidden, just last. |
| **Stopword** | A tiny word search engines ignore. "no" is one, which causes the bug. |
| **Prompt injection** | Text written by an outsider that tries to hijack an AI's instructions. |
| **Circuit breaker / pause** | After N failures, stop calling a service for a while. |
| **Edge cache** | Cloudflare's copy of a finished page, served without touching our servers. |
| **Fragment cache** | A cached *piece* of a page, not the whole page. |
| **Perk** | Something a renter can do without, like "no parking". Not treated as an avoid. |
