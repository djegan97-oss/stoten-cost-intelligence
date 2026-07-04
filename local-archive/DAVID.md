# About David Egan

## Background
Managing Director and Portfolio Manager at Stockbridge Capital Group.
Co-manages the Niche Logistics Fund (NLF) with Matt Jerry. Background
in institutional real estate investment management. Not a software
developer — evaluate technical decisions with that in mind and always
explain the "why" behind architectural choices, not just the "what."

## How to Work With Me
- I am not a developer. Do not use shortcuts, abbreviations, or
  assume I know what a term means. When in doubt, explain it.
- It is always better to over-explain than under-explain. If something
  could use more context, include it.
- Break explanations into chunks. If something has multiple steps or
  concepts, walk through them one at a time rather than combining
  them into a dense paragraph.
- Always use plan mode when starting a new feature or a significant
  piece of work. Use the ask user question tool to confirm you
  understand what I want before writing any code.
- I move fast once I understand what's happening. Get me to
  understanding first, then move quickly.
- If something is going to take more than one step, tell me upfront
  so I know what to expect.
- If you hit a decision point where there are two reasonable options,
  give me your recommendation and a one-line reason — don't just
  list the options and ask me to choose.
- I prefer clear confirmation that something works over a long
  summary of what you just did. Show me it works.

## Communication Style
- Plain language. No jargon without explanation.
- If you use a technical term, define it in the same sentence or
  the next one. Never assume I know what it means.
- I will sometimes be brief or terse — that's not frustration, just
  efficiency. But always give me the full explanation I need.
- If something isn't working and I say "this isn't working," start
  debugging immediately. Begin with the most likely cause and walk
  me through what you're checking and why.

## Domain Knowledge
- Deep expertise in IOS and industrial real estate. You do not need
  to explain what IOS is, what a truck terminal is, or how cap rates
  work. Treat me as an expert in the subject matter.
- Not familiar with software architecture, DevOps, or front-end
  development. Always explain these things from first principles.
  Never assume I know what a library, a route, a hook, or a build
  process is without explaining it.
- I think in terms of: what does the team actually need to do their
  job better? Keep that lens on everything we build.

## Starting Any New Feature
1. Enter plan mode first.
2. Use the ask user question tool to confirm you understand
   the goal, who will use it, and what success looks like.
3. Explain your plan in plain English before writing any code —
   what you're going to build, why, and what order you'll do it in.
4. Only then start building.

## Priorities for This Project
1. Data quality and accuracy above all else — a wrong comp is worse
   than a missing comp.
2. The team should be able to use the front end without any training.
3. Speed of getting to a working system matters — we have real data
   to ingest and real decisions to support.

## Things That Slow Me Down
- Technical shortcuts or assumed knowledge.
- Long blocks of unexplained jargon.
- Being asked to choose between options when one is clearly better —
  make the recommendation.
- Not knowing what step we're on or what comes next.

## Things David Has Caught That Improved the System
- Moved all math out of GPT into deterministic code
  (caught before scale caused silent errors)
- Identified that 50-record batch was too small for
  bulk loading (led to drain mode)
- Correctly identified Yard Dogs URL as ios-yarddogs.com
- Insisted on two equal upload paths on Add Comp page
  rather than upload buried at bottom
- Flagged that Fast Path should skip GPT for structured
  data (led to significant performance improvement)
- Caught that price_per_acre outliers could mean acreage
  is wrong, not just price (led to Rule 11 in Smart Review)
- Pushed for GPT judgment layer in Smart Review rather
  than purely deterministic rules

These are good instincts. When David flags something
that feels wrong about an approach, take it seriously
and investigate before defending the original design.
