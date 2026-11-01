# How the business works — a simple guide

No code in this file. Just people, and what they do.

---

## The business in one sentence

**Rentals.ca is a noticeboard.** Landlords put homes on it. Renters look at it. Landlords pay
to be seen more.

That is the whole business.

---

## How the money works

Rentals.ca does **not** take rent money. It does not take a cut of the lease.

It earns money in three ways:

| # | What | Who pays |
|---|---|---|
| 1 | **Paid placement** — your listing shows higher | Landlords and property managers |
| 2 | **Leads** — you get the renter's message | Landlords and partner sites |
| 3 | **Extra services** — credit checks on renters | Landlords |

Posting a few listings is **free**. Being seen more costs money.

> **Note:** the free-listing count and the exact upgrade prices come from Rentals.ca's public
> website, not from the internal documents. Treat them as roughly right, not exact.

### Why "paid placement" matters to you

This is the single most important business rule in the product:

> **Paid listings always come first. Nothing can change that.**

Not the newest listing. Not the cheapest. Not the best match. **Not semantic search.**

Your feature only reorders listings *inside* the group they are already in. If you ever break
that rule, the company stops being paid.

---

## Everyone involved

| Role | Inside the company? | What they want |
|---|---|---|
| **Renter** | no | Find a home to live in |
| **Landlord** | no | Fill an empty home, fast |
| **Property manager** | no | Fill many homes in many buildings |
| **Leasing agent** | no | Show units and sign the lease |
| **Feed partner** | no | Send many listings in automatically |
| **Admin / support** | yes | Keep the site clean and honest |
| **Product** | yes | Decide what gets built |
| **Developer** (you) | yes | Build it |
| **QA** | yes | Check it works before renters see it |

---

# The people outside the company

## 1. The renter

**Who they are:** a person looking for a home. They pay nothing. They usually do not even
make an account.

**What they do:**

```
1. Type what they want
2. Look at the results
3. Open the ones they like
4. Send a message to the landlord
5. (maybe) Save a search, and get emails when new homes appear
```

**What makes them leave:** a page full of homes that do not match. That is exactly the
problem your feature solves.

**The thing to understand:** the renter is **not the customer**. They pay nothing. But
without them there is no product, because landlords only pay to reach renters.

---

## 2. The landlord

**Who they are:** usually a small owner. One house, or a few units.

**What they do:**

```
1. Make an account
2. Write a listing — address, rent, bedrooms, description
3. Upload photos
4. Publish it
5. (maybe) Pay to be shown higher
6. Wait for messages from renters
7. (maybe) Run a credit check on a renter who applies
```

**What they want:** an empty home costs them money every single day. Speed is everything.

**One detail from the code:** there is a job that **reminds landlords about unfinished
listings**. So half-written listings are a known, common problem.

---

## 3. The property manager

**Who they are:** a company paid by owners to run buildings. They handle many homes at once,
sometimes hundreds.

**How they differ from a landlord:**

| | Landlord | Property manager |
|---|---|---|
| How many homes | 1 to 5 | 50 to 1,000+ |
| How they add listings | types them by hand | **sends them automatically** |
| Who they are | a person | a company |
| How much they spend | a little | a lot |

**This is the bigger customer.** They do not type listings one at a time. Their software
sends everything across automatically. That is the "feed partner" role below.

---

## 4. The leasing agent

**Who they are:** the person who actually shows the apartment and signs the paperwork.

**What they do:** answer renter messages, book viewings, sign the lease.

**Why they matter to the product:** they are the ones reading the **leads**. If leads arrive
slowly or badly, they complain.

---

## 5. Feed partners

**Who they are:** other companies whose software sends listings into Rentals.ca
automatically, in bulk.

From the internal documents, there are **8 feeds**:

| Partner |
|---|
| Rentsync (the parent company) |
| CREA MLS (the real estate board) |
| Zumper |
| RentFaster |
| Roomies |
| bstk |
| OSM (map data, not listings) |

**How it works:** a job runs every hour. It fetches each feed, reads it, and loads the
listings in. No human types anything.

**Why this matters:** **most listings on the site arrive this way**, not from a person
clicking "publish". So the quality of listing text is largely out of Rentals.ca's control.

That is also why so many listings have thin or empty descriptions — which is a problem your
semantic search has to live with.

**It goes both ways.** When a renter sends a message about a partner's listing, the message
is sent **back out** to that partner.

---

# The people inside the company

## 6. Admin and support

**What they do:** keep the noticeboard honest.

- Remove fake or duplicate listings
- Help landlords who are stuck
- Handle billing problems
- Check identity and run credit checks

**Tools they rely on** (seen in the code): Stripe for payments, Persona for identity, Certn
for credit checks.

---

## 7. Product

**What they do:** decide what gets built, and what "good enough" means.

**Real examples from your own documents:**

- Product wrote the spec for the zero-results feature, on 22 October
- Product must **sign off the pass mark** before your quality gate can run
- Product decides whether an AI grader is acceptable instead of people

**The lesson:** product can block your release even when the code is finished. Right now, they
are one of the three things blocking semantic search.

---

## 8. Developer — you

**What you do:** build it, measure it, prove it is better, then release it slowly.

Your team's rule, in their own words:

> "Semantic ranking should only reach renters if it **measurably beats** the keyword ranking,
> in both English and French."

---

## 9. QA

**What they do:** test the feature before renters see it.

On your team's board the columns are: To Do → In Progress → Peer Review → **Ready for QA** →
In QA → Final Review → Ready for Release → Done.

---

# The life of a listing

How a home gets onto the site, step by step.

```
   A landlord types it            OR        A partner's software sends it
            |                                            |
            v                                            v
      Photos uploaded                            Hourly import job
            |                                            |
            +---------------------+----------------------+
                                  |
                                  v
                      Saved in the database
                                  |
                                  v
                   Photos get tags added automatically
                                  |
                                  v
                    Added to the search list (hourly)
                                  |
                                  v
                    Turned into "meaning" numbers (hourly)
                                  |
                                  v
                         Renters can now find it
```

**The important part:** a landlord's change is **not** visible instantly.

| Where | How long it takes |
|---|---|
| Saved in the database | instant |
| Shows up in search | **up to 1 hour** |
| Works with the nearby-place filter | up to 1 hour 20 min |
| Works with meaning ranking | up to 1 hour 30 min |

So a landlord who fixes a typo may wait 90 minutes to see it. That is a known complaint.

---

# The life of a search

What happens when a renter types something.

```
   Renter types a sentence
            |
            v
   The site splits it into two parts
            |
            +---> FILTERS  (city, bedrooms, price)  --> removes listings
            |
            +---> LEFTOVER WORDS ("quiet", "sunny") --> decides the ORDER
                                  |
                                  v
                      Listings are put in order:
                        1. Paid listings first
                        2. Then exact word matches
                        3. Then the best score
                                  |
                                  v
                        The renter sees the page
                                  |
                                  v
                   They message a landlord (a "lead")
                                  |
                                  v
               The landlord or partner receives it
```

**Filters remove. Leftover words reorder.** That is the whole idea in one line.

---

# Where your feature fits, per role

| Role | What semantic search changes for them |
|---|---|
| **Renter** | Can type a real sentence. "Quiet place to work from home" now finds something. |
| **Landlord** | Their home gets found even if they used different words than the renter. |
| **Property manager** | Their bulk listings compete on *meaning*, not on who guessed the right keyword. |
| **Leasing agent** | Fewer pointless messages. Renters who arrive are a better fit. |
| **Feed partner** | Nothing to change. It works on the text they already send. |
| **Admin** | Nothing changes. Paid placement is untouched. |
| **Product** | A measurable improvement: +68% better first pages. |
| **You** | The thing you own. |
| **QA** | A new thing to test before release. |

**Nobody loses.** That is why this feature got funded.

---

# The five rules that never break

Memorise these. Every decision in the product respects them.

| # | Rule | Why |
|---|---|---|
| 1 | **Paid listings always come first** | It is how the company earns money |
| 2 | **Filters are promises** | If a renter says "under $2,000", never show $2,500 |
| 3 | **The page must never break** | If the AI fails, show the old ranking instead |
| 4 | **Prove it before shipping** | Measure, do not guess |
| 5 | **Release slowly** | One city, a few percent of renters, with an off switch |

---

# The simplest possible summary

> Landlords and partner companies put homes on a noticeboard.
> Renters search it for free.
> Landlords pay to be seen higher.
> Filters decide **which** homes appear.
> Your feature decides **what order** they appear in —
> but never above the ones that were paid for.

---

## Terms

| Term | Plain meaning |
|---|---|
| **Listing** | One rental advert on the site. |
| **Lead** | A message from a renter to a landlord. Landlords pay for these. |
| **Feed** | Listings sent in automatically by another company's software. |
| **Paid placement** | A landlord paying so their listing appears higher. |
| **Filter** | A hard rule that removes listings. "Montréal only." |
| **Leftover words** | The part of a sentence no filter could use. Your feature works on these. |
| **Property manager** | A company paid by owners to run buildings and find renters. |
| **Leasing agent** | The person who shows the home and signs the lease. |
| **Credit check** | Checking whether a renter is likely to pay rent. A paid extra. |
| **QA** | "Quality Assurance". People who test before release. |
| **Product** | The team that decides what gets built and what "good enough" means. |
