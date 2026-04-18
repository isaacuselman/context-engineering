# Exercise 02 — Context window budget

## Why this matters

Context is a finite, non-renewable resource within a single session. Attention does not scale linearly with tokens — transformer attention has n² pairwise relationships, training skews toward shorter sequences, and recall accuracy drops as the window fills. Anthropic calls this *context rot*. The practical consequence: past a certain point, adding more information to a session makes Claude worse, not better.

This exercise asks you to measure, audit, and shrink the context of a realistic Phase 0 working session. The payoff is a calibrated intuition for "how much am I already spending, and on what?"

## Read

- `docs/source-material.md` §2.4 (Anthropic — "Effective Context Engineering for AI Agents")
- `docs/source-material.md` §1.6 (the `@path/to/file.md` import / progressive disclosure pattern)
- `docs/source-material.md` §4.1 (Cherny on `/clear` and long-session hygiene)

Primary sources:

- <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>
- <https://research.trychroma.com/context-rot>

## Concepts

**Attention budget.** "LLMs have an 'attention budget' that they draw on when parsing large volumes of context." Treat tokens the way you'd treat working memory in a human: finite, not a free dumping ground.

**Context rot.** As context grows, recall accuracy drops. The architectural cause (quadratic attention + shorter training distributions) is not going away. Design sessions assuming the last 100k tokens matter less than the first 20k.

**The three long-horizon primitives** (Anthropic):

1. **Compaction** — summarize history and re-initialize. Claude Code's `/compact` preserves "architectural decisions, unresolved bugs, and implementation details while discarding redundant tool outputs" and keeps "the five most recently accessed files."
2. **Structured note-taking** — write a scratchpad file Claude updates across turns (the "Claude Plays Pokémon" pattern: *"for the last 1,234 steps I've been training my Pokémon in Route 1, Pikachu has gained 8 levels toward the target of 10"*).
3. **Sub-agent architectures** — each subagent can burn tens of thousands of tokens exploring and return 1,000–2,000 tokens of distilled summary.

**Progressive disclosure via `@imports`.** A lean `CLAUDE.md` skeleton points at deeper docs: `@agent_docs/building_the_project.md`, `@agent_docs/running_tests.md`. Each is indexed with a one-line description. The detail is fetched only when relevant.

**`/clear` is cheap.** Cherny's advice from the best-practices post: *"During long sessions, Claude's context window can fill with irrelevant conversation, file contents, and commands. Use the `/clear` command frequently between tasks to reset the context window."*

**Rough token math.** 1 token ≈ 4 characters ≈ 0.75 words for English prose. This is noisier for code and non-English text, but good enough for the back-of-envelope work in this exercise. Cherny's self-reported CLAUDE.md: ~100 lines, ~2,500 tokens — that's roughly 25 tokens per line.

## Exercise

**Audit the context budget of a realistic Phase 0 working session, then rewrite one component to make it leaner.**

### Step 1 — Inventory

Imagine (or stage) a session where you ask Claude Code to "add a `#/progress` route that shows a per-exercise bar chart." Before any turn happens, the session context already contains:

| Source | What it is | Rough tokens |
|---|---|---|
| Claude Code system prompt | ~50 rules, injected always | ~3,000 |
| Your project `CLAUDE.md` | the one you wrote in Exercise 01 | measure |
| Working dir listing | paths of `phase-0/` | ~200 |
| Your initial user prompt | "add a progress route…" | measure |
| Reference files you'd @-mention | e.g. `index.html` | measure |

For each row marked *measure*, compute a token estimate by counting characters and dividing by 4. Record the totals in your notes.

### Step 2 — Keep / evict / defer

Classify every item in your inventory into one of three buckets:

- **Keep** in context always (the line carries weight every turn)
- **Evict** — move out of context; it shouldn't have been there in the first place
- **Defer** — don't load upfront, but make it reachable on demand (via `@path` import, file read, or subagent call)

At minimum, find **two keeps** and **two evicts-or-defers**. Justify each in one sentence. If everything looks like a *keep*, you are under-scrutinizing.

### Step 3 — Refactor

Pick the largest *defer* candidate and refactor it:

- If it's CLAUDE.md bloat — extract a section to `agent_docs/{topic}.md` and replace the section with a one-line `@agent_docs/{topic}.md` pointer.
- If it's a file you'd always @-mention — remove the auto-mention; let the agent discover it via Glob or a file read when the task actually needs it.
- If it's historical chat bloat — note where you would have called `/clear` and why.

Write the before/after byte counts (or line counts) in your reflection.

### Step 4 — Plan the compaction moment

Imagine the session ran for 2 hours and filled ~120k tokens. Write two bullets on when you'd compact and two bullets on what must survive compaction (e.g. the `#/progress` route decision, the failing test you haven't fixed, which exercise file you're midway through editing).

## Reflection prompts

1. What did your *evict* pile include that you would have called "obviously relevant" before this exercise?
2. Did your refactored CLAUDE.md go under the 150–200 instruction budget (HumanLayer's ceiling)? If you had to guess the number of instructions in it, how would you count?
3. Next time you start a session, what is the first context-hygiene action you'll take? (Good answers: `/clear` between unrelated tasks; a TODO-list scratchpad file; a tight initial prompt instead of pasting a whole issue.)
