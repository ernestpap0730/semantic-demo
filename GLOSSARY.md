# Glossary

Every tech and business word we use, in plain English.
One line each. Newest terms get added as we go.

## Business terms

| Term | Plain meaning |
|---|---|
| **Rentals.ca** | The rental website we work on. People use it to find a home to rent in Canada. |
| **Rentsync** | The company that owns Rentals.ca. It also sells software to landlords. |
| **Listing** | One rental ad on the site. For example "2 bedroom apartment, Toronto, $2,400/month". |
| **ILS** | "Internet Listing Service". A website where many rental ads from many owners are collected in one place. Rentals.ca is one. |
| **Rentals.ca Network** | A group of rental websites all owned by Rentsync. Includes Rentfaster.ca, Louer.ca, RentCanada.com and others. |
| **Syndication** | Copying one listing out to many websites at once. The owner writes the ad once, and it appears on 20 sites. |
| **Landlord** | The person who owns the property and rents it out. |
| **Property manager** | A company paid by the landlord to run the building and find renters. |
| **Leasing agent** | The person who shows the unit to renters and signs the rental contract. |
| **Lead** | A renter who showed interest, for example by sending a message about a listing. Landlords pay to get leads. |
| **Multifamily** | A building with many rental units, like an apartment tower. The opposite is a single house. |
| **Unit** | One home inside a building. Apartment 501 is a unit. |
| **Promoted / Featured** | Paid upgrades. The landlord pays money so their listing shows higher in the list. |
| **Urbanation** | A research company that Rentsync bought. It studies rent prices. |
| **National Rent Report** | A monthly report from Rentals.ca about average rent prices in Canada. Good free marketing. |

| **Proximitii** | An outside company that sells neighbourhood data. Real estate sites pay them to show "what is nearby" on a listing page. |
| **Location intelligence** | Data about a place. Example: nearby schools, bus stops, crime rate, noise level. |
| **Vendor** | An outside company you pay for a service, instead of building it yourself. |

| **Building Stack** | Another Rentsync product. Software for signing leases and managing people who already live in a building. |
| **Competitor** | Another company selling the same kind of thing. For rental search: Apartments.com and RentCafe. |
| **Facet** | One filter choice in a search form. Example: a "2 bathrooms" checkbox. |

## Tech terms

| Term | Plain meaning |
|---|---|
| **Repo** | Short for "repository". The folder holding all the project's code and its history. |
| **Git** | The tool that saves every change to the code, so you can go back in time. |
| **Git clone** | Copying a repo from the internet down onto your own computer. |
| **API** | A way for one program to ask another program for data. Like a waiter taking your order to the kitchen. |
| **REST API** | The most common style of API. You ask for data using a normal web address (URL). |
| **Endpoint** | One specific web address in an API. Example: `/listings/1458073` returns one listing. |
| **HTTP status code** | A number the server sends back to say what happened. `200` = OK. `403` = you are not allowed. `404` = not found. |
| **Scraping** | Writing a program that reads a website's pages and pulls out the data. Often against the site's rules. |
| **Cloudflare** | A service sitting in front of a website. It speeds the site up and blocks bots. |
| **Managed challenge** | A Cloudflare test that checks if you are a real browser. Scripts fail it and get a `403`. |
| **robots.txt** | A text file at the top of a website. It tells search engines which pages they may read. |
| **Sitemap** | A file listing every page on a website. It helps Google find all the pages. |
| **URL** | A web address. Example: `https://rentals.ca/toronto`. |
| **Slug** | The human readable part of a URL. In `/toronto/2fifteen`, the slug is `2fifteen`. |
| **Query parameter** | Extra settings added after a `?` in a URL. In `?baths=2`, the parameter is `baths`. |
| **Pagination** | Splitting a long list of results across many pages. The `p=2` in a URL means "page 2". |
| **bbox** | Short for "bounding box". Four numbers describing a rectangle on a map. The site uses it to find listings inside the area you are looking at. |
| **Feed** | A machine readable file of listings, usually JSON or XML. Built for programs, not people. |
| **JSON** | A simple text format for data, used by almost every API. Looks like `{"rent": 2400}`. |
| **XML** | An older text format for data. Looks like `<rent>2400</rent>`. |
| **DNS** | The internet's phone book. It turns a name like `rentals.ca` into a number (an IP address). |
| **CNAME** | A DNS record meaning "this name is really just a nickname for that other name". |
| **TLS certificate** | The file that makes a site `https` instead of `http`. It proves the site is who it claims to be. |
| **SAN** | "Subject Alternative Name". The list of all domain names one TLS certificate covers. |
| **XHR / fetch** | How a web page asks the server for more data *after* the page has already loaded. |
| **CMS** | "Content Management System". Software for writing web pages without coding. HubSpot is one. |
| **PMS** | "Property Management System". Software landlords use to run buildings and track rent. Yardi and Buildium are two. |
| **Widget** | A small piece of someone else's web page that you drop into your own page. Example: an embedded map. |
| **Embed** | To place an outside widget inside your own page, usually with an `<iframe>` tag. |
| **Third party** | Code or data that comes from another company, not from us. |

## Search and AI terms

| Term | Plain meaning |
|---|---|
| **Keyword search** | Old style search. It looks for your exact words in the text. If the words are missing, you get nothing. |
| **Semantic search** | Search by *meaning*, not exact words. You type "quiet place near a park" and it understands you, even if the listing never says "quiet". |
| **Natural language** | Normal human sentences. The opposite of ticking boxes in a form. |
| **NLP** | "Natural Language Processing". Any computer work that reads or writes human sentences. |
| **Embedding** | A list of numbers that stands for the meaning of some text. Two texts about similar things get similar numbers. |
| **Vector** | Just another word for that list of numbers. |
| **Vector search** | Finding the listings whose numbers are closest to your question's numbers. Closest numbers = closest meaning. |
| **Vector database** | A database built to store those number lists and find the closest ones fast. |
| **pgvector** | A free add-on for PostgreSQL. It turns the normal database you already have into a vector database. |
| **Elasticsearch** | A popular separate search engine. Many listing sites use it for fast filtering. |
| **LLM** | "Large Language Model". The kind of AI behind ChatGPT. It reads and writes sentences. |
| **Re-ranking** | A second pass. You grab 100 roughly-good results fast, then use a smarter, slower model to put the best 10 on top. |
| **Index** | A prepared copy of your data, arranged so searching it is fast. You rebuild it when listings change. |

## Stack terms (what Rentals.ca is built with)

| Term | Plain meaning |
|---|---|
| **Stack** | The full set of tools a project is built with. Language, framework, database, and so on. |
| **PHP** | A programming language made for websites. The Rentals.ca backend uses it. |
| **Backend** | The part of the app running on the server. It holds the data and the rules. You cannot see it. |
| **Frontend** | The part running in your browser. The buttons and pages you actually see. |
| **Laravel** | The most popular PHP framework. A *framework* is a ready-made skeleton so you do not write everything from zero. |
| **Vue.js** | A JavaScript tool for building the frontend. A competitor to React. |
| **MySQL** | A common database. Stores data in tables, like a spreadsheet. |
| **PostgreSQL** | Another common database. Often called "Postgres". It has more advanced features than MySQL. |
| **AWS** | "Amazon Web Services". Amazon rents out servers and storage. Most companies run their site on it. |
| **GitLab** | A website for storing repos. Like GitHub, but a different company. |
| **CI/CD** | A robot that automatically tests your code and puts it live when you push a change. |
| **Unit test** | A small piece of code whose only job is to check that another piece of code works. |
| **TDD** | "Test Driven Development". Writing the test first, then writing the code to make it pass. |
| **Embedding model** | The model whose only job is turning text into numbers. It does not chat. |
| **Voyage AI** | A company that makes embedding models. Anthropic's docs recommend it, because Anthropic does not make one. |
| **Dimensions** | How many numbers are in one embedding. 1024 is a common size. More numbers = more detail, but more storage. |
| **Cosine similarity** | The maths for "how close are these two number lists?". Closer = more similar meaning. |
| **Nearest neighbour search** | Finding the few items closest to your item. This is the actual "search" in vector search. |
| **input_type** | A setting on the embedding call. You say whether the text is a `query` (what the renter typed) or a `document` (the listing). Getting this right improves results. |
| **Reranker** | A model that takes a query plus ~100 results and sorts them properly by relevance. Slower but smarter than vector search. |
| **Quantization** | Storing each number less precisely to save space. Can shrink your data 4x or 32x, with a small accuracy loss. |
| **Chunk** | A piece of a long text. You split long documents into chunks before embedding them. |
| **Token** | A piece of a word. Models count tokens, not characters. Roughly 1 token = 4 characters of English. |
| **RAG** | "Retrieval Augmented Generation". Search for relevant text first, then let an AI write an answer using it. |
| **Open weights** | A model you can download and run on your own server, for free. The opposite of paying an API per call. |
| **API key** | A secret password your code sends so the provider knows it is you, and knows who to bill. |
| **Backfill** | A one-time job that processes all your existing data. Example: embedding all 300,000 listings you already have. |

## Rentals.ca semantic search terms (from the internal docs)

| Term | Plain meaning |
|---|---|
| **AI search** | The existing Rentals.ca feature that reads a renter's sentence and turns parts of it into filters. |
| **Leftover words** | The part of the sentence AI search could NOT turn into a filter. Example: "sunny" and "nice view". These are what semantic search works on. |
| **Meaning index** | The stored number lists for every listing. Rebuilt every hour. |
| **Vector** | One listing's number list. "No vector" means that listing is not indexed yet. |
| **Parse** | Splitting a sentence into parts the computer can use. |
| **bge-m3** | The embedding model Rentals.ca chose. Free and open, hosted by Cloudflare. 1,024 numbers per text. |
| **Cloudflare Workers AI** | Cloudflare's service for running AI models. This is where bge-m3 runs. |
| **Django** | A website framework written in Python. The Rentals.ca website uses it. |
| **Rentals-API** | The internal service that does the ranking. |
| **Tier** | Placement level. Paid listings sit in higher tiers and always appear first. |
| **Keyword group** | Whether a listing contains the leftover words or not. Meaning is weighted differently in each group. |
| **Weight** | How much one factor counts in the final score. Weight 3 = strong. Weight 0.5 = tie-breaker only. |
| **Grade points** | The test score. 2 points per clear match, 1 per part match, counted in the top 10. |
| **Golden set** | A fixed list of test searches, re-run every time, so scores can be compared fairly between runs. |
| **Quality gate** | A pass/fail rule before shipping. Here: meaning must beat keywords by 5% or more, in both languages. |
| **Release gate** | The final check a feature must pass before going live. Needs a human, not just an AI. |
| **Develop** | The internal copy of the website, used for testing. |
| **Production** | The real, live website that renters actually use. |
| **Feature switch** | A setting that turns a feature on or off without changing code. Also called a feature flag. |
| **Negated wish** | A wish phrased backwards. Example: "no carpets", "sans tapis". The old search ranked these the wrong way round. |
| **POI** | "Point of interest". A place on a map, like a park, metro station, or school. |
| **Spot-check** | A human checking a sample of results by hand, instead of checking all of them. |
| **Commute-aware ranking** | The next planned feature. Listings closer to the metro or park the renter asked for rank higher. |
| **Fallback** | What the system does when something fails. Here: if meaning times out, rank by keywords alone. |
