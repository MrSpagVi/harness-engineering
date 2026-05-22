# Curso Interactivo: Harness Engineering desde Cero

## Qué es este proyecto

Un curso web interactivo en **un solo archivo HTML autocontenido** (sin build, sin dependencias externas que no sean CDN) que enseña harness engineering desde cero usando un proyecto guía llamado **BookClub** (gestión de biblioteca personal, libros premiados, recomendaciones).

## Audiencia objetivo

JV, Manager Analyst de Data en Newsweek (Buenos Aires). Background fuerte en datos (Microsoft Fabric, PySpark, Power BI, DAX). Conocimiento técnico medio. Habla español. Aprende mejor con analogías + ejemplos concretos + práctica.

## Stack

- HTML5 + CSS + JavaScript vanilla (**UN SOLO archivo**)
- Sin frameworks (no React, no Vue)
- LocalStorage para persistir progreso del usuario
- Diseño dark mode tipo terminal/Anthropic
- Responsive (mobile-friendly)
- Sin dependencias externas salvo Google Fonts si hace falta

## Idioma

Todo el contenido en **español rioplatense neutro** (usá "vos" no "tú"). Términos técnicos en inglés cuando es estándar (harness, hook, skill, sub-agente, MCP server, prompt, context window, etc.).

## Tono

- Directo, sin marketing, sin emojis excesivos (1-2 por sección máximo)
- Asumir CERO conocimiento técnico previo
- Cada concepto: definición → analogía cotidiana → ejemplo concreto en BookClub → checkpoint
- Frases cortas, párrafos cortos
- Evitar palabras como "simplemente", "obvio", "fácil"

## Filosofía pedagógica

El curso enseña **harness engineering como disciplina** — un conjunto de cuatro hábitos mentales (diseñar, diagnosticar, iterar, auditar) que se internalizan con reps diversos, no con un solo proyecto grande.

Por eso:
- BookClub **no es la espina dorsal** del curso. Es el **capstone** (Lección 14): un proyecto integrador al final.
- Cada pieza del harness (Lecciones 6-10) se aprende con un **mini-ejercicio en un dominio distinto** — para entrenar el músculo abstracto, no la receta de BookClub.
- Las Lecciones 11 y 12 entrenan las habilidades meta: diseñar desde cero y diagnosticar problemas.

## Referencias canónicas

Todas las fuentes externas consultadas — Anthropic oficial, Martin Fowler, Simon Willison, Geoffrey Huntley, Armin Ronacher, Mario Zechner — están consolidadas en **[REFERENCES.md](REFERENCES.md)** con:
- URLs canónicas por fuente
- Vocabulario oficial exacto
- Posts canónicos por lección
- Mapeo concepto → lección donde se usa
- Citas killer para usar como aperturas

**Tres principios universales** que cruzan todas las fuentes y deben permear el curso entero:

1. **El context window es el recurso fundamental** (Anthropic textual). Hilo de L5, L7, L9.
2. **Verification > generation** — *"the single highest-leverage thing you can do"* (Anthropic) / *"if you didn't see it run, it isn't a working system"* (Willison). Núcleo de L3, L11, L12.
3. **Simplicidad por defecto** — anti Christmas tree. Aplica al harness Y al curso mismo.

**Dos campos válidos del agentic engineering** que el curso debe exponer (no elegir):

- **Frontier / experimental** (Huntley): Ralph loop, autonomous push, build harness around it.
- **Pragmático / conservador** (Ronacher, Willison): hands-off con verificación, automatizar solo lo regular, evitar mental disengagement.

El estudiante va a transitar entre los dos según el contexto. L11 y L13 deben dar autoridad a ambos.

## Estructura del curso (17 lecciones)

### Bienvenida
- **Lección 0**: Bienvenida + filosofía pedagógica + cómo usar el curso

### Fundamentos (lecciones 1-5)
- **Lección 1**: ¿Qué es un agente de IA? (vs chatbot, loop agéntico)
- **Lección 2**: ¿Qué es un harness? (la regla "si no sos el modelo, sos el harness")
- **Lección 3**: La mentalidad del harness engineer (las 4 habilidades: diseñar, diagnosticar, iterar, auditar)
- **Lección 4**: Tu primera sesión con Claude Code (instalación, comandos, permisos)
- **Lección 5**: El archivo CLAUDE.md (jerarquía, qué meter, qué no)

### Piezas del harness (lecciones 6-10) — cada una con mini-ejercicio en dominio distinto
- **Lección 6**: Permisos y allow-listing — *ejercicio*: configurar permisos para un script de mantenimiento de archivos
- **Lección 7**: Skills personalizadas — *ejercicio*: skill que valida CUITs argentinos
- **Lección 8**: Hooks y guardrails — *ejercicio*: hook que bloquea `rm` fuera de `./tmp/`
- **Lección 9**: Sub-agentes — *ejercicio*: sub-agente "traductor ES→EN"
- **Lección 10**: MCP servers — *ejercicio*: MCP que devuelve la hora UTC

### Diseño y diagnóstico (lecciones 11-13)
- **Lección 11**: Diseñar un harness desde cero (framework de 5 preguntas + 3 escenarios: un scraper, un bot de soporte, un revisor de PRs)
- **Lección 12**: Diagnosticar problemas (taxonomía de fallas + 5 casos en vivo "el agente hizo X, ¿qué falta?")
- **Lección 13**: Patrones de oro y antipatrones (SDD, TDD, Ralph loop, worktrees, stop&reset + Christmas tree harness, etc.)

### Capstone (lección 14)
- **Lección 14**: Proyecto BookClub — integración completa de todas las piezas en una app real (SQLite, FastAPI, skills, hooks, sub-agentes, MCP)

### Más allá (lecciones 15-16)
- **Lección 15**: Costos y modelos nuevos (caching, prompts específicos, eval set personal, protocolo cuando sale un modelo nuevo)
- **Lección 16**: Mantenerte vigente + próximos pasos (Anthropic docs, Simon Willison, Geoffrey Huntley, Armin Ronacher, awesome lists, Discord oficial, comunidades · carrera, freelance, SaaS, posicionamiento interno)

## Elementos visuales en cada lección

1. **Header** con tag de sección (Fundamentos/Proyecto/Práctica/Carrera) + título + lead paragraph
2. **Definiciones formales** en cajas destacadas para términos clave
3. **Analogías** en cajas con icono distintivo (ej: 🐎 para "harness", 🔌 para MCP)
4. **Comparaciones** "❌ Sin harness vs ✅ Con harness" en grid de 2 columnas
5. **Bloques de código** con botón de copiar (todos copiables)
6. **Tablas** para comparativas
7. **Callouts**: tip (azul), warning (amarillo), success (verde), danger (rojo)
8. **Project steps**: cajas con número de paso para el proyecto BookClub
9. **Checkpoint quiz** al final de cada lección: 1 pregunta multiple choice (4 opciones) con feedback al clickear
10. **Diagramas SVG** cuando ayuden (loop agéntico, jerarquía CLAUDE.md, etc.)
11. **Navegación** entre lecciones (anterior/siguiente)

## Persistencia (LocalStorage)

- Lección actual del usuario
- Lecciones completadas (marca ✓ en sidebar)
- Respuestas a checkpoints (correctas/incorrectas)
- Botón "resetear progreso" en algún lugar

## Sidebar fija (izquierda)

- Logo "📚 Harness Engineering"
- Subtítulo: "Curso desde cero · Proyecto BookClub"
- Navegación agrupada por sección
- Indicador de lección actual (highlight)
- Check ✓ en lecciones completadas
- Scroll independiente del contenido

## Header

- Barra de progreso fija arriba (% del curso completado)

## Reglas duras (NUNCA romper)

- **NUNCA** dividir en múltiples archivos. UN solo `.html`.
- **NUNCA** usar build steps, npm, ni frameworks.
- **NUNCA** dejar lecciones con menos de 600 palabras de contenido real.
- **NUNCA** copiar contenido genérico de internet — todo el contenido es original y específico a BookClub.
- **SIEMPRE** incluir checkpoint quiz en cada lección (excepto la 0 y la 16).
- **SIEMPRE** incluir al menos 1 ejemplo de código real en cada lección de fundamentos en adelante.
- **SIEMPRE** dar contexto narrativo antes de mostrar código (no tirarlo seco).
- **SIEMPRE** generar un reporte de handoff detallado (`handoff.md` o actualización de `walkthrough.md`) antes de agotar los tokens o la cuota (tokens/quota) para asegurar la continuidad agéntica sin pérdida de contexto y continuar con el proyecto sin problemas.

## Paleta de colores

| Rol | Hex |
|---|---|
| Background principal | `#0f1419` (casi negro) |
| Background cards | `#1a2027` |
| Background elevado | `#232b35` |
| Borders | `#2d3743` |
| Texto principal | `#e6e9ef` |
| Texto secundario | `#9ba5b3` |
| Acento principal | `#d95c41` (naranja Anthropic) |
| Success | `#4ade80` |
| Warning | `#fbbf24` |
| Info | `#60a5fa` |
| Código bg | `#11151c` |

## Tipografía

- **Sistema**: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`
- **Código**: `'SF Mono', Monaco, 'Cascadia Code', monospace`
- **Line-height**: 1.7 en cuerpo, 1.3 en headings

## Skills disponibles

- `actualizame` — reporte semanal de novedades en harness engineering desde fuentes canónicas (REFERENCES.md). Mantiene ledger anti-duplicación en `reports/_ledger.json`. Output: `reports/YYYY-MM-DD-update.md`. Invocar con `/actualizame` o diciendo "actualizame" / "novedades" / "ponéme al día".

## Comandos para verificar

Abrir el `.html` en un browser y verificar:

- Todas las lecciones se ven (16 total)
- Sidebar funciona (clicks navegan)
- Botón "Siguiente" avanza
- Checkpoints responden con feedback
- Progreso se guarda al recargar
- Responsive en mobile (sidebar colapsa o se mueve arriba)
- Código tiene botón "copiar" funcional
