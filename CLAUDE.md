# Working with me

## Who I am

I am a junior developer. I recently joined the team that builds **Rentals.ca**.
Explain things to me like I am 15 years old and new to coding.
I do not know many tech or business terms yet.
Dense, compressed English is hard for me to read. Keep it simple.

## How to write to me

Follow these rules in every reply:

1. Use plain English. Use short sentences.
2. Put one idea in one sentence. Do not pack three ideas into one line.
3. Explain things step by step, in the order they happen.
4. Give a real example after any idea that is abstract.
5. Say what a thing *is* first. Then say why it matters.
6. Avoid long sentences full of commas and dashes.
7. Avoid clever or fancy words. Pick the boring, clear word instead.
8. Use short lists and small tables. They are easier to read than paragraphs.

## Always explain the terms

This is important to me.

- Every time you use a tech term or a business term, explain it in simple words.
- Do it right there, in the same place. Do not make me go look it up.
- A short form is fine. Example: "Cloudflare (a service that blocks bots)".
- At the end of a long reply, add a small **Terms** table listing the new words.
- Also add every new term to `GLOSSARY.md` in this project, so it builds up over time.

## Example of bad and good

Bad. This is too dense:

> The entire site sits behind a Cloudflare managed challenge, so server-side HTTP
> scraping is a dead end and any sync built on requests will fail on request one,
> not at scale.

Good. This is plain and step by step:

> The website is protected by a service called Cloudflare.
> Cloudflare's job is to block bots.
> It checks whether a real browser is asking for the page.
> Our Python script is not a real browser. So Cloudflare blocks it.
> This means even the first request fails.
> We cannot fix this by slowing the script down.

## Detail level hotkeys

I will type one letter to tell you how much detail I want.
**The letter controls your writing AND your code.**

### When you write to me

| Letter | What I want |
|---|---|
| `C` | Very concise and simple. Just the main points. |
| `M` | More than `C`. More content and a few examples. |
| `L` | Full detail. Explain everything, with many examples. |

### When you write code

| Letter | What I want |
|---|---|
| `C` | The smallest code that works. Few comments. No extra features. Just show me the changed part. |
| `M` | Working code. Comments on the tricky lines. Basic error handling. Show me how to run it. |
| `L` | Full code. Comments on every block. Type hints. Error handling. Tests. A line-by-line walkthrough after the code. |

### Default

If I do not type a letter, use `C`.

I may type the letter by itself. Or I may type it with a question.
Example: `L why is Cloudflare blocking us?`

## About code style

The rules above change how you *talk* to me.
They do not change the project's code style.
Always match the style of the code already in this project.
