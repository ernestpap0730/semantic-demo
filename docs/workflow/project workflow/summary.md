# `project workflow` — summary

**Source:** `D:\screenshots\code\workflow\project workflow` · 15 images
**The document:** "Rentals.ca System Map" · an architecture review dated **2026-10-26**

---

## What this document is

It is a **map of the whole website**. Not just semantic search — everything.

Its own opening line says it best:

> "Every part of the site, from the renter's browser to the database and the nightly jobs,
> what each part is built with and why, and where it is slow or fragile, with a fix for each."

It was written by reading four repositories: `RentalsCa`, `Rentals-API`, `Rentals-Deploy`
and `rentals_ai_search_rs`. Every number comes from real code, config or docs.

**Why it matters to you:** this is the single best onboarding document in all 57 screenshots.
If you want to understand the system you just joined, start here.

It has five parts: System diagram · One search in order · Data freshness · Bottlenecks and
fixes · Each component.

---

## Part 1 — The system diagram

The site is built in five layers. Here they are, top to bottom.

### Layer 1: the browser

Vue 3 "islands" sitting on server-rendered HTML. An *island* means most of the page is plain
HTML from the server, and only small parts are live Vue components.

Map pins and page 2 come through `/graphql`. A tracking SDK runs alongside.

### Layer 2: the edge

**Cloudflare.** It caches results pages for 1 hour, caches `/graphql` answers for 1 hour, and
blocks attacks (DDoS and WAF).

Then **AWS load balancer → Traefik → an nginx proxy**. The proxy splits traffic:

| Search type | Goes to |
|---|---|
| Map searches (`bbox=`) | read-only pods |
| Everything else | read-write pods |

### Layer 3: our services (Kubernetes, EKS, ca-central-1)

| Service | Built with | Size |
|---|---|---|
| **RentalsCa** | Django 5.2 on ASGI, Wagtail CMS | 8–24 + 6–24 pods, **1 process and 1 CPU each** |
| **Rentals-API** | FastAPI, Strawberry GraphQL, SQLAlchemy async, asyncpg | 12–20 pods |
| **AI search service** | Rust, axum | 2 pods |
| **User tracking** | Rust, SQS | — |

The AI search service tries three things in order: **rules → Jev → the language model**. It
runs on per-ticket sites only, and it does **not return leftover words yet**. That last point
turns out to be the biggest blocker in the whole system.

### Layer 4: the data

| Store | What it holds |
|---|---|
| **Aurora PostgreSQL 14, writer** | Everything. PostGIS, h3, pgvector, full-text, the `quick_*` search views |
| **Aurora reader** | Same zone. Only serves map pages and typeahead |
| **Redis** | RentalsCa has 1 GB — **unused**. Rentals-API uses it for lead throttling and highlights |
| **S3** | Photos, static files, feed artifacts, nightly database dump |

**One database holds everything** — listings, leads, billing, CMS, places and vectors.

### Layer 5: the scheduled jobs

Kubernetes CronJobs. **There is no queue and no broker.**

| Job | When | State |
|---|---|---|
| Feed imports | 8 feeds, hourly to daily | running |
| Search views | full refresh hourly at :00 | running |
| POI distances | hourly at :20 | ⚠️ **off in production** |
| Meaning index | bge-m3, hourly at :30 | ⚠️ **off in production** |
| Everything else | alerts, emails, exports, photo tags nightly, OSM weekly | running |

Feed sources: Rentsync · CREA MLS · Zumper · RentFaster · Roomies · bstk · OSM

### Outside services they pay for

Cloudflare AI Gateway (OpenAI gpt-6-luna for parsing; Workers AI for Jev and bge-m3; Llama
3.3 as fallback) · Google Places and Geocoding · Geoapify for commute times · SendGrid for
email · CREA DDF and Zumper for leads · Stripe, Persona and Certn for billing and credit
checks · Rentsync tagger and restb.ai for photo tagging.

---

## Part 2 — One search, in order

This traces a real search: *"quiet 2 bed in Toronto, no carpets"*.

| # | What happens | Time |
|---|---|---|
| 1 | Browser asks Cloudflare for the page | |
| 2 | **Cache hit?** Page comes straight back, no server work at all | 0 s |
| 3 | Cache miss → RentalsCa reads the form and the rollout flag | ≤ 2 s |
| 4 | Keywords are embedded by bge-m3 → a vector comes back | ≤ 2 s |
| 5 | First 48 listing ids fetched (only for "no …" searches) | |
| 6 | Jev is asked, 2 calls at once | 0.64 s median |
| 7 | The real ranked search runs, with `demote` | |
| 8 | 24 listings come back | |
| 9 | **If 0 results**, filters are dropped one at a time — up to **13 more searches** | |
| 10 | HTML is rendered and cached for an hour | |

The document's own conclusion about this flow:

> Without a Django cache in production (bottleneck 1), the rollout flag, the keyword vector
> and Jev's answers are **fetched again on every request** the edge did not serve.

---

## Part 3 — Data freshness

How long a landlord's change takes to actually show up:

| Where | Delay |
|---|---|
| In the database | instant |
| In search results | **up to 60 minutes** |
| In the nearby-place filter | up to 80 minutes |
| In meaning ranking | up to 90 minutes |

And the feed import itself runs hourly (or daily), **before** all of that.

So a landlord who fixes a typo may wait an hour and a half to see it. The cause is that
search reads from materialized views that are rebuilt in full once an hour.

---

## Part 4 — The 13 bottlenecks

This is the heart of the document. Every problem has a named file and line number.

### The four big ones

**1. The RentalsCa cache is switched off.**
Production runs Django's no-op cache — `REDIS_CONN=""` **since 2024-04-16**. That is over two
years. Every rollout flag, keyword vector, geocode, commute time and Jev answer is recomputed
on every request Cloudflare did not serve. Meanwhile a 1 GB Redis pod sits unused.
*Impact: high. Change: small.* **This is the best first fix in the whole system.**

**2. Search runs on the writer database.**
The writer also handles feed imports and the hourly view refresh. The reader only serves map
pages and typeahead. *Impact: high. Change: small.*

**3. No query time limits and no connection pooler.**
Rentals-API sets no limit, so calls use aiohttp's default 300-second timeout. 12–20 API pods
with pools up to 30, plus every RentalsCa pod, all connect straight to Aurora.
*Impact: high. Change: medium.*

**4. One process per pod.**
Each pod runs one uvicorn process on one CPU. The AI parse and rent-report views are
*synchronous* on a shared thread. Stalls have already killed pods (RN-4817) and stalled the
pool (RN-5293). Nodes sit at about 89% of their CPU limits.
*Impact: high. Change: medium.*

### The rest

| # | Problem | Key fact |
|---|---|---|
| 5 | AI search parse is slow | The language model takes 1.5–2.1 s. The Rust service takes 0.48 s but **returns no leftover words**, so semantic ranking cannot run on it |
| 6 | Freshness | Views rebuilt in full hourly instead of only what changed |
| 7 | Leads sent inline | `submitLead` calls SendGrid/CREA/Zumper **inside the request**, so a slow vendor makes the renter wait |
| 8 | Jobs that are off | POI distances and the meaning index are defined but off. Nearby-place search uses a live join: **1.07 s instead of 0.15 s** |
| 9 | **Can't see slowness** | Trace export commented out, Sentry off, **no alert rules at all**. A feed hung 5 days and a backup re-uploaded a stale dump for 2 weeks before anyone noticed |
| 10 | Zero-result pages | Up to 13 extra searches for one page |
| 11 | Shared request headers | One renter's IP header can leak onto another renter's request. Marked **correctness** |
| 12 | Idle work | A job runs every 2 hours on 2 CPUs calling a command that does nothing |
| 13 | Single zone | Writer and reader in one availability zone. The TLS certificate is renewed **by hand** |

---

## Part 5 — Each component

Ten components, each with the same four sections: what it is built with, **why** that was
chosen, what else was considered, and how to improve it.

The "why" column is the valuable part. Some examples:

| Choice | Reason given |
|---|---|
| Vue islands, not Nuxt | A Node server would be a second runtime to deploy and scale |
| Cloudflare, not CloudFront | Cloudflare already runs their DNS and AI Gateway |
| pgvector, **not** a vector database | Exact scan takes about 20 ms — "not needed" |
| Postgres full-text, **not** Elasticsearch | Better scoring, but a second store to keep in sync |
| CronJobs, not Celery | "No queue or broker to run" |
| Rentals-API exists at all | To take load off the Django ORM — RDS cost, CPU spikes, a 5-replica limit |

Note the honesty: for Elasticsearch they wrote "better text scoring" — admitting the thing
they *didn't* pick is better at one job, and explaining the trade anyway.

---

## The five things to remember

1. **The cache has been off for two years.** One config line. High impact. Start here.
2. **Nobody can see when things are slow.** No traces, no Sentry, no alerts in production.
   Problems are found by accident, days later.
3. **Two jobs the semantic work depends on are off in production** — POI distances and the
   meaning index.
4. **The Rust AI search service does not return leftover words.** Without them, semantic
   ranking cannot run on that path at all.
5. **One database does everything**, and search reads from the writer, which is also busy
   importing feeds.

---

## How this connects to your job

You own semantic search. Three bottlenecks here are directly yours:

| # | Why it is your problem |
|---|---|
| **8** | The meaning index job is **off in production**. Your feature has no data there. |
| **5** | The fast Rust parser returns no leftover words. No leftover words = no semantic ranking. |
| **1** | With the cache off, every search re-embeds the same keywords instead of reusing them. |

Bottleneck 5 is the one to watch. The document states it plainly: production uses the slow
language model, and the fast Rust service "returns no leftover words, so keyword and meaning
ranking cannot run on its answers."

So if the team moves production to the Rust service before it returns leftover words, **your
feature silently stops working.**

---

## New terms

| Term | Plain meaning |
|---|---|
| **Pod** | One running copy of an app inside Kubernetes. |
| **Writer / reader** | The main database, and a read-only copy that shares its storage. |
| **Materialized view** | A saved, pre-computed query result. Fast to read, but must be rebuilt. |
| **ASGI** | The modern Python way of serving web requests, which allows async. |
| **uvicorn** | The program that actually runs a Python web app. |
| **Island** | A small live component on an otherwise plain HTML page. |
| **WAF** | "Web Application Firewall". Blocks malicious requests. |
| **Connection pooler** | Software that shares a few database connections among many apps. |
| **Advisory lock** | A Postgres flag ensuring only one copy of a job runs at a time. |
| **Typeahead** | The suggestions that appear as you type in a search box. |
| **bbox** | "Bounding box". The map rectangle you are looking at. |
| **Circuit breaker** | After N failures, stop calling a service for a while. |
| **Terraform** | Code that describes your servers, so they can be rebuilt reliably. |
