# 19 · ¿Qué leer ahora?

← [Volver al índice](index.md) · ← [Anterior: Resolución de problemas comunes](18-troubleshooting.md)

---

> **Cómo usar esta página**: ya pasaste por la guía para principiantes. Si ahora querés profundizar en algo específico, esta es **la entrada al resto de la documentación del repo**. No leas todos los docs avanzados de un saque: andá al que responde tu pregunta del momento.

---

## El mapa visual

```
docs/beginner/             ← acabás de leer esta guía
   │
   ├── overview general    → README.md
   │
   ├── conceptos mentales  → intended-usage.md
   │
   ├── docs por agente
   │   ├── todos          → agents.md
   │   ├── Pi             → pi.md
   │   ├── Kiro IDE       → kiro.md
   │   └── Antigravity    → antigravity-sdd-workaround.md
   │
   ├── docs por componente
   │   ├── Engram         → engram.md
   │   ├── skill registry → skill-registry.md
   │   ├── OpenCode profs → opencode-profiles.md
   │   ├── OpenSpec       → openspec-config.md
   │   └── catálogo       → components.md
   │
   ├── docs por tarea
   │   ├── uso completo   → usage.md
   │   ├── plataformas    → platforms.md
   │   ├── CI / scripts   → non-interactive.md
   │   ├── backups        → rollback.md
   │   └── GGA en Windows → gga-powershell-shim.md
   │
   ├── docs de autoría
   │   ├── crear skill    → skill-style-guide.md
   │   └── contribuir     → CONTRIBUTING.md
   │
   └── docs profundos (devs)
       ├── arquitectura   → architecture.md
       ├── mapa código    → CODEBASE-GUIDE.md
       ├── PRD principal  → PRD.md
       └── PRD agent-bldr → PRD-AGENT-BUILDER.md
```

---

## Por pregunta concreta

### "Quiero entender el modelo mental completo de Gentle-AI"

📘 **[docs/intended-usage.md](../intended-usage.md)** — la página única que explica cómo usar Gentle-AI según la intención de sus autores. Lectura **muy recomendada** después de esta guía. Cubre el comportamiento esperado del orquestador, las reglas de delegación, la regla de oro ("cuanto menos pienses en gentle-ai, mejor funciona"), y la diferencia entre skills incluidas y skills de framework.

### "Quiero ver TODOS los flags del CLI y todas las pantallas de la TUI"

📘 **[docs/usage.md](../usage.md)** — referencia completa de:

- `gentle-ai install` con todos los flags.
- `gentle-ai sync` y su scope.
- `gentle-ai update` / `upgrade`.
- `gentle-ai skill-registry refresh`.
- `gentle-ai uninstall` (managed uninstall).
- Personas, presets, componentes.

Ideal para tener a mano cuando estás scripteando algo o curioseando opciones.

### "Quiero los detalles exactos de un agente específico"

📘 **[docs/agents.md](../agents.md)** — matriz completa con paths de config por agente, comportamiento de delegación, soporte de MCP, multi-mode, slash commands. Notas detalladas por cada uno de los 14 agentes.

Hay dos docs específicos para agentes con complejidad propia:

- **[docs/pi.md](../pi.md)** — todo sobre el agente Pi: comandos, paquetes, persona, modelos, troubleshooting.
- **[docs/kiro.md](../kiro.md)** — Kiro IDE: paths, MCP en `settings/`, sub-agents nativos, multi-modelo nativo, specs workflow.
- **[docs/antigravity-sdd-workaround.md](../antigravity-sdd-workaround.md)** — particularidades del workaround SDD en Antigravity.

### "Estoy en Windows y algo no anda"

📘 **[docs/platforms.md](../platforms.md)** — paths de config por agente en Windows, instalación con Scoop, notas sobre PowerShell vs Git Bash, verificación de firma.

📘 **[docs/gga-powershell-shim.md](../gga-powershell-shim.md)** — específico para entender cómo funciona el shim `gga.ps1` y por qué GGA anda en PowerShell.

### "Quiero correr Gentle-AI en CI / automatizar"

📘 **[docs/non-interactive.md](../non-interactive.md)** — modo no-interactivo, flags soportados, comportamiento por plataforma, manejo de errores, ejemplos para scripts.

### "Quiero entender los backups a fondo"

📘 **[docs/rollback.md](../rollback.md)** — política de retención, formato de snapshots (nuevo y legacy), comportamiento de restore, qué cubre y qué NO cubre, comportamiento ante fallos de verificación.

### "Quiero personalizar mis perfiles SDD en OpenCode"

📘 **[docs/opencode-profiles.md](../opencode-profiles.md)** — referencia completa: dos estrategias (`generated-multi` y `external-single-active`), creación por TUI/CLI, gestión de perfiles, reasoning effort levels, comportamiento del sync por estrategia.

### "Quiero entender mejor el skill registry"

📘 **[docs/skill-registry.md](../skill-registry.md)** — runtime flow, refresh flow, contrato del index, comportamiento de scoping (project vs user), authoring flow para crear nuevas skills.

📘 **[docs/skill-style-guide.md](../skill-style-guide.md)** — cómo escribir un `SKILL.md` bien: convenciones del frontmatter, estructura del cuerpo, triggers efectivos.

### "Quiero ver el catálogo completo de componentes, skills y presets"

📘 **[docs/components.md](../components.md)** — listado canónico de:

- 8 componentes con sus IDs.
- 20 skills incluidas (10 SDD + 10 foundation).
- 4 presets (`full-gentleman`, `ecosystem-only`, `minimal`, `custom`).
- Comportamiento de GGA.

Útil cuando estás dudando si una skill o componente está incluido.

### "Quiero saber a profundidad cómo funciona Engram"

📘 **[docs/engram.md](../engram.md)** — referencia completa de comandos CLI + lista de tools MCP (`mem_*`), comportamiento de detección de proyecto, sincronización de equipo, y casos avanzados.

Para más profundidad técnica: [github.com/Gentleman-Programming/engram](https://github.com/Gentleman-Programming/engram).

### "¿Cómo se integra Gentle-AI con OpenSpec?"

📘 **[docs/openspec-config.md](../openspec-config.md)** — config a nivel proyecto que las fases SDD pueden usar para convenciones, Strict TDD, y metadata de testing.

### "Quiero contribuir al proyecto"

📘 **[CONTRIBUTING.md](../../CONTRIBUTING.md)** (en la raíz del repo) — cómo contribuir, qué espera el equipo, estándares de PR. Lo expandimos en la **página 20** de esta guía.

📘 **[docs/CODEBASE-GUIDE.md](../CODEBASE-GUIDE.md)** — mapa de mantenedores para ownership por carpeta, *architecture boundaries*, y guardrails de review.

📘 **[docs/architecture.md](../architecture.md)** — layout del código (`cmd/`, `internal/`), testing (unit, E2E con Docker), relación con Gentleman.Dots.

### "Quiero leer el PRD original — el documento de producto"

📘 **[PRD.md](../../PRD.md)** (en la raíz del repo) — el PRD principal de Gentle-AI. **Lectura larga** (~70k caracteres). Describe la visión, los problemas, las decisiones de diseño. Útil para entender el "por qué" detrás de cada feature.

📘 **[PRD-AGENT-BUILDER.md](../../PRD-AGENT-BUILDER.md)** — PRD específico del **Agent Builder** (el flow para crear tu propio agente desde la TUI). También extenso.

### "Quiero ver cómo se testean los flujos E2E"

📘 **[docs/docker-e2e-testing.md](../docker-e2e-testing.md)** — cómo se corren los tests E2E en contenedores Ubuntu y Arch.

---

## Por nivel de profundidad

Una forma alternativa de pensarlo, si preferís un orden lineal:

### Nivel 1 · Usuario casual (ya estás acá)

- Esta guía completa.
- Snippets puntuales del **README**.

### Nivel 2 · Usuario que quiere dominar la herramienta

- **`intended-usage.md`** — modelo mental.
- **`usage.md`** — referencia CLI/TUI.
- **`engram.md`** — comandos de memoria.
- **`agents.md`** — detalles del agente que usás.

### Nivel 3 · Power user / equipo

- **`opencode-profiles.md`** — si usás OpenCode/Kilo.
- **`skill-registry.md`** + **`skill-style-guide.md`** — si querés crear skills propias.
- **`non-interactive.md`** — si automatizás.
- **`rollback.md`** — si manejás backups con disciplina.

### Nivel 4 · Contribuidor

- **`CONTRIBUTING.md`**.
- **`CODEBASE-GUIDE.md`**.
- **`architecture.md`**.
- **`PRD.md`** + **`PRD-AGENT-BUILDER.md`** (opcional, contexto histórico).

---

## Tres lecturas que te recomendamos elegir una y leer ya

Si tuviera que recomendarte **una sola** lectura inmediata después de esta guía, basado en tu situación:

### Si querés "dominar" Gentle-AI

➡️ **[intended-usage.md](../intended-usage.md)**.

Razón: es la única página que conecta todo el modelo mental. Si entendés esa página, entendés Gentle-AI.

### Si vas a configurar muchos proyectos / equipos

➡️ **[usage.md](../usage.md)**.

Razón: es la referencia CLI completa. Te vas a ahorrar tiempo cada vez que necesites un flag.

### Si vas a contribuir o crear cosas propias

➡️ **[CONTRIBUTING.md](../../CONTRIBUTING.md)** seguido de **[architecture.md](../architecture.md)**.

Razón: te ahorrás idas y vueltas en code review entendiendo qué espera el equipo y dónde vive cada cosa.

---

## Comunidad y soporte

Cuando los docs no alcanzan:

- **Issues / preguntas**: [github.com/Gentleman-Programming/gentle-ai/issues](https://github.com/Gentleman-Programming/gentle-ai/issues)
- **Repos relacionados**:
  - [engram](https://github.com/Gentleman-Programming/engram) — el binario de memoria.
  - [Gentleman-Skills](https://github.com/Gentleman-Programming/Gentleman-Skills) — skills de framework (React, Angular, etc.).
  - [gentle-pi](https://github.com/Gentleman-Programming/gentle-pi) — el harness Pi.
  - [agent-teams-lite](https://github.com/Gentleman-Programming/agent-teams-lite) — predecesor (archivado, ya no se mantiene).
- **Plugins de comunidad** mencionados en el README:
  - [sub-agent-statusline](https://github.com/Joaquinvesapa/sub-agent-statusline)
  - [sdd-engram-plugin](https://github.com/j0k3r-dev-rgl/sdd-engram-plugin)

---

## ¿Y los issues "good first" para empezar a contribuir?

Buscá en GitHub:

```
is:issue is:open label:"good first issue"   repo:Gentleman-Programming/gentle-ai
```

O por status aprobado:

```
is:issue is:open label:"status:approved"    repo:Gentleman-Programming/gentle-ai
```

La página 20 entra en detalle.

---

## Resumen

- **Esta guía** te dejó listo para usar Gentle-AI con criterio.
- A partir de acá, los **docs avanzados del repo** responden preguntas puntuales — no son lectura lineal.
- Tres recomendaciones inmediatas según tu perfil:
  - **Modelo mental** → `intended-usage.md`.
  - **Referencia técnica** → `usage.md`.
  - **Contribuir** → `CONTRIBUTING.md` + `architecture.md`.
- El **PRD** existe y es largo. Lectura opcional: útil para entender el "por qué", no necesaria para usar.

---

## Siguiente paso

➡️ **[20 · Cómo contribuir siendo principiante](20-contribuir.md)** — la última página de esta guía. Issues etiquetados como `good first issue`, cómo correr los tests, qué espera `CONTRIBUTING.md`, y cómo proponer una nueva skill o documento.

---

← [Volver al índice](index.md) · ← [Anterior: Resolución de problemas comunes](18-troubleshooting.md)
