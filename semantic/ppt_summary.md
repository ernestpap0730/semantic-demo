# Slide deck — plain English summary

Full text: [`ppt.md`](./ppt.md) · 13 slides · October 2026

---

## In one sentence

Rentals.ca built semantic search, tested it, and it works — but it is **switched off in
production** and waiting for one human check.

---

## The problem they were solving

A renter types a full sentence:

> "sunny apartment with a nice view in Montreal under 2000"

The old search turns part of that into filters:

- Montréal ✅
- Apartments ✅
- Rent under $2,000 ✅

But two words are **left over**: `sunny` and `nice view`.

Those words were simply thrown away. **28 of 42 test searches had leftover words.**

Why they could not just match the words? Because French listings say "ensoleillé" or
"lumineux". Neither word is "sunny". So the old search found nothing.

---

## How their system works

There are two halves.

### Half 1: every hour, ahead of time

1. Take the listing text (description, photo captions, photo tags).
2. Strip out personal details (phones, emails, links).
3. Send it to the **meaning model** (bge-m3, running on Cloudflare Workers AI).
4. Store the result in the database. This is the **meaning index**.

### Half 2: on every search, live

1. Take the leftover words (what the filters could not use).
2. Strip out personal details again.
3. Send them to the **same model**. There is a 2 second time limit. The answer is cached
   for a day.
4. Rank the results.

**Their stack for this:** Django (website) · Rentals-API (ranking) · PostgreSQL with
pgvector (index) · bge-m3 (model).

---

## How a listing's position is decided

Three steps, in this order:

1. **Paid tier first.** Landlords who paid always appear above those who did not. Meaning
   never changes this.
2. **Has the words?** Inside a tier, listings containing the leftover words come first.
3. **Score.** Inside those groups, five factors decide the order.

| Factor | Higher when | Weight |
|---|---|---|
| Freshness | listed more recently | 1 |
| Value | rent is lower than similar units | 1 |
| Rotation | it is this listing's turn today | 0.5 |
| Keywords | its text has more of the words | 3 |
| **Meaning** | its meaning is closer to the words | **3** |

**The key point:** meaning is just *one factor among five*. It does not replace the old
search. It is added to it.

---

## Does it actually work? Yes.

They ran a quality test on 103 searches. An AI model graded the results blind.

| Measure | Result |
|---|---|
| Better first page overall | **+68%** (score 0.42 → 0.71 out of 1) |
| Searches that improved | **93 of 103** (6 the same, 4 worse) |
| English | +72% (65 better, 2 worse) |
| French | +58% (28 better, 2 worse) |
| Listings indexed | **26,117** — all of them, refreshed hourly |

### Three real examples

| Renter typed | Filters only | With meaning |
|---|---|---|
| "appartement près d'un parc et des espaces verts" | 0 of 10 matched | **3 of 10** matched |
| "appartement calme pour le télétravail" | 0 of 10 matched | **2 of 10** matched |
| "walking distance to the subway, renovated kitchen" | 0 of 10 matched | **1 of 10** matched |

Look at the left column. **Zero every time.** The old search was returning nothing useful
for these sentences. Anything above zero is a win.

What meaning found that keywords missed:

- "face à un beau et grand Parc" → matched "espaces verts"
- "immeuble tranquille" → matched "calme"
- "Insonorisation supérieure" (good soundproofing) → matched "quiet"
- "Cuisine et salle de bain entièrement rénovées" → matched "renovated kitchen"

Notice something important: the system matched **across languages**. An English listing
saying "Bright, Quiet" was found by a French search.

---

## The "no carpets" fix

A separate ticket (RN-5389) solved a funny bug.

A renter types "apartment with **no** carpets". The old system saw the word "carpets" and
ranked carpeted apartments at the top. The exact opposite of what was wanted.

Now it understands the "no" and ranks **hardwood floors** instead.

Results: +25% on new tests, +31% on the quality test.

Also note: **0 of 48** negated wishes were handled by filters. So filters were useless
here. Only meaning could fix it.

---

## Which model they chose, and why

They tested two and picked the cheaper one.

| | bge-m3 (chosen) | text-embedding-3-small (dropped) |
|---|---|---|
| Who | Cloudflare Workers AI | OpenAI |
| Numbers per text | 1,024 | 1,536 |
| Price per million tokens | **$0.0118** | $0.02 |
| Quality | passed the gate | "no better than bge-m3" |

**The reason:** OpenAI's model cost almost twice as much and was not better. So they kept
bge-m3.

They also built a **quality gate**. Meaning had to beat keywords by 5% or more in *both*
languages, or it would not ship. It scored +6.0% English and +8.4% French. It passed.

Useful detail: *"models switch without a deploy."* They can swap the model by changing a
setting. They are not locked in.

---

## Privacy

Four rules, from seven rounds of privacy review:

1. Personal details (names, phones, emails, addresses) are removed **before** the words
   leave their servers.
2. The renter's words are **never recorded**. Logs only say whether meaning ranked the page.
3. Error reports drop the words and any map location.
4. If anything fails, the page still works. There is a 2 second limit, and any failure
   falls back to keywords alone.

Only made-up test searches were ever sent to outside providers.

---

## Status: built, tested, and switched OFF

This is the part that matters most for you.

There are **three switches, and all are off in production**:

1. **Master switch** — off in production, on in develop.
2. **City list** — they will turn it on one city at a time.
3. **Percentage of searches** — can be set to 0% to stop it instantly, with no deploy.

| State | Where |
|---|---|
| ✅ Working, full index, quality test passed | develop (internal copy) |
| ❌ Switched off | production (the real website) |

### What is blocking it

> "Before production, **a person spot-checks its grades**."

All the grading so far was done by an AI model. A human has to confirm the AI graded
fairly before it goes live.

### What comes after

The next ticket is **commute-aware ranking** (RN-5122): listings closer to the metro or
park the renter asked for rank higher. It reuses distances already stored by RN-5114.

---

## What this means for you

1. **The feature already exists.** You are not building it from scratch. You are joining
   a project that is nearly finished.
2. **The model decision is already made.** bge-m3, on Cloudflare Workers AI.
3. **The next real task is the human grading check.** That is the only thing between this
   and production. It is a great first job: low risk, high visibility, and it forces you
   to read every part of the system.
4. **After that, commute-aware ranking** is the next ticket on the board.

---

## Terms used on these slides

| Term | Plain meaning |
|---|---|
| **Leftover words** | The part of the renter's sentence that could not become a filter. |
| **Meaning index** | The stored number-lists for all listings. Rebuilt every hour. |
| **bge-m3** | The model that turns text into numbers. Free and open, hosted by Cloudflare. |
| **pgvector** | The PostgreSQL add-on that stores and searches those numbers. |
| **Workers AI** | Cloudflare's service for running AI models. |
| **Quality gate** | A pass/fail rule. Ship only if the score beats the old way by 5%+. |
| **Golden set** | A fixed list of test searches, re-run every time, so scores compare fairly. |
| **Develop** | The internal copy of the site, for testing. Not the real website. |
| **Production** | The real, live website that renters use. |
| **Negated wish** | A wish phrased backwards. Example: "no carpets". |
| **Feature switch** | A setting that turns a feature on or off without changing code. |
