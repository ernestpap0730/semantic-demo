# Summary — what each folder means, in plain English

Full transcript: [`origin.md`](./origin.md) · 57 screenshots · 9 folders · read 2026-10-25

---

## Read this first

All 57 screenshots are about **one** piece of work.

The work has a name: **epic RN-5120, "Ranking and meaning"**.

An *epic* is a big job split into smaller tickets. Here are the tickets:

| Ticket | What it does | Status |
|---|---|---|
| RN-5109 | Measure how slow search is today | ✅ Done |
| RN-5114 | Work out each listing's distance to parks, metros, shops | ⏸️ Built, but switched off everywhere |
| RN-5121 | Build the ranking engine (order by score, not by newest) | ✅ Done |
| RN-5122 | Rank closer listings higher | 🚫 Blocked |
| RN-5123 | Build the meaning index | 🟡 Ready for QA |
| **RN-5124** | **Add semantic ranking, behind a quality gate** | 🟡 **Peer review** |
| RN-5389 | Handle "no carpets" style searches | 🔵 In progress |

**RN-5124 is the one you care about.** It is the semantic search ticket. Nearly every
screenshot is about it.

---

## The 9 folders, one by one

### 1. `chat/` — a Slack message

**What it is:** one message from Chris to the team, 21 October, telling everyone where things
stand.

**Why it matters:** it is the shortest, clearest explanation of the whole project. If you read
one thing, read this.

**The key line:** you can try it yourself at `rn-5124.cd.rentals-dev.io` (Montréal only). Type
a sentence, press Enter. Add `&semantic_ranking=0` to the address to see the same search
*without* meaning. That is the fastest way to understand what the feature does.

---

### 2. `jira/rn_5122/` — the sprint board and the RN-5389 ticket

⚠️ **The folder name is wrong.** It says `rn_5122`, but the screenshots show the team's board
plus the **RN-5389** ticket. RN-5122 only appears as one card.

**The board** shows the whole team's work in columns: To Do → In Progress → Peer Review →
Ready for QA → In QA → Final Review → Ready for Release → Done.

Right now the last four columns are **empty**. Nothing has reached QA or release.

**RN-5389 is the "no carpets" problem.** It is a funny bug worth understanding:

- A renter types "apartment with **no** carpets".
- Keyword search ignores the word "no" and looks for "carpets".
- The meaning model also reads "no carpets" as very close to "carpets".
- So both systems rank **carpeted** apartments first. The exact opposite.

They fixed it in two steps. Step 1 is built: stop turning "no pets" into a *pet-friendly*
filter. Measured result: **0 of 48** negated wishes now become a filter.

Step 2 uses an AI model called **Jev**. Jev only answers yes or no. It is asked "does this
home have carpets?" for the top 48 results, then those listings get pushed down.

**The honest part you should notice:** the ticket lists what they tried and **threw away** —
AI rewriting of the search, a meaning penalty, a contrast score, and a reranker. All were
"worse or even". That is good engineering. They measured instead of guessing.

---

### 3. `jira/Rn_5124/` — the main semantic search ticket

**What it is:** the full Jira ticket for RN-5124, "Add semantic ranking behind a quality gate".

**The one sentence that defines the whole project:**

> "Semantic ranking should only reach renters if it measurably beats the keyword ranking, in
> both English and French."

That is the rule the whole team works under. Build it, prove it is better, *then* ship it.

**Status:** Peer Review. 8 story points. High priority. Assigned to Christopher Da Silva.

**It is blocked by two things:** RN-5121 (done ✅) and RN-5123 (ready for QA 🟡).

**Its three acceptance boxes are all unticked:**

- [ ] Beats the baseline in English and French
- [ ] Rollout % configurable, kill switch tested
- [ ] Unparsed-words share reported

⚠️ **Notice a mismatch.** The ticket says the plan is to "replay real past searches". But the
spec file (folder 4) says that replay **was retired on 2026-10-16**, because events no longer
record a search's words. So the ticket text is out of date. The team switched to a *golden
set* instead — a fixed list of made-up test searches. Good thing to ask about.

---

### 4. `project/` — the actual code repository

**What it is:** screenshots of the `RentalsCa` repo open in an editor.

**This is the most useful folder for you as a developer.** It tells you what the codebase
looks like.

**What you learn:**

| Thing | Detail |
|---|---|
| Language | Python |
| Framework | Django |
| Main app folder | `rentalsca_phoenix/` |
| Specs live in | `specs/rn-5120-ranking-and-meaning/` |
| There is a `CLAUDE.md` | So the team already uses Claude Code |
| There is an `AGENTS.md` | Instructions for AI agents working in the repo |

The specs folder holds: `spec.md`, `tasks.md`, `design.md`, `research.md`, `evaluation.md`,
`README.md`, and `rn-5122-plan.md`.

⚠️ **`spec.md` is mostly empty.** It has all the headings — Problem, Goals, Requirements — but
no text under them. Only the "Done when" checklist is filled in. The real detail lives in the
plan files instead.

**`rn-5122-plan.md` is worth reading in full.** It is a model of how this team writes a plan:
goal, what already exists (with measured numbers), the design, a numbered task list, risks,
and open decisions. Copy this structure when you write your own.

**One small finding hidden in it that you would never guess:**

> PostgreSQL's English stemmer reads "parking" as "park".

So a renter searching for a park gets every listing that mentions *parking* — 17,693 out of
26,202 listings. A bug caused by grammar, not by AI. It is marked "worth shipping now".

---

### 5. `semantic/github/` — the pull request

**What it is:** pull request **#518** on the `Rentals-API` repo. A *pull request* (PR) is a
request to merge your code into the main codebase.

**The size:** 18 commits, 36 files changed, **+1,946 lines added, −84 removed**. Still open.
Reviewer: Brendan Dagys.

**What the code actually does:** it adds a new sort called `semantic`. The caller sends a list
of numbers (the search's meaning). Rentals-API compares it to each listing's numbers.

**One design choice worth remembering:**

> "The caller sends the query's vector; Rentals-API never calls an embedding provider."

The ranking service never talks to an AI. It only does maths on numbers it is given. That
keeps it fast and predictable.

**Three careful details that show good engineering:**

1. A listing with **no** meaning score gets the *average* of the others. So missing data
   never helps or hurts a listing.
2. The renter's words and numbers are **hidden from error messages** in production.
3. If the system cannot read the meaning index, meaning just **turns off**. The search still
   works.

⚠️ **The most important thing in this folder is the commit history.** Look at these two:

| Date | Commit |
|---|---|
| Oct 17 | RN-5124: **weigh meaning less** among keyword matches |
| Oct 21 | RN-5124: **weigh meaning fully** among keyword matches |

**They changed their minds.** See the "One contradiction to know about" section below.

There is also a **second repo**: `rentals_ai_search_rs`. It is written in **Rust**, not
Python. Its job is to turn a plain-language search into a filtered results URL. Rocky Burt
owns it. So the team runs more than one language.

---

### 6. `semantic/pptx/` — the slide deck

**What it is:** a 13-slide presentation. Probably shown to managers or the wider team.

**Why it matters:** it is the *simplest* version of the story. Problem → how it works →
examples → numbers → privacy → what is left.

**Full plain-English breakdown:** [`../semantic/ppt_summary.md`](../semantic/ppt_summary.md)

**The three numbers to remember:** +68% better first page · 93 of 103 searches improved ·
26,117 listings indexed.

---

### 7. `semantic/quality gate/` — how to actually run the gate

**What it is:** an instruction page. It explains the test that must pass before semantic
search can go live, and gives the exact commands to run it.

**This is the folder that tells you what is blocking the release.**

**The gate has four rules. All four must pass:**

| Rule | Meaning |
|---|---|
| 5% better | Meaning must beat today's ranking by 5%+, in English **and** French separately |
| 20+ searches | At least 20 compared searches per language, so one lucky result can't carry it |
| 90% graded | At least 90% of each top-10 must have a grade |
| More wins than losses | Beyond what random chance would give (a *sign test*) |

**How it runs, in four steps:**

1. **Collect** — run every test search twice, with and without meaning (10–30 min)
2. **Export** — put every listing into one spreadsheet
3. **Grade** — an AI grades every listing 0, 1 or 2, blind. 10% goes to a human
4. **Score** — compare and print a verdict

**The status at capture:**

| | Item |
|---|---|
| ✅ Done | Meaning index built and live on the test site |
| ✅ Done | Ranking by meaning works |
| ✅ Done | The gate tool is written |
| 🟡 Built | 44 test searches — **not deployed yet** |
| 🟡 Built | AI grading — **not merged yet** |
| 🔴 To do | Get approval to send listing text to the grader |
| 🔴 To do | Product must agree the pass mark |
| ⏭️ Next | Actually run it |

**So: the gate has never been run.** The tool exists. The tests exist. Nobody has pressed go.

⚠️ **The page says this directly, and it is important:**

> **"Early signal, not the gate."** The 32 → 78 result everyone quotes came from a local copy
> with only **282 listings**, graded by Claude in a conversation, with **no blind setup and no
> spot-check**.

In plain words: the headline number people repeat is **not** from a proper test. The real
test is still waiting.

---

### 8. `semantic/search field test/` — the detailed experiment

**What it is:** the full test report from run 4, 17 October. 25 searches compared, each run
twice.

**Full plain-English breakdown:**
[`../semantic/semantic_results_summary.md`](../semantic/semantic_results_summary.md)

**The main lesson:** more AI is not always better. Turning meaning's influence *down* (from
weight 3 to 0.5) in one specific group was the fix that made run 4 work.

**Three weaknesses it reports honestly:**

1. Listings with almost no text score too high. One listing whose entire text is "Additional
   amenities: Central Heat" scored highest in 3 of 25 searches.
2. The sentence-splitter behaves differently between runs, so some comparisons are unreliable.
3. Most of the gain is "part matches", not exact ones.

---

### 9. `semantic/search quality report/` — the final report

**What it is:** a polished report dated **21 October**. Published as a Claude artifact. This is
the newest document of all 57 screenshots.

**This is the version you should trust most,** because it is the most recent and the most
thorough.

**The numbers:**

| Slice | Before | After | Change |
|---|---|---|---|
| English (71 searches) | 0.427 | 0.735 | **+72%** |
| French (32 searches) | 0.407 | 0.645 | **+58%** |
| All (103 searches) | 0.421 | 0.707 | **+68%** |

**Scale note:** they started with **355** test searches, but only **103** had leftover words
to rank by. Semantic search only does something when the sentence contains a wish the filters
cannot hold. The other 252 searches were unaffected.

**It also names a weak spot, honestly:**

> "street-address searches, where meaning can push the exact building down the page"

If a renter types an exact address, meaning can bury the right building. Nobody has fixed that.

**The "Try it yourself" table is the best thing in this folder.** Six real searches with
scores. Run them on the test site and you will *see* the difference instead of reading about
it.

---

## One contradiction to know about

This will confuse you if nobody warns you. So here is the warning.

**The question:** when a listing already contains the renter's exact words, how much should
meaning count?

Two documents give **opposite** answers.

| Date | Source | Weight | Reasoning given |
|---|---|---|---|
| **Oct 17** | Field test run 4 | **0.5** | At weight 3, a listing that only *resembled* the search beat one that *named* it |
| **Oct 21** | Quality report | **3.0** | At full weight it picks the best fit among them, which lifted it past the 5% bar |

The GitHub commits confirm the flip:

- Oct 17 — "weigh meaning **less** among keyword matches"
- Oct 21 — "weigh meaning **fully** among keyword matches"

**What happened:** on 17 October they turned meaning down to fix one problem. By 21 October
they turned it back up, because at 0.5 the gain was only +2.4% English and +4.8% French —
**below the 5% bar**. At full weight it reached +6.0% and +8.4% and passed.

**So the current, correct answer is 3.0.** The deck and the quality report agree. The field
test document is the older view.

**Why you should care:** if you read the field test first and believe it, you will be working
from a decision that was reversed four days later. Always check the date.

---

## The real state of the project

Putting all 9 folders together:

| Thing | State |
|---|---|
| Does semantic search work? | **Yes.** Measurably, in both languages. |
| Is it on the live website? | **No.** All switches off in production. |
| Has the formal quality gate run? | **No.** The tool is built but never executed. |
| What is blocking it? | Two approvals, a merge, and about 45 minutes of running |

**The three blockers, exactly:**

1. **Approval** to send listing text to the grading provider (Anthropic or OpenAI)
2. **Product sign-off** on the pass mark and on using an AI grader
3. **A merge** — the 44 test searches and the grading code are on a branch, not deployed

Then someone runs 4 commands (~30 min) and one person spends ~15 minutes spot-checking a
spreadsheet.

---

## What I would do in your first week

1. **Open the test site.** `rn-5124.cd.rentals-dev.io`, Montréal. Run the six searches from
   folder 9's table. Then add `&semantic_ranking=0` and run them again. Feel the difference
   yourself before reading more.
2. **Read folder 1 (Slack), then folder 9 (quality report).** Shortest and newest. Skip the
   rest until you need it.
3. **Ask about the human spot-check.** It is 15 minutes of work and it unblocks the release.
   It is the perfect first contribution: zero risk, high value, and it forces you to read
   every part of the system.
4. **Ask your three questions:**
   - Has anyone fixed the short-listing bug (a listing with no text scoring highest)?
   - RN-5124's ticket still describes the replay harness, but the spec says replay was
     retired on Oct 16. Should the ticket be updated?
   - The field test says meaning weight 0.5, the quality report says 3.0. 3.0 is current,
     right?

---

## New terms in this batch

| Term | Plain meaning |
|---|---|
| **Epic** | A big job split into several tickets. RN-5120 is the epic here. |
| **Sprint** | A fixed period of work, usually two weeks. Theirs is Oct 22 – Nov 5. |
| **Story points** | A rough guess of effort. RN-5124 is 8 points. |
| **Blocked** | This ticket cannot start until another one finishes. |
| **Pull request (PR)** | A request to merge your code into the main codebase. Others review it first. |
| **Commit** | One saved change, with a message explaining it. |
| **Peer review** | Another developer reading your code before it is merged. |
| **nDCG@10** | A score from 0 to 1 for how good a top-10 list is. Good matches near the top count most. |
| **P@5** | The share of the top 5 results that were judged relevant. |
| **Sign test** | A maths check that your win count is real and not luck. |
| **Golden set** | A fixed list of test searches, re-run every time so scores compare fairly. |
| **Baseline** | The old way, which the new way must beat. |
| **Kill switch** | A setting that instantly turns a feature off with no code deploy. |
| **Rollout percentage** | Show the feature to only some renters. 5% first, then more. |
| **Jev** | An AI model used for yes/no questions only. Used in the "no carpets" fix. |
| **POI** | "Point of interest". A park, metro, shop, gym, school. |
| **Stemmer** | Code that cuts words to their root. It reads "parking" as "park", which caused a bug. |
| **LATERAL** | A SQL technique for running a small lookup once per row. |
| **Cron** | A job that runs automatically on a schedule. RN-5114's cron is currently off. |
| **Bastion** | A locked-down server you connect through to reach production systems. |
| **kubectl** | The command-line tool for running things on a Kubernetes server. |
| **SSR** | "Server-side rendering". Building the page HTML on the server, not in the browser. |
| **GraphQL** | A way for the website to ask the API for exactly the data it needs. |
| **Rust** | A fast programming language. The `rentals_ai_search_rs` service uses it. |
