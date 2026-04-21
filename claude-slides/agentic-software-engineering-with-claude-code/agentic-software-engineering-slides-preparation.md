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

- **Session** = one chat + its context window. Fills up → Claude gets dumber.
- **Reset**
  - `/clear` — wipe context, start fresh (do this between unrelated tasks)
  - `/compact` — compress context in place
- **Branch & aside**
  - `--fork-session` — split off to explore without polluting main thread
  - `/btw` — one-off side question, doesn't add to history
  - `claude --continue` — resume last session
- **Rewind:** `Esc` twice — undo Claude's last actions + restore files

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

**When to use:**
- Recurring domain knowledge (e.g., "how to add a Jasper report")
- Multi-step workflows (deploy, generate playwright script from squash test cases)
- Specialized — too narrow for CLAUDE.md

**Why skills beat MCP for most cases:**
- Load **only when invoked** — zero idle cost
- Plain markdown — easy to write, review, version in git

Invocation: auto-trigger (keyword) · manual (`/skill-name`)

---

## MCP: Powerful but Specific

- **MCP** lets Claude call external services safely
- Example: DB queries via whitelisted functions (no raw SQL writes)
- Tool **names** always load; schemas now **deferred** (loaded on use)
- More servers = more upfront context — enable only what you need
- **CLI beats MCP when available** (`gh`, `psql`, `aws`)

---

## Subagents + Hooks

**Subagents** — isolated Claude instances with own context
- Built-in: `Explore` (codebase search), `Plan` (architecture), `general-purpose`
- Custom: own tools, permissions, prompts
- Use for deep research without polluting main chat

**Hooks** — shell commands on tool events
- Auto-format on write, block `rm -rf`, Slack alert on commit
- Configured in `settings.json`

---

## Trust, But Verify

- LLMs non-deterministic. Production must be deterministic.
- **Testing bridges the gap** — unit, E2E, screenshots
- UI flows → **Playwright** (we have `/squash-e2e`)

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

## Habits Worth Building

1. **Don't whip a dead horse.** Wrong fix twice → `Esc` twice, rewind, restart with a better prompt.
2. **Refactor in small, testable increments.**
3. **Innovate relentlessly.** Ask *"Could AI do this?"* — codify patterns into skills.
4. **Keep your setup fresh.** Prune `CLAUDE.md` and skills regularly.

---

## Thank You
