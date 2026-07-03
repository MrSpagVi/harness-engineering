---
name: actualizame
description: Genera un reporte semanal con las novedades de los últimos 7 días en harness engineering, fetcheando fuentes canónicas (Anthropic, Willison, Huntley, Ronacher, Reddit, HackerNews, MCP spec, X/Twitter). Mantiene un ledger anti-duplicación para no repetir info ya cubierta. Output en HTML limpio (reports/YYYY-MM-DD-update.html). Usar cuando el usuario diga "actualizame", "novedades", o pida ponerse al día.
when_to_use: el usuario dice "actualizame", "qué hay nuevo", "novedades", "ponéme al día" sobre harness engineering, agentes, Claude Code, MCP, o pide un update semanal
allowed-tools:
  - WebFetch(domain:anthropic.com)
  - WebFetch(domain:claude.com)
  - WebFetch(domain:code.claude.com)
  - WebFetch(domain:simonwillison.net)
  - WebFetch(domain:ghuntley.com)
  - WebFetch(domain:lucumr.pocoo.org)
  - WebFetch(domain:modelcontextprotocol.io)
  - WebFetch(domain:news.ycombinator.com)
  - WebFetch(domain:hn.algolia.com)
  - WebFetch(domain:reddit.com)
  - WebFetch(domain:github.com)
  - WebFetch(domain:raw.githubusercontent.com)
  - WebSearch
  - Read(./REFERENCES.md)
  - Read(./reports/**)
  - Read(./.claude/skills/actualizame/report-template.html)
  - Write(./reports/**)
  - Bash(mkdir -p reports)
  - Bash(date *)
---

# Skill: actualizame

Genera un reporte semanal con las novedades en harness engineering (últimos 7 días), filtrando contra un ledger para no repetir info ya cubierta. **Output: `reports/YYYY-MM-DD-update.html`** — HTML limpio y legible, NO markdown.

## Procedimiento (9 pasos)

### Paso 1 · Calcular el cutoff UTC

Computar `CUTOFF = now_utc - 7 days` en formato ISO 8601 (ej: `2026-05-10T14:00:00Z`).
Usar `date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ` o equivalente. Si el shell no lo soporta, calcularlo a partir de `date -u` y restar manualmente.

También computar `CUTOFF_UNIX` (timestamp Unix) para las APIs que lo requieren (HN Algolia, Reddit).

### Paso 2 · Leer REFERENCES.md (source of truth)

Leer `./REFERENCES.md`. Extraer todas las URLs canónicas agrupadas por autor:

- **Anthropic**: URLs bajo "Anthropic (oficial)"
- **Martin Fowler**: el artículo de harness engineering (rara vez cambia, baja prioridad esta semana)
- **Simon Willison**: además del blog general, agregar su feed Atom canónico: `https://simonwillison.net/atom/everything/`
- **Geoffrey Huntley**: ghuntley.com (probar RSS en `/rss/` o `/feed/`)
- **Armin Ronacher**: feed Atom canónico: `https://lucumr.pocoo.org/feed.atom`
- **MCP spec**: modelcontextprotocol.io
- **Repos de skills (TRACKING)**: las URLs bajo la sección "Repo de skills (TRACKING)" de REFERENCES.md (hoy: github.com/mattpocock/skills)

NO hardcodear URLs en esta skill — siempre leerlas de REFERENCES.md. Si en el futuro se agregan/quitan fuentes ahí, esta skill se adapta sola.

### Paso 3 · Leer el ledger anti-duplicación

Intentar leer `./reports/_ledger.json`. Si no existe (primera corrida), inicializar en memoria:

```json
{
  "schema_version": 1,
  "last_run_utc": null,
  "items": {},
  "topics_covered": []
}
```

Guardar la fecha del último run (`last_run_utc`) — la vas a mostrar en el reporte.

### Paso 4 · Fetchear fuentes core (RSS-first)

**Simon Willison** (RSS Atom):
- Fetchear `https://simonwillison.net/atom/everything/`
- Parsear entries. Filtrar por `<published>` o `<updated>` >= CUTOFF
- Para cada entry: extraer `title`, `link`, `published`, primeras 2-3 oraciones del summary

**Armin Ronacher** (RSS Atom):
- Fetchear `https://lucumr.pocoo.org/feed.atom`
- Mismo procesamiento que Willison

**Geoffrey Huntley** (intentar RSS, fallback a HTML):
- Intentar primero: `https://ghuntley.com/rss/`, después `https://ghuntley.com/feed/`
- Si ambos fallan, fetchear `https://ghuntley.com/` (homepage) y parsear los últimos posts visibles con sus fechas
- Filtrar por fecha >= CUTOFF

**Anthropic** (HTML — no hay RSS público obvio):
- Fetchear `https://www.anthropic.com/news` y `https://claude.com/blog`
- Para release notes: `https://code.claude.com/docs/en/release-notes`
- Parsear las fechas de cada post visible y filtrar

### Paso 5 · Fetchear fuentes extra

**Reddit** (JSON público, sin auth):
- `https://www.reddit.com/r/ClaudeAI/new/.json?limit=50`
- `https://www.reddit.com/r/LocalLLaMA/new/.json?limit=50`
- Filtrar items: `created_utc >= CUTOFF_UNIX` AND `score > 10`

**Hacker News** (Algolia API):
- Hacer estas queries (una por cada keyword):
  - `https://hn.algolia.com/api/v1/search?query=Claude+Code&tags=story&numericFilters=created_at_i>CUTOFF_UNIX`
  - `https://hn.algolia.com/api/v1/search?query=MCP&tags=story&numericFilters=created_at_i>CUTOFF_UNIX`
  - `https://hn.algolia.com/api/v1/search?query=agentic+coding&tags=story&numericFilters=created_at_i>CUTOFF_UNIX`
- Filtrar: `points > 20`. Deduplicar por `objectID`.

**MCP spec**:
- Fetchear `https://modelcontextprotocol.io/` y buscar links a "Updates", "Changelog", o "What's new"
- Si encontrás alguna sección de versiones recientes, parsear por fecha

**Repos de skills (GitHub)** — leer las URLs de la sección "Repo de skills (TRACKING)" de REFERENCES.md (hoy: `https://github.com/mattpocock/skills`):
- Fetchear la página del repo y mirar commits recientes / carpetas de skills (ej. `skills/productivity/`)
- Si hay una **skill nueva** o un cambio relevante desde el último run (comparar contra el ledger), reportarla como item NUEVO (source: `github`) con link al SKILL.md
- Filtrar ruido: solo skills/cambios sustantivos, no bumps de versión triviales

**X / Twitter** (best-effort, sin API):
- Usar `WebSearch` con query: `site:twitter.com OR site:x.com (claude code OR mcp OR anthropic) after:YYYY-MM-DD`
- Tomar top 10 resultados
- Si menos de 3 resultados parecen relevantes, marcar la sección como "cobertura limitada (sin API)"

### Paso 6 · Filtrar contra el ledger

Para cada item recolectado (de cualquier fuente), aplicar esta lógica:

| Estado en ledger | Acción |
|---|---|
| URL ya está + mismo `pub_date` | **Descartar** (ya cubierto, no cambió) |
| URL ya está + `pub_date` cambió | Marcar `ACTUALIZADO` (incluir) |
| URL no está | Marcar `NUEVO` (incluir) |

Mantener un contador: `n_nuevos`, `n_actualizados`, `n_descartados_por_duplicado`.

### Paso 7 · Aplicar filtros de ruido (solo HN y Reddit)

Para items de HN y Reddit que pasaron el filtro de ledger, aplicar además:
- **Keywords en title o cuerpo (whitelist)**: claude code, anthropic, mcp, agentic, harness, skill, hook, prompt injection, eval, sub-agent, sub-agente, llm tool
- Si el title no menciona ninguna keyword, descartar
- Si el title menciona una keyword pero el cuerpo es spam/promo obvio, descartar

### Paso 8 · Generar el reporte (HTML, NO markdown)

**Determinar path del archivo**:
- Base: `reports/YYYY-MM-DD-update.html` (usar fecha UTC de hoy)
- Si existe, usar sufijo: `reports/YYYY-MM-DD-update-2.html`, `-3.html`, etc.

Asegurar que `reports/` existe: `mkdir -p reports`.

**Usar el template**: leer `./.claude/skills/actualizame/report-template.html` y reemplazar los placeholders `{{...}}`. El template ya trae el CSS (dark mode estilo Anthropic, igual paleta que el curso). NO inventar estilos nuevos — solo llenar el contenido. Reglas de formato HTML:

- Cada **fuente** va en un `<div class="source">` con su `<h3>`.
- Cada **item** es un `<li>` con badge: `<span class="badge nuevo">NUEVO</span>` o `<span class="badge upd">ACTUALIZADO</span>`, seguido del `<a href="url">título</a>`, la síntesis (2-3 oraciones), y la acción en `<span class="action">· acción: leé / mirá / FYI</span>`.
- **Callouts**: usar `<div class="callout tip|warn|ok|danger">` con `<span class="lbl">Etiqueta</span>` para destacados.
- El **TL;DR** va en `<div class="tldr">` con un `<ul>`.

**Caso A — hay items NUEVO o ACTUALIZADO**. Placeholders del template:
- `{{TITULO}}` (tag `<title>`) y `{{H1}}` → `Update Harness Engineering · YYYY-MM-DD` (en H1, envolver "Harness Engineering" en `<span class="acc">`).
- `{{SUBTITULO}}` → `Ventana cruda: últimos 7 días · Filtrado contra ledger (último update: YYYY-MM-DD HH:MM UTC, o "primera corrida")`
- `{{CONTENIDO}}` → el cuerpo, con estas secciones en este orden (mismo contenido que la versión markdown previa, ahora en HTML):
  1. `<div class="tldr">` **TL;DR — qué hacer esta semana**: 3 acciones concretas y aplicables.
  2. `<h2>Lo más importante (NUEVO)</h2>` — 1 a 3 piezas que más rinden, 2-3 oraciones cada una (NO listado seco).
  3. `<h2>Actualizado desde el último reporte</h2>` — solo si hay items ACTUALIZADO; si no, omitir.
  4. `<h2>Por fuente</h2>` con un `<div class="source">` por cada una: Anthropic oficial, Simon Willison, Geoffrey Huntley, Armin Ronacher, Reddit (r/ClaudeAI, r/LocalLLaMA), HackerNews, MCP spec, X/Twitter (o "cobertura limitada (sin API)" si <3 resultados).
  5. `<h2>Cambios a considerar para el curso</h2>` — mapear cada novedad relevante → lección de `curso.html`. Si no aplica: "Sin cambios necesarios al curso esta semana." No inflar.
  6. `<h2>Fuentes no disponibles</h2>` — solo si alguna falló (nombre, error, causa). El ledger NO se actualiza para esas.
  7. `<h2>Para practicar esta semana</h2>` — 2 sugerencias accionables en <30 min.
- `{{FOOTER}}` → `Ledger actualizado: N items agregados / M actualizados. Total acumulado: T items.`

**Caso B — sin items NUEVO ni ACTUALIZADO**. Llenar el template con:
- `{{H1}}` → `Update Harness Engineering · YYYY-MM-DD`
- `{{SUBTITULO}}` → `Sin novedades nuevas desde la última corrida (YYYY-MM-DD HH:MM UTC)`
- `{{CONTENIDO}}` → un `<div class="callout ok">` explicando que las fuentes canónicas no publicaron nada nuevo en la ventana que no esté ya en el ledger, un link al último reporte con novedades reales, y un `<h2>Fuentes consultadas</h2>` con la lista (✓ por fuente OK, ⚠ + error por fuente fallida).
- `{{FOOTER}}` → `Sin cambios en el ledger.`

### Paso 9 · Actualizar el ledger

Para cada item NUEVO o ACTUALIZADO incluido en el reporte:

- Agregar / actualizar entry en `ledger.items[url]`:
  ```json
  {
    "title": "...",
    "source": "willison|huntley|ronacher|anthropic|reddit|hn|mcp|x",
    "first_seen": "<timestamp UTC actual SI es NUEVO, sino mantener original>",
    "last_seen": "<timestamp UTC actual>",
    "pub_date": "<pub_date del item>",
    "covered_in_report": "<path del reporte recién creado>"
  }
  ```

**IMPORTANTE — fuentes que fallaron**: NO actualizar `ledger.items` con entries que no se pudieron verificar. Si Reddit falló entero, no agregues nada de Reddit al ledger. La próxima corrida los va a re-encontrar como nuevos.

Setear `ledger.last_run_utc = <timestamp UTC actual>`.

Escribir `reports/_ledger.json` (sobreescribir el anterior).

## Reglas críticas

1. **NO inventes contenido.** Si una fuente no devuelve resultados, decílo. Si no podés verificar la existencia de un post (la URL devuelve 404), no lo incluyas en el reporte.

2. **NO copies bloques largos.** Síntesis en 2-3 oraciones por item, escritas con tus propias palabras. Linkear al original.

3. **Priorizá ACCIONABLE sobre exhaustivo.** El TL;DR debe ser concreto:
   - ✅ Bien: *"Probá la nueva opción `--auto-approve` en una rama de prueba — Anthropic dice 30% menos prompts."*
   - ❌ Mal: *"Leé las novedades de Anthropic."*

4. **Si una fuente falla, seguí con las demás.** Reportala en "Fuentes no disponibles" con el error específico. El ledger NO se actualiza para esa fuente — se reintentará.

5. **Para la sección "Cambios para el curso"**: solo mencionar lecciones específicas si detectaste una novedad que claramente las afecta. Si no aplica, decir explícitamente "sin cambios necesarios esta semana". No inflar.

6. **Output al usuario en el chat (después de escribir el reporte)**:
   ```
   ✓ Reporte generado: reports/YYYY-MM-DD-update.html (abrilo en el browser)

   TL;DR esta semana:
   - [Acción 1]
   - [Acción 2]
   - [Acción 3]

   [Si aplicable: ⚠️ Fuentes que fallaron: ...]
   [Si Caso B: ℹ️ Sin novedades nuevas — el ledger ya tiene todo lo de los últimos 7 días.]
   ```

## Notas de mantenimiento

- El ledger crece con el tiempo. Si en algún momento pasa de 1000 entries, considerá agregar un step para podar items con `last_seen` más viejo que 6 meses.
- Si una fuente nueva se agrega a `REFERENCES.md`, esta skill la va a incluir automáticamente en la próxima corrida — siempre que la URL esté bajo un dominio incluido en `allowed-tools`. Si es un dominio nuevo, actualizar `allowed-tools` en este SKILL.md.
