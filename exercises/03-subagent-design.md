# Exercise 03 — Subagent design

## Why this matters

A subagent in Claude Code is a scoped second process with its own system prompt, its own tool allowlist, and its own model choice. Spawned well, it absorbs 30k+ tokens of exploration and hands back a 1–2k-token summary, sparing the main thread. Spawned badly — with every tool granted, vague description, wrong model — it either burns context, makes the wrong decisions, or never gets auto-delegated to because its description is too generic.

Design is mostly about constraints, not capabilities: *what can it not do, and why is that deliberate?*

## Read

- `docs/source-material.md` §3.1 (official frontmatter schema — the whole table)
- `docs/source-material.md` §3.2–3.5 (VoltAgent code-reviewer, security-auditor, test-automator; Nuttall's PRD writer)
- `docs/source-material.md` §3.6 (PubNub synthesis — single goal, tight tools, model tiering)
- `docs/source-material.md` §2.3 (Cognition's "share full traces" tension — know what you're giving up when you split work)

Primary sources:

- <https://docs.claude.com/en/docs/claude-code/sub-agents>
- <https://github.com/VoltAgent/awesome-claude-code-subagents>
- <https://cognition.ai/blog/dont-build-multi-agents>

## Concepts

**Frontmatter is the API.** `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`. The description is the basis for auto-delegation — write *when to use this*, not just *what it does*. "Reviews code" does not match a user task; "Use this agent when you need a security-focused review before merging a PR" does.

**Tools are an allowlist, not a default.** Omit `tools` and the subagent inherits everything — including any destructive action. Scope tightly by role:

| Role | Typical tools | Model | Why |
|---|---|---|---|
| Reviewer / auditor | `Read, Grep, Glob` | `opus` | Analytical; no mutations |
| Researcher | `Read, Grep, Glob, WebFetch` | `haiku` or inherit | Fast read-only exploration |
| Implementer | `Read, Write, Edit, Bash, Grep` | `sonnet` | Executes changes |
| Spec writer | `Read, Write, Grep, Glob` | `opus` | Writes one artifact |

The VoltAgent code-reviewer is an instructive counterexample: it grants `Write, Edit, Bash`. Defensible if you want it to *apply* fixes, but the stricter Anthropic minimal example (`Read, Glob, Grep` only) is safer — it forces suggestions back into the main thread for review.

**Model tiering matters.** `opus` for analytical roles where judgment is the product. `sonnet` for roles where throughput matters. `haiku` for fast narrow exploration. `inherit` to match whatever the parent is running.

**Cognition's two principles** (§2.3) are the counterweight. Split work → subagents don't share context → decisions diverge → the final result has two miscommunications baked in. The mitigation is to use subagents for *research and collection* ("provide a small amount of summary back" — Adam Wolf) rather than for parallel decision-making. Reviewer, auditor, researcher subagents are safe. Parallel implementers are where trouble starts.

**Subagents cannot spawn subagents.** The main thread is the only place that does orchestration. Don't try to build a tree.

## Exercise

**Write a complete subagent definition — frontmatter plus system prompt body — for one of these roles, then justify every design choice.**

Pick one:

- `curriculum-reviewer` — reads a Phase 0 exercise file and flags where claims aren't grounded in `docs/source-material.md`
- `exercise-scaffolder` — given a title and a source-material section, produces the six-header exercise skeleton (`## Why this matters` through `## Reflection prompts`)
- `link-auditor` — walks every exercise file, extracts external links, and reports any that are dead or redirect to a different domain
- `localstorage-sanity-checker` — a browser-automation subagent that opens `http://localhost:8000/`, clicks through a representative user journey, and verifies localStorage keys are written correctly

### Template

```markdown
---
name: your-kebab-case-name
description: "Use this agent when … (one concrete triggering situation). Include an <example> block if the trigger is subtle."
tools: Tool1, Tool2, Tool3
model: opus | sonnet | haiku | inherit
---

You are a <role>. Your job is to <one sentence>.

## When you're invoked
…

## Your workflow
1. …
2. …
3. …

## What you return
A single artifact: <file path or structured output>. No side effects beyond that.

## Definition of done
- [ ] …
- [ ] …
```

### Constraints

- `description` must be concrete enough that Claude Code can auto-delegate. If "I'd pick this subagent for task X" is ambiguous, the description is too vague.
- `tools` must be the *minimum* sufficient set. If `Write` or `Bash` is in the list, write one sentence justifying it in your notes.
- Exactly one model choice, with a one-sentence justification.
- The body must fit on one printed page (~300 lines is too long; 30–80 is realistic).
- End with an explicit "definition of done" checklist — the agent should know when to stop.

### Critique pass

After writing it, answer:

1. Does the description trigger on the *right* tasks and *not* on adjacent ones? (Auto-delegation is pattern-matching on the description.)
2. Is there any tool in `tools` you could remove and still have the subagent work?
3. Could this subagent be replaced by a single well-written user prompt in the main thread? If yes, it probably should be.
4. What context does the *main thread* need to pass in when spawning this subagent? Is that passing easy or awkward?

## Reflection prompts

1. Which constraint (description specificity, tool scoping, model, body length) was hardest to get right?
2. Where did Cognition's "don't split decision-making" principle bite — i.e., what were you tempted to have the subagent decide that should really come back to the main thread as a question?
3. If you had to pick one subagent to actually install in `~/.claude/agents/` and use this week, which is it and why?
