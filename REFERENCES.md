# Referencias canónicas del curso

Material consultado para diseñar el curso. Organizado por autor → fuentes → conceptos clave → mapeo a lecciones donde se usa.

---

## Tres principios universales (cruzan todas las fuentes)

1. **El context window es el recurso fundamental que hay que gestionar.** (Anthropic textual.) Hilo conductor de L5, L7, L9.
2. **Verification > generation.** "The single highest-leverage thing you can do" (Anthropic). "If you didn't see it run, it isn't a working system" (Willison). Núcleo de L3, L11, L12.
3. **Simplicidad por defecto.** "Add complexity only when it demonstrably improves outcomes" (Anthropic). Anti-Christmas tree. Conecta con Fowler.

---

## 1. Anthropic (oficial)

### URLs canónicas
- **Building effective agents** — https://www.anthropic.com/research/building-effective-agents
- **Claude Code overview** — https://code.claude.com/docs/en/overview (la doc canónica)
- **Best practices** — https://code.claude.com/docs/en/best-practices
- **Permissions** — https://code.claude.com/docs/en/permissions
- **Permission modes** — https://code.claude.com/docs/en/permission-modes
- **Skills** — https://code.claude.com/docs/en/skills
- **Sub-agents** — https://code.claude.com/docs/en/sub-agents
- **Hooks guide** — https://code.claude.com/docs/en/hooks-guide
- **MCP (Claude Code)** — https://code.claude.com/docs/en/mcp
- **MCP spec** — https://modelcontextprotocol.io/introduction
- **How Anthropic teams use Claude Code** — https://claude.com/blog/how-anthropic-teams-use-claude-code (ORO para la audiencia data, muestra equipos no-eng)
- **Building agents with Claude Agent SDK** — https://claude.com/blog/building-agents-with-the-claude-agent-sdk
- **Índice completo de la doc** — https://code.claude.com/docs/en/llms.txt

### Vocabulario oficial EXACTO
- **Agentic systems** (paraguas), **Workflows** (paths predefinidos) vs **Agents** (LLM dirige dinámicamente) — categorías distintas
- **Agentic loop** (canónico, 4 fases): **gather context → take action → verify → repeat**
- **Agent-computer interface (ACI)** — la interfaz entre modelo y tools
- **Memory** = CLAUDE.md (+ auto memory)
- **Skills** = SKILL.md con frontmatter
- **Subagents** (escriben así o "sub-agents"): built-in son **Explore** (Haiku, read-only), **Plan**, **general-purpose**
- **Hooks** = "shell commands at lifecycle points, provide deterministic control"
- **MCP servers** = "open standard for AI-tool integrations"
- **Slash commands** + **Plugins** (bundle de skills/hooks/subagents/MCP)
- **Permission modes** (los 6): `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`
- **Permission rules**: `Tool(specifier)`, evaluación `deny → ask → allow`
- **Checkpointing / /rewind** (doble Esc)
- **Headless mode** = `claude -p "..."`
- **Worktrees**, **Background agents**, **Agent teams**, **Routines**

### Los 6 patrones canónicos de "Building effective agents"
1. **Prompt chaining** — pasos secuenciales con gates
2. **Routing** — clasifica input → especialista
3. **Parallelization** — *sectioning* o *voting*
4. **Orchestrator-workers** — orquestador descompone dinámicamente
5. **Evaluator-optimizer** — generador + evaluador en loop
6. **Autonomous agents** — loops con feedback ambiental, pasos imprevisibles

### Anti-patrones nombrados oficialmente (van a L13)
- **"Kitchen sink session"** — mezclar tareas no relacionadas → `/clear`
- **"Correcting over and over"** — tras 2 correcciones fallidas, `/clear` y replantear
- **"Over-specified CLAUDE.md"** — *"If Claude keeps doing something you don't want despite a rule, the file is probably too long and the rule is getting lost"*
- **"Trust-then-verify gap"** — código plausible que no maneja edge cases
- **"Infinite exploration"** — pedir "investigate" sin scope → usar subagents
- **Framework trap** — abstracciones que ocultan prompts/responses
- **Agentic by default** — usar agentes "only when simpler solutions fall short"

### Heurísticas oficiales clave
- **Plan mode**: "If you could describe the diff in one sentence, skip the plan"
- **CLAUDE.md**: "For each line, ask: 'Would removing this cause Claude to make mistakes?' If not, cut it"
- **CLAUDE.md bloated**: "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"
- **Skills vs CLAUDE.md**: "CLAUDE.md is loaded every session... For domain knowledge or workflows that are only relevant sometimes, use skills instead"
- **Tools**: "Tell Claude Code to use CLI tools like `gh`, `aws`, `gcloud`, and `sentry-cli`. CLI tools are the most context-efficient way to interact with external services"
- **Hooks**: "Use hooks for actions that must happen every time with zero exceptions. Unlike CLAUDE.md instructions which are advisory, hooks are deterministic"
- **Subagents**: "Subagents work within a single session" — distinto de background agents y agent teams

---

## 2. Martin Fowler

### URL
- https://martinfowler.com/articles/harness-engineering.html

### Conceptos clave (varios ya integrados)
- **Guides vs Sensors** (feedforward vs feedback) ✅ integrado en L2
- **Computational vs Inferential** (eje ortogonal) ✅ integrado en L2
- **Steering loop** ✅ integrado en L3
- **Tres categorías de harness**: maintainability (madura) / architecture fitness (intermedia) / behaviour (frontera) → L13
- **Harnessability + ambient affordances** → L11
- **Ley de Ashby (variedad requisita)** → L11
- **Keep quality left** → L8 + L13

### Citas clave
- *"Separately, you get either an agent that keeps repeating the same mistakes (feedback-only) or an agent that encodes rules but never finds out whether they worked (feed-forward-only)."*
- *"Harnesses are an attempt to externalise and make explicit what human developer experience brings to the table, but it can only go so far."*

---

## 3. Simon Willison — simonwillison.net

### Posts canónicos por lección

| Lección | Post | URL |
|---|---|---|
| L1 (Agente) | AI agents can mean a lot of different things | /2025/Sep/18/agents/ |
| L3 (Mentalidad) | Vibe engineering | /2025/Oct/7/vibe-engineering/ |
| L6 (Permisos) | The Lethal Trifecta | /2025/Jun/16/the-lethal-trifecta/ |
| L7 (Skills) | Claude Skills are a bigger deal than MCP | /2025/Oct/16/claude-skills/ |
| L11 (Diseñar) | Designing agentic loops | /2025/Sep/30/designing-agentic-loops/ |
| L11 (Diseñar) | Embracing parallel coding agents | /2025/Oct/5/parallel-coding-agents/ |
| L12 (Diagnosticar) | Here's how I use LLMs to help me write code | /2025/Mar/11/using-llms-for-code/ |
| L15 (Mantenerte) | **Agentic Engineering Patterns** (guía-libro, lectura paralela) | /guides/agentic-engineering-patterns/ |

### Conceptos clave a integrar
- **"An LLM agent runs tools in a loop to achieve a goal"** — definición canónica para L1
- **Lethal Trifecta**: (1) acceso a datos privados + (2) exposición a contenido no confiable + (3) capacidad de comunicación externa = exfiltración. Va en L6.
- **Vibe coding vs Vibe engineering / Agentic Engineering** — distinción para L3
- **"Designing agentic loops is a critical new skill"** — frame de L11
- **Criterios de problema agentic-adecuado** (L11): criterios de éxito claros + componente prueba-error + test suite automatizado
- **YOLO mode + sandboxing**: fórmula "más autonomía ⇔ menor blast radius" → L6
- **AGENTS.md sobre MCP** para CLI tools (Playwright, ffmpeg) — sutil pero útil para L10
- **"Send out a scout"** patrón parallel → L13

### Citas killer para usar como aperturas
- *"An LLM agent runs tools in a loop to achieve a goal."*
- *"An AI agent is an LLM wrecking its environment in a loop."* (Solomon Hykes, vía Simon)
- *"Designing agentic loops is a critical new skill."*
- *"If you didn't see it run, it isn't a working system."*
- *"LLMs amplify existing expertise."*
- *"Once you start mixing and matching tools yourself there's nothing those vendors can do to protect you."*
- *"If an LLM wrote every line of your code but you reviewed, tested, and understood it all — that's not vibe coding, that's using an LLM as a typing assistant."*

### Recursos para L15
- Blog completo + RSS: https://simonwillison.net
- Tags clave: `/tags/agents/`, `/tags/claude-code/`, `/tags/llm-tool-use/`, `/tags/prompt-injection/`, `/tags/evals/`
- Newsletter: https://simonw.substack.com/
- CLI `llm`: https://llm.datasette.io

---

## 4. Geoffrey Huntley — ghuntley.com

### Posts canónicos
| Post | URL | Para qué lección |
|---|---|---|
| Ralph Wiggum as a 'software engineer' | /ralph/ | L11 (case study), L13 (patrón) |
| Everything is a Ralph loop | /loop/ | L13 |
| Don't waste your back pressure | /pressure/ | L13 — concepto **backpressure** |
| The six-month recap | /six-month-recap/ | L11, L16 |
| `/specs` + Groundhog | /specs/ | L11 — spec-driven |
| stdlib | /stdlib/ | L11 — composición de prompts |
| Cursed (3 meses de loop) | /cursed/ | L11 case study |
| Secure codegen anti-patterns | /secure-codegen/ | L6 / L13 |
| Cognitive security | /cogsec/ | L16 |

### Conceptos clave (a usar en L11 y L13)
- **Ralph loop** (jul 2025): `while :; do cat PROMPT.md | claude --dangerously-skip-permissions; done`. **Solo para greenfield.** Estructura: `loop.sh` + `PROMPT_build.md` + `PROMPT_plan.md` + `AGENTS.md` + `IMPLEMENTATION_PLAN.md` + `specs/`
- **Stack determinista upstream + backpressure downstream**: contexto fresco por iteración, tests/builds/linters como compuerta
- **"On the loop, not in the loop"**: el operador se mueve fuera del loop a observar
- **JTBD → Topic of Concern → Spec → Task** (jerarquía)
- **`/specs` + stdlib**: composición de prompts versionados como código
- **Test de scope**: *"Can you describe the topic in one sentence without 'and'?"*

### Citas
- *"Ralph is a technique. In its purest form, Ralph is a Bash loop."*
- *"Software is now clay on the pottery wheel and if something isn't right then I just throw it back on the wheel."*
- *"There's no way in heck would I use Ralph in an existing code base."* (caveat crítico)
- *"It's not if it gets popped, it's when. What's the blast radius?"*
- *"If anyone pitches you on the idea that you can achieve secure code generation via an MCP tool or Cursor rules, run, don't walk."*

---

## 5. Armin Ronacher — lucumr.pocoo.org

### Posts canónicos
| Post | URL | Para qué lección |
|---|---|---|
| Agentic Coding Recommendations | /2025/06/12/agentic-coding/ | L11 — workflow hands-off |
| Tools: Code Is All You Need | /2025/7/3/tools/ | L10 |
| Your MCP Doesn't Need 30 Tools | /2025/8/18/code-mcps/ | L10 — **Ubertools** |
| Agentic Coding Things That Didn't Work | /2025/7/30/things-that-didnt-work/ | L13 |
| Agent Design Is Still Hard | /2025/11/21/agents-are-hard/ | L11, L12 |
| Skills vs Dynamic MCP Loadouts | /2025/12/13/skills-vs-mcp/ | L7, L10 |
| The Final Bottleneck | /2026/2/13/the-final-bottleneck/ | L12, L13 |
| A Year Of Vibes | /2025/12/22/a-year-of-vibes/ | L16 |
| In Support of Shitty Types | /2025/8/4/shitty-types/ | L11 — elección de lenguaje |

### Conceptos clave (a usar en L7, L10, L12, L13)
- **Ubertools**: MCP que expone UNA tool que ejecuta código stateful — alternativa al MCP con 30 tools
- **"Code over tools"**: generar código vence a invocar tools porque compone vía operadores del lenguaje, es verificable, escala sin inferencia
- **Skills > Dynamic MCPs**: skills no cargan tool definitions en contexto
- **Things that didn't work** (lista directa para L13): slash commands pre-escritos, hooks en yolo-mode, `--print` mode, sub-agents para tareas read/write mixtas, tipos demasiado expresivos en el loop
- **"Only automate what you do regularly"** — heurística de longevidad
- **"Mental disengagement → quality drops"** — riesgo central
- **The Final Bottleneck**: la revisión, no la generación, es el cuello
- **Cache management explícito** (estilo Anthropic): un cache point tras system prompt, dos en conversation tail
- **Reinforcement injections**: usar respuestas de tools para inyectar recordatorios

### Citas
- *"There is a big hidden risk with automation through LLMs: it encourages mental disengagement. When you stop thinking like an engineer, quality drops."*
- *"You can review the formula, not the calculated result."* (sobre code-over-tools)
- *"To replace oneself with a shell script."*
- *"If the machine writes the code, the machine better review the code at the same time."*
- *"Agents struggle with exceptions, they are afraid of them."* (sobre lenguajes para agentes)

---

## 6. Otros autores del círculo (mencionar en L16)

- **Mario Zechner** — autor de **Pi**, agente minimalista (Read/Write/Edit/Bash, system prompt más corto conocido). Referencia para "harness mínimo".
- **Solomon Hykes** — cita canónica: *"An AI agent is an LLM wrecking its environment in a loop."*
- **Dexter Horthy / HumanLayer** — "A Brief History of Ralph" en humanlayer.dev
- **Boris Cherny** — creador Claude Code (Anthropic)
- **Andrej Karpathy** — acuñó "vibe coding" (feb 2025)
- **Thariq Shihipar** (equipo Claude Code): *"Long running agentic products like Claude Code are made feasible by prompt caching."*

---

## Mapeo concepto → lección (consolidado)

### L1 — Agente
- ✅ Hecho: definición + loop diagram
- A integrar: callout con Simon's "tools in a loop to achieve a goal" + Anthropic 4-phase (gather/take action/**verify**/repeat) + nota workflows vs agents

### L2 — Harness
- ✅ Hecho: anatomía + guides/sensors (Fowler) + computacional/inferencial

### L3 — Mentalidad
- ✅ Hecho: 4 habilidades + steering loop (Fowler)
- A integrar: sección "Esto NO es vibe coding" (Willison)

### L4 — Claude Code primera sesión
- ✅ Hecho

### L5 — CLAUDE.md
- A integrar: callout con "bloated CLAUDE.md" anti-pattern de Anthropic + heurística "would removing this cause mistakes?"

### L6 — Permisos (a escribir)
- Material: 6 permission modes, deny→ask→allow, Tool(specifier) syntax, protected paths
- Mini-ejercicio: script de mantenimiento de archivos
- **Lethal Trifecta** (Willison) como frame de seguridad
- YOLO mode + sandboxing

### L7 — Skills (a escribir)
- Material: heurística oficial "when you keep pasting", SKILL.md + frontmatter, lifecycle, locations
- Mini-ejercicio: validador de CUITs
- Skills vs CLAUDE.md (Anthropic) + Skills > MCP (Willison + Ronacher)

### L8 — Hooks (a escribir)
- Material: 9+ eventos, "must happen every time with zero exceptions"
- Mini-ejercicio: bloquear `rm` fuera de `./tmp/`
- Keep Quality Left (Fowler)
- Anti-patterns: hooks en yolo-mode (Ronacher), paths relativos (usar `$CLAUDE_PROJECT_DIR`)

### L9 — Sub-agentes (a escribir)
- Material: Explore/Plan/general-purpose built-in, "side task floods context", cannot nest
- Mini-ejercicio: sub-agente traductor ES→EN
- Failure isolation pattern (Ronacher)

### L10 — MCP (a escribir)
- Material: spec (servers/clients/hosts/primitives), "connect when copying data", security/prompt injection
- Mini-ejercicio: MCP que devuelve hora UTC
- Ubertools (Ronacher) + Code-over-tools + AGENTS.md sobre MCP (Willison)

### L11 — Diseñar harness desde cero (a escribir)
- Framework de 5 preguntas (mío)
- **Designing agentic loops** (Willison) — criterios agentic-adecuados
- **6 patrones canónicos** (Anthropic) — el estudiante debe poder reconocerlos
- **Harnessability + ambient affordances** (Fowler)
- **Ley de Ashby**
- **Ralph loop** (Huntley) como case study con caveat "solo greenfield"
- **Hands-off workflow** (Ronacher) como contraste
- 3 escenarios distintos: scraper, bot de soporte, revisor de PRs

### L12 — Diagnosticar (a escribir)
- "If you didn't see it run" (Willison)
- "Anti-antropomorfizar" (Willison)
- Anti-patterns Anthropic: kitchen sink, infinite exploration, trust-then-verify, correcting over and over
- 5 casos en vivo

### L13 — Patrones y antipatrones (a escribir)
- **Patrones**: Spec-driven (Huntley), TDD with LLMs, Ralph loop (Huntley), Worktrees, Stop&reset (`/clear`), Backpressure (Huntley), Code-over-tools (Ronacher)
- **Antipatrones**: Christmas tree (Fowler), Kitchen sink (Anthropic), Trust-then-verify gap (Anthropic), Bloated CLAUDE.md (Anthropic), Guides-only / Sensors-only (Fowler), Lethal trifecta composition (Willison), Mental disengagement (Ronacher), Slash commands pre-escritos (Ronacher), Things that didn't work (Ronacher list)
- 3 categorías de harness (Fowler)

### L14 — Capstone BookClub (a escribir)
- Integración completa, aplicar todo lo aprendido

### L15 — Costos y modelos nuevos (a escribir)
- Tiers de modelos, prompt caching como enabler
- Evals approach (Willison): empezar chico, SQLite cache, LLM-as-judge con cuidado
- "Is Claude Code going to cost $100/month?" (Willison)
- Local models (Ronacher + ds4.c)
- Protocolo para evaluar modelos nuevos cuando salen

### L16 — Mantenerte vigente + próximos pasos (a escribir)
- **Lista curada de fuentes**:
  - Oficial: code.claude.com, anthropic.com/engineering, claude.com/blog, modelcontextprotocol.io
  - Fowler: martinfowler.com/articles/harness-engineering.html
  - Willison: blog + agentic-engineering-patterns guide + newsletter + tags
  - Huntley: ghuntley.com canonical posts
  - Ronacher: lucumr.pocoo.org canonical posts
  - Mario Zechner / Pi
- **Heurística de Ronacher** ("the center has a bias"): seguir a quienes USAN intensivamente Y critican con especificidad
- **Rutina de actualización**: feeds RSS + newsletter + community digest
- **Próximos pasos profesionales** (plan original): carrera, freelance, SaaS, posicionamiento interno
