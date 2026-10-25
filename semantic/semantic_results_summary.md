# Field test document — plain English summary

Full text: [`semantic_results.md`](./semantic_results.md) · ticket RN-5124 · Run 4,
2026-10-17 · Montréal

---

## What this document is

It is a **test report**. Not a plan, not a design. It records one experiment.

The experiment: take 42 real search sentences, run each one **twice**, and compare.

- **Run A — the old way:** rank results by keywords only.
- **Run B — the new way:** rank by keywords **plus meaning**.

Then grade both result lists and see which is better.

This is **run 4**. Runs 1, 2 and 3 came before, using the same searches. So they can see
whether things are improving over time.

Of the 42 searches, **25 could be compared**. The rest could not, for reasons explained
below.

---

## How they measured it

Each of the first 10 results gets a grade:

| Mark | Meaning | Points |
|---|---|---|
| ✓ Clear match | The text says what was asked | 2 points |
| ◐ Part match | Half of it, something close, or only a photo tag | 1 point |
| ✗ No match | Nothing | 0 points |

Add the points up. That total is called **grade points**.

**Who graded?** Claude, reading each listing's text. The document is honest about this:

> "This is not a substitute for the human-graded release gate."

In plain words: an AI graded this. A human still has to check before it ships.

---

## The main result

| Number | What it means |
|---|---|
| **32 → 78** | Grade points more than doubled for listings that did **not** contain the words. 15 searches better, 1 worse, 5 the same. |
| **55 → 56** | Almost no change for listings that **did** contain the words. This is on purpose (see below). |
| **24 → 32** | Clear matches in the top 10 went up across all 25 searches. |
| **39 → 70** | Part matches nearly doubled. |
| **475 ms** | Typical time to understand a new search. Only 1 ms if the same search repeats. |

**The one-line takeaway:** meaning helps a lot where keywords found nothing, and is
deliberately kept quiet where keywords already worked.

---

## The main change in this run: the weight fix

This is the most important idea in the document. Let me build it up step by step.

### The setup

Listings are split into two groups:

- **Group A:** listings that do **not** contain the renter's leftover words.
- **Group B:** listings that **do** contain them.

### What went wrong in earlier runs

Meaning had weight **3** in both groups. That broke Group B.

Here is why. In Group B, every listing already contains the words. So they all look
roughly equally good to the meaning model. The meaning scores end up very close together.

At weight 3, those tiny differences got multiplied into big differences. So a listing that
only *resembled* the search could jump above a listing that *literally said it*.

The document says it plainly:

> "at 3 a listing that only resembled the search passed one that named it."

### The fix in run 4

They added a second, lower weight for Group B.

| Group | Meaning weight | Why |
|---|---|---|
| A — does **not** contain the words | **3** | Meaning is the only signal. Let it work. |
| B — **does** contain the words | **0.5** | Keywords already did the job. Meaning only breaks ties. |

The setting is called `ranking_meaning_keyword_band_weight`, and it lives in Rentals-API.

### Did it work? Yes.

| Group B, first 10 | Keywords only | Meaning at 3 (old) | Meaning at 0.5 (new) |
|---|---|---|---|
| Run 2's results | 42 | 31 ❌ | 41 |
| Run 3's results | 42 | 39 ❌ | 42 |
| Run 4's results | 55 | 52 ❌ | **56** ✅ |

Read the "Meaning at 3" column. In **every** run, it scored *lower* than keywords alone.
Meaning was actively making Group B worse.

At 0.5 that stops. In run 4 it finally scores higher than keywords alone (56 vs 55).

**The lesson:** more AI is not always better. Turning its influence *down* in the right
place was the fix.

---

## A real example, start to finish

The document walks through one French search in full detail.

**Renter typed:** "appartement près d'un parc et des espaces verts à Montréal"
*(apartment near a park and green spaces in Montréal)*

**What the search turned it into:**

- Filter: apartments
- Filter: park nearby
- **Leftover words:** "espaces verts"

**The scale of it:** 881 results, but only 129 had a meaning score. Understanding took
617 ms.

### Before — keywords only

The top 6 in the group where meaning acts:

| Listing | Note |
|---|---|
| Listing 1545237 | No park or green space mentioned |
| 146 rue Pilon, Saint-Eustache | **empty text** |
| 1256 Saint-André | **No usable text** |
| Listing 1544085 | No park or green space mentioned |
| Listing 1555360 | no park or green space mentioned |
| Listing 1556553 | No park or green space mentioned |

Six results. **Zero** mention a park. Two have no text at all. This is a useless page.

### After — keywords plus meaning

| Listing | What meaning found | Verdict |
|---|---|---|
| Listing 1562241 | "face à un beau et grand Parc" | ✓ clear |
| Listing 1545689 | "Jardin botanique, parc Maisonneuve" nearby | ✓ clear |
| Listing 1561847 | almost no text | ✗ no match |
| Listing 1527201 | "pistes cyclables du parc Maisonneuve" | ✓ clear |
| Listing 1545704 | no park or green space mentioned | ✗ no match |

Score went from **0 clear matches to 3**.

Notice none of those listings contain the phrase "espaces verts". They say "Parc",
"Jardin botanique", "pistes cyclables du parc". Keywords could never have found them.

---

## Four honest findings

The document does not only report good news. That is a good sign.

**1. Meaning helps most where keywords find nothing.** ✅
Grade points went 32 → 78. It catches wording keywords miss: "immeuble tranquille" for
"calme pour le télétravail", "huge terrazzo" for a large terrace.

**2. Where keywords already match, meaning now only breaks ties.** ✅
This is the weight fix working. "Near restaurants" was the worst case before (12 → 7). It
now gains (12 → 14).

**3. Very short listings still float up.** ⚠️ **This is a real bug.**
Listing #1561847's entire text is *"Additional amenities: Central Heat"*. It has the
**highest** meaning score in 3 of the 25 searches.

Think about why. A very short text has little meaning in it, so it ends up sitting in the
middle of everything, close to every question. It is a false positive. Nobody has fixed
this yet.

**4. Gains are mostly part matches, not clear matches.** ⚠️
Clear matches: 24 → 32 (+8). Part matches: 39 → 70 (+31).

So most of the improvement is "close enough", not "exactly right". The win is real but
softer than the headline suggests.

---

## Two searches got worse

Honest reporting. Two of the 25 lost a point:

| Search | Change |
|---|---|
| heritage building with character and high ceilings in Montreal | −1 |
| quartier tranquille pour une famille à Montréal | −1 |

Small, but worth knowing.

---

## A measurement problem you should understand

This is subtle, and the document is careful about it.

The thing that splits the sentence into filters and leftover words is called **AI search**.
**It does not behave the same way every time.**

In 13 of the 42 searches, it left **different words** than it did in run 3.

Example: "near restaurants in montreal"

- Run 3 left: `restaurants`
- Run 4 left: `near restaurants`

That looks tiny. But it means run 3 and run 4 were not asking the same question. So when
the score jumps from 7 to 14, you cannot be sure the ranking improved. The input changed.

Where this happened, the document marks it:

> "another group compared; 0 of 10 listings in common"

**Zero listings in common.** Those two lists have nothing to do with each other.

**Why this matters to you:** before you trust any number in this report, check whether the
leftover words stayed the same between runs. If they changed, the comparison is shaky.
The document warns about this itself:

> "read across a row with care"

---

## Things to be careful about

| Issue | Why it matters |
|---|---|
| **Only 282 listings had a meaning score** on this local test copy | The deck says 26,117 on develop. This one test ran on a much smaller set. |
| **Most top-page listings have no meaning score at all** | Paid tiers fill page 1, and those mostly have no vector. So meaning often changes nothing on page 1. |
| **Page 1 only changed in 15 of 25 searches** | In 10 searches, the renter would see no difference. |
| **The grader was an AI, not a person** | The document says so directly, twice. |
| **Short/empty listings score too high** | Finding 3. Unfixed. |

---

## Three questions worth asking your team

1. **Has anyone fixed the short-listing bug (#1561847)?** A listing with almost no text
   should not out-score real matches.
2. **Why does AI search parse differently between runs?** If that is not stable, every
   comparison is unreliable.
3. **Why do so few listings have a meaning score in the top tiers?** If paid listings
   never get a vector, meaning never touches page 1 — where it would matter most.

---

## Terms used in this document

| Term | Plain meaning |
|---|---|
| **Run** | One full pass of the test. Run 4 is the fourth attempt. |
| **Leftover words** | The part of the sentence that could not become a filter. |
| **Grade points** | The score. 2 per clear match, 1 per part match, in the top 10. |
| **Tier** | Placement level. Paid listings sit in higher tiers and always come first. |
| **Keyword group** | Whether a listing contains the leftover words or not. |
| **Weight** | How much one factor counts in the final score. Bigger weight = more influence. |
| **Vector** | The list of numbers holding a listing's meaning. "No vector" = not indexed yet. |
| **Parse** | Splitting a sentence into parts the computer can use. |
| **Rentals-API** | The service that does the ranking. |
| **Develop** | The internal test copy of the site. Not the live one. |
| **Release gate** | The check that must pass before a feature goes live. |
