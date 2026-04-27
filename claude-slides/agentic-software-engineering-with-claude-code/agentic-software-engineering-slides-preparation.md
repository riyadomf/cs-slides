---
marp: true
theme: default
paginate: true
---

# Agentic Software Engineering

*Md Omar Faruqe · April 21, 2026*

---

## What Is an Agent?

- **Agent = model + tools + harness**
- **Model** — reasons (Claude)
- **Tools** — act (read files, run commands, search web)
- **Harness** — glue that gives the model context, memory, tool access → **Claude Code**

---

## The Agent Loop

![agent loop](https://mintcdn.com/claude-code/c5r9_6tjPMzFdDDT/images/agentic-loop.svg?fit=max&auto=format&n=c5r9_6tjPMzFdDDT&q=85&s=5f1827dec8539f38adee90ead3a85a38)

**Brief → Plan → Act → Review**

Review is non-negotiable.

---

## How Tools Work

![how tools work](https://mintcdn.com/langchain-5e9cc07a/-_xGPoyjhyiDWTPJ/oss/images/agent.png?fit=max&auto=format&n=-_xGPoyjhyiDWTPJ&q=85&s=bd8da41dbf8b5e6fc9ea6bb10cb63e38)

- Tools make Claude Code agentic
- Each tool result feeds back into the loop
- Decide → call tool → observe → decide again → stop when goal met

---

## What Tools Can Claude Use?

| Category | What Claude can do |
|----------|-------------------|
| **File ops** | Read, edit, create, rename |
| **Search** | Glob, regex, codebase exploration |
| **Execution** | Shell, tests, git |
| **Web** | Search, fetch docs, lookup errors |
| **Code intel** | Type errors, definitions, references (via plugins) |

---

## Context Is Everything

- Context window is finite — fills up, gets dumber as it fills
- **CLAUDE.md** — always-loaded project context
  - `.claude/CLAUDE.md` (project) · `~/.claude/CLAUDE.md` (personal)
- **Memory** — facts that persist across sessions
- Rule: **if you explain it twice, codify it** (CLAUDE.md, skill, or memory)

---

## Prompting: Bad vs. Good

**Bad:**
> "fix the loan approval"

**Good:**
> "In `LoanApplicationView.submit()`, validation throws `ExceptionHolder` when attachments are missing. Show a field-level warning. Check `BaseView.addClientErrorMessage` for the pattern. Write a test first."

**Formula:** goal + file path + constraint + done criteria

- Precise prompt → fewer corrections
- Provide context **proactively** — or the agent wastes turns reading unnecessary files

---

## Plan Mode

- **Research + propose before edit**
  - `Shift+Tab` cycles: default → auto-accept → **plan**
- Runs in isolated Plan subagent — main context stays clean
- Use carefully — reads a lot of files. Skip for trivial edits.
- Let Claude use `AskUserQuestion` during planning — answer honestly

---

## Session Management

- **Session** = one chat + its context window
- Context fills up → Claude gets dumber → output quality drops
- **Three moves to fight context bloat:**
  - **Reset** when topic changes
  - **Branch** when you need to side-explore
  - **Rewind** when Claude goes off-rails

*Commands on the cheatsheet near the end of the deck.*

---

## The Five Context Primitives

| Primitive | Purpose | When to use |
|-----------|---------|-------------|
| **CLAUDE.md** | Always-on project context | Conventions, base patterns, gotchas |
| **Skills** | On-demand workflows | Repeatable tasks (deploy, scaffold, review) |
| **MCP** | External system connections | DB queries, Linear, GitHub, Slack |
| **Subagents** | Isolated research | Deep exploration without polluting main chat |
| **Hooks** | Automation on tool events | Auto-lint after edit, block risky commands |


*Plugins* = bundle of the above, one install.

---

## Skills: When & Why

**Skill** = reusable instructions, loaded only when needed.

**When to use:**
- Recurring domain knowledge (e.g., "how to add a Jasper report")
- Multi-step workflows (deploy, generate playwright script from squash test cases)
- Specialized — too narrow for CLAUDE.md

**Why skills beat MCP for most cases:**
- Load **only when invoked** — zero idle cost
- Plain markdown — easy to write, review, version in git

**Invocation:** auto-trigger (when the user prompt matches the skill's `description`) · manual (`/skill-name`)

---

## Creating a Skill

**Anatomy:**
- Lives in `.claude/skills/{name}/SKILL.md`
- Frontmatter:
  - `name` — slash-command handle (`/name`)
  - `description` — what auto-invoke matches against. Write it carefully.
- Body: instructions, examples, links to helper docs

**Two ways to write one:**
- `/skill-generator` — Claude walks you through it
- Manual — write the markdown, commit, done

**Progressive disclosure:** `SKILL.md` can reference sibling files in the same folder — loaded only when the skill runs. Keeps the auto-loaded part small.

*Rule:* explain a workflow twice → write a skill.

---

## MCP: Powerful but Specific

- **MCP** lets Claude call external services safely
- Example: DB queries via whitelisted functions (no raw SQL writes)
- Tool **names** always load; schemas now **deferred** (loaded on use)
- More servers = more upfront context — enable only what you need
- **CLI beats MCP when available** (`gh`, `psql`, `aws`)

---

## Subagents

- **Isolated Claude instance with its own context window**
- Use when: deep research or task that'd pollute main chat
- Returns a summary, not the 40 file reads it took to get there
- **Built-in types:**
  - `Explore` — codebase search / investigation
  - `Plan` — architecture + implementation planning
  - `general-purpose` — open-ended tasks
- **Custom subagents** — markdown file with frontmatter
  - Own tools, permissions, system prompt
  - Lives in `.claude/agents/` (project) or `~/.claude/agents/` (user)

*Example:* "Find all tests calling the deprecated `LegacyAuth` API" → delegate to `Explore`; main chat stays clean.

---

## Hooks

- **Shell commands fired on Claude Code events**
- Configured in `.claude/settings.json`

**Common event types:**

| Event | Fires | Use case |
|-------|-------|----------|
| `PreToolUse` | Before a tool runs | Block risky commands, add confirmations |
| `PostToolUse` | After a tool completes | Auto-format, run linter, notify Slack |
| `UserPromptSubmit` | On user message | Inject context, log prompts |
| `SessionStart` / `Stop` | Session lifecycle | Setup / teardown, metrics |

*Example:* `PreToolUse` on `Read` → if path matches `.env*`, block the read. Keeps secrets out of the agent's context.

---

## Trust, But Verify

- LLMs non-deterministic. Production must be deterministic.
- **Testing bridges the gap** — unit, E2E, screenshots
- If you can't verify it, don't ship it.
- Read every diff. Summary ≠ reality.

---

## Testing Strategy: Scope & Tooling

| Layer | Tool | When |
|-------|------|------|
| **Unit tests** | JUnit + Mockito | Business logic, services, helpers |
| **Ad-hoc UI testing** | Playwright CLI via Claude (agent-driven) | One-off — "does this flow work right now?" |
| **Regression E2E** | Claude-generated Playwright scripts in `e2e/` | Durable suite — run on every PR, nightly, forever |

**Why generate scripts for regression (vs. agent-run every time)?**

- **Deterministic** — script is code: same input, same steps, same assertions each run
- **Accumulates** — scripts stack into a suite; each new feature adds a spec
- **Cheap** — one-time generation cost, then free to replay
- **CI-friendly** — scripts run in pipelines; agent sessions don't
- **Auditable** — reviewable code in git

**Agent-driven testing shines for exploratory checks** — quick smoke, new-feature sanity, reproducing a bug interactively. When the check needs to survive past today, convert it into a spec.

**QA path:** `/squash-e2e {test-case-id}` → Claude writes Playwright spec from Squash TM → review → commit → runs forever.

*Principle:* **generate once, run forever.**

---

## Parallel Work with Worktrees

- Multiple Claude sessions on same repo = conflicts
- **Solution:** git worktrees — isolated copies, shared history
- Claude Code supports worktree-isolated agents out of the box
- One session refactors auth, another writes tests — in parallel

---

## Ecosystem: Tools Worth Knowing

| Tool | Purpose |
|------|---------|
| [superpowers](https://github.com/obra/superpowers) | Curated skills / hooks / subagents |
| [caveman](https://github.com/JuliusBrussee/caveman) | Prompt compression (~75% fewer tokens) |
| [rtk](https://github.com/rtk-ai/rtk) | Smarter bash execution |
| [claude-mem](https://github.com/thedotmack/claude-mem) | Cross-session memory |
| [graphify](https://github.com/safishamsi/graphify) | Codebase context via graph |

---

## Cheatsheet

**Session control**

| Command | What it does |
|---------|--------------|
| `/context` | Visualize context usage |
| `/clear` | Wipe context, keep session |
| `/compact` | Compress context in place |
| `/btw` | Side question, doesn't pollute history |
| `/export [file]` | Save conversation to file (or clipboard) |
| `Esc` × 2 | Rewind — restores chat + files |

**Resume & branch (CLI)**

| Command | What it does |
|---------|--------------|
| `claude --continue` | Resume last session |
| `claude --resume` | Pick a session from a list |
| `claude --continue --fork-session` | Branch — preserve history, new ID |

**Modes & shortcuts**

| Command | What it does |
|---------|--------------|
| `Shift+Tab` | Cycle permission modes (default → auto-accept → plan) |
| `@filename` | Reference a file inline in your prompt |
| `!<command>` | Run a shell command from the prompt — output lands in chat |
| `/loop 5m /cmd` | Run on interval (`/loop /cmd` = self-paced) |

**Manage**

`/agents` · `/memory` · `/skills` · `/mcp` · `/help`

---

**Further reading:**
- 📘 [Official Anthropic cheatsheet](https://support.claude.com/en/articles/14553413-claude-code-cheatsheet) — concepts + canonical commands
- 📚 [cc.storyfox.cz](https://cc.storyfox.cz/) — community deep reference, auto-updated daily (CLI flags, hooks, env vars)

---

## Habits Worth Building

1. **Don't whip a dead horse.** Wrong fix twice → `Esc` twice, rewind, restart with a better prompt.
2. **Refactor in small, testable increments.**
3. **Innovate relentlessly.** Ask *"Could AI do this?"* — codify patterns into skills.
4. **Keep your setup fresh.** Prune `CLAUDE.md` and skills regularly.

---

## Thank You
