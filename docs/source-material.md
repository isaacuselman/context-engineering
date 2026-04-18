# Source material for a Phase 0 context-engineering curriculum

This report compiles primary-source material across four categories — CLAUDE.md examples, agent-failure analyses, subagent design patterns, and task decomposition / spec writing — collected for a self-paced curriculum on context engineering for Claude Code. Each category presents sources with verbatim content where possible, structural analysis, and credibility caveats. A consolidated **gaps section** at the end flags claims that lack a verifiable primary source so you know where you'll need to write original content.

The most important top-level finding: the widely repeated **"33% higher success rate when planning is separated from execution"** claim has no clear primary source. The closest real research (Plan-and-Act, arXiv 2503.09572) reports a roughly **20-percentage-point absolute gain** (9.85% → 29.63% on WebArena-Lite) from adding a planner. Use those real numbers in the curriculum rather than the apocryphal 33%.

---

## Category 1: Real-world CLAUDE.md examples and design wisdom

### 1.1 Anthropic — `claude-code-action/CLAUDE.md` (44 lines, 3.45 KB)

**Source:** `github.com/anthropics/claude-code-action/blob/main/CLAUDE.md`

This is the cleanest canonical example of the Boris Cherny style — short, scannable, structured around "what / why / how" rather than rules. Section structure:

```
## Commands              (build, test, typecheck — literal code block)
## What This Is          (2 sentences + pointer to key file)
## How It Runs           (1 paragraph tracing entry point through major modules)
## Key Concepts          (bolded one-liners + file pointers, no code)
## Things That Will Bite You   (bullet list of gotchas)
## Code Conventions      (4–5 short bullets: Bun runtime, moduleResolution, retry helper)
```

**What makes it good:** uses **file-path pointers instead of embedded code snippets** (snippets go stale); front-loads commands so the agent doesn't have to discover them; "Things That Will Bite You" surfaces non-obvious project-specific traps (strict TypeScript with `noUnusedLocals`, discriminated unions requiring `isEntityContext()` first); declares the runtime is **Bun, not Node**, which is exactly the kind of override that prevents wasted exploration. Notably absent: any style/formatting rules (those live in Biome/Prettier).

**Useful for the exercise on:** demonstrating concise structure, pointer-over-snippet discipline, and the "gotchas" pattern.

### 1.2 OpenAI — `codex/AGENTS.md` (211 lines, 16.5 KB)

**Source:** `github.com/openai/codex/blob/main/AGENTS.md`

The opposite end of the length spectrum, and the canonical AGENTS.md example. It's long because the repo is the agent CLI's own dev guide — multiple languages (Rust, Node, TypeScript), nested AGENTS.md files (the repo has roughly **88 AGENTS.md files** scattered across subdirectories using nearest-file lookup).

**What makes it instructive:** demonstrates the **nested / progressive-disclosure pattern**. The root file covers cross-cutting policy; child directories carry their own AGENTS.md for module-specific rules. Codex (and Claude Code) walk up the tree from the working directory and load the nearest file. Child files do not duplicate parent content. Codex also supports `AGENTS.override.md` for monorepo subpath overrides.

**Useful for the exercise on:** showing how to scale CLAUDE.md guidance in a monorepo without one giant file, and explaining the AGENTS.md ecosystem (Codex, Cursor, Aider, Gemini CLI, Jules, Amp, Factory all read it; ~60k+ open-source repos have adopted it).

### 1.3 HumanLayer — root `CLAUDE.md` (≈60 lines)

**Source:** `github.com/humanlayer/humanlayer` plus the companion blog post **"Writing a good CLAUDE.md"** at `humanlayer.dev/blog/writing-a-good-claude-md` (Nov 25, 2025).

The HumanLayer post is the most rigorous public guide on CLAUDE.md design and is paired with a working repo example. Key claims:

- **Less is more.** They cite arXiv 2507.11538: frontier thinking LLMs reliably follow ~150–200 instructions; Claude Code's own system prompt already uses ~50. Each added instruction degrades adherence to *all* instructions, not just newer ones.
- **Universal applicability.** Claude Code wraps CLAUDE.md in a `<system-reminder>` tag that explicitly tells the model "this context may or may not be relevant…you should not respond to this context unless it is highly relevant." Irrelevant content is double-penalized: it's both ignored and pollutes the context.
- **Conditional instructions.** Wrap domain-specific sections in XML tags like `<important if="you are writing or modifying tests">…</important>`. Don't wrap foundational items (tech stack, directory map). Make the condition narrow — `if="you are writing code"` defeats the purpose.
- **Don't make Claude be a linter.** Style/formatting rules belong in Biome/Prettier and `Stop` hooks; LLMs already learn style from existing code patterns via in-context learning.

**Useful for the exercise on:** providing the "instruction budget" frame, the conditional-instruction pattern, and the linter-vs-CLAUDE.md decision rule.

### 1.4 Boris Cherny's CLAUDE.md philosophy (the "~100 lines" claim)

**Sources:** Boris Cherny's public threads (Jan 2026), Pragmatic Engineer interview (Mar 2026), and the official **"Claude Code: Best practices for agentic coding"** post at `anthropic.com/engineering/claude-code-best-practices` (April 18, 2025).

Cherny — creator of Claude Code — reports his typical CLAUDE.md is around **100 lines / ~2,500 tokens**. His self-improvement loop is the most-quoted operational rule:

> "Anytime we see Claude do something incorrectly, we add it to CLAUDE.md so it doesn't repeat next time."

Operationally: when Claude misbehaves, prompt it with `Update CLAUDE.md so you don't make that mistake again` — Claude is good at writing rules for itself. Check CLAUDE.md into git and update it multiple times a week. The "How Anthropic teams use Claude Code" PDF reinforces: "the better the Claude.md files, the better Claude Code performs."

**Caveat to flag in the curriculum:** the actual full text of Cherny's personal ~100-line CLAUDE.md has not been published verbatim. The 100-line / 2,500-token figure is his self-reported number. Use it as guidance, not as a citable measurement. The `claude-code-action` example above (44 lines) is the closest publicly available proxy.

### 1.5 Anti-patterns consolidated from multiple sources

The strongest pattern across HumanLayer, Anthropic, and community discussion is that the CLAUDE.md anti-patterns are mostly about **what to leave out**, not what to add. The recurring offenders:

| Anti-pattern | Why it fails |
|---|---|
| Auto-generating with `/init` and shipping unedited | This is the highest-leverage file in the harness; bad lines compound across every session |
| Embedded code snippets | They go stale; use `file:line` pointers instead |
| Code-style rules (formatting, naming) | Burns instruction budget that linters handle deterministically |
| Schema-specific or feature-specific rules | Not universally applicable; degrades adherence to genuinely universal rules |
| Settings-level config in prose ("NEVER add Co-Authored-By") | Should live in `.claude/settings.json` where it's deterministically enforced |
| Caps-lock threats on every line ("YOU MUST ALWAYS…") | Defeats emphasis tuning; Reddit threads document Claude ignoring CLAUDE.md when it's all-caps everywhere |
| 500–1000+ line files with every command | Triggers exponential degradation; community warning threshold is ~10k words, hard warning at ~40k |

### 1.6 File hierarchy and security

Claude Code load order (precedence): `~/.claude/CLAUDE.md` (global) → `./CLAUDE.md` (project root, committed) → `./CLAUDE.local.md` (gitignored personal overrides) → parent dirs (monorepo) → child dirs (loaded on demand). **Never put secrets in CLAUDE.md** — it's committed. As of Bun v1.2.17, `bun init` auto-generates a CLAUDE.md when the `claude` CLI is detected and symlinks Cursor's `.cursor/rules/` to it.

The `@path/to/file.md` import syntax (supported since mid-2025) lets you keep the CLAUDE.md skeleton lean while pulling in detailed docs on demand — the basis of the **progressive disclosure** pattern (`agent_docs/building_the_project.md`, `agent_docs/running_tests.md`, etc., indexed from CLAUDE.md with one-line descriptions).

---

## Category 2: Published analyses of agent failure modes

### 2.1 Andrej Karpathy — the canonical "context engineering" tweet

**Source:** `x.com/karpathy/status/1937902205765607626` (June 25, 2025, quote-tweeting Tobi Lütke)

> "+1 for 'context engineering' over 'prompt engineering'. People associate prompts with short task descriptions you'd give an LLM in your day-to-day use. When in every industrial-strength LLM app, context engineering is the delicate art and science of filling the context window with just the right information for the next step."

> "Science because doing this right involves task descriptions and explanations, few shot examples, RAG, related (possibly multimodal) data, tools, state and history, compacting... Too little or of the wrong form and the LLM doesn't have the right context for optimal performance. Too much or too irrelevant and the LLM costs might go up and performance might come down."

> "people's use of 'prompt' tends to (incorrectly) trivialize a rather complex component. You prompt an LLM to tell you why the sky is blue. But apps build contexts (meticulously) for LLMs to solve their custom tasks."

**Credibility:** former Tesla AI director, OpenAI founding member. Framed as expert opinion, not empirical study. Useful as the canonical naming event for the field and an opening citation for the curriculum.

### 2.2 Tobi Lütke — the originating tweet

**Source:** `x.com/tobi/status/1935533422589399127` (June 18, 2025)

> "I really like the term 'context engineering' over prompt engineering. It describes the core skill better: the art of providing all the context for the task to be plausibly solvable by the LLM."

Longer-form on the **ACQ2 podcast** (`acquired.fm/acq2-episodes/how-to-live-in-everyone-elses-future-with-shopify-ceo-tobi-lutke`):

> "I like the term context engineering because I think the fundamental skill of using AI well is to be able to state a problem with enough context, in such a way that without any additional pieces of information, the task is plausibly solvable… Now with a lot of agents, the agents end up being pretty good at figuring out what's missing and go get it themselves. But I think the crazy thing about AI is it can do this. I think the skill people should build is trying to not require it to do it."

The one-sentence definition ("the art of providing all the context for the task to be plausibly solvable by the LLM") is the cleanest working definition available. Pair with Karpathy's enumeration for a complete opening.

### 2.3 Cognition AI — "Don't Build Multi-Agents"

**Source:** `cognition.ai/blog/dont-build-multi-agents` (June 12, 2025, by Walden Yan)

The essay's two principles:

> "Principle 1: Share context, and share full agent traces, not just individual messages."

> "Principle 2: Actions carry implicit decisions, and conflicting decisions carry bad results."

> "I would argue that Principles 1 & 2 are so critical, and so rarely worth violating, that you should by default rule out any agent architectures that don't abide by them."

**The Flappy Bird teaching vignette (paraphrased):** Decompose "build a Flappy Bird clone" into Subtask 1 (background with green pipes) and Subtask 2 (controllable bird). Subagent 1 misreads and builds a Mario-style background; Subagent 2 builds a non-game-asset bird that doesn't move correctly. Even when the original task is shared with both subagents, neither sees the other's work, so styles and decisions diverge — the final agent inherits two miscommunications. This is the cleanest published example of why naive parallel decomposition fails.

> "As of June 2025, Claude Code is an example of an agent that spawns subtasks. However, it never does work in parallel with the subtask agent, and the subtask agent is usually only tasked with answering a question, not writing any code. The designers of Claude Code took a purposefully simple approach."

> "Running multiple agents in collaboration only results in fragile systems. The decision-making ends up being too dispersed and context isn't able to be shared thoroughly enough between the agents."

**Empirical backing:** none. The essay is principled argument from production experience with Devin, not measurement. Treat claims as informed opinion. It is nonetheless treated as near-canonical in the field, so the curriculum should engage with it directly — including its tension with Anthropic's multi-agent research post.

### 2.4 Anthropic — "Effective Context Engineering for AI Agents"

**Source:** `anthropic.com/engineering/effective-context-engineering-for-ai-agents` (Sept 29, 2025)

The official Anthropic statement, released alongside Claude Sonnet 4.5. Key concepts:

> "Context engineering refers to the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference, including all the other information that may land there outside of the prompts."

> "Context, therefore, must be treated as a finite resource with diminishing marginal returns. Like humans, who have limited working memory capacity, LLMs have an 'attention budget' that they draw on when parsing large volumes of context."

The post introduces **"context rot"** — a needle-in-a-haystack finding (citing `research.trychroma.com`) showing that as context grows, recall accuracy drops. Architectural cause: transformer attention has n² pairwise relationships, and training distributions skew toward shorter sequences.

Three named long-horizon primitives, all directly relevant to Claude Code:

1. **Compaction** — summarize history and re-initialize. Claude Code's implementation "preserves architectural decisions, unresolved bugs, and implementation details while discarding redundant tool outputs" and continues with "the five most recently accessed files."
2. **Structured note-taking** — the "Claude Plays Pokémon" example: maintained tallies across thousands of game steps, e.g. "for the last 1,234 steps I've been training my Pokémon in Route 1, Pikachu has gained 8 levels toward the target of 10."
3. **Sub-agent architectures** — "Each subagent might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000-2,000 tokens)."

Concrete failure modes named in the post:

- **System prompts at the wrong altitude.** Hardcoded if-else logic causes "fragility and increases maintenance complexity." Vague high-level guidance "fails to give the LLM concrete signals."
- **Bloated tool sets.** "If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better."
- **Few-shot stuffing.** Teams "stuff a laundry list of edge cases into a prompt" — recommend curated diverse canonical examples instead.

Companion posts worth assigning: **"Writing Effective Tools for AI Agents"** (notes that Claude Code restricts tool responses to 25,000 tokens by default; recommends `user_id` over `user`); **"Building Effective Agents"** (workflows-vs-agents taxonomy); the **multi-agent research system** post (Anthropic's counterpoint to Cognition).

### 2.5 Sean Goedecke — agent failure modes from a GitHub engineer

**Sources:** `seangoedecke.com/ai-agents-and-code-review/`, `seangoedecke.com/good-code-reviews/`, `seangoedecke.com/programming-with-ai-agents-as-theory-building/`, `seangoedecke.com/ai-security/`.

The most quotable framing of the agent-as-junior-engineer failure pattern:

> "Working with AI agents is like working with enthusiastic juniors who never develop the judgement over time that a real human would."

> "If you don't have the technical ability to spot when the LLM is going down the wrong track, you'll rapidly end up stuck. Trying to make a badly-designed solution work costs time, tokens, and codebase complexity. All of these things cut into the agent's ability to actually solve the problem. Once two or three of them pile up, the app is no longer tractable for the agent and the whole thing grinds to a halt."

A concrete failure example (VicFlora): Codex spent a lot of effort trying to reverse-engineer frontend code for a dichotomous key when the raw data was readily available elsewhere. The lesson is structural review beats diff nitpicking — "is this even the right place for the code at all?"

Self-reported workflow ratio: "about 10% of agent output is actually making its way into my output." Anecdotal but useful as an honest practitioner number.

Security gotchas worth quoting in a Phase 0 module: "No model is immune to prompt injection." "Allowing third-parties to fill out your Cursor or Copilot rules presents the same risks as allowing them to contribute code directly."

### 2.6 METR — actual measurements of agent capability

Two METR sources are the strongest empirical material in the entire research base:

**Time horizon paper** (`metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/`): fits a logistic curve of P(success) vs log2(human task time). 50%-time-horizon doubling roughly every 7 months historically, possibly accelerating to ~4 months in 2024. Claude 3.7 Sonnet has roughly a 1-hour time horizon at 50% success on software tasks. Updated in **Time Horizon 1.1** (Jan 2026): post-2023 doubling time estimated at 130.8 days (~4.3 months).

**Developer productivity RCT** (`metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/`, paper at arXiv 2507.09089): 16 experienced open-source developers, 246 tasks, mature repos (avg 22k+ stars, 1M+ LOC). Headline result:

> "When developers use AI tools, they take 19% longer than without—AI makes them slower."

And the perception gap: developers predicted a 24% speedup; even after experiencing the slowdown, they still believed they were 20% faster. This is the most rigorous available data point and a useful sober counterweight to vendor claims.

### 2.7 Closest real research on planning/execution separation

**Source:** Plan-and-Act, arXiv 2503.09572 (Erdogan et al., ICML 2025)

The paper's actual ablation on WebArena-Lite:

- **Base executor, no planner: 9.85% success**
- **Adding a planner: 29.63% success** (~20-percentage-point absolute gain, ~3× relative)
- **Full pipeline (planner + finetuning + synthetic data + plan expansion + dynamic replanning + CoT): 57.58%** (state-of-the-art)
- WebVoyager text-only state-of-the-art: 81.36%

> "Adding a PLANNER increases success rate from 9.85% to 29.63%. This validates our core hypothesis that explicit planning helps bridge the gap between high-level user intentions and low-level actions."

**Use this in place of the apocryphal "33%" claim.** It is a real, citable, peer-reviewed measurement of planning-vs-execution separation.

The most rigorous coding-agent study on plan/context effects is **"From Plan to Action"** (arXiv 2604.12147), using SWE-bench Verified and Pro across 16,991 trajectories: "providing the standard plan improves issue resolution," "periodic plan reminders can mitigate plan violations," and crucially "a subpar plan hurts performance even more than no plan at all."

---

## Category 3: Subagent design patterns for Claude Code

### 3.1 Official Anthropic schema (verbatim)

**Source:** `docs.claude.com/en/docs/claude-code/sub-agents` (also at `code.claude.com/docs/en/sub-agents`)

Minimal example exactly as published:

```
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

> "The frontmatter defines the subagent's metadata and configuration. The body becomes the system prompt that guides the subagent's behavior. Subagents receive only this system prompt (plus basic environment details like working directory), not the full Claude Code system prompt."

**Full frontmatter field reference:**

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Unique identifier, lowercase letters and hyphens |
| `description` | Yes | When Claude should delegate to this subagent (basis for auto-delegation) |
| `tools` | No | Allowlist of tools. Inherits all if omitted |
| `disallowedTools` | No | Denylist; applied first, then `tools` is resolved against the remaining pool |
| `model` | No | `sonnet`, `opus`, `haiku`, full model ID (e.g. `claude-opus-4-7`), or `inherit` (default) |
| `permissionMode` | No | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan` |
| `maxTurns` | No | Cap on agentic turns |
| `skills` | No | Skills loaded at startup (full content injected; not inherited from parent) |
| `mcpServers` | No | MCP servers available (string reference or inline) |
| `hooks` | No | Lifecycle hooks scoped to this subagent |
| `memory` | No | `user`, `project`, or `local` |
| `background` | No | `true` to always run as background task (default `false`) |
| `effort` | No | `low`, `medium`, `high`, `xhigh`, `max` (model-dependent) |
| `isolation` | No | `worktree` to run in temporary git worktree |
| `color` | No | UI color: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `initialPrompt` | No | Auto-submitted first user turn when run as main session agent |

**Tool restriction examples (verbatim):**

```
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
---
```

```
---
name: no-writes
description: Inherits every tool except file writes
disallowedTools: Write, Edit
---
```

To restrict which subagents a main-thread agent can spawn: `tools: Agent(worker, researcher), Read, Bash`. If `Agent` is omitted entirely, the agent cannot spawn any subagents at all. Note the rename: `Agent` replaced `Task` in Claude Code v2.1.63; `Task` still works as alias.

**File-location precedence:**

| Location | Scope | Priority |
|---|---|---|
| Managed settings | Organization-wide | 1 (highest) |
| `--agents` CLI flag | Current session | 2 |
| `.claude/agents/` | Current project (commit to git) | 3 |
| `~/.claude/agents/` | All your projects (personal) | 4 |
| Plugin's `agents/` directory | Where plugin is enabled | 5 |

Plugin subagents do NOT honor `hooks`, `mcpServers`, or `permissionMode` — these are silently ignored for security reasons.

**Invocation:** Auto-delegation matches the user's task to each subagent's `description` field — this is why the description should describe *when* to use the subagent, not just what it does. Explicit invocation: `Use the code-reviewer subagent to analyze my latest commits`. **Subagents cannot spawn other subagents** (prevents infinite nesting).

**Built-in subagents:** `Explore` (Haiku, read-only, file discovery), `Plan` (inherits model, read-only, used inside plan mode), `general-purpose` (inherits, all tools), plus helpers `statusline-setup` and `Claude Code Guide`.

### 3.2 Code reviewer example — VoltAgent collection

**Source:** `github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/04-quality-security/code-reviewer.md`

```yaml
---
name: code-reviewer
description: Use this agent when you need to conduct comprehensive code reviews focusing on code quality, security vulnerabilities, and best practices.
tools: Read, Write, Edit, Bash, Glob, Grep
model: opus
---
```

The system prompt body (~280 lines) frames the subagent as a senior code reviewer with expertise across correctness, performance, maintainability, and security. Three explicit phases: Context Gathering → Review Execution → Delivery. Uses a JSON "Communication Protocol" for inter-agent messaging (`requesting_agent`, `request_type`, `payload`) and emits progress JSON (`files_reviewed`, `issues_found`, `critical_issues`).

**Worth noting** for the curriculum: this example uses `opus` (analytical role) and *does* grant `Write` and `Edit` — which is more permissive than many reviewer subagents. A stricter alternative pattern is the official Anthropic minimal example, which restricts to `Read, Glob, Grep` only. Both are defensible; the choice illustrates the design axis "should the reviewer be able to suggest fixes by writing them, or only describe them?"

### 3.3 Spec / PRD writer — Ian Nuttall collection

**Source:** `github.com/iannuttall/claude-agents/blob/main/agents/prd-writer.md`

```yaml
---
name: prd-writer
description: Use this agent when you need to create a comprehensive Product Requirements Document (PRD) for a software project or feature. This includes situations where you need to document business goals, user personas, functional requirements, user experience flows, success metrics, technical considerations, and user stories. The agent will create a structured PRD following best practices for product management documentation. Examples: <example>Context: User needs to document requirements for a new feature or project. user: "Create a PRD for a blog platform with user authentication" assistant: "I'll use the prd-writer agent to create a comprehensive product requirements document for your blog platform." <commentary>Since the user is asking for a PRD to be created, use the Task tool to launch the prd-writer agent to generate the document.</commentary></example>
tools: Task, Bash, Grep, LS, Read, Write, WebSearch, Glob
color: green
---
```

System prompt opening (verbatim):

> "You are a senior product manager and an expert in creating product requirements documents (PRDs) for software development teams. Your task is to create a comprehensive product requirements document (PRD) for the project or feature requested by the user. You will create a prd.md document in the location requested by the user. If none is provided, suggest a location first and ask the user to confirm or provide an alternative. Your only output should be the PRD in Markdown format."

The output sections include business goals, user personas, functional requirements, UX flows, success metrics, technical considerations, and user stories.

**Useful for the exercise on:** showing the `<example>` block style of description (verbose but disambiguates auto-delegation), and demonstrating that writer-style subagents need `Write` access while still being constrained from arbitrary tool use.

### 3.4 Security auditor — VoltAgent collection

**Source:** `github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/04-quality-security/security-auditor.md`

```yaml
---
name: security-auditor
description: Use this agent when conducting comprehensive security audits, compliance assessments, or risk evaluations across systems, infrastructure, and processes. Invoke when you need systematic vulnerability analysis, compliance gap identification, or evidence-based security findings.
tools: Read, Grep, Glob
model: opus
---
```

The body (287 lines) includes detailed checklists for SOC 2, ISO 27001, HIPAA, PCI DSS, GDPR, NIST, CIS benchmarks; vulnerability assessment domains (network, app, config, patch, access, encryption, endpoint, cloud); a JSON Communication Protocol; a 3-phase workflow (Audit Planning → Implementation → Audit Excellence); finding classification (Critical/High/Medium/Low/Observations); and explicit integration handoffs to other named agents (`security-engineer`, `penetration-tester`, `compliance-auditor`).

**Design lesson:** principle of least privilege exemplified — read-only tools (`Read, Grep, Glob`) for an auditor role, paired with `opus` for highest-capability analytical reasoning. The contrast with the test-automator (which gets `Write, Edit, Bash` and runs on `sonnet`) makes the "analyze with opus, execute with sonnet" pattern very visible.

### 3.5 Test automator — VoltAgent collection

**Source:** `github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/04-quality-security/test-automator.md`

```yaml
---
name: test-automator
description: Use this agent when you need to build, implement, or enhance automated test frameworks, create test scripts, or integrate testing into CI/CD pipelines.
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
---
```

System prompt frames the subagent as a senior test automation engineer; uses the same Communication Protocol pattern; three-phase workflow (assess → build → deliver). Closing directive: "Always prioritize maintainability, reliability, and efficiency while building test automation that provides fast feedback and enables continuous delivery."

### 3.6 Community design wisdom

**Source:** PubNub blog, "Best practices for Claude Code subagents" (`pubnub.com/blog/best-practices-for-claude-code-sub-agents/`)

Synthesized recommendations:

1. **Single clear goal per subagent.** "Give each subagent one clear goal, input, output, and handoff rule." Write action-oriented descriptions: "Use after a spec exists; produce an ADR and guardrails."
2. **Pipeline of specialists.** PubNub's 3-stage canonical pipeline: `pm-spec` (writes spec, sets `READY_FOR_ARCH`) → `architect-review` (validates, sets `READY_FOR_BUILD`) → `implementer-tester` (builds + tests, sets `DONE`). Uses a shared queue file (`enhancements/_queue.json`).
3. **Scope tools tightly per role.** Read-heavy roles (PM, architect) get search/docs/MCP only. Implementer gets `Edit/Write/Bash`. "If you omit `tools`, you're implicitly granting access to all available tools. Be intentional."
4. **Hooks suggest, humans approve.** Register `SubagentStop` hooks that print the next suggested command; a human pastes it to proceed. Prevents runaway chains.
5. **Definition of Done per agent.** End each system prompt with a checklist (PM: acceptance criteria; Architect: ADR + guardrails; Implementer: code + passing tests + summary).
6. **Context hygiene** (attributed to Adam Wolf, Claude Code team): "Subagents work best when they just look for information and provide a small amount of summary back to main conversation thread." Use subagents for research and collection; let the main agent execute.
7. **Model tiering.** Opus for reviewer/auditor (analytical), Sonnet for implementer/test-writer (execution), Haiku for fast read-only exploration.

---

## Category 4: Task decomposition and spec-writing patterns

### 4.1 Anthropic's recommended workflows (from "Claude Code Best Practices")

**Source:** `anthropic.com/engineering/claude-code-best-practices` (Boris Cherny, April 18, 2025)

The flagship workflow is **explore, plan, code, commit**:

1. **Explore.** Have Claude read relevant files/images/URLs but "explicitly tell it not to write any code just yet." Use subagents to investigate specific questions early.
2. **Plan.** Use the magic words: **"think" < "think hard" < "think harder" < "ultrathink"** — "Each level allocates progressively more thinking budget for Claude to use." Have it write the plan to a document or GitHub issue so you can reset to that state.
3. **Code.** Implement the solution.
4. **Commit.** Create a PR.

> "Steps #1-#2 are crucial—without them, Claude tends to jump straight to coding a solution… asking Claude to research and plan first significantly improves performance for problems requiring deeper thinking upfront."

Three other named workflows:

- **TDD.** "Be explicit about the fact that you're doing test-driven development so that it avoids creating mock implementations." Ask subagents to verify the implementation isn't overfitting to the tests.
- **Visual iteration.** Write code → screenshot result → iterate (typically 2–3 iterations).
- **Safe YOLO mode.** `claude --dangerously-skip-permissions` in a container.

**Specificity beats vagueness.** Cherny gives a concrete contrast:

- Bad: `add tests for foo.py`
- Good: `write a new test case for foo.py, covering the edge case where the user is logged out. avoid mocks`
- For complex features: `look at how existing widgets are implemented... HotDogWidget.php is a good example to start with. then, follow the pattern...`

**Context management:** "During long sessions, Claude's context window can fill with irrelevant conversation, file contents, and commands. This can reduce performance and sometimes distract Claude. Use the `/clear` command frequently between tasks to reset the context window." For long multi-step tasks: have Claude maintain a markdown checklist as a working scratchpad; "instruct Claude to address each issue one by one, fixing and verifying before checking it off."

**Course-correction tools** (worth a slide on their own): (1) ask for a plan first; (2) press Escape to interrupt; (3) double-tap Escape to jump back in history and edit a prior prompt; (4) ask Claude to undo.

### 4.2 GitHub Spec Kit — the published spec template ecosystem

**Source:** `github.com/github/spec-kit` (MIT-licensed, official GitHub project, ~69k stars)

Spec Kit defines a four-phase **Spec-Driven Development** workflow:

1. `/speckit.constitution` → `memory/constitution.md` (non-negotiable project principles)
2. `/speckit.specify` → `specs/###-feature/spec.md` (what/why; *no* tech stack)
3. `/speckit.plan` → `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` (how, with tech stack)
4. `/speckit.tasks` → `tasks.md` (small, executable tasks)

**Feature specification template (verbatim headings):**

```
# Feature Specification: [FEATURE NAME]

**Feature Branch**: [###-feature-name]
**Created**: [DATE]
**Status**: Draft
**Input**: User description: "$ARGUMENTS"

## User Scenarios & Testing
### Primary User Story
### Acceptance Scenarios
### Edge Cases

## Requirements
### Functional Requirements
- FR-001: System MUST [specific capability]
- FR-002: ...
### Key Entities (if data involved)

## Success Criteria

## Out of Scope

[NEEDS CLARIFICATION: specific question]   ← max 3 such markers
```

**Specification quality checklist (verbatim):**

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

**Plan template fields (verbatim):**

```
**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75]
**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM]
**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]
**Testing**: [e.g., pytest, XCTest, cargo test]
**Target Platform**: [e.g., Linux server, iOS 15+, WASM]
**Project Type**: [e.g., library/cli/web-service/mobile-app]
**Performance Goals**: [e.g., 1000 req/s, 10k lines/sec, 60 fps]
**Constraints**: [e.g., <200ms p95, <100MB memory, offline-capable]
**Scale/Scope**: [e.g., 10k users, 1M LOC, 50 screens]
```

**Philosophy (verbatim from `spec-driven.md`):**

> "We're moving from 'code is the source of truth' to 'intent is the source of truth.' With AI the specification becomes the source of truth and determines what gets built. This isn't because documentation became more important. It's because AI makes specifications executable."

> "Focus on WHAT users need and WHY; ❌ Avoid HOW to implement (no tech stack, APIs, code structure)."

**Constitutional TDD article (verbatim):** "All implementation MUST follow strict Test-Driven Development. No implementation code shall be written before: 1. Unit tests are written 2. Tests are validated and approved by the user 3. Tests are confirmed to FAIL (Red phase)."

This is the most polished, opinionated, and credible published spec template for AI agents. It is the single best artifact to base spec-writing exercises on.

### 4.3 Harper Reed's workflow

**Source:** `harper.blog/2025/02/16/my-llm-codegen-workflow-atm/`

The key planning prompt (verbatim) — useful as a teachable artifact for the spec-writing exercise:

> "Draft a detailed, step-by-step blueprint for building this project. Then, once you have a solid plan, break it down into small, iterative chunks that build on each other. Look at these chunks and then go another round to break it into small steps. review the results and make sure that the steps are small enough to be implemented safely with strong testing, but big enough to move the project forward. Iterate until you feel that the steps are right sized for this project."

This is the "iterate-on-task-size-until-it-feels-right" pattern in one sentence.

### 4.4 Feature decomposition example

**Source:** `github.com/snarktank/ai-dev-tasks` plus Addy Osmani's "Self-Improving Coding Agents"

Two templates drive the decomposition: `create-prd.md` (feature idea → PRD) and `generate-tasks.md` (PRD → step-by-step task list). The process is "iterative implementation: guiding the AI to tackle one task at a time, allowing you to review and approve each change."

**Atomic task decomposition example (verbatim from Osmani):**

> "Each task should be small enough to fit in one AI session and have unambiguous pass/fail criteria. For example, instead of 'Build the entire dashboard' you'd have tasks like 'Add a navigation bar with links to Home, About, Contact' with acceptance criteria specifying exact expectations (e.g. 'the current page link is highlighted in blue'). This granular way makes sure the agent knows what 'done' looks like for each step."

The end state is a `tasks.json` file consumed by an agent loop that "continuously picks tasks, writes code, runs tests, and updates the task list until all tasks pass." Geoffrey Huntley / Ryan Carson named this the **"Ralph Wiggum" loop**.

### 4.5 Task granularity guidance

**The strongest available numerical claim** (use with the caveat below):

> "Granularity is measurable: reported evaluations show multi-file tasks at around 19% accuracy versus about 87% for single-function tasks, largely because smaller tasks fit within an agent's effective working set, whereas the spec carries the long-horizon intent."

Source: Augment Code guide. **Caveat:** the "reported evaluations" are unsourced in the article — this is practitioner folklore until verified, but the directional claim (smaller is more reliable) is consistent across all sources.

**Parallel-agent sweet spot (Claw101/OpenClaw blog):** "Too fine: Spawning 20 agents for 20 small file edits creates more coordination overhead than you save from parallelism. Sweet spot: 3-8 parallel tasks. Each taking 3-10 minutes."

**Eclipse/Theia "Task Engineering" framing (Sep 2025):**

> "A caution: over-dividing can fragment your mental model and bring down the overall efficiency. Aim for the smallest coherent unit that preserves architectural clarity."

> "Task size affects context needs: Smaller, well-divided tasks typically need less, sharper context. Context friction signals task issues: If it's hard to collect or provide the concise, focused context for a task, adjust the granularity or design of the task."

**Addy Osmani on spec-writing:**

> "Simply throwing a massive spec at an AI agent doesn't work — context window limits and the model's 'attention budget' get in the way."

> "Scaffold your code with `// TODO` comments that describe what needs to be done, and have the agent fill them one by one. Each TODO essentially acts as a mini-spec for a small task."

### 4.6 Cognition's position on decomposition (cross-reference)

Cognition's "Don't Build Multi-Agents" essay (covered in Category 2.3) is the canonical published critique of naive task decomposition. Their alternative for very long tasks: a single-threaded linear agent with a dedicated LLM that compresses history into key events/decisions (they've fine-tuned a smaller model for this in production). Parallel decomposition is acceptable only for read-only investigative subtasks.

### 4.7 Spec-format comparison (PRD vs user story vs technical spec)

**No single rigorous published comparison exists** of which spec format works best for AI agent consumption. The strongest signals from the assembled material:

- GitHub Spec Kit's **layered approach** — separate documents for spec (what/why), plan (how/tech), tasks (atomic execution units) — appears to be the emerging best practice.
- The **PRD-style** template (Nuttall's `prd-writer`, snarktank's `create-prd.md`) is widely used for the "spec" layer.
- Functional requirements with **FR-001-style numbering and "MUST" language** (from Spec Kit) maps well onto agent-checkable acceptance criteria.
- **User stories alone are insufficient** for agents: they encode WHY but not WHAT in checkable terms.
- **Technical specs alone are also insufficient**: they encode HOW but tend to skip user-facing acceptance criteria the agent can verify.

This is a topic where you'll likely need to write original synthesis for the curriculum rather than cite a primary source.

---

## Gaps — what couldn't be sourced

These are the items the curriculum will need to either generate originally or cite with explicit caveats.

**The "33% higher success rate when planning is separated from execution" claim has no verifiable primary source.** No paper, blog post, Anthropic study, or Cognition study uses that specific number for that specific claim. Closest real findings: Plan-and-Act (arXiv 2503.09572) reports a +20-percentage-point absolute gain (9.85% → 29.63%) on WebArena-Lite from adding a planner; LLaMAR (OpenReview Y1rOWS2Z4i) reports "30% higher success rate" but in multi-agent robotics, not coding. **Recommendation: drop the 33% number entirely and cite Plan-and-Act's actual measurements.**

**Boris Cherny's full ~100-line CLAUDE.md has not been published verbatim.** The "100 lines / 2,500 tokens" figure is his self-report from social posts and the Pragmatic Engineer interview. The closest public proxy is `anthropics/claude-code-action/CLAUDE.md` (44 lines) — use that as the worked example.

**Anthropic's "90.2% multi-agent improvement" claim** is referenced in secondary sources (Cosine.sh, Phil Schmid's blog) citing Anthropic's "How we built our multi-agent research system" post. The exact phrasing and methodology in the primary post should be confirmed before quoting in curriculum material.

**The "80% reduction in research time"** claim from Anthropic's "How Anthropic teams use Claude Code" PDF is vendor-internal with no published methodology. Treat as anecdotal.

**Augment Code's "19% multi-file vs 87% single-function accuracy"** numbers are unsourced in the article that publishes them. Directionally consistent with everything else in the literature, but should not be cited as a measurement.

**Cognition's empirical numbers are absent.** "Don't Build Multi-Agents" contains no success rates, no benchmark scores, no measurements — purely principled argument from production experience. The field treats it as canonical anyway, but the curriculum should make its anecdotal nature explicit.

**No long-form Karpathy essay on context engineering exists.** His contributions are tweets and the AI Startup School keynote (June 17, 2025, YouTube) — not a written piece. Plan to cite the tweets directly.

**No long-form Tobi Lütke essay either.** Material is the originating tweet plus the ACQ2 podcast appearance.

**No published PRD-vs-user-story-vs-technical-spec comparison for AI agent consumption.** The Spec Kit's layered approach is the best implicit answer, but a head-to-head comparison study does not appear to exist. This is a place to write original analysis.

**No primary citation for the "30–35% completion rate on multi-step tasks"** number sometimes attributed to a CMU/Salesforce study. Likely refers to TheAgentCompany benchmark (CMU, Xu et al., 2024/2025) but that primary was not retrieved here — worth a follow-up fetch if you want to use the figure.

**Geoffrey Huntley's writing on the "Ralph Wiggum" loop** is referenced in secondary sources but his primary blog posts on it were not directly fetched in this research pass — worth a follow-up if you want to use his exact phrasing.

**Anthropic docs page** for subagents was partially captured: the minimal frontmatter/body example and the full field table are verbatim, but the in-page **example subagents** (Code reviewer, Debugger, Data scientist, Database query validator) were only confirmed as headings rather than captured with full body content. Fetch the docs page directly if you need those for an exercise.
