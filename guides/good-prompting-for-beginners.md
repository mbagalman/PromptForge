# A Guide to Good Prompting for Beginners

*How to get better answers from ChatGPT, Claude, Gemini, and other AI chat tools. Written for people just getting started, current as of May 2026.*

---

## Introduction

If you've been using an AI chat tool — ChatGPT, Claude, Gemini, Copilot — and feeling like the results are hit-or-miss, this guide is for you. There's no secret syntax, no magic phrase that unlocks better answers. But there are a few habits that consistently produce better results, and most of them take less effort than what you're probably doing now.

A lot of "prompt engineering" advice you'll see online is either out of date, made up, or aimed at people building software with AI rather than chatting with it. This guide is just about chat — the kind of conversation you have by typing into a browser window or app.

The whole thing should take ten minutes to read.

---

## The single most important idea

**Tell the AI what you want, who it's for, and what shape the answer should take.**

That's it. That's the core of every good chat prompt. The longer version of this guide is mostly elaboration on that one sentence.

Compare these two prompts:

> *"Write something about climate change."*

> *"Help me write a 200-word explainer about climate change for my 12-year-old nephew. He likes science but gets bored by long lectures. End with one thing he can actually do."*

The second prompt isn't longer because it's more polite or more formal. It's longer because it gives the AI three things the first prompt doesn't: a length, an audience, and a goal for the ending. The output will be dramatically better, and you won't need three follow-up messages to fix it.

The rest of this guide is just specific ways to apply that idea.

---

## Five habits that make a real difference

### 1. State the audience and the goal

The single biggest improvement most people can make is telling the AI who the answer is for and what they're trying to accomplish. "Explain inflation" produces a mediocre answer. "Explain inflation to a smart friend who failed economics in college and is now worried about their grocery bill" produces a much better one.

Audience and goal don't have to be elaborate. A few words is usually enough:

- "...for a job interview tomorrow"
- "...for my parents, who aren't technical"
- "...as a starting point I'll edit later"
- "...for a quick email I need to send in five minutes"

These small additions change the tone, depth, and vocabulary of the response. They're the cheapest improvement available.

### 2. Say what shape you want the answer in

If you want a bulleted list, ask for one. If you want a table, ask for a table. If you want three options ranked by feasibility, ask for that exactly. If you want a single paragraph, say so.

The AI will pick *some* format if you don't specify, and it's usually one of two defaults: a long structured response with headers and bullets (often more formatting than you wanted) or a conversational paragraph (often less structured than you wanted). Telling it the shape you actually want saves a round trip.

Useful phrases:

- "Give me three options, ranked from safest to riskiest."
- "Answer in one paragraph, no bullet points."
- "Format as a table with columns for X, Y, and Z."
- "Keep it under 100 words."

### 3. Front-load anything that's non-negotiable

If a constraint is critical — a length limit, a forbidden topic, a tone requirement — put it at the start of the prompt, not at the end. "Write me a cover letter, but please keep it under 250 words" works less reliably than "I need a cover letter under 250 words. Here's the context..."

AI models give more weight to instructions near the start. This isn't a rule you have to follow religiously, but for the one or two things that really matter, putting them up front is a free improvement.

### 4. Tell the AI what to do if it doesn't know

This is the most underused trick in chat prompting. AI models, by default, will try to give you *some* answer to almost any question, even when they shouldn't. They make things up. The cure is simple: tell them what to do when they're uncertain.

Examples:

- "If you're not sure about a specific fact, say so rather than guessing."
- "If you need information I haven't given you, ask me before answering."
- "If this question is outside what you can answer reliably, tell me."

Adding a sentence like this to a fact-heavy prompt cuts confident-sounding wrong answers significantly. It also makes the AI more useful when the right answer is "I'd need to check that" — because now it's allowed to say so.

### 5. Show, don't just tell, when style matters

If you want the AI to match a specific style — your writing voice, a particular tone, a brand's house style — give it an example. One paragraph of "here's how I usually write" does more than five sentences of "I write in a casual, friendly, conversational tone with occasional dry humor."

This is especially useful for:

- Emails that should sound like you wrote them
- Marketing copy that should match a brand
- Translations that should match a register (formal, casual, technical)
- Anything where "you'll know it when you see it" is the only honest description

One or two examples is usually enough. More than three starts to anchor the AI on incidental features of the examples rather than the pattern you wanted.

---

## What's changed since 2023

If you learned about prompting a few years ago, two things have changed enough to be worth flagging.

**"Let's think step by step" is no longer needed.** The classic 2023 trick of asking the AI to "think step by step" or "show your reasoning" used to meaningfully improve answers on hard problems. Modern AI models (Claude 4.x, GPT-5, Gemini 2.5 and 3) already reason internally before answering. Adding the phrase usually doesn't help and can occasionally make answers worse by producing long, performative "thinking" you didn't want. If you want the AI to think carefully, just say "this is important — take your time and get it right."

**Elaborate role-playing prompts are no longer needed either.** You don't need "You are a world-class expert with 20 years of experience in..." A simple "help me with..." works just as well. Defining a role still helps when you actually want to constrain the answer to a specific perspective ("answer as a tax accountant would"), but the elaborate setup is mostly leftover from older models.

---

## Things that still trip people up

A few common patterns that consistently produce worse results.

**Stacking constraints over multiple messages.** When the AI gets something wrong, the instinct is to send a follow-up like "no, make it shorter," then "also more formal," then "also no bullet points." After five messages the AI is juggling contradictory instructions and producing worse results than if you'd started over with one good prompt. If you're more than two follow-ups in, consider starting a new conversation with a better prompt.

**Asking for too many things at once.** "Write me a business plan, a marketing strategy, a financial projection, and a pitch deck" is going to produce four mediocre things. Asking for them one at a time, with context carried forward, produces better results in roughly the same total time.

**Trusting facts without checking.** Modern AI is impressively accurate on common facts and impressively wrong on specific ones — dates, statistics, names of obscure people, recent events, legal details, medical specifics. Treat any factual claim that matters as "probably right, worth verifying." This is especially true for anything time-sensitive (the AI's training data has a cutoff date) or anything you'd be embarrassed to be wrong about.

**Sharing things you wouldn't share publicly.** Most chat AI services use conversations to improve their products by default. Even when they don't, conversations are stored on company servers and can be subject to subpoenas, breaches, or policy changes. Don't paste in passwords, full credit card numbers, your home address with other identifying details, or sensitive personal information about other people.

**Asking for opinions and treating them as facts.** When you ask "is X a good idea?" the AI will usually give you a confident-sounding answer. That answer reflects patterns in its training data, not independent judgment. For important decisions, the AI is useful as a brainstorming partner and unreliable as an oracle.

---

## A simple template that works for most things

If you want a starting structure for harder requests, this one works across topics:

> **What I'm trying to do:** [one sentence]
>
> **Who it's for / context:** [one or two sentences]
>
> **What I want from you:** [the actual request]
>
> **Format:** [length, shape, any constraints]
>
> **What to do if you're stuck:** [ask me, flag uncertainty, etc.]

You don't need to use the labels — just covering those five things in any order works fine. For simple requests, skip it entirely. The template is for the times when you've gotten a disappointing answer and want to try again with more structure.

A worked example:

> *I'm trying to send my landlord a polite but firm email about a maintenance issue that's been ignored for three weeks. The relationship has been fine until now, so I don't want to escalate too hard, but I do want a response. Write me a draft I can edit. Keep it under 150 words, neutral tone, no threats or legal language. If you need details I haven't given you (like dates), use placeholders in brackets.*

This is a complete specification of a small task. No mystery about what good output looks like.

---

## When to start a new conversation

A practical question that doesn't get enough attention: how long should a chat conversation get before you start a new one?

A few signs it's time to start fresh:

- You've corrected the AI multiple times and it keeps making the same mistake.
- The conversation has drifted across several unrelated topics.
- You're getting answers that seem to be "remembering" earlier context in confused ways.
- The AI is producing repetitive or formulaic responses.

Long conversations accumulate context, and that context can start working against you. A fresh conversation with a well-written prompt usually beats a 30-message conversation that's gone off the rails.

---

## What this guide doesn't cover

This is the simplified version. A few things are deliberately left out:

- **Building custom assistants** (ChatGPT's Custom GPTs, Claude Projects, Gemini Gems). These are saved instruction sets you can share — useful, but a different kind of writing.
- **Working with AI coding agents** (Claude Code, Cursor, Copilot Workspace). Different rules apply when the AI is actually executing tasks rather than answering questions.
- **API usage and developer-focused prompting.** If you're writing code that talks to an AI, the considerations are different.
- **Platform-specific quirks.** Claude, ChatGPT, and Gemini have small differences in how they respond to certain kinds of prompts, but at the beginner level these don't matter much.

If you want to go deeper, the full version of this guide — covering all of the above — is in the same repo: [prompting-best-practices-2026.md](./prompting-best-practices-2026.md).

---

## The short version

If you skim everything else, take these five things:

1. Tell the AI who the answer is for and what you're trying to accomplish.
2. Say what shape you want the answer in.
3. Put critical constraints at the start, not the end.
4. Tell the AI what to do when it doesn't know.
5. Show an example when style matters.

That's most of good prompting. Everything else is refinement.

---

*Last updated: May 2026. AI tools change frequently; specific product features should be verified against current documentation.*

---

*Compiled and maintained by [Michael Bagalman](https://michaelbagalman.com/). For the philosophy behind this work, see [michaelbagalman.com/philosophy.html](https://michaelbagalman.com/philosophy.html).*
