# Exercise 06 — Capstone

## Why this matters

The capstone is not a new topic. It is a **retrospective**: what you just did for five exercises you also did (if you built this platform yourself with Claude Code) while shipping the platform that's rendering this markdown right now.

The goal is to convert what you've practiced into something durable:

1. A calibrated sense of which source-material claims held up under contact with real work, and which ones were more aspirational than operational.
2. A short list of changes you'd make to your own CLAUDE.md, subagent shelf, and default workflow.
3. A read on what Phase 1 should actually contain — rather than guessing up front.

## What to read (or re-read)

- `docs/source-material.md` §1.4 (Cherny's self-improvement loop — the point of this exercise is to run one pass of it on yourself)
- `docs/source-material.md` §2.5 (Goedecke on tractability: *"Once two or three of [bad decisions] pile up, the app is no longer tractable for the agent"*) — watch for the moments where this almost happened to you
- `docs/source-material.md` §2.6 (METR's 19% slowdown result — an honest counterweight to your own self-assessment of speedup)
- `docs/source-material.md` §4.1 (the explore/plan/code/commit arc — were your real sessions shaped like this?)

## The exercise: structured retrospective

Answer each prompt in the textarea. Aim for one tight paragraph per prompt — four to six sentences. If you find yourself writing more than that, either the claim is big enough to deserve its own CLAUDE.md line, or you're generalizing too early.

### Section A — source-material audit

**A1.** Pick the source-material claim that matched your experience most cleanly. Quote or cite it, and describe the specific session where it showed up. (Example: *"Anthropic's 'bloated tool sets' warning — I gave the reviewer subagent `Write` and `Edit` thinking it would be useful, and it applied a fix I didn't want."*)

**A2.** Pick a claim that **didn't** match your experience, or that you weren't able to test. Which is it, and what would you need in order to check it? (This is how you keep your reading honest.)

**A3.** The gaps section of `docs/source-material.md` flags several numbers that can't be sourced cleanly (the apocryphal "33% planning uplift", Augment Code's "19% vs 87%", etc.). Did you catch yourself citing or silently believing any of these during the exercises? Which one, and how will you re-ground next time?

### Section B — changes to your own setup

**B4.** Your `CLAUDE.md` from Exercise 01 — what's one line you'd add now that you've been through the subagent and spec exercises? What's one line you'd cut?

**B5.** Your subagent from Exercise 03 — did you actually install it? If yes, what did it do well or badly in use? If no, what stopped you?

**B6.** Your spec and task list from Exercises 04–05 — if you implemented the feature, how many of the FRs survived contact with the code unchanged? Where did the spec need to be revised mid-implementation, and what does that tell you about the spec?

### Section C — context engineering in practice

**C7.** Describe the single moment in this whole arc when your **context went bad** — context rot, wrong CLAUDE.md line firing, subagent returning too much, stale file in the working set, whatever it was. Be specific. This is the incident you should actually remember.

**C8.** What's the *one* piece of context hygiene you will now do by default — automatically, without thinking about it, on every new Claude Code session? (Good answers are concrete: "I'll `/clear` between unrelated tasks," "I'll run Explore before Plan on anything >2 files," "I'll keep my CLAUDE.md under 60 lines.")

### Section D — Phase 1

**D9.** Based on what was thin in Phase 0, what should Phase 1 cover? Propose three exercise topics, each in one sentence. Candidates to consider:

- Hooks (`PreToolUse`, `SubagentStop`, `Stop`) — the deterministic half of the harness
- Plan mode, Explore mode, and the Agent tool — when to use which
- Evaluating a CLAUDE.md quantitatively (instruction-count, session drift)
- Multi-repo / monorepo CLAUDE.md strategy (nested files, `@imports`)
- Prompt injection and agent security (Goedecke §2.5)
- Writing your own MCP server
- Benchmarking your own agent throughput against METR's time-horizon numbers

**D10.** What is the *smallest possible Phase 1*? If you only had time for one of those topics, which would it be and why?

## Final artifact

When you're done, mark this exercise complete. Your notes on this page — together with your updated `CLAUDE.md`, your subagent file, your spec, and your task list — are the Phase 0 deliverable.

There is no further exercise. The *platform* that rendered this page is the capstone artifact: you (or past-you) wrote its CLAUDE.md, decomposed its build, and reviewed Claude's output while building it. You are now looking at your own work, from the inside.

## Reflection prompts

*(These are lighter than the structured sections above — use them only if you want to keep writing.)*

1. If you had to teach someone else Phase 0 in 30 minutes, what's the one thing you'd make sure they walked out with?
2. What's the claim in the Phase 0 curriculum you now *don't* believe, after actually doing the work?
3. Which exercise felt most like "fake homework" versus most like "this will change how I work tomorrow"? The honest answer is the one worth writing down.
