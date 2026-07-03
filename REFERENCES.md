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

## 6. Ecosistema agentic — snapshot junio 2026

*Sección agregada el 2026-05-22. Actualizada 2026-06-27. Mantener fresca con cada reporte semanal de `actualizame`.*

### Modelos vigentes (junio 2026)

- **Claude Opus 4.8** — **nuevo default** de Claude Code (Max, Team Premium, Enterprise pay-as-you-go, API) desde Week 22 (2026-05-25). High effort por defecto; `/effort xhigh` para lo más duro. Fast mode corre sobre 4.8 a $10/$50 por MTok. Reemplazó a Opus 4.7 (que había sido frontier desde abril/Week 16).
- **Claude Sonnet 4.6** — workhorse balanceado. Default para sub-agentes y para la mayoría de Routines (cloud).
- **Claude Haiku 4.5** — fastest + cheapest. Default del subagent Explore (read-only).
- **Claude Fable 5 / Mythos 5** — frontier nueva de Anthropic (~2026-06-09). **Suspendidos días después (2026-06-12/13) por una directiva de export control del gobierno de US** citando un supuesto "jailbreak" (que era code review estándar). NO es un modelo usable establemente hoy — no reorganizar workflows alrededor. Saga relacionada: "silent guardrails" del system card de Fable (intervenciones que degradaban en silencio temas de frontier AI), revertidos tras backlash → ahora flagged requests caen a Opus 4.8 y devuelven el motivo del refusal.
- **Competencia**: **Gemini 3.5 Flash** (Google, speed-first, base de Antigravity 2.0); **GPT-5.6** (OpenAI, serie Sol/Terra/Luna, preview limitada, jun 2026); presión local-first creciente (DeepSeek V4 Pro a ~5% del costo de Claude, Qwen3.6-27B para coding local).

### Plataformas competidoras a Claude Code

- **Antigravity 2.0** (Google, lanzado 19-mayo-2026 en I/O) — standalone desktop app + CLI + SDK + Managed Agents API. Multi-agent architecture (manager + subagents) builtin. Pricing $25/mes Pro vs Claude Max $200/mes. SWE-bench: Claude Code mantiene ventaja en calidad, Antigravity gana en velocidad/precio. Google deprecó Gemini CLI; todo va a Antigravity CLI desde 18-jun-2026. Heuristic: NO migrar proyectos productivos, vale curiosidad en sandbox.
- **Cursor** — IDE fork de VS Code, foco en autocomplete + agent. Maduro pero más expensive y menos extensible que Claude Code.
- **Windsurf** — Cascade agent. Foco en flow continuo. Niche.
- **GitHub Copilot** — más antiguo, foco autocomplete; ahora agregando agents pero detrás en quality.

### Claude Code 2.1.x — features clave de 2026 (hasta junio / Week 26)

Tracking desde Week 13 (marzo) hasta Week 26 (junio). Para detalle por semana ver https://code.claude.com/docs/en/whats-new

**Junio 2026 (Weeks 21-26) — lo nuevo desde el snapshot de mayo:**
- **Opus 4.8 nuevo default** + **dynamic workflows** (orquestar decenas/cientos de subagentes desde un script) + **security-guidance plugin** (Week 22).
- **Sub-agentes anidados** — un sub-agente puede spawnear sub-agentes; árbol en `/agents`; cap 5 niveles en background (Week 24). **Corrige el "cannot nest" anterior.**
- **`--safe-mode`** — arranca sin CLAUDE.md/skills/plugins/hooks/MCP para aislar config rota (Week 24). Herramienta de diagnóstico (L12).
- **`/cd`** (mover sesión sin romper cache) + **`fallbackModel`** (hasta 3 fallbacks) (Week 24).
- **Artifacts** (output de sesión → página live compartible) + **deny/ask rules con `Tool(param:value)`** ej. `Agent(model:opus)` + **auto mode bloquea git destructivo** (Week 25).
- **`claude mcp login/logout`** (OAuth de MCP desde la shell) + **shell mode responde al `!`** (`! npm test` → explicación) + **background subagents piden permiso en sesión principal** + **`sandbox.credentials`** + **`autoMode.classifyAllShell`** (Week 26).
- **Auto mode en Bedrock/Vertex/Foundry** (Week 23) + **Auto mode en Pro** + **`/code-review`** + **`/usage`** (Week 21).

**Marzo-mayo 2026 (Weeks 13-20):**

- **Auto mode** (Week 13, marzo) — clasificador maneja permission prompts. Safe actions corren sin interrupción; risky bloqueadas. Middle ground entre approve-all y `--dangerously-skip-permissions`.
- **Computer use en CLI** (Week 14, marzo) — Claude abre apps nativas, clickea UI, verifica cambios desde la terminal.
- **Ultraplan** (Week 15, abril) — draft de plan en cloud, review en web editor, ejecutar remoto o pull local.
- **`/ultrareview`** (Week 17, abril) — fleet de agentes bug-hunting que corre en cloud y devuelve findings al CLI/Desktop.
- **Routines** (Week 16, abril) — templated cloud agents que se disparan por schedule cron, GitHub event, o API call. Necesita Pro/Max. **Esto es lo que usamos para activar `actualizame` semanal.**
- **`claude project purge`** + **PowerShell tool nativo** + **Windows sin Git Bash** (Week 18, abril) — Windows-first improvements.
- **Plugins desde .zip y URL** (Week 19, mayo) — `--plugin-dir` acepta `.zip`, `--plugin-url` fetchea archivos. Plugin marketplace.
- **Hard deny rules en auto mode** (Week 19) — bloquean acciones unconditionally, no se overridean con allow.
- **Agent view (`claude agents`)** (Week 20, mayo) — panel de todas las sesiones interactivas + background con estado.
- **`/goal`** (Week 20) — Claude trabaja entre turns hasta cumplir completion condition.
- **Skills + Hooks juntos en frontmatter** (2.1.x) — Skills pueden incluir hooks PreToolUse/PostToolUse/Stop directamente en YAML. Agent-Scoped Hooks: hooks que solo corren durante el lifecycle de un sub-agente específico.
- **Skills visibles como `/commands`** (2.1.x) — aparecen junto a comandos en el menú `/`. Esconder con flag en frontmatter.
- **Skill hot-reload** (2.1.x) — cambios en `~/.claude/skills` o `.claude/skills` se aplican sin reiniciar sesión.
- **Cambio breaking (2.1.77)** — Agent tool ya NO soporta `resume` parameter. Continuaciones via `SendMessage` con agent ID/name en `to`. New Agent calls siempre arrancan fresh.

### MCP ecosystem destacado

- **Semble** (https://github.com/MinishLab/semble, 442 puntos HN 2026-05-17) — code search server con embeddings + BM25. **98% menos tokens que grep+read**. Zero config, sin API keys. Caso real de "Ubertool" (Ronacher) aplicado a code search.
- **Self-hosted sandboxes + MCP tunnels** (Anthropic Managed Agents, 2026-05-19) — cloud agents pueden conectar a tus propios MCP servers via túneles.

### Anthropic moves recientes

- **Stainless acquired** (2026-05-18) — refuerza el Agent SDK con SDK generation cross-language desde OpenAPI specs.
- **KPMG deploy 276K employees** (2026-05-19) — datapoint enterprise. Claude Code deploy-ready a escala.

### Cómo mantenerse vigente (sistema operativo)

1. **Reports semanales `actualizame`** — `reports/YYYY-MM-DD-update.md` generado cada lunes 9am BA via Routines. Cubre Anthropic, Willison, Huntley, Ronacher, Reddit, HN, MCP spec, X.
2. **Suscribirse a feeds RSS** — Willison `https://simonwillison.net/atom/everything/`, Ronacher `https://lucumr.pocoo.org/feed.atom`.
3. **Cuentas X clave** — @simonw, @aliveevolve (Mario Zechner), @arcaranth (Ronacher), @ClaudeCodeLog (changelog automático del system prompt).
4. **Para Antigravity / Gemini**: blog.google/developers + Logan Kilpatrick en X.

---

## 7. Otros autores del círculo (mencionar en L16)

- **Mario Zechner** — autor de **Pi**, agente minimalista (Read/Write/Edit/Bash, system prompt más corto conocido). Referencia para "harness mínimo".
- **Solomon Hykes** — cita canónica: *"An AI agent is an LLM wrecking its environment in a loop."*
- **Dexter Horthy / HumanLayer** — "A Brief History of Ralph" en humanlayer.dev
- **Boris Cherny** — creador Claude Code (Anthropic)
- **Andrej Karpathy** — acuñó "vibe coding" (feb 2025)
- **Thariq Shihipar** (equipo Claude Code): *"Long running agentic products like Claude Code are made feasible by prompt caching."*

---

## 8. Matt Pocock — aihero.dev

### URL
- **5 Agent Skills I Use Every Day** — https://www.aihero.dev/5-agent-skills-i-use-every-day
- **Learn anything with my teach skill** — https://www.aihero.dev/learn-anything-with-my-teach-skill

### Repo de skills (TRACKING — revisar en cada `actualizame`)
- **Repo oficial**: https://github.com/mattpocock/skills — chequear commits recientes / skills nuevas en `skills/productivity/` y otras carpetas. Si aparece una skill nueva relevante, reportarla en el update semanal.
- Skills ya adoptadas a nivel usuario (`~/.claude/skills/`): `grill-me` (=grilling), `to-prd`, `to-issues`, `tdd`, `improve-codebase-architecture`, `teach`, `handoff`, `writing-great-skills`. Ver [[reference-skills-pipeline]].

### Conceptos clave (a usar en L7, L13)
- **Tesis central**: los agentes no tienen memoria entre sesiones → encodeá tus workflows en **skills** reusables. Una skill bien hecha sube la calidad del output más que cualquier prompt suelto.
- **Pipeline de skills encadenadas** (caso real de *prompt chaining*, patrón 1 de Anthropic): `/grill-me → /to-prd → /to-issues → /tdd`, más `/improve-codebase-architecture` cuando la base está frágil.
  1. **`/grill-me`** — interrogatorio de diseño antes de codear (≈ las 5 preguntas de diseño llevadas a skill).
  2. **`/to-prd`** — conversación → PRD con user stories Agile (contexto comprimido que sobrevive entre sesiones).
  3. **`/to-issues`** — PRD → issues como **vertical slices** (corte fino que atraviesa todas las capas) con relaciones de bloqueo para paralelizar. Anti-patrón: horizontal slices.
  4. **`/tdd`** — red-green-refactor; *"the most consistent way to improve agent outputs"* (= verification > generation hecho hábito).
  5. **`/improve-codebase-architecture`** — profundizar módulos shallow, fronteras de testeo.

### Citas
- *"If you have a garbage code base, the AI will produce garbage within that code base."*
- *"Interview me relentlessly about every aspect of this plan until we reach a shared understanding."* (directiva de `/grill-me`)

### Nota
Sesgo TS/web (GitHub + tests automatizados). Para audiencia data (Fabric/PySpark) el spirit aplica pero los vertical slices y el TDD se ven distinto (testear transformaciones puras separadas del I/O de Spark). Las 5 skills están instaladas a nivel usuario en `~/.claude/skills/` (versión adaptada al español/setup de JV).

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
