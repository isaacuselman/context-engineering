# Exercise 04 — Spec writing

## Why this matters

GitHub's Spec Kit frames the shift bluntly: *"We're moving from 'code is the source of truth' to 'intent is the source of truth.' With AI the specification becomes the source of truth and determines what gets built."*

A spec is not a design doc or a list of user stories; it is the artifact that survives when the code gets regenerated. The most rigorous coding-agent study on this — *From Plan to Action* (arXiv 2604.12147), 16,991 SWE-bench trajectories — reports that "providing the standard plan improves issue resolution" *and* that "a subpar plan hurts performance even more than no plan at all." The cost of a bad spec is higher than the cost of no spec.

This exercise asks you to write a Spec-Kit-style `spec.md` for a feature that was explicitly excluded from the Phase 0 MVP, so you have to decide *what the feature actually is* before any code gets written.

## Read

- `docs/source-material.md` §4.2 (GitHub Spec Kit templates, verbatim) — this is the single most important section for this exercise
- `docs/source-material.md` §4.1 (Anthropic's explore/plan/code/commit workflow; "think" < "think hard" < "think harder" < "ultrathink")
- `docs/source-material.md` §4.3 (Harper Reed's iterate-on-task-size prompt)
- `docs/source-material.md` §4.7 (caveat: no rigorous published PRD-vs-user-story-vs-technical-spec comparison exists)

Primary sources:

- <https://github.com/github/spec-kit>
- <https://harper.blog/2025/02/16/my-llm-codegen-workflow-atm/>

## Concepts

**Four phases, four artifacts** (Spec Kit):

1. `/speckit.constitution` → `memory/constitution.md` — non-negotiable project principles
2. `/speckit.specify` → `spec.md` — **what and why, no tech stack**
3. `/speckit.plan` → `plan.md`, `research.md`, `data-model.md`, `contracts/` — **how, with tech stack**
4. `/speckit.tasks` → `tasks.md` — atomic executable units

The separation is load-bearing. Mixing WHAT with HOW is the most common failure.

**FR-### language.** "System MUST [specific capability]." The MUST is not optional. "FR-001: System MUST persist per-exercise completion state across page reloads" is checkable. "Should remember progress" is not.

**Max 3 `[NEEDS CLARIFICATION]` markers** per spec. If you have more, you do not understand the feature well enough to spec it. Stop and go ask.

**The content quality checklist** (verbatim from Spec Kit):

```
## Content Quality
- [ ] No implementation details (languages, frameworks, APIs)
- [ ] Focused on user value and business needs
- [ ] Written for non-technical stakeholders
- [ ] All mandatory sections completed

## Requirement Completeness
- [ ] No [NEEDS CLARIFICATION] markers remain
- [ ] Requirements are testable and unambiguous
```

**Harper Reed's iteration prompt** — the sentence to say when a spec or plan feels too coarse or too fine:

> "Look at these chunks and then go another round to break it into small steps. Review the results and make sure that the steps are small enough to be implemented safely with strong testing, but big enough to move the project forward. Iterate until you feel that the steps are right sized."

## Exercise

**Write a Spec-Kit-style `spec.md` for one feature that the Phase 0 MVP explicitly excluded.**

Pick one (all are real rejects from the MVP):

- **A. Progress bar in the sidebar.** A visual indicator at the top of `aside` showing `N/6 completed`.
- **B. Answer-key collapsibles inside exercises.** Each exercise can optionally include a `<details>` block showing a worked example, collapsed by default.
- **C. `manifest.json` replaces the hardcoded `EXERCISES` array.** The app fetches the manifest at startup instead of relying on an in-line JS constant.
- **D. Import/export progress as JSON.** User can back up their localStorage state to a file and restore it.

### Template — save as `specs/001-your-feature/spec.md`

```markdown
# Feature Specification: [FEATURE NAME]

**Feature Branch**: 001-your-feature-slug
**Created**: YYYY-MM-DD
**Status**: Draft
**Input**: User description: "what the user literally asked for, in one sentence"

## User Scenarios & Testing

### Primary User Story
As a <user>, I want <capability> so that <benefit>.

### Acceptance Scenarios
1. Given <precondition>, when <action>, then <observable outcome>.
2. …

### Edge Cases
- <the localStorage is full / cleared / corrupt case>
- <the file not found / malformed / partial case>
- <the no-JavaScript / Safari Private / old browser case>

## Requirements

### Functional Requirements
- FR-001: System MUST …
- FR-002: System MUST …
- FR-003: System MUST …
(aim for 4–8 FRs; more is probably HOW leaking in)

### Key Entities (if data involved)
- <Entity name>: <what it represents, not its schema>

## Success Criteria
- <measurable outcome #1>
- <measurable outcome #2>

## Out of Scope
- <feature adjacent to this one that you explicitly are not building>
- <related feature you thought of but are deferring>

[NEEDS CLARIFICATION: …]   ← at most three of these; each must be a real question you cannot answer yourself
```

### Constraints

- **No tech stack.** Do not mention `localStorage`, `<details>`, `fetch`, JSON schema, or HTML. The spec has to read cleanly if you switched the whole app to Elm tomorrow.
- **Every FR must be testable in one sentence.** "The system persists notes" — how do you test that? "The system MUST restore a previously-saved note when the user revisits the exercise in the same browser profile" — now you can write an acceptance scenario for it.
- **Every acceptance scenario must use Given/When/Then.** If you can't fit the scenario into that shape, the FR is probably too vague.
- **Run the content-quality checklist** at the end of the template. Four yes/no boxes, four answers.

### Cross-check

After you write it, read your spec and ask:

1. A stranger reads this with no context on Phase 0 — can they implement the feature from the spec alone? If no, which FR is underspecified?
2. Can I test each acceptance scenario in a browser within 30 seconds? If no, the scenario is either too abstract or too tech-entangled.
3. Does my "Out of Scope" section include at least one adjacent temptation? (If it's empty, you probably scope-crept into the FRs.)

## Reflection prompts

1. What leaked between WHAT and HOW in your first draft? (Tech stack names, data formats, UI library choices, CSS class names are the usual culprits.)
2. Which FR did you end up splitting into two because it wasn't atomically testable?
3. Harper Reed's prompt says "iterate until you feel that the steps are right sized." Did you iterate on your FRs? If not, run one more pass now and write what changed.
