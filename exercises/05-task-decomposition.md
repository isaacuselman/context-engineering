# Exercise 05 — Task decomposition

## Why this matters

A spec tells you *what* to build; a task list tells you *how to eat the spec one bite at a time.* The mistake most people make is either:

- Too coarse — one task says "build the dashboard," the agent picks the wrong abstraction, and you spend an hour trying to salvage it.
- Too fine — twenty tasks for twenty single-line edits, each with its own context-load and coordination cost, and the project takes longer than just doing it yourself.

Cognition's *Don't Build Multi-Agents* gives the canonical failure mode: decompose "Build Flappy Bird" into Subtask 1 (background with green pipes) and Subtask 2 (controllable bird). Subagent 1 misreads and builds a Mario-style background; Subagent 2 builds a non-game-asset bird. Neither can see the other's work. The final agent inherits two miscommunications.

The pattern that works: tasks at the **smallest coherent unit that preserves architectural clarity** (Eclipse "Task Engineering"), each with **unambiguous pass/fail criteria** (Osmani), executed **in sequence with review between them** (Anthropic's explore/plan/code/commit).

## Read

- `docs/source-material.md` §2.3 (Cognition — the Flappy Bird lesson; share full traces, not subtask prompts)
- `docs/source-material.md` §4.4 (snarktank + Osmani — atomic tasks with acceptance criteria)
- `docs/source-material.md` §4.5 (task granularity; Eclipse "Task Engineering"; the 3–8 parallel sweet spot)
- `docs/source-material.md` §4.6 (Cognition's prescription for long tasks: single-threaded with context compression)
- `docs/source-material.md` §4.1 (Cherny on markdown-checklist scratchpads — the Ralph Wiggum pattern in practice)

## Concepts

**Atomic task** (Osmani, verbatim):

> "Each task should be small enough to fit in one AI session and have unambiguous pass/fail criteria. For example, instead of 'Build the entire dashboard' you'd have tasks like 'Add a navigation bar with links to Home, About, Contact' with acceptance criteria specifying exact expectations (e.g. 'the current page link is highlighted in blue')."

**The smallest coherent unit** (Eclipse): *"Aim for the smallest coherent unit that preserves architectural clarity. Over-dividing fragments your mental model and brings down overall efficiency."* — the corrective to "more tiny tasks is always better."

**Context friction is a signal** (Eclipse): *"If it's hard to collect or provide the concise, focused context for a task, adjust the granularity or design of the task."* — if a task needs you to explain half the codebase, it's the wrong task.

**Parallel sweet spot** (Claw101): 3–8 parallel tasks, each 3–10 minutes. Twenty parallel one-line edits create more coordination overhead than they save. Keep in mind Cognition's rule: parallel is safe for read-only investigation, risky for parallel decision-making.

**The Ralph Wiggum loop** (Huntley/Carson): a loop where an agent reads `tasks.json`, picks the next task, edits, runs tests, marks it done, and repeats. Works only if the tasks are genuinely atomic and the acceptance criteria are machine-checkable.

**The subpar-plan warning** (*From Plan to Action*, SWE-bench):

> "A subpar plan hurts performance even more than no plan at all."

Implication: spending extra time to make the task list *right* has convex returns.

## Exercise

**Take the spec you wrote in Exercise 04 and decompose it into a `tasks.md` file that a Ralph Wiggum loop could execute.**

If you didn't write Exercise 04 yet, do that first.

### Template

```markdown
# Tasks: [Feature Name]
Source spec: `specs/001-your-feature/spec.md`

## Dependencies
- [ ] External library updates / migrations needed first
- [ ] Prerequisite refactors (do these before T001)

## Tasks

### T001 — [short imperative title, <10 words]
**What:** one-sentence description of the change.
**Files:** `path/to/file.html`, `path/to/other.md`.
**Context needed:** the FR(s) and acceptance scenarios this task addresses.
**Done when:** one sentence stating the pass/fail condition — ideally something you could verify in 30 seconds in a browser or one test command.

### T002 — …
…
```

### Constraints

- **3 to 8 tasks.** If you have 2, the tasks are too coarse. If you have 15, you are either micro-slicing or the spec was too broad.
- **Every task must reference at least one FR** from the spec. If a task has no FR, either the task is waste or the spec is missing a requirement.
- **"Done when" is one sentence** and it must be testable without loading the codebase into your head. "Done when the sidebar shows `N/6` above the exercise list and `N` equals `localStorage.getItem('completed-1') === 'true' + ...` for all six keys" is testable. "Done when progress looks right" is not.
- **Order matters.** T001 must be possible to do first. T004 must be possible once T001–T003 are done.
- **Mark any `[PARALLEL-OK]` tasks explicitly.** If two tasks touch different files and have no dependency, annotate them. Otherwise assume sequential.

### Calibration pass

Read your task list and answer:

1. **Could a junior engineer who has never seen this repo execute task T001 with only your `tasks.md` + `spec.md` + the repo itself?** If no, add context.
2. **Is there a task whose "Done when" is vague?** Rewrite it. (The fastest fix is usually "Done when `<expected visible thing>` appears at `<expected location>`.")
3. **Did any task end up doing two things?** Split it. The seam is usually where your "Done when" sentence has an "and" in it.
4. **Did any task end up being a 5-minute job that could just live as a `// TODO` comment inside a larger task?** Merge it.
5. **Harper Reed's iterate-on-size prompt:** do one more pass asking whether each step is "small enough to be implemented safely with strong testing, but big enough to move the project forward."

### Dry-run

Walk through the list top to bottom, narrating out loud (or in your notes) what you would do for each task. If you get lost, the decomposition is wrong.

## Reflection prompts

1. How many revision passes did the task list go through before you were satisfied? (Two to four is typical. Zero means you didn't calibrate.)
2. Where did the Flappy Bird failure mode nearly bite you — what decision were you tempted to split across two tasks that really needed to live in one?
3. If you actually ran this plan in a Ralph-Wiggum-style loop, which task's "Done when" is the one most likely to be incorrectly self-reported as passing? How would you harden its check?
