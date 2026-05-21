# 10 · Sub-agentes y delegación

← [Volver al índice](index.md) · ← [Anterior: Skills: capacidades curadas](09-skills.md)

---

> **Idea central**: un solo hilo de conversación con mucho contexto **se ensucia**, y un agente con contexto sucio toma malas decisiones. Gentle-AI cambia eso con una regla simple: cuando la tarea crece, **el orquestador delega** en sub-agentes con **contexto propio**, le pasa **skills exactas**, y recibe **resultados estructurados**. Vos no operás nada; el flujo es automático.

---

## El problema: contexto que se ensucia

Imaginá que arrancás una sesión de Claude/OpenCode/Cursor para arreglar un bug chico. Una hora después, te encontrás haciendo:

1. Refactor de un módulo grande.
2. Una migración de DB.
3. Documentación.
4. Un PR.

Todo **en el mismo hilo**. El agente acumuló:

- el código que leyó (50+ archivos),
- las decisiones que tomaron,
- los intentos fallidos,
- los errores de tests,
- los logs de comandos,
- la conversación.

Resultado: el modelo se confunde, mezcla decisiones de tareas distintas, repite errores que ya cometió y arrastra un context window inflado que cuesta plata y baja la calidad. Esto es el **monolithic orchestrator anti-pattern**.

La solución de Gentle-AI: **el orquestador no ejecuta todo. Delega.**

---

## ¿Qué es un sub-agente?

Un **sub-agente** es un agente completo, con su propio contexto, herramientas y permisos, que el orquestador lanza para **una tarea acotada**. Cuando termina, devuelve un resultado y muere (el contexto se descarta).

No es un "script tonto" ni un prompt embebido. Es **otra instancia del agente**, con un rol claro:

| Sub-agente | Su única función |
| --- | --- |
| `sdd-explore` | Investigar el código y devolver un resumen estructurado. |
| `sdd-propose` | Tomar la exploración y proponer un approach. |
| `sdd-design` | Tomar la propuesta y diseñar técnicamente. |
| `sdd-apply` | Implementar **una** tarea concreta. |
| `sdd-verify` | Validar el cambio (tests, lint, criterios). |
| `judgment-day` | Revisión adversaria del cambio en dos jueces paralelos. |

> Hay más, pero estos son los más visibles. El nombre del sub-agente coincide con la skill que ejecuta (página 9).

---

## "Super" sub-agentes

Tres cosas distinguen a los sub-agentes de Gentle-AI de "agentes hijos" cualquiera:

### 1) El orquestador resuelve skills una sola vez y se las pasa al sub-agente

El orquestador lee `.atl/skill-registry.md` **al inicio de la sesión**, identifica las skills relevantes, y le pasa al sub-agente las **rutas exactas** del `SKILL.md` que tiene que leer. No le pasa un resumen, no le pasa una versión digerida: le pasa **el archivo entero**, que el sub-agente lee antes de actuar.

### 2) Se adaptan al proyecto en tiempo real

Un `sdd-apply` corriendo en un proyecto React recibe las skills de React. El mismo `sdd-apply` en un proyecto Go recibe `go-testing` y las convenciones Go. **No es un sub-agente "hardcodeado"**: las reglas se resuelven por contexto.

### 3) Persisten su trabajo en Engram

Cada sub-agente, al terminar, guarda su artefacto en Engram con un `topic_key` predecible (ver página 8). Eso significa que **el siguiente sub-agente arranca donde el anterior dejó**, incluso si pasaron horas o días entre uno y otro.

---

## Las 6 reglas que disparan delegación

Estas son **reglas explícitas** del orquestador. Aparecen en la doc oficial como *delegation stop rules*. No las tenés que memorizar, pero entenderlas te ayuda a leer el comportamiento del agente:

| Regla | Condición | Acción |
| --- | --- | --- |
| **4-file rule** | Necesita leer **4 o más archivos** para entender un flujo. | Delegar exploración o correr fase `sdd-explore`. |
| **Multi-file write rule** | Va a tocar **2 o más archivos no triviales**. | Usar **un solo writer**, o pedir fresh review antes de cerrar. |
| **PR rule** | Antes de **commit / push / PR** después de tocar código. | Correr fresh review (salvo diff trivial de docs/texto). |
| **Incident rule** | `cwd` equivocado, accidente de git/worktree, recovery de merge, test/env confuso. | **Parar**, correr fresh audit antes de seguir. |
| **Long-session rule** | ~20 tool calls, 5 lecturas exploratorias, o 2 ediciones no mecánicas con complejidad acumulada. | Pausar, delegar, replan, o justificar por qué no. |
| **Fresh review rule** | Review adversaria de diffs, conflictos, PR readiness, incidentes. | Usar **contexto fresco** si el agente lo soporta. |

> Lo que vas a notar: cuando una tarea crece, el agente **te avisa** que va a delegar (*"voy a lanzar sdd-explore para mapear esto"*) en vez de seguir tirando línea de comandos en el mismo hilo. Esa es la regla aplicándose.

---

## Cómo se ve la delegación en cada agente

No todos los agentes delegan igual. Esto lo cubrimos en la **página 3**, pero acá lo aterrizamos al comportamiento concreto:

| Agente | Cómo lanza sub-agentes |
| --- | --- |
| **Claude Code** | Vía la **Task tool** nativa. Cada sub-agente arranca con `Agent({ ... })` y context window propio. |
| **OpenCode / Kilo Code** | **Overlay multi-agente** con 11 agentes nombrados en `opencode.json` (`gentle-orchestrator` + 10 fases SDD). Cada uno con su modelo, sus tools y sus permisos. |
| **Cursor** | Archivos `~/.cursor/agents/sdd-{phase}.md`. Cursor auto-delegar según el `description` en el frontmatter. |
| **VS Code Copilot** | Tool `runSubagent` con soporte de **ejecución paralela**. |
| **Kiro IDE** | Sub-agents nativos en `~/.kiro/agents/sdd-{phase}.md` con `model:` por fase. |
| **Qwen / Kimi / Gemini CLI** | Sub-agents nativos del agente. |
| **Pi** | Sub-agents administrados por el paquete `pi-subagents`. |
| **Codex / Windsurf / Antigravity / OpenClaw** | **Solo-agent**: no hay sub-agents custom. SDD corre **inline** en la misma sesión; Engram cumple el rol de "memoria entre fases". |

### Solo-agent ¿es peor?

No necesariamente. Para tareas chicas/medianas, anda bien. La diferencia se siente:

- En features grandes (muchas fases SDD) → full-delegation gana por aislamiento de contexto.
- En revisiones adversarias (donde **querés** contexto fresco) → full-delegation gana.
- En tareas chicas → da lo mismo.

---

## Cómo se ve en la práctica

Pedido típico:

> *"Quiero agregar autenticación con magic links."*

Flujo en un agente con full-delegation (Claude Code, por ejemplo):

```
[orchestrator] tarea reconocida como mediana/grande → propongo SDD

[orchestrator] → delegating to sdd-explore
   skill: sdd-explore (read-only)
   target: /api/auth, /lib/session, /middleware
   ─────────────────────────────────────────────
   [sdd-explore] inicia con contexto limpio
   [sdd-explore] lee 8 archivos
   [sdd-explore] guarda artefacto en engram: sdd/magic-links/exploration
   [sdd-explore] returns → resumen estructurado

[orchestrator] recibo resumen, lo muestro al usuario
[orchestrator] gate de aprobación

✓ usuario aprueba

[orchestrator] → delegating to sdd-propose
   ...
```

Cosas para notar:

1. **El orquestador no leyó los 8 archivos**. El sub-agente sí, y devolvió un resumen. Tu sesión principal queda **liviana**.
2. **El sub-agente arrancó con contexto limpio**. No arrastra historial irrelevante de antes.
3. **Engram persiste el artefacto**. Si cerrás y volvés mañana, el siguiente sub-agente lo lee y sigue.

---

## ¿Qué hago yo en todo esto?

**Nada**. En serio.

- No tenés que **decidir** cuándo delegar (el orquestador lo hace).
- No tenés que **lanzar** sub-agentes a mano (los lanza el orquestador).
- No tenés que **memorizar** los nombres (el orquestador los selecciona por skill).
- No tenés que **coordinar** entre ellos (Engram es el pegamento).

Lo único que hacés vos es **aprobar en los gates** (página 8) y **redirigir** si algo no te cierra. El resto fluye solo.

> Si en algún punto querés ver qué pasó, está todo en Engram. Buscá con `engram tui` o `engram search "magic-links"`.

---

## Cuándo conviene que pidas delegación a propósito

Casos donde **vos** querés que el agente delegue:

- *"Antes de tocar nada, explorá el código y devolveme un resumen."* → fuerza `sdd-explore`.
- *"Revisá este diff con ojo crítico, contexto fresco."* → dispara la *fresh review rule*.
- *"Hacé judgment-day sobre este cambio."* → corre **dos jueces paralelos** y compara.
- *"Mirá esto desde cero, ignorá nuestra conversación previa."* → contexto limpio.

Son atajos para forzar buenas prácticas cuando el orquestador no las disparó solo.

---

## Mitos y aclaraciones

### "Más sub-agentes = más caro"

A veces sí, a veces no. Cada sub-agente arranca con contexto fresco, así que **no paga el token tax** del hilo monolítico. En sesiones largas, **delegar baja el costo total** porque evita arrastrar 50 archivos leídos en cada turno.

En OpenCode/Kiro multi-mode podés además usar **modelos más baratos** en fases exploratorias (página 15).

### "Si delego, pierdo control"

Falso. Los **gates de aprobación** entre fases (página 8) son exactamente eso: control. Vos aprobás, redirigís, retrocedés. Lo único que dejás de hacer es **leer las 50 líneas de exploración por tu cuenta**.

### "Si el sub-agente se equivoca, ¿cómo me entero?"

El orquestador valida la salida (estructura del resultado, archivos modificados, criterios). Si algo huele mal, te avisa o re-lanza con instrucciones más claras. Y siempre podés ver el rastro en Engram.

### "Sub-agente ≠ paralelismo"

En la mayoría de los agentes los sub-agentes corren **uno detrás del otro**. Solo VS Code Copilot soporta **paralelo real** vía `runSubagent`. OpenCode tiene un *background-agents plugin* opcional. El resto, secuencial.

---

## Errores comunes

### "El agente no delega y mete todo en el mismo hilo"

1. Verificá que tengas SDD instalado: `gentle-ai install --component sdd` (o un preset que lo incluya).
2. Verificá el skill registry: `gentle-ai skill-registry refresh --force`.
3. Forzá explícitamente: *"delegá esto a sdd-explore en vez de hacerlo inline"*.

### "El agente delega para tareas triviales"

Decile: *"esto no necesita SDD, hacelo directo"*. O reformulá tu prompt para que el orquestador lo perciba como chico.

### "Lanzó un sub-agente y no veo qué hizo"

Mirá los logs del agente (cada agente tiene su forma de mostrar la traza). Y revisá Engram:

```bash
engram tui
# proyecto → memorias → buscá topic_key "sdd/<change>/..."
```

---

## Resumen

- **Un solo hilo monolítico** se ensucia → menos calidad, más costo.
- El **orquestador delega** en sub-agentes con **contexto propio**, **skills exactas** y **persistencia en Engram**.
- Hay **6 reglas explícitas** que disparan delegación (4 archivos, 2 writes, antes de PR, incidentes, sesiones largas, fresh review).
- En agentes con **full-delegation** (Claude Code, OpenCode, Cursor, Kiro, etc.) el sub-agente arranca limpio. En **solo-agent** (Codex, Windsurf, Antigravity, OpenClaw) corre inline; Engram cumple el rol de memoria entre fases.
- Vos **no operás** la delegación. Solo aprobás en gates y redirigís si algo no cierra.

---

## Siguiente paso

➡️ **[11 · Personas y permisos](11-personas-permisos.md)** — la persona "teaching-first" con permisos *security-first*: qué puede y qué no puede hacer tu agente por defecto, y cómo cambiarlo.

📚 Doc de referencia (avanzado): **[docs/intended-usage.md](../intended-usage.md)** — sección *Sub-Agents -- Smarter Than You Think* y *Delegation Stop Rules*.

---

← [Volver al índice](index.md) · ← [Anterior: Skills: capacidades curadas](09-skills.md)
