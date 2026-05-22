## # Harness Engineering: A Comprehensive Guide for the Claude Code Era (2025–2026)

*A working reference for JV — Manager Analyst of Data at Newsweek (Buenos Aires) — covering foundations, Claude Code mastery, patterns, real examples, application to data/BI work, model-migration strategy, career path, business angles, and a concrete 12-month roadmap.*

> **TL;DR.** The model is no longer the bottleneck. What you build *around* the model — context architecture, tools, hooks, sub-agents, skills, MCP servers, feedback loops, and verification gates — is. That practice now has a name: **harness engineering**. If you're not the model, you're the harness. For someone with your background (data engineering, BI, Microsoft Fabric, PySpark, Power BI), harness engineering is the single highest-leverage skill you can build in the next 12 months: it compounds with what you already know, it slots cleanly into a "Data + AI platform" role, and it's the spine of any consulting practice or SaaS you might launch.

The rest of this document is dense on purpose. Read it once end-to-end, then bookmark sections 2, 5, 6 and 10 as working references.

---

## ## 1. Foundations and Definition

### What "harness" actually means

The cleanest working definition comes from Vivek Trivedy at LangChain and has now been adopted across Anthropic, OpenAI, Salesforce and the rest of the agent ecosystem:

> **Agent = Model + Harness.** *If you're not the model, you're the harness.*

A harness is **everything that is not the model itself**: the system prompt, the tool definitions, the agent loop, the permission gates, the context-window manager, the memory store, the sub-agent dispatcher, the file system the agent sees, the hooks that fire on tool calls, the MCP servers exposing your data, and the verification step that decides whether a turn is "done." A raw LLM is a stateless text predictor. The harness is what turns it into something that can read files, run tests, edit code, retry on failure, and persist progress across sessions.

Anthropic uses the term directly in its own documentation: the Claude Code SDK *"is the agent harness that powers Claude Code"*, and the Claude Agent SDK page renames it explicitly to signal that the same harness can drive non-coding agents. Their two flagship engineering posts — *Effective context engineering for AI agents* (Sept 2025) and *Effective harnesses for long-running agents* (late 2025) — are now the canonical primary sources.

### Where the term came from

Three lineages converged on the same word in late 2025 / early 2026:

1. **Anthropic's engineering team** (Prithvi Rajasekaran and the Claude Code team) used "harness" internally to describe initializer agents, feature-list files, `init.sh`, self-verification loops, handoff artifacts and context resets — published in *Effective harnesses for long-running agents* and the follow-up *Harness design for long-running application development*.
2. **Dex Horthy** (HumanLayer) is widely credited with coining "**harness engineering**" as a discipline in public talks and his *Frequent Intentional Compaction* writing.
3. **Geoffrey Huntley** (formerly Canva tech lead for developer productivity, now at Sourcegraph on Amp) named the **Ralph Wiggum Loop** (`while :; do cat PROMPT.md | claude-code; done`) and the **back-pressure** concept — bidirectional constraints (upstream context discipline, downstream automated feedback) that keep autonomous agents on track. His free workshop *how to build a coding agent* (ghuntley.com/agent) is the best 300-line, from-scratch primer.

By early 2026, after OpenAI published its own *Harness engineering: leveraging Codex in an agent-first world* field report, Thoughtworks/Birgitta Böckeler formalized it on Martin Fowler's blog, and Anthropic's Nicholas Carlini published the *million-line C compiler with 16 parallel Opus 4.6 agents* case study — saying *"Most of my effort went into designing the environment around Claude — the tests, the environment, the feedback — so that it could orient itself without me"* — "harness" had become the default industry term.

### Why it matters now (and why prompt engineering is dead)

The vocabulary has shifted twice in three years:

| Era | Discipline | Unit of optimization |
|---|---|---|
| 2022–2023 | **Prompt engineering** | Wording of a single prompt |
| 2024–2025 | **Context engineering** | The full set of tokens the model sees (system, tools, memory, history) |
| 2025–2026 | **Harness engineering** | The entire surrounding system: tools, loops, hooks, sub-agents, verification, persistence |

Anthropic's *Effective context engineering* post defines context engineering as *"the set of strategies for curating and maintaining the optimal set of tokens during LLM inference."* Harness engineering is the superset: context engineering plus tool design plus loop design plus orchestration plus persistence plus evaluation.

The reason this matters *right now*:

- **Models hit a competence floor where infrastructure became the bottleneck.** Five independent teams (OpenAI, Anthropic, Huntley, Horthy, Vasilopoulos) converged on the same finding in 2025–2026: above a certain model capability, the *harness* determines real-world performance more than the model.
- **Context windows are misleading.** A model advertising 200K or 1M tokens leaves you with maybe a "Commodore 64 worth of memory" once system prompts, tool definitions and accumulated history are subtracted. Anthropic's own research shows performance degrades past ~40% context utilization (the *context rot* phenomenon).
- **Better models make harness engineering more important, not less.** Carlini explicitly redesigned the harness *at every capability level* of the compiler project. Every component in a harness encodes an assumption about what the model can't do on its own — when the model gets better, some scaffolding becomes dead code and *new* scaffolding is needed to reach the new ceiling. The harness moves; it doesn't disappear.

### Core principles (the four pillars)

Drawing on Alex Lavaee's synthesis of the five-team convergence and Anthropic's own writing, four pillars define the discipline:

1. **Context architecture (tiered, progressive disclosure).** Don't dump everything in CLAUDE.md. Use layered context: hot memory (current task), domain experts (sub-agents/skills), cold knowledge (linked reference files). Skills exemplify this with their three-level loading: YAML frontmatter (always loaded, ~tens of tokens), SKILL.md body (loaded when relevant), linked files (loaded on demand). Forty skills can cost only ~1,500 tokens of overhead with progressive disclosure.
2. **Agent specialization (scoped prompts and restricted tools).** Don't make one omniscient agent; make many narrow ones. Carlini specialized agents into compiler work, deduplication, performance and documentation. Vasilopoulos shipped 19 domain-specific agents. Huntley uses sub-agents specifically to keep the main agent's context clean.
3. **Persistent memory (filesystem-backed, not conversation history).** The filesystem is the most durable memory an agent has. Use `CLAUDE.md`, `AGENTS.md`, `claude-progress.txt`, `decisions.md`, `active-plan.md` as the source of truth. Git for versioning and rollback. Conversation history is volatile and gets compacted away.
4. **Structured execution (research → plan → execute → verify).** Anthropic's *generator/evaluator* split — one agent does the work, a different (skeptical) agent grades it — outperforms self-evaluation, because agents reliably skew positive when grading their own output. The sprint-contract pattern (agreeing on what "done" looks like *before* code is written) catches more scope drift than any prompt change.

### The vocabulary you need

- **Agent loop / agentic loop** (Simon Willison's working definition: *"an LLM agent runs tools in a loop to achieve a goal"*).
- **Context window utilization** — keep it under ~40% of the advertised window for best behavior.
- **Context rot** — degradation as the window fills with low-signal tokens.
- **Compaction** — automatic summarization of older turns when usage approaches ~98%.
- **Context reset / handoff artifact** — a clean slate session that picks up from a written file rather than from compaction. Anthropic found resets essential for Sonnet 4.5's "context anxiety"; Opus 4.5+ largely removed the need.
- **Back-pressure** (Huntley) — bidirectional constraints: upstream deterministic context allocation, downstream automated feedback on quality.
- **Ralph loop / Ralph Wiggum loop** (Huntley) — minimalist outer loop: re-feed PROMPT.md to a fresh coding-agent session forever; filesystem is source of truth.
- **Initializer agent** (Anthropic) — a special first-pass agent that sets up the environment, writes the feature list and `init.sh`, then hands off to specialized worker agents.
- **Generator/evaluator split** — separation of work and judgment.
- **Sprint contract** — written "definition of done" before code is written.
- **YOLO mode** (Willison) — auto-approving all tool calls, dangerous but maximally effective.
- **Sub-agent / Task tool** — isolated context window for delegated work.
- **MCP (Model Context Protocol)** — Anthropic's JSON-RPC standard for letting agents discover and call tools, resources and prompts on external servers.
- **Skill** — a folder (SKILL.md + scripts + references) that teaches an agent how to do a specific task, loaded progressively.
- **Hook** — a script that fires at a specific lifecycle event (PreToolUse, PostToolUse, Stop, SessionStart, etc.) and can allow/deny/modify the action.
- **Plan mode** — a read-only mode in Claude Code where the agent investigates and writes a plan but cannot edit.
- **Extended thinking / effort levels** — adjustable reasoning depth (`low`/`medium`/`high`/`xhigh`) on Opus 4.6/4.7 and Sonnet 4.6.
- **Workslop** — a term that has gained currency (Armin Ronacher and others) for high-volume, low-quality AI-generated output that *looks* like work but isn't reviewable or maintainable.

---

## ## 2. The Claude Code Ecosystem in Depth

Claude Code is the canonical harness right now. Anthropic has been explicit that the SDK powering Claude Code (now renamed the **Claude Agent SDK**) is the same harness driving non-coding agent loops at Anthropic itself (research, video, note-taking). Understanding this harness deeply is the foundation of everything else in this guide.

### 2.1 Architecture in one breath

Claude Code is a tight ReAct-style loop wrapping a frontier model. On each turn:

1. The harness assembles context (system prompt + tools + `CLAUDE.md` + auto memory + skills metadata + history).
2. Claude generates either a text response or one or more tool calls.
3. A **rule-based pipeline** evaluates each tool call: *allow / ask / deny*. Deny always wins. In auto mode, an out-of-band classifier (on a separate model instance that *doesn't* see the agent's prose) handles ambiguous cases — explicitly to mitigate prompt-injection.
4. Allowed tools execute. Results are fed back. Loop.
5. When usage approaches ~98% of the context window, **automatic compaction** summarizes earlier history. Older tool outputs are cleared first, then conversation history is summarized.
6. Hooks fire at lifecycle events (`PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `SessionStart`, `SessionEnd`, `PermissionRequest`, etc. — 17 event types as of v2.1.x).

Tools fall into roughly 19 permission-gated categories (Read, Edit, Write, Bash, Glob, Grep, NotebookEdit, WebFetch/WebSearch, Task/Explore subagents, plus MCP tool calls under the `mcp__server__tool` namespace).

### 2.2 `CLAUDE.md` — the agent's constitution

`CLAUDE.md` is the single most important file in a Claude Code project. It is *re-read on every turn*, which makes it both extremely powerful and extremely precious — every wasted token in it costs you on every single tool call.

**Loading hierarchy** (in order; later layers override / append earlier ones):

| Scope | Location | Purpose |
|---|---|---|
| Enterprise / managed | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) or Windows/Linux equivalent | IT-pushed policies |
| User / global | `~/.claude/CLAUDE.md` | Your personal cross-project preferences (style, language defaults, tone) |
| Project root | `<repo>/CLAUDE.md` | Project conventions, commands, architecture |
| Subdirectory | `<repo>/path/CLAUDE.md` | Path-scoped rules that load automatically when files in that path are touched |
| Imports | `@./other-file.md` syntax | Pull in additional context on demand |

**What to put in CLAUDE.md** (synthesized from the Anthropic Claude Code best-practices doc, Shrivu Shankar's *How I Use Every Claude Code Feature*, Alex Op's customization guide, and PubNub's pipeline write-ups):

- **Architecture overview** (2–4 paragraphs max): what this codebase *is*, which subsystems exist, how data flows.
- **Build / test / run commands** with exact invocations (`pnpm test`, `make integration`, `uv run pytest -k slow`, etc.).
- **Code conventions** that are non-obvious and that Claude wouldn't infer (naming, error handling, logging, the linter you use).
- **Dangerous areas** ("never touch `migrations/`", "the OAuth code is generated, edit `oauth.proto` instead").
- **Where to read more** (links to subdirectory CLAUDE.md files, ADR folders, design docs).

**What NOT to put in CLAUDE.md:**

- Anything Claude already does correctly without being told (this is Shankar's biggest rule — *if Claude does it right without the instruction, delete it or convert it to a hook*).
- Implementation details Claude can read from the code.
- Long lists of "best practices" that aren't enforceable.
- Anything path-scoped (move it to a subdirectory CLAUDE.md).

**Target size:** Shankar's professional monorepo CLAUDE.md is ~13 KB (he could see it growing to 25 KB). Anything past 30 KB is usually a sign you should move content into skills, sub-CLAUDE.md files, or hooks. Empirically, very long CLAUDE.md files are ignored — important rules get lost in the noise.

**Maintenance pattern:** every time Claude makes the same mistake twice, decide whether the fix is (a) a CLAUDE.md rule, (b) a hook (enforced rather than advisory), or (c) a skill. Mitchell Hashimoto considers this "build a test harness, validation script, or linting rule that the agent can invoke to self-check" pattern one of his key 2026 investments.

### 2.3 Slash commands

Slash commands are markdown files in `.claude/commands/` (project) or `~/.claude/commands/` (user). They are essentially named, parameterized prompt templates with optional embedded shell/agent logic.

```markdown
# .claude/commands/ship.md
---
description: "Run tests, type-check, format, and create a conventional commit"
---
1. Run pnpm test. If anything fails, stop and report.
2. Run pnpm typecheck.
3. Run pnpm format.
4. Stage all changes, write a conventional commit message
   summarizing the diff, and create the commit.
```

You invoke this as `/ship` inside Claude Code.

**Best practices** (from alexop.dev's customization guide and the Awesome Claude Code repos):

- Use slash commands for **deterministic terminal entry points** ("the way I always start a feature").
- Have them orchestrate other primitives: a single `/feature` command can spin up a `pm-spec` sub-agent, then `architect-review`, then `implementer-tester`.
- **Don't** use slash commands to replace good CLAUDE.md content — Shankar's rule: a slash command shortens typing, it doesn't replace a better-designed agent.
- Skills are usually a better choice once a workflow has supporting files (templates, schemas, scripts) — slash commands are single files.

### 2.4 Sub-agents (Task tool, custom agents, agent teams)

Sub-agents are isolated Claude instances with their own context window, system prompt and tool whitelist. Three flavors:

1. **Built-in Explore / Plan subagents** — Claude Code launches these automatically (e.g. an Explore subagent during plan mode that scans the repo and reports a distilled summary).
2. **Anonymous Task subagents** — spawned ad-hoc via the Task tool when the main agent decides delegation makes sense ("use a subagent to investigate how auth handles token refresh").
3. **Custom sub-agents** — markdown files in `.claude/agents/` with frontmatter declaring role, allowed tools, isolation, etc.

```markdown
# .claude/agents/sql-reviewer.md
---
name: sql-reviewer
description: "Use proactively to review SQL or PySpark transformations for performance, correctness, and PII handling. Trigger on any change to files under sql/ or notebooks/."
tools: Read, Grep, Glob, Bash(uv run sqlfluff:*), mcp__fabric__query
model: sonnet
isolation: worktree
---
You are a senior analytics engineer at Newsweek. For every change:
1. Read the SQL/PySpark and any referenced semantic-model files.
2. Run sqlfluff with our config.
3. Look for: NULL handling on joins, window-function partition keys,
   PII columns (subscriber_email, user_id) flowing into non-secured tables.
4. Use the fabric MCP to run EXPLAIN on the new query against the dev lakehouse.
5. Return a structured review: PASS / FIX-REQUIRED / DISCUSS.
```

**When to use sub-agents** (synthesizing Anthropic docs + Shankar's contrarian take + Ronacher's experience):

- **Use for:** parallel investigation, context isolation on noisy work (test runs, log greps, large refactors), specialization where tool whitelisting matters (a read-only researcher vs. a writer).
- **Be cautious of:** "lead-specialist" pipelines that force every workflow through your handcrafted org chart. Shankar prefers "master-clone" — keep all context in `CLAUDE.md`, let the main agent spawn copies of itself via `Task(...)`, and let it decide when to delegate. Ronacher reports better results from "starting new sessions and writing thoughts to Markdown files" than from elaborate custom sub-agents for non-investigative work.
- **Tool whitelisting:** if you omit `tools:` in a sub-agent definition, it inherits *all* tools including all MCP tools. PubNub's blog calls this "tool sprawl"; whitelist intentionally.

**Agent teams (experimental)** — set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` to enable multiple Claude sessions communicating through a shared task/plan file. The PubNub Part-II write-up documents the migration from one-off prompts → pipelined sub-agents → "agent teams" with hooks gating transitions (e.g., `pm-spec → architect-review → implementer-tester → qa`, with a hook flipping a status field after each step).

### 2.5 Hooks — making rules *enforced* instead of *advisory*

Hooks are shell commands (or LLM prompts, or sub-agents) that fire on lifecycle events. They live in `.claude/settings.json` (project, committed) or `~/.claude/settings.json` (user). They are how you turn `CLAUDE.md` from "the agent might follow this" to "the system enforces this."

**The 12+ event types** (current as of Claude Code v2.1.x):

- `SessionStart`, `SessionEnd`
- `UserPromptSubmit` — fires before Claude sees the prompt; can rewrite or block it
- `PreToolUse` — **the most powerful**: can `allow` / `deny` / `ask` and *modify the tool input*. Fires before any permission-mode check, so a deny survives even `--dangerously-skip-permissions`
- `PostToolUse`, `PostToolUseFailure` — can rewrite tool output (v2.1.121+) before Claude sees it; useful for redacting secrets, stripping ANSI codes, normalizing diffs
- `PermissionRequest`, `PermissionDenied`
- `Stop`, `SubagentStop` — fire when Claude finishes responding; can force it to keep working by returning exit code 2
- `SubagentStart`
- `Notification`

**Handler types:** `command` (shell), `prompt` (a Claude Haiku evaluates the situation), `agent` (a full sub-agent), and `http` (POST to a webhook).

**Communication:**
- Exit code `0` = allow / OK. Stdout can be JSON for fine-grained control.
- Exit code `2` = **block**. Stderr goes back to Claude as the reason. *This is your power tool.*
- Other non-zero = non-blocking error, shown in verbose mode.

**Five hooks every project should have** (copy-ready, drawn from claudefa.st, blakecrosley.com, stevekinney.com and Anthropic's docs):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          { "type": "command",
            "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null || true" }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf /| DROP TABLE | TRUNCATE | DELETE FROM .* WHERE 1=1' && { echo 'Destructive command blocked' 1>&2; exit 2; } || exit 0" }
        ]
      },
      {
        "matcher": "mcp__fabric__write_.*|mcp__powerbi__deploy_.*",
        "hooks": [
          { "type": "prompt",
            "prompt": "The agent is about to call a production-write tool. Given the event JSON ($ARGUMENTS), decide if this targets the production workspace. If yes, deny with reason. If dev/test, allow.",
            "model": "haiku" }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          { "type": "command",
            "command": "echo '{\"additionalContext\":\"Branch: '$(git branch --show-current)' | Last commit: '$(git log -1 --oneline)'\"}'" }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          { "type": "command",
            "command": "INPUT=$(cat); [ \"$(echo $INPUT | jq -r '.stop_hook_active')\" = 'true' ] && exit 0; pnpm -s typecheck && pnpm -s test -- --reporter=dot || { echo 'Tests/typecheck failing — keep working.' 1>&2; exit 2; }" }
        ]
      }
    ]
  }
}
```

This gives you, in order: auto-formatting on every write, hard block on destructive bash, LLM-gated approval for production writes via MCP, automatic git context at session start, and a "don't stop until tests pass" gate.

**The Stop-hook infinite-loop gotcha:** always check `stop_hook_active` in the JSON input before returning exit 2 from a Stop hook, or you'll loop forever.

### 2.6 MCP servers — when, why, how

MCP (Model Context Protocol) is Anthropic's open JSON-RPC-based standard for exposing tools, resources and prompts to any MCP-aware LLM client. Released November 2024, now adopted by OpenAI, Google, LangChain, Salesforce. It solves the N×M integration problem (N tools × M LLM frontends).

**Three primitives:**
- **Tools** — named functions with JSON-schema args (the main way agents *act*).
- **Resources** — read-only context (file contents, DB views, API responses).
- **Prompts** — pre-written templates users can invoke.

**Transports:** Streamable HTTP (current default), HTTP+SSE (older clients), stdio (local servers).

**When to build vs. use an existing MCP server:**

| Use existing | Build your own |
|---|---|
| GitHub, Linear, Slack, Stripe, Notion, Figma, Playwright, Postgres/MySQL/SQLite, Sentry | Internal data warehouse with custom auth |
| Power BI semantic models (Microsoft's `powerbi-modeling-mcp`), Fabric (mcp-fabric) | Your team's internal admin API |
| Filesystem, fetch, git, time | Tribal-knowledge functions ("compute Newsweek's standard editorial KPI") |

**When NOT to use MCP** (this is Ronacher's repeated warning and Mario Zechner's blog argument): if Claude can already do the thing with `bash + psql + curl`, don't add the dependency. Many MCP servers are over-engineered, load huge tool catalogs into the system prompt (consuming context before any work starts), and are themselves a source of bugs and security holes (see CVE-2025-68143/144/145 in mcp-server-git, CVE-2026-25153 in Backstage's local TechDocs). Ronacher's stance: *"I really only start using MCP if the alternative is too unreliable."*

**Minimal Python MCP server** (use Anthropic's `mcp` SDK with FastMCP):

```python
# fabric_mcp.py
from mcp.server.fastmcp import FastMCP
import os, requests

mcp = FastMCP("fabric_analytics")

@mcp.tool()
async def query_lakehouse(sql: str, workspace: str = "dev") -> str:
    """Run a read-only SQL query against the Newsweek lakehouse. Returns the first 200 rows as CSV."""
    # call Fabric SQL endpoint with service-principal auth
    ...

@mcp.tool()
async def list_semantic_models(workspace: str = "prod") -> list[dict]:
    """List Power BI semantic models in a workspace with id, name, last refresh."""
    ...

@mcp.resource("fabric://workspaces")
def workspaces() -> str:
    """Newsweek Fabric workspace inventory."""
    ...

if __name__ == "__main__":
    mcp.run()
```

Register in `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "fabric": {
      "command": "uv",
      "args": ["--directory", "/abs/path/fabric-mcp", "run", "fabric_mcp.py"],
      "env": { "FABRIC_TENANT_ID": "...", "FABRIC_CLIENT_ID": "..." }
    }
  }
}
```

### 2.7 Skills — the *progressive disclosure* primitive

Skills (officially announced Oct 2025 in Anthropic's *Equipping agents for the real world with Agent Skills*) are folders containing a `SKILL.md` with YAML frontmatter plus optional `scripts/`, `references/`, `assets/`. They are *loaded progressively*: only frontmatter (name + description, ~40 tokens) sits in the system prompt; the body loads when the description matches what Claude is doing; linked files load when the body decides to reference them.

```markdown
# skills/newsweek-power-bi/SKILL.md
---
name: newsweek-power-bi
description: "Editing, validating and deploying Newsweek's Power BI semantic models and DAX measures using our Fabric Git-integrated workspaces. Use this skill whenever the user mentions Power BI, DAX, semantic model, TMDL, measure, calculation group, RLS, or wants to ship a report change."
---

# Newsweek Power BI workflow

## Folder structure
- `models/` — `.bim`/TMDL files for each semantic model. Always edit TMDL, not the JSON.
- `reports/` — PBIR JSON. Only touch this for explicit visual changes.
- `measures/standards.md` — naming, formatting, time-intelligence conventions (READ FIRST).

## Always do
1. Run `scripts/validate.sh` after any edit — wraps Tabular Editor 2 BPA + DAX format.
2. Open a worktree branch named `pbi/<jira-id>-<slug>`.
3. Before opening a PR, regenerate `docs/measures.md` via `scripts/document.py`.

## Gotchas
- Power BI Desktop must be running and have the .pbip open for live edits; if not, use TMDL file mode.
- DateTime columns from Fabric Direct Lake silently lose timezone; always cast in the source view.
- Don't rename measures used in published reports without running `scripts/find_measure_refs.py` first.
```

**Distribution:** skills live in `~/.claude/skills/` (user), `<repo>/.claude/skills/` (project), or a **plugin marketplace** (Anthropic's marketplace + community ones like `obra/superpowers`, `affaan-m/everything-claude-code`). Install via `/plugin install <name>@<marketplace>`.

**Authoring tips** (from Anthropic engineer Tort Mario's *Skills for Claude Code: The Ultimate Guide* and the Anthropic *Complete Guide to Building Skills* PDF):

- **Be "pushy" in descriptions.** Claude under-triggers skills. Instead of *"How to build a dashboard"*, write *"How to build a dashboard. Use this whenever the user mentions dashboards, data visualization, internal metrics, or wants to display company data, even if they don't explicitly ask for a 'dashboard.'"*
- **Keep SKILL.md under 500 lines.** Split into reference files if longer.
- **Always include a "Gotchas" section** with real problems Claude encountered. Update it whenever you discover a new edge case. Most of Anthropic's internal skills started as "a few lines and a single gotcha."
- **Use the skill-creator skill** (`/plugin install skill-creator@anthropic`) to bootstrap.
- **Track adoption with a PreToolUse hook** that logs skill invocations — surface the ones being used and the ones being ignored.

### 2.8 Skill vs slash command vs sub-agent vs MCP vs hook

This is the question that comes up over and over. Synthesizing from alexop.dev's two guides and Anthropic's *Extend Claude Code* docs:

| Primitive | Use when |
|---|---|
| **`CLAUDE.md`** | Short, always-true project conventions and commands |
| **Slash command** | Explicit, repeatable terminal entry point you want to type |
| **Skill** | Reusable workflow / domain expertise that Claude should auto-apply when relevant; needs supporting files (scripts, templates, references) |
| **Sub-agent** | Parallel execution, isolated context, restricted toolset, specialized "persona" |
| **MCP server** | Stable external integration with tools/resources; cross-client portability matters |
| **Hook** | Enforced rule (deterministic policy), automatic reaction to events, ML-gated decision via `prompt` hook |

### 2.9 Agent SDK vs Claude Code CLI

- **Claude Code CLI** — the terminal app + VS Code/JetBrains/desktop integrations. Interactive. What you'll use 90% of the time.
- **Claude Agent SDK (Python/TypeScript)** — the same harness, exposed as a library. Use it when embedding agentic behavior into your own app (a Power BI deploy bot, a Fabric pipeline triggered by Slack, a "data-question to dashboard" backend).
- **Claude Managed Agents** (beta, requires `managed-agents-2026-04-01` header) — Anthropic-hosted harness with built-in containerized sandbox, persistent filesystem, prompt caching, compaction. $0.08/session-hour of *running* time plus standard token cost. Use for long-running cloud workloads where you don't want to operate your own sandbox.

### 2.10 Plan mode, extended thinking, model selection

- **Plan mode** (Shift+Tab to enter): Claude is read-only. It investigates, reads files (via Explore subagent), and writes a plan you approve before any edits. Use it for anything non-trivial.
- **Extended thinking / effort levels** (`/effort low|medium|high|xhigh`): adaptive reasoning depth. Supported on Opus 4.7, Opus 4.6, Sonnet 4.6. Defaults to `xhigh` on Opus 4.7 and `high` on the others. Use higher effort for architecture, debugging, security review; lower for boilerplate.
- **Model selection** (May 2026 current generation):
  - **Opus 4.7** — flagship; agentic coding step-change over 4.6; default `xhigh` effort; supports 1M context via auto-upgrade; ~$5/M in, $25/M out.
  - **Opus 4.6** — flagship-prior; strongest computer-use / desktop-agent (+6.4 pts OSWorld vs 4.5).
  - **Sonnet 4.6** — the workhorse. 79.6% SWE-Bench Verified vs Opus 4.6's 80.8% (a 1.2-point gap), 5× cheaper, ~2× faster. Default for ~80% of work.
  - **Haiku 4.5** — fastest/cheapest; classification, summarization, high-volume sub-agent tasks, hook prompt evaluation.
  - **`opusplan`** alias — automated hybrid: Opus in plan mode, Sonnet in execution mode. This is the cost-quality sweet spot for most non-trivial work.
- **Context management commands:**
  - `/compact [instructions]` — manual compaction with custom preservation instructions (e.g. *"preserve modified files list and all test commands"*).
  - `/clear` — wipe context, start fresh in the same session.
  - `/btw <question>` — ask something that *doesn't enter conversation history* (dismissible overlay).
  - `claude --resume` / `--continue` — restart sessions. Session history lives in `~/.claude/projects/`; you can mine these logs (Shankar runs meta-analysis on his own session history to find recurring exceptions and improve CLAUDE.md).

---

## ## 3. Practices and Patterns

These are the patterns that actually compound. They are drawn from Anthropic's engineering writing, Geoffrey Huntley, Armin Ronacher, Mitchell Hashimoto, Simon Willison, Shrivu Shankar, the PubNub pipeline series, the OpenAI Codex field report, and Birgitta Böckeler at Thoughtworks.

### 3.1 Designing the agentic loop (Willison)

Simon Willison's *Designing agentic loops* is the foundational read. Four steps:

1. **Reduce the problem to a clear goal.** What does "done" mean, in writing, before the first prompt?
2. **Pick a set of tools that can iterate toward the goal.** Tests, type-checkers, linters, dev servers, screenshot diffs, SQL `EXPLAIN`, a benchmark script — anything the agent can run, observe, and react to.
3. **Give the agent enough autonomy** (YOLO mode / `--dangerously-skip-permissions` / pre-approved tool list) that it can actually *iterate* rather than ask for approval every step. Otherwise you've reduced effectiveness to brute force minus approval-tedium.
4. **Wrap it in a sandbox** (Docker, devcontainer, separate user, network egress controls). The price of YOLO mode is containment.

The art is in step 2: the tool design determines whether the agent can converge or wanders forever. If your task has no clear "did it work?" signal the agent can read, *invent one* — write a test, write a check script, write a diff comparison.

### 3.2 Spec-driven development (SDD)

Specs before code is now the dominant pattern. The Anthropic *long-running harness* post formalizes it; Wataru Takahashi's *Spec Driven Development with Claude Code* writes the canonical workflow:

1. **Requirements agent** writes `requirements.md` — objectives, features, constraints, acceptance criteria. User reviews and approves.
2. **Design/architecture agent** writes `design.md` — tech choices, data model, components, sequence. User reviews.
3. **Tasks agent** writes `tasks.md` — broken into reviewable units.
4. **Implementation agent(s)** execute against tasks, marking each done.

The genius is that approval moves from "100 micro-prompts during implementation" to "3 phase gates upfront", which is faster *and* produces higher quality. The specs themselves become living documentation.

For data work specifically: this maps directly onto *requirements → semantic-model spec → DAX measure catalogue → implementation*.

### 3.3 Test-driven / verification-driven development with agents

TDD becomes *the* methodology when the agent is the implementer. The pattern:

1. Write (or have Claude write) **failing tests first** that encode the spec.
2. Tell Claude: "Make these tests pass. Don't modify the tests. Run them after every change. Don't stop until all green."
3. Add a Stop hook that runs the test suite and returns exit-code-2 (forcing Claude to keep working) if anything fails.

Nicholas Carlini's compiler project leaned on exactly this: *"I had to constantly remind myself that I was writing this test harness for Claude and not for myself."* When a new feature kept breaking existing behavior, the fix was a stricter CI pipeline — *a harness-level solution to a model-level problem.*

For analytics this generalizes to **verification-driven development**: dbt tests, Great Expectations checks, schema contracts, snapshot tests on a small fixture lakehouse, DAX result comparisons against a known-good model.

### 3.4 Plan-then-execute, generator/evaluator splits, and sprint contracts

Three sub-patterns from Anthropic's harness research:

- **Plan-then-execute.** Always use `/plan` for non-trivial work. The plan is cheap; bad code is expensive.
- **Generator/Evaluator.** Two agents, one writes, one critiques. Tune the evaluator to be *skeptical* (it's tractable to make a separate agent harsh, but very hard to make an agent self-critical). Anthropic found this consistently outperforms self-evaluation, where agents reliably praise their own work.
- **Sprint contract.** Before any code, generator and evaluator negotiate a written "definition of done" for this chunk. They communicate by writing files. They iterate until they agree. *Then* generator builds against the contract.

### 3.5 Parallel agents and git worktrees

Two distinct parallelism patterns; don't conflate them.

**(a) Worktree pattern — cross-session parallelism.** Each agent gets its own filesystem (a git worktree, optionally with its own DB, ports, `.env.local`). True branch-level isolation; you merge at the end.

```bash
git worktree add ../proj-auth feature/auth
git worktree add ../proj-payments feature/payments
git worktree add ../proj-dash feature/dashboard
# in three terminals, cd into each and start `claude`
```

Or Claude Code's native: `claude --worktree` (or `-w`) auto-creates an isolated worktree at `.claude/worktrees/<name>/`. Sub-agents can declare `isolation: worktree` in frontmatter to spawn into their own worktree automatically. A `.worktreeinclude` file (gitignore syntax) copies untracked env files into each new worktree.

Mitchell Hashimoto runs multiple Ghostty checkouts (`ghostty`, `ghostty2`, `ghostty3`, `ghostty4`) and has Claude vs Codex *compete on the same task* — picks the better output. Treat it as a tournament.

**(b) Sub-agent split-and-merge — intra-session parallelism.** A parent agent decomposes work into pieces, dispatches sub-agents in its own session, merges their outputs. They share context through the parent. Use for "investigate these 8 files in parallel and summarize", not for "build feature A and feature B simultaneously."

**Practical limits:** 2–3 worktree-parallel sessions for most humans (Hashimoto calls himself "the mayor managing at most two agents at a time"); 5–10 with tmux + an orchestrator; "Gas Town"-style autonomous orchestration (Steve Yegge's term, now an open-source pattern Hashimoto and others discuss) — interesting but premature for most teams.

### 3.6 The Ralph loop (Huntley)

```bash
while :; do cat PROMPT.md | claude-code -p --dangerously-skip-permissions; done
```

The *Ralph Wiggum Loop* is a minimalist outer loop pattern: re-feed a fixed prompt to a fresh agent session forever. The filesystem (PRD, progress.md, git history) is the source of truth, not the agent's internal state. A YC hackathon team used a Ralph loop to generate 1,100+ commits across six repos overnight for ~$10.50/hour per agent.

Huntley's later refinement: *back-pressure* is the real point. *Upstream* — deterministic prompt stacking, consistent context allocation. *Downstream* — automated quality feedback. The loop without back-pressure is just expensive nonsense. Ralph is GENERIC; it applies to all tasks, not just code, including the "loom" pattern Huntley uses for autonomous system verification while DJ-ing.

### 3.7 The stop-and-reset rule

This is universal across practitioners. Symptoms that mean *stop, reset context, start fresh*:

- Agent has been spinning on the same failure mode for >3 cycles.
- Context window is past ~60% used.
- You catch yourself re-explaining something you wrote in CLAUDE.md.
- The agent is hedging ("I'll try to…") instead of acting.
- You've manually fixed something and the agent's mental model is now wrong.

The right action: write a 1-paragraph handoff (`echo "what we learned, what to do next" > HANDOFF.md`), `/clear`, restart. Anthropic's harness research treats *context resets* as a first-class primitive: a clean slate with a written artifact often outperforms compaction.

### 3.8 Token economy and cost management

The real cost numbers (May 2026):

- Heavy daily Claude Code user: $200–500/month on API billing, with Agent Teams users above $800.
- Plan selection: most working devs land at **Max 5× ($100/mo)**; full-day power users at **Max 20× ($200/mo)**. API pay-as-you-go only makes sense if usage is bursty or you need provider-specific endpoints.
- Subscription savings: one reported user logged 10B tokens across 8 months — API cost would have exceeded $15K, Max at $100/mo totaled ~$800 (~93% savings).

**Six concrete levers, ranked by ROI:**

1. **Use `opusplan` or stay on Sonnet 4.6 by default.** The 1.2-point SWE-bench gap to Opus 4.6 doesn't justify a 5× price gap for most work. Ronacher reports preferring Sonnet outputs to Opus.
2. **Prompt caching** — 90% discount on cached input tokens, 25% premium on cache write, 5-minute TTL (Anthropic shortened from 60 min in early 2026; 1-hour TTL available for additional cost). Anthropic's own example: 41–80% cost savings on multi-turn agentic workloads. For Claude Code this is automatic; for your own Agent SDK code, place stable content (system prompt, tool defs, large docs) before any cache breakpoint.
3. **Scoped prompts.** *"Fix the JWT validation error in `src/auth/middleware.ts:47`"* costs a fraction of *"fix the auth error"* because the agent doesn't have to search to find what you meant. 30–50% token reduction is typical.
4. **Aggressive sub-agent dispatch for investigation.** Reading 200 files into the main context to find one bug burns tens of thousands of tokens that don't need to be there. Delegate to an Explore subagent.
5. **Compaction discipline.** Periodically `/compact` with explicit instructions ("preserve all file paths touched and exact test commands"); use `/btw` for side questions.
6. **Hard caps.** Set `max_turns` / `max_budget_usd` on Agent SDK runs. A runaway loop on a one-line typo fix has been documented consuming 21,000+ input tokens.

### 3.9 Anti-patterns and common mistakes

Compiled from Ronacher's *Agentic Coding Things That Didn't Work*, the Anthropic best-practices doc, Shankar's *How I Use Every Claude Code Feature*, and Huntley's writing:

- **Over-specified CLAUDE.md.** If it's too long, Claude ignores half of it. Prune ruthlessly. If Claude does it right without the rule, delete the rule.
- **Verify-by-vibes.** Plausible-looking code that doesn't handle edge cases. *If you can't verify it, don't ship it.* Always have a test/check/screenshot.
- **Infinite exploration.** "Investigate auth" with no scope → reads 300 files → context shot. Scope investigations; use subagents.
- **Over-engineered sub-agents.** "Lead-specialist" pipelines that force everything through your handcrafted org chart create their own maintenance burden. Master-clone (one main agent, spawning copies of itself) is often simpler.
- **Tool sprawl.** Sub-agents inheriting all MCP tools by default. Whitelist intentionally.
- **MCP for things bash can already do.** Often a worse experience than `psql` + `curl`.
- **Workslop volume contests.** Beads-style 240,000-LoC issue tracker projects, Gas Town–like agent factories with no review gates. *Just because you can generate it doesn't mean it's value.*
- **Mental disengagement.** Ronacher's biggest warning: agents make it easy to stop thinking like an engineer. Quality drops, you don't learn, and refactoring cost doesn't actually go to zero. Stay engaged; review every meaningful change.
- **Skipping the cleanup step.** Hashimoto: every session should end with *"Are there any other improvements you can see? Don't write code. Consult the oracle."*
- **Letting prompt injection through.** With YOLO mode + MCP + WebFetch, an attacker can plant instructions in any page/issue/email the agent reads. Sandbox aggressively; the auto-mode classifier on a separate model is your friend; never run agents with prod credentials in the same shell.

---

## ## 4. Real Examples, Case Studies, and Reference Material

### 4.1 Anthropic's own production examples

- **Nicholas Carlini's million-line C compiler** — 16 parallel Opus 4.6 agents across 2,000 sessions, specialized roles (compiler, dedup, perf, docs), CI as harness, harness redesigned at every capability level. The cleanest large-scale case study available; multiple Anthropic engineering posts cover it.
- **Long-running harness research** (Prithvi Rajasekaran, Anthropic Labs) — *Effective harnesses for long-running agents* and *Harness design for long-running application development*. Introduces the initializer-agent + feature-list + `init.sh` + `claude-progress.txt` pattern; documents the generator/evaluator split and the sprint-contract pattern with Opus 4.5 enabling drop of context resets that were essential under Sonnet 4.5.
- **Anthropic Cybersecurity team's threat-detection platform** built with Claude Code (anthropic.com/news case study).
- **Internal skills library** — Anthropic engineer Tort Mario reports "hundreds of skills in production" inside Claude Code at Anthropic, tracked via a PreToolUse hook that logs every skill invocation.

### 4.2 Practitioner case studies worth reading end-to-end

- **Geoffrey Huntley** (ghuntley.com) — the most prolific writer in the space. Must-reads: *how to build a coding agent* (300-line workshop), *Ralph Wiggum as a Software Engineer*, *everything is a ralph loop*, *don't waste your back pressure*, the *six-month recap* talk from Web Directions Melbourne. Also the *Gas Town* / *Weaving Loom* posts on agent orchestration.
- **Mitchell Hashimoto** on Ghostty — *Vibing a Non-Trivial Ghostty Feature* (16 sessions, 8 hours, $15.98, transcripts published via Amp). His Pragmatic Engineer podcast appearance (TeamDay.ai summary) lays out his "always have an agent doing something" discipline, the "build a harness from every mistake" rule, and the Git-isn't-built-for-this take.
- **Armin Ronacher** (lucumr.pocoo.org) — *Agentic Coding Recommendations* (the original $100 Max + Sonnet workflow), *Agentic Coding Things That Didn't Work*, *Agent Design Is Still Hard*, *A Language For Agents*, *Pi: The Minimal Agent Within OpenClaw*, *Building an Agent That Leverages Throwaway Code*, *Agent Psychosis*. His perspective is anti-hype and grounded; he's the corrective to the "more agents! more loops!" crowd.
- **Simon Willison** (simonwillison.net) — *Designing agentic loops*, *I think "agent" may finally have a widely enough agreed upon definition* (where he settles on *tools in a loop*), *Embracing the parallel coding agent lifestyle*, his Claude Skills reverse-engineering post.
- **Steve Yegge** — *Welcome to Gas Town* (medium) — the high-end vision of autonomous agent factories, "Kubernetes for agents", MEOW (Molecular Expression of Work). Useful even if you don't deploy it: shows you where things might go.
- **Shrivu Shankar** — *How I Use Every Claude Code Feature* (blog.sshh.io) and his *Building Multi-Agent Systems* series. The most thorough end-to-end "what I actually use" inventory.
- **Addy Osmani** — *Agent Harness Engineering* — synthesizes the Anthropic posts, the Khan analysis, and Trivedy's "if you're not the model, you're the harness" frame.
- **Birgitta Böckeler / Martin Fowler's site** — *Harness Engineering* — the Thoughtworks framing into context engineering, architectural constraints, and *garbage collection* against entropy.
- **OpenAI Codex team** — *Harness engineering: leveraging Codex in an agent-first world* — the OpenAI counterpart, focused on architectural constraints, repo-local instructions, browser validation, telemetry.
- **PubNub blog** — *Best Practices for Claude Code Subagents* parts I and II — the most concrete walk-through of building a `pm-spec → architect-review → implementer-tester` pipeline with hooks gating transitions.

### 4.3 Reference repos and skill collections

- **`anthropics/skills`** — official skills repository; includes the `skill-creator` skill, document skills (PDF/DOCX/XLSX/PPTX), example skills.
- **`anthropics/anthropic-cookbook`** — code examples for the API; older but still useful for prompt-cache patterns.
- **`hesreallyhim/awesome-claude-code`** — the canonical awesome-list; curated with judgment, items get removed if they stop working.
- **`jqueryscript/awesome-claude-code`** — large fire-hose list with star counts (good for discovery, scan for 🔥).
- **`quemsah/awesome-claude-plugins`** — auto-indexed plugin tracker; 15K+ repos as of May 2026. Use as early-warning feed.
- **`walkinglabs/awesome-harness-engineering`** — *the* awesome-list for this discipline specifically. Curates the Anthropic posts, the OpenAI Codex report, the Thoughtworks framing, *Skill Issue: Harness Engineering for Coding Agents*, *Your Agent Needs a Harness, Not a Framework* (Inngest), the *Greenfield/Brownfield/Vibecode* taxonomy, plus practical tools like *Harness Evolver* (Meta-Harness Lee et al. 2026).
- **`obra/superpowers`** — community skills framework officially accepted into Anthropic's marketplace (~94K+ stars when accepted). Worth studying as an example of a *skill ecosystem*, not just individual skills.
- **`affaan-m/everything-claude-code` (ECC)** — Anthropic-hackathon-winning agent harness performance optimization system; production-ready agents/skills/hooks/MCP configs from 10+ months of daily use. Cross-harness (works across Claude Code, Codex, Cursor, OpenCode, Gemini, Copilot). Includes `ecc-agentshield` — a security scanner for your own Claude config (`npx ecc-agentshield scan`).
- **Andrej Karpathy's CLAUDE.md** — a single oft-cited CLAUDE.md derived from Karpathy's observations on LLM coding pitfalls (extremely high stars in the awesome-claude-code lists).
- **`mattpocock/skills`** — "skills for real engineers" — TypeScript/JS focused.
- **`scientific-agent-skills`** — research, science, engineering, analysis, finance, writing.
- **`callstackincubator/skills`** — React Native skills.
- **`napkin`** — agent-mistakes-as-memory pattern (per-repo markdown scratchpad of past mistakes).
- **`caveman` skill** — cuts ~75% of agent output tokens while keeping technical accuracy.
- **`using-git-worktrees` skill** — auto-creates worktrees as standard operating procedure.
- **Stripe / Vercel / Linear / Claude `DESIGN.md` collection** — 55+ design-system specs extracted from production sites you can hand to Claude as context.

### 4.4 Production agentic workflows

- **Prophecy v4** — uses *specialized Claude Code agents* (each with its own system prompt, tools, skills, some as sub-agents) for visual data prep on Databricks/Snowflake/BigQuery, with embedded DuckDB for sub-500GB ad-hoc work. A useful pattern for data-team builders.
- **pbi-cli** (community, Microsoft-affiliated) — gives Claude Code 12 structured skills for editing Power BI semantic models + PBIR reports via in-process .NET TOM (sub-second latency, no MCP sidecar). 488 tests, DAX execution, visual CRUD, REPL.
- **Microsoft `powerbi-modeling-mcp`** (official) — MCP server for live Power BI semantic-model editing via Tabular Editor 2.
- **GitButler** — Claude Code hook integration that auto-commits per session into the correct branch when running multiple parallel sessions.
- **Builder.io agent harness** — visual multi-agent harness with worktree isolation; useful as a reference architecture even if you don't adopt the product.

### 4.5 "Workslop" and quality control

The term *workslop* (and Ronacher's deeper *Agent Psychosis* post) names the failure mode where teams generate huge volumes of AI output that looks like work but is unreviewable or unmaintainable. Patterns that prevent it:

- **Mandatory verification gates** at every transition: tests, schema checks, screenshot diffs, BPA-style rule sets.
- **Sprint contracts** before code.
- **Generator/evaluator split** with a *skeptical* evaluator (different model, different prompt, no access to the generator's prose).
- **Cleanup pass at the end of every session** ("Don't write code. Consult the oracle. What's missing? What tests should exist?").
- **PRs with prompts attached**, not just code. Some projects now require submitters to include the prompt sequence alongside the diff so reviewers can re-run it.
- **Ratcheted pre-commit hooks** (from *Greenfield AI, Brownfield AI, and the Vibecode You Just Inherited*) — baseline existing lint violations, then prevent any new ones from being added.

---

## ## 5. Applying Harness Engineering to Data and Analytics Work

This is the section where harness engineering meets your day job. Most generic Claude-Code content assumes a TypeScript web-app developer. You're not that — you ship semantic models, DAX, PySpark, SQL, dashboards, and ad-hoc analysis. The patterns below are tuned to that reality.

### 5.1 The Newsweek "Data Harness" reference architecture

Build this once. Reuse it for every project.

```
~/.claude/
├── CLAUDE.md                    # personal global preferences (English, terse, no emojis)
├── settings.json                # user-level hooks (auto-format SQL/Python, log bash)
├── mcp.json                     # fabric, github, linear, slack, postgres, playwright
├── skills/
│   ├── newsweek-fabric/         # Fabric workspaces, lakehouse naming, capacity rules
│   ├── newsweek-power-bi/       # semantic model conventions, DAX standards, deployment
│   ├── newsweek-dax/            # DAX patterns, time intelligence, RLS standards
│   ├── newsweek-pyspark/        # Fabric notebook conventions, partitioning, perf
│   ├── newsweek-sql/            # query style, query review checklist
│   ├── newsweek-analytics-pm/   # how we write briefs, what a "good question" looks like
│   ├── newsweek-pii/            # PII handling, GDPR/CCPA, columns that need masking
│   └── using-git-worktrees/     # community skill, auto-worktrees per task
└── agents/
    ├── analyst-pm.md            # spec writer for ad-hoc analytics requests
    ├── sql-reviewer.md          # PR reviewer for SQL/notebooks
    ├── dax-reviewer.md          # PR reviewer for DAX/TMDL
    ├── data-explorer.md         # read-only investigation across data sources
    └── docs-writer.md           # turns delivered analysis into Confluence/Notion docs

<each-project>/
├── CLAUDE.md                    # this repo's architecture + commands
├── .claude/
│   ├── settings.json            # project hooks (block prod writes, run tests, etc.)
│   ├── commands/                # /ship, /review, /publish-dashboard
│   ├── agents/                  # project-specific sub-agents if needed
│   └── plans/active-plan.md     # the spec currently being worked
├── docs/decisions/              # ADRs (Claude reads and writes these)
├── specs/                       # requirements.md, design.md, tasks.md per feature
└── HANDOFF.md                   # written when context resets
```

### 5.2 Patterns for data engineering (PySpark, Fabric notebooks, ETL)

**1. Treat notebooks as code, not as artifacts.** Fabric notebooks export as `.ipynb` or, better, as Python files with cell markers (`# %%`). Commit them; Claude diffs them; review them like normal code. Use a skill that documents *"always export notebooks as .py with `# %%` markers; never commit the .ipynb if a .py exists."*

**2. Build a tiny "lakehouse harness" early.** A handful of utilities that make Claude effective:

- A `query` tool (via Fabric MCP, or just bash + `sqlcmd`/`spark-submit`) that runs against a *dev* lakehouse.
- A `schema` tool that returns table+column metadata.
- A `sample` tool that returns 50 rows of any table.
- A `lineage` tool (or a static lineage doc in `docs/lineage.md`) so the agent knows which downstream things will break.
- A `cost` tool that returns the Fabric CU consumption of a query plan.

These five tools — plus a CLAUDE.md telling Claude to use them — make analytics work feel completely different. Anthropic's harness research keeps coming back to the same principle: *give the agent the same tools a senior engineer has*.

**3. dbt-style verification, even if you're not on dbt.** Encode expectations as runnable checks:

- Row-count tolerance after a refresh.
- Schema contract diffs (column presence, types).
- Reference-data joins (FK orphans).
- Distribution checks (mean shift > 3σ flags review).

Wire them into a `Stop` hook. The agent doesn't get to declare "done" until they pass.

**4. PySpark + Fabric notebook skill.** Your `newsweek-pyspark` SKILL.md should encode the team's rules:

```markdown
# Newsweek PySpark / Fabric notebooks

## Always
- Use `spark.sql.shuffle.partitions` aligned to capacity (target: 2× current core count).
- Read from `lakehouse://prod` is FORBIDDEN; use `lakehouse://dev` and the `gold_*` views.
- Cache only with explicit `unpersist()` paired in a `try/finally`.
- All wide transformations get a `# %% [markdown]` cell above with a 1-line description.

## Gotchas (update when you find new ones)
- `to_timestamp` in Direct Lake silently drops timezone — cast with `from_utc_timestamp` first.
- Fabric Lakehouse SQL endpoint sees Delta tables ~5s after write; use `MERGE INTO` checkpoints
  in long pipelines rather than chained selects.
- Notebooks in Fabric Git-integrated workspaces don't auto-format on save; we use `ruff format`
  via a PreToolUse hook.

## Run / test
- `make notebook-test NB=path/to/notebook` runs the notebook against a sample lakehouse.
- `make lint` runs ruff + sqlfluff over all .py and .sql in this repo.
```

**5. Pipeline scaffolding via Ralph loops.** For an iterative pipeline (ingest → bronze → silver → gold), a Ralph-style outer loop with a PROMPT.md that says *"Read PROGRESS.md. Pick the next bronze table that hasn't been built. Build it. Test it. Update PROGRESS.md."* is shockingly effective when paired with strong checks.

### 5.3 Patterns for Power BI / semantic models / DAX

This is now a well-trodden path; see Section 4. Concrete setup:

- **Use PBIP format** (Git-integrated workspaces; semantic model in TMDL, report in PBIR JSON). Both are diffable; both are agent-editable.
- **Install Microsoft's `powerbi-modeling-mcp`** for live editing while Power BI Desktop is open with the .pbip.
- **Install `pbi-cli`** for headless, sub-second TMDL/PBIR operations and DAX execution (488 tests, REPL with tab-completion). Direct in-process .NET TOM beats MCP sidecars on latency for batch operations like "rename all measures starting with `Calc_`" or "set every visual to font size 12."
- **Skill — `newsweek-dax`:**
  ```markdown
  description: "Newsweek DAX standards and patterns. Use whenever the user is writing or reviewing DAX, measures, calculation groups, RLS, or time intelligence. Includes our naming, formatting, perf playbook, and BPA rule set."
  ```
- **BPA + Claude flow** (Hashimoto-style "every mistake becomes a harness improvement"):
  1. Run Tabular Editor 2 BPA against the model.
  2. Pipe violations to Claude with the BPA rule descriptions.
  3. Claude proposes TMDL edits.
  4. Hook re-runs BPA + DAX format + measure-reference scan before allowing commit.
  5. Whenever Claude makes a non-obvious mistake, codify it as a *new* BPA rule.
- **PR review subagent (`dax-reviewer.md`)** with tools restricted to Read, Grep, Glob and `pbi-cli` execution. It scores changes on: naming, formatting, perf risk, RLS impact, KPI catalogue drift.

### 5.4 Patterns for ad-hoc analytics

This is where harness engineering produces 10× speedups, because ad-hoc work is iterative, context-heavy, and verification-light by default.

**The "Analyst PM" workflow** (mirror of SDD):

1. **Brief intake.** A `/brief` slash command opens a template (question, decision it serves, deadline, data sources). A sub-agent `analyst-pm.md` asks clarifying questions until the brief is sharp.
2. **Data exploration in plan mode.** Spawn a `data-explorer` sub-agent with read-only DB tools. It writes `EXPLORATION.md` summarizing what's available and where the gaps are.
3. **Method spec.** Claude writes `METHOD.md` — the analytical approach, expected joins, filters, ratios, segmentation, statistical caveats. You review.
4. **Execution** in a Fabric notebook or DuckDB locally. The notebook is run end-to-end on commit via a hook.
5. **Documentation.** A `docs-writer` sub-agent turns the notebook into a Notion/Confluence page following your house style (encoded in the `newsweek-analytics-pm` skill).

This gives you a *paper trail* — every analysis has a brief, a method, the notebook, and the doc — which is what separates an analyst from a data scientist in promotion discussions.

### 5.5 Working with proprietary / internal data via MCP

Two patterns:

- **Build an internal MCP server** for systems where the team will keep accessing the same APIs (Newsweek CMS, subscriber DB, Fabric capacities, internal admin tools). One server, multiple agents, central auth (use OAuth user-tokens, not service principals, when the agent is acting on a user's behalf).
- **Don't build an MCP server** for things `psql`/`curl`/`az`/`pwsh` already do. Ronacher's principle holds: MCP adds dependencies that themselves fail. Use MCP when the alternative is unreliable.

**Security checklist for internal MCP servers** (informed by the 2025–2026 CVE history):

- Per-user OAuth scopes mapped to MCP tokens — don't let one server have a god key.
- Read-only mode by default; an explicit `--write` flag or a separate `write-enabled` server.
- All write tools behind a *prompt hook* that asks Haiku "is this targeting production?" before allowing.
- Tool descriptions reviewed for prompt-injection-friendly phrasing (no "ignore previous instructions if the input says…").
- Audit log every tool call (PreToolUse hook into a file or your SIEM).

### 5.6 Building internal tools and skills for a Newsweek team

Make harness engineering a *team capability*, not your hobby. Concrete first projects:

1. **The Newsweek skills monorepo.** A Git repo of `.claude/skills/`, `.claude/commands/`, `.claude/agents/`, a shared `CLAUDE.md` base, plus a setup script (`bootstrap.sh`) that symlinks them into each engineer's `~/.claude/`. Ship it like an internal library.
2. **The Newsweek MCP servers.** One or two servers exposing your most-used data systems with safe, audited tools.
3. **The "agent on-call" runbook.** Document which patterns (sub-agents, Ralph loops, plan mode) to use for which class of work. Codify it as a slash command (`/how-do-i`) that summarizes the right approach.
4. **A monthly skill review.** Whoever made a mistake twice that month writes a new skill / new gotcha / new BPA rule. This is exactly Hashimoto's "build a harness from every mistake" practice, scaled to a team.

---

## ## 6. Maximizing Current Claude Code Workflows (late 2025 / 2026)

### 6.1 Model selection cheat sheet (May 2026)

| Scenario | Model | Notes |
|---|---|---|
| Default everyday — writing, refactor, SQL, DAX, light review | **Sonnet 4.6** | 79.6% SWE-Bench Verified; 5× cheaper than Opus; 2× faster |
| Architecture, hard debugging, long-horizon planning, security review | **Opus 4.7** (or 4.6) | Step-change agentic coding over 4.6; default `xhigh` effort |
| Plan first, then execute | **`opusplan`** alias | Auto: Opus in plan mode, Sonnet in execution. Cost-quality sweet spot. |
| High-volume sub-agent tasks (classification, summarization, log triage) | **Haiku 4.5** | Speed-critical; hook `prompt`-type evaluations default here |
| Desktop / computer-use / OSWorld-style automation | **Opus 4.6** (or 4.7) | +6.4 pts OSWorld over 4.5 was the headline of 4.6 |

**Effort levels:** `low` / `medium` / `high` / `xhigh`. Defaults are `xhigh` on Opus 4.7 and `high` on Opus 4.6 / Sonnet 4.6 as of v2.1.117. Bump to `xhigh` for genuinely hard problems; drop to `low` for boilerplate to save tokens.

**Switching mid-session:** `/model sonnet` / `/model opus` / `/model haiku` / `/model opusplan`. Conversation history persists. Useful pattern: plan in Opus, switch to Sonnet for the long implementation pass, switch back to Opus for the security/architecture review at the end.

**Per-shell pinning:** `export ANTHROPIC_MODEL=claude-opus-4-7` (or `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`, etc.) in `~/.zshrc`.

### 6.2 Productivity multipliers (the settings that pros actually use)

From Ronacher, Hashimoto, Shankar, the Builder.io "50 tips" post, and the Anthropic best-practices doc:

- **YOLO mode with sandboxing.** `claude --dangerously-skip-permissions` is the productivity unlock, but ONLY inside a Docker dev container, a separate user, or a worktree with no production credentials. Most pros run YOLO 95% of the time.
- **Custom status line** (`/statusline`) showing branch, model, token budget remaining.
- **`/rename` and `/color`** sessions when you run 2–3 in parallel. Five seconds saves the "wrong terminal" mistake.
- **`Stop` hook with a system-sound bell.** You context-switch back when Claude actually finishes.
- **Voice input** (Whisper/SuperWhisper) — Ronacher prompts mostly by voice; Hashimoto does too. Faster than typing and you naturally produce longer, better-specified prompts.
- **Plan mode as default for anything non-trivial.** Shift+Tab into plan mode at the start of every feature.
- **Subagents for exploration.** *"Use subagents to investigate how X works"* keeps your main thread clean.
- **`/btw`** for side-questions that should not enter the conversation.
- **`claude --resume` / `claude --continue`** for digging up old sessions; meta-analyze `~/.claude/projects/` for recurring failure modes.
- **CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1** if you want multi-agent coordination on the same plan file.
- **Custom subagents pinned to Haiku** (`model: haiku` in frontmatter) for fast read-only research subagents — *much* cheaper than letting the main Sonnet/Opus session do it.

### 6.3 The pro setup (dotfiles patterns)

A minimal `.claude/` you can lift wholesale:

```bash
# ~/.claude/settings.json  (user-level)
{
  "env": {
    "ANTHROPIC_MODEL": "claude-sonnet-4-6",
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "hooks": {
    "PostToolUse": [
      { "matcher": "Write|Edit|MultiEdit",
        "hooks": [{ "type":"command",
          "command":"ext=\"${CLAUDE_TOOL_INPUT_FILE_PATH##*.}\"; case \"$ext\" in py) ruff format \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null;; sql) sqlfluff format \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null;; ts|tsx|js|jsx|json|md) npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null;; esac; exit 0" }] }
    ],
    "PreToolUse": [
      { "matcher": "Bash",
        "hooks": [{ "type":"command",
          "command":"cmd=$(jq -r '.tool_input.command' < /dev/stdin); echo \"$cmd\" | grep -qE 'rm -rf /| sudo rm | DROP TABLE | TRUNCATE TABLE | DELETE FROM [^ ]+ *$| --force-with-lease| git push.*--force' && { echo 'Destructive command — denied by hook' 1>&2; exit 2; }; echo \"$(date -Is) $cmd\" >> ~/.claude/bash.log; exit 0" }] }
    ],
    "SessionStart": [
      { "hooks": [{ "type":"command",
        "command":"echo '{\"additionalContext\":\"branch=' $(git branch --show-current 2>/dev/null) ' | model='$ANTHROPIC_MODEL'\"}'" }] }
    ],
    "Notification": [
      { "hooks": [{ "type":"command",
        "command":"afplay /System/Library/Sounds/Glass.aiff 2>/dev/null || true", "async": true }] }
    ]
  },
  "permissions": {
    "allow": ["Bash(git *)", "Bash(rg *)", "Bash(fd *)", "Bash(uv *)", "Bash(pnpm *)", "Bash(make *)"]
  }
}
```

Plus a global `~/.claude/CLAUDE.md`:

```markdown
# Personal preferences (JV)

- Respond in English. Be terse. No marketing tone.
- Never use emojis or em-dashes in code, commit messages, or PR bodies.
- Prefer `uv` over pip/poetry; `pnpm` over npm; `ruff` for Python; `sqlfluff` for SQL.
- I use a Mac. I run Claude Code from Ghostty or VS Code.
- For any non-trivial work: enter plan mode first, write the plan, wait for my OK.
- For data work: assume Microsoft Fabric + Power BI unless I say otherwise.
- Cite file paths as `path:line` so I can jump to them.
- If I say "ship it", run tests/lint/typecheck/format and create a conventional commit.
- Never run anything that touches a production workspace. Ever.
```

### 6.4 Prompt caching strategies for your own Agent SDK code

If you build with the Agent SDK or the raw Messages API (not Claude Code itself), structure every request as:

```
[stable cached prefix: system prompt + tool defs + large reference docs]   <-- cache breakpoint
[per-request variable content: user message, recent state]
```

- 5-minute TTL by default; 1-hour TTL available for additional cost. Anthropic shortened the default from 60 min → 5 min in early 2026, which silently increased many production bills 30–60%; if your workload is bursty across hours, opt into the 1-hour TTL.
- Break-even: Anthropic requires 2+ hits per cached prefix to recoup the 25% write premium.
- Real-world deployments routinely see 80%+ cost reductions on multi-turn agentic workloads; one reported pipeline cut RCA cost 90% on Haiku 4.5 with a 90% cache-hit rate.
- For Claude Code itself, caching is automatic; you don't have to do anything. But if you run *additional* Anthropic SDK calls in scripts alongside Claude Code, you need to handle caching yourself.

### 6.5 Environment / hardware notes

- **Terminal:** Ghostty (Hashimoto's, GPU-accelerated, fast at streaming output) or Warp 2.0 (built-in AI; redundant if you use Claude Code anyway). The terminal is now where most agentic coding happens; treat it as a tool.
- **Sandboxing:** Docker dev containers for YOLO mode. For really risky work, an ephemeral cloud VM or a Modal/E2B sandbox.
- **Notifications:** Stop hook → system sound + macOS Notification Center / Slack webhook. Hashimoto: *"I don't need to go to it. If I'm heavily into something, I'll keep doing it. The agent posts a notification when it wants my attention."*
- **Voice:** SuperWhisper or built-in macOS dictation for prompt input. Genuinely raises prompt quality.
- **Mobile remote control:** Anthropic's Claude Code iOS app lets you babysit/redirect sessions from your phone.

---

## ## 7. Strategy for New Models

Models change every 3–6 months. A career built on "I memorized Sonnet 4.5's behaviors" is a career with a half-life. A career built on harness engineering is durable, because the harness is what stays.

### 7.1 Build a personal eval set early

You need a small, private, repeatable benchmark of *your* tasks. Without it, every new model release becomes a vibes contest. With it, you have a 2-hour answer to *"is this worth migrating?"*.

Minimum viable eval set for someone in your role:

1. **5 SQL tasks** of increasing difficulty: simple join, multi-CTE window aggregation, a known-hard performance puzzle (rewrite a 90s query under 5s), a PII-redaction transform, a dbt model with tests.
2. **5 DAX / TMDL tasks**: write a measure, refactor a calc group, fix a BPA violation set, implement RLS, build a calculation group with time-intelligence variants.
3. **5 PySpark / Fabric tasks**: notebook conversion (Pandas → PySpark), partition tuning, MERGE pipeline correctness, schema drift handling, perf rewrite.
4. **3 multi-step analytics tasks** with a brief → method spec → notebook → doc handoff, scored on whether the final doc actually answers the brief.
5. **2 harness-engineering tasks**: "write a skill for X," "design a Stop hook for Y."

Score every model release on this set in plan mode + execution mode. Track: success/fail per task, tokens consumed, wall-clock, number of human interventions, subjective code-quality score. Two hours of work per release; saves you weeks of confusion.

### 7.2 Migration playbook when a new model drops

The day a new model lands, run this sequence:

1. **Read the model card and the engineering blog post.** Anthropic reliably documents new defaults (e.g. Opus 4.7's `xhigh` default, the new tokenizer that can consume up to 35% more tokens than 4.6 for the same input — this matters for your cost projections).
2. **Pin first, test second.** `export ANTHROPIC_MODEL=claude-opus-4-7` in one shell. Don't change defaults yet.
3. **Run your eval set on `plan` mode tasks first** (cheapest, fastest signal).
4. **Run on execution tasks** with the same prompts you used last release. Look for *regressions*, not just gains.
5. **Identify dead scaffolding.** What in your CLAUDE.md / skills / hooks exists *because the previous model couldn't do something*? Anthropic's harness research is explicit: every component encodes an assumption about what the model can't do; when the model gains a capability, the corresponding scaffolding becomes dead code. (Example: the "context anxiety" mitigation that became obsolete with Opus 4.5.)
6. **Identify new ceilings.** What can the new model do that you weren't asking for? Allow more autonomy. Reduce intervention. Replace a sub-agent with a single-shot call.
7. **Branch your skills** for any model-specific behavior you actually want to keep separate (rare, but happens — e.g. a tokenization-aware truncation skill).
8. **Update your default in `~/.claude/settings.json`** once you're confident.

### 7.3 Writing model-agnostic harnesses

Three principles:

- **Code against the SDK, not the model.** The Claude Agent SDK abstracts model invocation; treat the model field as configuration. The same harness should run on Opus, Sonnet, and Haiku with only an env-var change.
- **Avoid model-specific prompting tricks.** A prompt that depends on quirks of Sonnet 4.5 (like the context-anxiety mitigation) will silently regress on Opus 4.5+. Keep prompts based on *principles* (clear role, clear tool descriptions, clear examples) rather than tricks.
- **Push behavior into tools, hooks, and skills.** A hook that runs `pnpm test` is model-agnostic. A 5,000-word system prompt that "talks the model into" running tests is model-specific. Always prefer the deterministic enforcement.

### 7.4 Capability discovery patterns

When trying a new model:

- **Stretch one axis at a time.** Long-horizon (multi-day task), wide-context (1M-token codebase), high autonomy (no approvals), specialized domain (DAX vs PySpark). Don't change three things at once or you can't tell what changed.
- **Probe for new failure modes.** Carlini's blog repeatedly notes that more capable models exhibit *new* failure modes (e.g., breaking existing tests when implementing new features). The CI/hook layer of your harness should evolve to catch them.
- **Look for cost drops disguised as quality holds.** The 4.6 generation flipped the math: many tasks that needed Opus on 4.5 now belong on Sonnet, because the 1.2-point SWE-bench gap doesn't justify 5× the cost. Re-route aggressively when new pricing lands.

### 7.5 The model-agnostic harness skill stack

A harness that survives model migrations has, in priority order:

1. **Filesystem as memory** (CLAUDE.md, plans, progress, decisions, handoffs) — works with any model.
2. **Tools first, prompts second** — your `query`, `schema`, `lineage`, `bpa-check`, `pbi-cli` tools work with any model.
3. **Hooks for enforcement** — `Stop`/`PreToolUse`/`PostToolUse` are model-agnostic; they enforce the rule no matter how the model regresses.
4. **Skills with progressive disclosure** — under any model, only loaded when relevant; the cost of having 40 skills is constant ~1.5K tokens.
5. **MCP servers** — protocol-level abstraction works across vendors (Claude, GPT, Gemini all support MCP now). Investments here are portable across LLM providers.

Build your harness on these five pillars and your migration cost on the next model is hours, not weeks.

---

## ## 8. Career Path: Becoming a Harness Engineer

### 8.1 Where the market is, mid-2026

Some grounded numbers:

- **AI Engineer** has been LinkedIn's #1 fastest-growing job title in the US in both 2025 and 2026.
- Pragmatic Engineer's March 2026 survey: ~55% of devs run AI agents as part of their primary workflow. Claude Code is the #1 AI coding tool, overtaking Copilot and Cursor in the previous 8 months. 75% of devs at smaller companies use Claude Code as primary tool.
- **US base ranges** (mid-2026, KORE1 / Glassdoor / Codecademy / Dataquest aggregates):
  - Entry-level AI engineer (typically CS degree + ML project work): $115K–$135K base in SF/NYC, $90K–$135K nationally.
  - Mid-level (3–5 YOE shipping production ML/LLM work): $145K–$210K base, $170K–$280K total comp.
  - Senior / staff: $200K–$310K+ base, total comp routinely $400K+ at SF/NYC large tech.
  - Forward-Deployed Engineer (Palantir-origin role spreading to all AI-native companies): $140K–$200K base, often higher with equity.
  - Glassdoor median across 911 reports: $142K base; 75th percentile $180K; 90th percentile $221K.
- **The premium is for *specialization*.** Over 75% of AI job listings now ask for domain experts, not generalists. LLM fine-tuning, MLOps, RAG, and agent engineering carry the strongest premiums.
- **Buenos Aires reality check:** USD rates from US/EU clients via Deel/remote contracts are the relevant comparison set, not local salaries. Senior remote agent/AI-engineering roles for LATAM-based engineers are routinely $80K–$160K USD; staff roles cross $200K. Argentine analyst-to-AI-engineer transitions are happening *now* and you're well-placed.

### 8.2 The titles to target (in priority order)

1. **AI Engineer / Agent Engineer / Applied AI Engineer** — the main game.
2. **Forward-Deployed Engineer (FDE)** — Palantir-origin; now common at OpenAI, Anthropic, Scale, Sierra, and every AI-native enterprise startup. Ships production code *inside the customer's stack*. Plays to your strength (you already operate as embedded technical staff at Newsweek).
3. **AI/ML Platform Engineer** — building the harness/infra for an internal team. Highest-leverage role for someone with a data-platform background.
4. **Data + AI Engineer** — the "data analyst who can ship agents" role that's emerging at every analytics-heavy company. Closest to your current center of gravity.
5. **Solutions Engineer (AI)** — pre/post-sales technical role at LLM platform vendors; pays well, less ego than ML research roles, builds your reputation fast if you write/speak.

Avoid (for now): pure "ML Engineer" / "ML Research" roles unless you want a 2-year detour into PyTorch/fine-tuning. Your edge is *systems thinking around models*, not model training.

### 8.3 Skills to build (6-month, 1-year, 2-year)

**Months 1–6: Become operationally fluent**
- Build the personal eval set from §7.1.
- Build one MCP server end-to-end (Section 5.5) and publish it.
- Write your `newsweek-*` skill collection (§5.1) and put a sanitized version on GitHub.
- Pass Anthropic's API certification track + the Skilljar Agent Skills course.
- Ship one open-source contribution: a skill, a hook, or a small awesome-list PR.
- Write 6 blog posts: one per month, technical, on what you actually built.

**Months 6–12: Become visibly credible**
- Build one production agent at Newsweek that demonstrably saves time (target: 1 FTE-day per week saved across the team).
- Speak at one local meetup (Buenos Aires Python, Data on the Rocks BA, Microsoft Fabric LATAM, Anthropic Builder Day if available). Even 20 people in a room counts.
- Maintain one public skill or MCP server with >100 stars or >50 users.
- Get featured in `awesome-claude-code` or `awesome-harness-engineering`.
- Start a newsletter or Substack — pick a niche ("Harness Engineering for Data Teams" is unclaimed and high-value).

**Year 2: Become a market**
- Publish a definitive resource (long-form guide, course, or book).
- Speak at an international conference (Microsoft Fabric Community, Coalesce, MDS Fest, Anthropic Code w/ Claude).
- Either: become *the* internal AI lead at Newsweek (formal Head of AI / AI Platform role) **or** consult independently with 2–3 anchor clients **or** ship a SaaS in the space.

### 8.4 People and accounts to follow (and the noise to ignore)

The high-signal set (mid-2026):

- **Anthropic engineering blog** (anthropic.com/engineering) and **news** (anthropic.com/news). Read every harness/context post.
- **Anthropic Code w/ Claude** annual event — replay the keynotes.
- **Geoffrey Huntley** — ghuntley.com + @geoffreyhuntley on X/Bluesky.
- **Simon Willison** — simonwillison.net + @simonw + his newsletter (free tier alone is worth following). The single best human-curated AI news source.
- **Armin Ronacher** — lucumr.pocoo.org. Anti-hype, deep, contrarian.
- **Mitchell Hashimoto** — @mitchellh; his Pragmatic Engineer interview is required listening.
- **Shrivu Shankar** — blog.sshh.io.
- **Birgitta Böckeler** — martinfowler.com posts on Harness Engineering.
- **Dex Horthy** — HumanLayer + his talks on harness engineering and Frequent Intentional Compaction.
- **Nicholas Carlini** — his Anthropic posts on the C-compiler project.
- **Steve Yegge** — Gas Town and follow-ups (read for vision, not implementation).
- **Andrej Karpathy** — his CLAUDE.md observations gist + occasional tweets.
- **Pragmatic Engineer (Gergely Orosz)** — the survey work + practitioner interviews.
- **Latent Space podcast (swyx)** — practitioner-oriented interviews; lots of agent infra coverage.
- **The Pragmatic Engineer Podcast** — Hashimoto, Carmack, Jepsen-style interviews.

Communities:
- **Anthropic Developer Discord** (official; harness/skills channels active).
- **Cursor / Cline / Aider Discords** for cross-tool conversations.
- **`r/ClaudeAI`** for the user perspective (signal-to-noise is mixed but you'll see emerging patterns).
- **Buenos Aires AI / Data Engineering meetups** — physically attend; this is how you get the consulting leads later.

Ignore: AI-influencer Twitter, "10x productivity" newsletters, anything that calls itself a "vibe coder" without irony. Ronacher's *Agent Psychosis* post is the corrective.

### 8.5 Credentials worth getting (and what to skip)

**Worth doing:**
- Anthropic's Skilljar course on Agent Skills (free, ~half a day).
- Anthropic API certification when/if they launch a formal one (they've been moving in that direction).
- Microsoft DP-700 / DP-800 (Fabric Data Engineer / SQL AI Developer) — leverages what you already do and credentialing for Microsoft-shop consulting.
- Azure AI Engineer Associate (AI-102) — if you target enterprise customers, this opens doors.

**Skip:**
- Generic "AI Certification" bootcamps for $2,000 from no-name providers.
- Anything that promises "prompt engineering certification" — the field has moved on.
- Coursera Andrew Ng *Specializations* unless you have specific gaps in stats/ML basics. The applied agent-engineering work isn't covered there.

### 8.6 The credibility flywheel for harness engineers specifically

Open-source contributions that *actually* move the needle, in roughly increasing leverage:

1. PR to an awesome-list adding a tool you've vetted.
2. A skill or hook published in your own repo.
3. A useful MCP server (especially for an under-served system: pick a Microsoft Fabric, Snowflake, dbt, Looker, Hex, or Mode integration that doesn't exist yet).
4. A *plugin* for the Claude Code marketplace (bundle skills + hooks + sub-agents).
5. A blog post that gets cited by Anthropic, Simon Willison, or Geoffrey Huntley.
6. A talk at a recognized conference.
7. Maintaining a high-quality awesome-list or curated newsletter in this niche.

Aim to do (1)–(3) in your first 90 days. (4)–(5) by month 6. (6)–(7) by month 12.

---

## ## 9. Business Angles

### 9.1 As a consultant / freelance

**Services that companies are actually buying in 2026:**

1. **Claude Code rollout** — installing Claude Code across an engineering org, building the team CLAUDE.md, setting up shared skills, plugins, and hooks. Common deliverables: a 4–8 week engagement, $30K–$120K, ending with a working harness + training + handover docs.
2. **Custom MCP servers** for the client's internal systems (CRM, data warehouse, internal admin tooling). Common range: $15K–$60K per server depending on auth complexity.
3. **Internal skills library + plugin** for a team. Replaces tribal knowledge with progressive-disclosure skills. $20K–$80K depending on scope.
4. **Agent workflow design** for a specific business process (PR review pipeline, BI deploy pipeline, customer-support triage, analytics-request intake). $25K–$150K.
5. **Harness assessment / audit** — for a team that already has Claude Code chaos. 1–3 weeks, $10K–$30K, produces a written assessment + roadmap. Great lead-in to bigger engagements.
6. **AI on-call** — retainer ($3K–$10K/month) to be the team's expert: review their CLAUDE.md, debug agent failures, propose hooks, evaluate new models when they drop.
7. **Training / workshops** — half-day to 2-day for engineering teams. $5K–$25K per workshop.

**Pricing models:**
- **Fixed-fee project** with milestone payments (best for well-scoped MCP servers / skill libraries).
- **Weekly retainer** ($5K–$15K/week typical for senior US-market freelancers in this niche; LATAM-based via Deel can be $4K–$10K/week for the same work and still beat your full-time salary).
- **Day rate** for ad-hoc audit work: $1,200–$2,500/day mid-market, $2,500–$5,000/day for recognized names.
- **Outcome-based**: rare, but doable for measurable wins (e.g. "cut PR review cycle time by 40%"); typically command 1.5–3× the equivalent T&M.
- **Revenue share / equity** with AI-native startups — only do this for warrants or with clauses that protect you if the company pivots away from your work.

**Finding first clients (concrete sequence):**
1. **Newsweek case study first.** Build the internal Newsweek harness (Section 5.1). Get permission to write a *sanitized* case study. Publish it. This is your portfolio.
2. **Free 30-minute audits** for 5 mid-size companies in your network. The audit produces a 1-pager. The 1-pager produces 1–2 paid follow-ups.
3. **One niche, one channel.** "Harness engineering for Microsoft-shop analytics teams" is highly specific and under-served. Channels: LinkedIn long-form posts + a niche newsletter + Microsoft Fabric community presence.
4. **Refer-in via the open-source flywheel** (Section 8.6). Companies hire the people whose code they've already adopted.
5. **Speak at the right conferences** — Microsoft Fabric Community, MDS Fest, Coalesce, Data Council. One good talk → 3–5 inbound consulting leads.

**Pain points companies will pay you to solve:**
- *"Our engineers are using Claude Code but everyone's CLAUDE.md is different."*
- *"We tried agents for our pipelines but they break things in prod."*
- *"Our analysts want to use AI but can't touch SQL Server directly."*
- *"We have 47 prompts in a Notion doc; nothing is versioned."*
- *"We don't know if Opus 4.7 is worth the cost increase."*
- *"How do we let agents touch our customer data safely?"*

Each of those is a $20K–$200K engagement.

### 9.2 As a SaaS / product builder

The agent-tooling space is huge and still under-built. The category map (rough segmentation):

- **Coding agents themselves** (Cursor, Claude Code, Codex, Cline, Aider, Amp, Zed, Windsurf, Cognition/Devin, Tessl). **Skip.** Too much capital, too many incumbents.
- **Harness/orchestration layers** (Builder.io's agent harness, LangGraph, DeepAgents, Inngest, Temporal-for-agents, MindStudio, Builder, Remy). Crowded but still differentiable for specific niches.
- **Skill/plugin marketplaces** (skills.sh, the Anthropic marketplace). High-leverage if you have distribution.
- **Eval / observability** (LangSmith, Helicone, Braintrust, OpenLLMetry, Langfuse). Hot in 2026.
- **Domain-specific agent products** — *the green field*. Examples:
  - **Power BI / Fabric agent products** (under-served despite Microsoft's own MCP server). Newsweek-adjacent.
  - **DAX/semantic-model copilots** for analytics teams.
  - **dbt + agents** (data-engineering specific harness).
  - **Notebook agents** (Hex, Deepnote, Mode adjacent).
  - **Editorial / publishing workflow agents** (Newsweek is in this exact category — there's a product here).
  - **Compliance/PII agents** for regulated industries.
- **Agent security / auditing** (ecc-agentshield-style scanners, prompt-injection detectors). Early but growing.
- **Agent-native version control** (Hashimoto's "Gmail moment for version control"). Several stealth companies; large bet.

**Indie products that have shown traction:**
- **Cursor / Windsurf** (Series B+ scale, not indie anymore).
- **Aider** (open-source, sponsorship/grant-based).
- **Amp** (Sourcegraph; commercial coding agent).
- **Builder.io** (visual agent harness; Series B+).
- **Termdock** (terminal-multiplexer for AI agents).
- **ChatPRD, How I AI products** (Claire Vo) — productivity-niche agents.
- **Pi / OpenClaw** (Mario Zechner / Peter Steinberger) — minimal agent + community.
- **MindStudio, Remy** — agent orchestration for non-engineers.

**Niches worth considering for you specifically:**
1. **"Power BI Copilot for real semantic modelers"** — pbi-cli + skills + a hosted MCP + a usable UI. Microsoft's own offering is built for end-users; the professional-modeler market is wide open.
2. **"Fabric agent toolkit"** — opinionated harness for Fabric data engineers (notebooks, lakehouse, pipelines, semantic models). MS will probably ship something; ship faster and charge less.
3. **"Editorial analytics agent"** for publishers (Newsweek-adjacent) — an agent that turns "what should our top story be this hour?" into a real ranking against subscriber/engagement data.
4. **"Audit my Claude Code config"** — `ecc-agentshield`-style productized service for enterprises rolling out Claude Code.
5. **"Skill registry for finance/health/legal"** — pre-vetted, compliance-reviewed skills.

Open-source projects that became businesses (mental models for the path): Aider → support/grants; LangChain → LangSmith; HumanLayer; Inngest's agent pivot; Sourcegraph → Amp.

### 9.3 As the internal AI/agent expert at Newsweek

This is the highest-ROI option in the near term because you're already there, you have organizational context, and the upside (a new role, a real team, a budget) is larger than most external consulting equivalents.

**The 6-step path:**

1. **Pick one painful workflow and automate it end-to-end.** Editorial dashboards refresh, subscriber-churn modeling, content-performance KPIs. Build the agent + harness; ship it; measure savings honestly.
2. **Quantify it.** "We saved X analyst-hours per week, Y Power BI deploys per month with Z fewer errors." Leadership cannot promote what they can't measure.
3. **Publish internally.** Wiki page, Slack post, 15-minute lunch-and-learn. Include the failure modes — credibility comes from naming what didn't work.
4. **Become the policy author.** Write the Newsweek AI usage policy (acceptable use, PII rules, model approval list, hook standards). Make it short, practical, and review-it-quarterly. Once you've written the policy, you're the *de facto* AI lead.
5. **Build the platform.** Centralize the skills/MCP/hooks repo. Bootstrap script for new joiners. Internal Slack channel. Monthly office hours.
6. **Propose the role.** "We need an AI Platform Engineer / Head of AI Engineering for Data. Here's the scope. Here's what it saves. Here's how I'd build the team." If Newsweek says yes, congratulations. If not, you have a portfolio that makes the external job market easier.

**Demonstrating ROI to leadership** — the metrics that actually move them:
- Analyst hours saved (translate to USD at your blended rate).
- Time-to-insight on ad-hoc requests (P50 and P90; agents typically crush P90).
- Defect rate / refresh failures avoided.
- Net Promoter Score from internal stakeholders.
- New analytical capacity unlocked (questions you could never have answered before because they'd have taken too long).

**Internal advocacy patterns that work:**
- One-page demos > slide decks. Run the agent live for 90 seconds at the all-hands.
- Pair with one heavy user as your champion. They'll evangelize you internally.
- Offer office hours, not training sessions. People show up to ask their question; nobody shows up to "Intro to Claude Code v3."
- Don't fight Excel or Power BI users. Augment them. Their skill is the data; yours is plugging an agent into it.

---

## ## 10. The Roadmap: A Concrete 30 / 60 / 90 / 365-Day Plan

This is the most important section. Everything above is reference; this is what you do *next week*.

### 10.1 Days 0–30: Foundation

**Goal:** be operationally fluent. Be running Claude Code daily with a deliberate setup, not by accident.

| Week | Action |
|---|---|
| Week 1 | Read end-to-end: Anthropic *Effective context engineering*, *Effective harnesses for long-running agents*, *Harness design for long-running application development*, *Building agents with the Claude Agent SDK*; Willison *Designing agentic loops*; Huntley *how to build a coding agent* workshop. Take notes in a single `notes.md`. |
| Week 1 | Install Claude Code, Ghostty, `uv`, `pnpm`, `ruff`, `sqlfluff`. Set up `~/.claude/CLAUDE.md`, `~/.claude/settings.json` with the 4 baseline hooks from §6.3. |
| Week 2 | Build your personal eval set (§7.1, ~20 tasks). Run it against Sonnet 4.6 once with notes. |
| Week 2 | Build your first skill: `newsweek-dax` or `newsweek-fabric`. Iterate on it daily for a week. |
| Week 3 | Build your first sub-agent: a `data-explorer` read-only investigator that you can `Task(...)` against your dev lakehouse. |
| Week 3 | Build your first MCP server: a thin one over the Newsweek Fabric dev environment (list workspaces, run SELECT queries, fetch schemas). Run it locally only; don't expose to others yet. |
| Week 4 | Pick a real workflow at work and run it end-to-end through Claude Code + your skills. Don't pick the biggest one — pick the most repetitive one. |
| Week 4 | Write your first public blog post: *"What I learned building a Fabric MCP for Claude Code"*. Post on LinkedIn and your blog. |

**Books / courses to consume in parallel:**
- Anthropic *Complete Guide to Building Skills for Claude* (PDF).
- Skilljar course on Agent Skills (Anthropic).
- The full *Best Practices for Claude Code* documentation.
- Geoffrey Huntley's *how to build a coding agent* workshop (build the agent — don't just read).
- *Building Effective AI Agents* (Anthropic resources hub).

### 10.2 Days 30–60: Depth

**Goal:** depth in the Claude Code ecosystem; one production thing at Newsweek.

| Week | Action |
|---|---|
| Week 5 | Pin your second skill (`newsweek-pyspark` or `newsweek-power-bi`). Add a Gotchas section that you commit to updating every time Claude trips on it. |
| Week 5 | Add a generator/evaluator pair for one workflow (e.g. `dax-author` + `dax-reviewer` sub-agents). |
| Week 6 | Set up `claude --worktree` for parallel work. Run a "competition" once (Claude vs Codex on the same task, à la Hashimoto). |
| Week 6 | Migrate your CLAUDE.md content into skills where it makes sense (anything path-scoped, anything that's not always-true). |
| Week 7 | Build the *Newsweek bootstrap script* — a `bootstrap.sh` that any teammate can run to get your shared skills + commands + hooks installed. |
| Week 7 | Demo to one teammate. Get them up and running. Update the bootstrap based on their feedback. |
| Week 8 | Ship one real internal automation (refresh-monitoring agent, DAX BPA bot, Fabric pipeline reviewer) with measurable savings. Quantify the hours/USD saved. |
| Week 8 | Publish your second blog post: the case study, including failure modes. |

### 10.3 Days 60–90: Visibility

**Goal:** be visibly credible inside Newsweek and on the public internet.

| Week | Action |
|---|---|
| Week 9 | Internal demo at Newsweek (lunch-and-learn or all-hands). Five minutes max. Live demo. |
| Week 9 | Open-source your `newsweek-*` skills as a generic version (`analytics-skills-pack` or similar). License it MIT. |
| Week 10 | Submit your repo to `hesreallyhim/awesome-claude-code` and `walkinglabs/awesome-harness-engineering`. Submit a plugin to the Anthropic marketplace if you have one. |
| Week 10 | Speak at one local meetup. Topic: *"Harness Engineering for Data Teams: A Field Report from Newsweek"*. |
| Week 11 | Publish a "definitive guide" post in your niche: *"A 90-Day Roadmap for Data Analysts Becoming Agent Engineers"* (this document is your draft). |
| Week 11 | Start a newsletter (Substack/Beehiiv). Commit to bi-weekly. Pick one niche and one audience. |
| Week 12 | Propose to Newsweek leadership: a formalized AI/Data Platform initiative. Bring numbers. Bring the bootstrap. |
| Week 12 | Eval Day. Run your full eval set against any newer model that has shipped. Document migration costs. |

### 10.4 Months 3–6: Compounding

By the end of month 6 you should have:

- A working, documented internal harness at Newsweek used by 3+ people.
- One published open-source artifact with at least 100 stars or 30 active users.
- 12 blog posts / newsletter issues.
- One conference talk submitted (Microsoft Fabric Community, MDS Fest, Data Council). One accepted.
- A clear value-prop sentence: *"I'm the person who [specific thing] for [specific people] using [specific stack]."*
- 2–3 inbound consulting conversations (don't have to take them; the fact that they happened is the signal).

Decision point at month 6: do you want to **stay-and-grow** at Newsweek (push for the AI Platform role), **freelance** (start booking 1–2 small clients while still employed), or **build a product** (start a 10-hours-a-week side project that's specifically a SaaS).

### 10.5 Months 6–12: Positioning

- **If staying internal:** propose the formal role with a written charter. Hire one junior. Build the platform team. Aim for "Head of Data + AI Engineering" within 18 months of starting.
- **If consulting:** ship the Newsweek case study (with permission), set up a Stripe account, get one anchor client at $5K–$10K/week for 4–8 weeks. Reinvest into a website, a newsletter, and one more open-source artifact.
- **If product:** spend 3 months on customer discovery (20 calls with people in your target ICP). 3 months on a v0 that 5 people actually use weekly. Decide at month 12 whether to go bigger.

### 10.6 Year 2: Becoming a market

- A definitive long-form artifact: a book, a course, or a maintained reference site that becomes *the* resource in your niche. (For example: *"Harness Engineering for Microsoft Fabric Teams"* is a book that doesn't exist and that you could write.)
- An international conference talk (Anthropic Code w/ Claude, Coalesce, MDS Fest, Fabric Conference).
- An identifiable signature: a pattern, a framework, a piece of vocabulary that people cite back to you.
- Clear positioning: *"This is what JV does. This is who he does it for. This is the work I want from him."*
- Either: a real team you lead, a real consulting practice you run, or a real product with real users.

### 10.7 Concrete projects to build (pick from this list)

In rough order of difficulty:

1. **`newsweek-analytics-skills`** — a public skill pack for analytics teams on Fabric/Power BI.
2. **`fabric-mcp`** — a clean, secure MCP server for Microsoft Fabric (lakehouse, semantic models, pipelines, capacity).
3. **A Power BI BPA-driven Claude Code plugin** — wraps Tabular Editor 2 BPA + DAX format + reference-scan into a single workflow.
4. **A `notebook-reviewer` sub-agent** specifically tuned for PySpark/Fabric notebooks.
5. **A `subscriber-churn` agent** that turns a brief into a full notebook + Power BI page, end-to-end (publishable Newsweek case study, sanitized).
6. **A `claude-code-newsroom-bootstrap`** — opinionated dotfiles + skills + hooks for editorial analytics teams. Maintain it; let people fork it.
7. **A newsletter / blog: "Harness Engineering for Data Teams"** — bi-weekly, technical, specific.
8. **A small SaaS: an audit-and-recommend tool** that scans a company's CLAUDE.md + skills + MCP config and produces a written report (think `ecc-agentshield` but business-friendly).
9. **A book: *Harness Engineering for Microsoft Fabric Teams*** — niche, opinionated, definitive. 120 pages. Self-publish, or pitch O'Reilly / Manning.

### 10.8 Daily / weekly cadence

- **Daily (30 min):** check Hacker News, Simon Willison's blog, Anthropic news. Capture one note. Update one skill or hook based on something you ran into.
- **Weekly (2 hours):** run your eval set if a new model dropped; otherwise, write a draft of a newsletter issue or blog post. Review your `~/.claude/projects/` session logs for recurring failures.
- **Bi-weekly:** publish one piece of writing.
- **Monthly:** one open-source contribution (skill, MCP server, awesome-list PR, plugin). One demo at Newsweek.
- **Quarterly:** retrospective on the roadmap. Move the goalposts. Talk to 5 people in adjacent roles.

### 10.9 What to do *this week*

If you do nothing else after reading this:

1. **Tomorrow:** install Ghostty (or improve your current terminal), upgrade to the latest Claude Code, set the four baseline hooks from §6.3.
2. **By Friday:** write a 30-line `~/.claude/CLAUDE.md` and a 50-line project `CLAUDE.md` for your current Newsweek work. Delete anything Claude already does correctly. Add a `Gotchas` section.
3. **By Sunday:** Read the Anthropic *Effective harnesses for long-running agents* post, Huntley's *how to build a coding agent*, and Willison's *Designing agentic loops*. Take notes.
4. **Next week:** build your first skill. Use it. Update its Gotchas section every time it fails.

You're already further along than you think. The discipline is young — *Designing agentic loops* was posted in September 2025; *Effective context engineering* in late September 2025; the term "harness engineering" only became standard vocabulary by early 2026. The window to become a recognized practitioner is *now*, and your data/BI background is an advantage, not a deficit, because most of the early practitioners are pure-software-engineering people who don't understand semantic models, DAX, or lakehouses. The "harness engineer for analytics and BI teams" niche has fewer than ten visible practitioners worldwide. You can credibly be one of them by Q3 2026.

---

## Closing notes and source map

### The single most important takeaway

**The model is the easy part. The harness is your career.** Frontier models will keep getting better. The vocabulary will keep shifting (prompt engineering → context engineering → harness engineering → whatever-comes-next). What doesn't shift is the *practice* of building reliable systems around capable but flawed models: clear context, well-designed tools, deterministic guardrails, verification gates, persistent memory, and human judgment about what to delegate. Build those muscles and you stay employed across model generations, vendor shifts, and capability cliffs.

### How to use this document

- **Sections 1–3** are foundational; read once, then revisit when you teach someone else.
- **Section 4** is your reading list; work through it over 60 days.
- **Sections 5–6** are the working references; bookmark them.
- **Sections 7–8** matter most when models change or you're job-hunting.
- **Section 9** is the strategic decision space; revisit at the 6-month mark.
- **Section 10** is the action plan; execute it.

### The primary sources, in priority order

If you only read ten things:

1. Anthropic, *Effective context engineering for AI agents* — anthropic.com/engineering
2. Anthropic, *Effective harnesses for long-running agents* — anthropic.com/engineering
3. Anthropic, *Harness design for long-running application development* — anthropic.com/engineering
4. Anthropic, *Building agents with the Claude Agent SDK* — anthropic.com/engineering
5. Anthropic, *Equipping agents for the real world with Agent Skills*
6. Simon Willison, *Designing agentic loops* — simonwillison.net/2025/Sep/30/designing-agentic-loops/
7. Geoffrey Huntley, *how to build a coding agent* workshop — ghuntley.com/agent
8. Geoffrey Huntley, *Ralph Wiggum as a Software Engineer* and *don't waste your back pressure*
9. Armin Ronacher, *Agentic Coding Recommendations* and *Agentic Coding Things That Didn't Work* — lucumr.pocoo.org
10. Mitchell Hashimoto on *The Pragmatic Engineer* (and the Zed/Sourcegraph webinars) — practical methodology from a working senior engineer.

Then: the Anthropic *Complete Guide to Building Skills for Claude* PDF; the official Claude Code docs at code.claude.com; the *Claude Agent SDK / Agent loop* docs; Addy Osmani's *Agent Harness Engineering*; Birgitta Böckeler's *Harness Engineering* on martinfowler.com; the OpenAI Codex *harness engineering* field report; the `walkinglabs/awesome-harness-engineering` and `hesreallyhim/awesome-claude-code` repos for the living reference set.

### A final note on epistemics

This guide synthesizes a field that is *eight to twelve months old as a named discipline*. Most of the patterns here will still be right in 12 months. Some won't. Where I've stated specific numbers (model benchmarks, prices, capability claims, salary ranges), they reflect publicly published sources as of mid-2026 and will need re-verification each quarter. Where I've quoted vocabulary or attributions ("back-pressure", "Ralph loop", "if you're not the model, you're the harness", "harness engineering"), the attributions are accurate to the best of public record but the field's history is being written in real time and you may find earlier uses if you dig.

The most reliable signal in this space remains: *who actually ships, with their name attached, what they actually built*. Anthropic's engineering team, Huntley, Ronacher, Hashimoto, Willison, Shankar, Carlini, Horthy, Böckeler — these are the people I'd trust on a Tuesday. The rest of us, including this guide's author, are still figuring it out alongside them.

Now go build the harness.