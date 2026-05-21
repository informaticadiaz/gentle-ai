# 1 · ¿Qué es Gentle-AI?

← [Volver al índice](index.md)

---

## En una frase

Gentle-AI **no instala** agentes de IA. Lo que hace es **configurar** los agentes que ya tenés (o vas a instalar) para que dejen de ser "un chatbot que escribe código" y pasen a tener **memoria, skills, flujos de trabajo, herramientas externas (MCP) y una persona que te enseña** mientras trabajan.

> Pensalo como la diferencia entre **comprar un auto** y **darle ABS, GPS, asistente de carril y caja de herramientas**. El auto ya andaba; ahora hace lo mismo, pero mucho mejor y con menos accidentes.

---

## El problema que resuelve

Cuando instalás un agente de IA para programar (Claude Code, OpenCode, Cursor, Gemini CLI, Codex, Windsurf, etc.) la experiencia inicial suele ser parecida a esta:

- Te escribe código, pero **se olvida** de las decisiones de la sesión anterior.
- Cada vez que abrís el editor tenés que **volver a explicarle** el stack, la convención de tests, la estructura del repo.
- Si la tarea es grande, **se mete en cualquier dirección** sin un plan claro.
- Lee 15 archivos en un solo hilo y **se confunde** con lo que ya leyó.
- No tiene una forma estándar de **revisar** lo que él mismo escribió antes del commit.
- No conoce las **convenciones** de tu proyecto a menos que se las pegues a mano cada vez.

Esa es la experiencia "chatbot". Funciona para tareas chicas y se rompe rápido cuando el proyecto crece.

---

## Lo que cambia con Gentle-AI

Gentle-AI te configura el agente para que tenga, **de fábrica**, varias capas que normalmente tendrías que armar a mano y mantener vos:

| Capa | Qué aporta | Sin Gentle-AI |
| --- | --- | --- |
| **Engram (memoria persistente)** | Guarda decisiones, bugs resueltos y contexto entre sesiones. | El agente "empieza de cero" cada vez. |
| **SDD (Spec-Driven Development)** | Flujo: explorar → proponer → especificar → diseñar → implementar → verificar. | El agente improvisa en tareas grandes. |
| **Skills curadas** | Capacidades reutilizables (PR, commits, issues, docs cognitivas, etc.) que el agente descubre solo. | Vos copiás prompts de internet. |
| **Servidores MCP** | Herramientas externas (filesystem, GitHub, búsqueda, etc.) conectadas al agente. | No tiene acceso a herramientas más allá del chat. |
| **Persona "teaching-first"** | El agente explica lo que hace y aplica permisos *security-first* por defecto. | Hace cambios sin contarte y a veces se va al pasto. |
| **Sub-agentes + delegación** | Cuando una tarea crece, el orquestador delega en sub-agentes con su propio contexto. | Un solo hilo monolítico que se ensucia. |
| **Per-phase model assignment** *(OpenCode)* | Modelo barato para explorar, modelo potente para diseñar. | Pagás el modelo caro para todo. |

Resultado: el mismo agente que ya usabas, ahora **recuerda**, **planifica**, **delega**, **revisa** y **enseña**.

---

## Lo que Gentle-AI **NO** es

Esto es tan importante como lo anterior:

- **No es un agente de IA.** No reemplaza a Claude Code, OpenCode, Cursor, etc. Los configura.
- **No es un instalador de modelos.** No descarga modelos ni gestiona tus claves de API: eso lo hace el agente.
- **No es un IDE.** Seguís usando tu editor (VS Code, Cursor, terminal, lo que sea).
- **No es un wrapper/proxy.** El agente sigue hablando directo con su proveedor (Anthropic, OpenAI, OpenRouter, etc.).
- **No te obliga a usar SDD.** Si la tarea es chica, el agente la hace y listo. SDD aparece solo cuando hace falta.
- **No reemplaza a [Agent Teams Lite (ATL)](https://github.com/Gentleman-Programming/agent-teams-lite)** — lo **sucede**. Todo lo que daba ATL está acá, mejor instalado, con auto-update y con memoria persistente.

---

## La regla de oro

> **Cuanto menos tengas que pensar en Gentle-AI después de instalarlo, mejor está funcionando.**

Lo corrés una vez, elegís tu(s) agente(s) y tu preset, y a partir de ahí vos seguís trabajando como siempre. El agente, por debajo, ya tiene:

- memoria que persiste,
- skills que se cargan solas,
- un flujo SDD que se activa cuando la tarea lo amerita,
- una persona que te explica los porqués,
- delegación a sub-agentes cuando la cosa se complica.

No hay comandos nuevos para memorizar. Hay comandos disponibles **por si los querés**, pero el día a día es: **abrir el agente y trabajar**.

---

## ¿A quién está dirigido?

Gentle-AI tiene sentido si:

- Usás (o querés usar) un agente de IA **para tareas de programación reales**, no solo para preguntas sueltas.
- Querés que tu agente **mantenga contexto** entre días/semanas en un mismo proyecto.
- Trabajás en **más de un proyecto** y te cansa explicarle cada uno desde cero.
- Te interesa un **flujo serio** (planificar antes de codear, revisar antes de commitear) sin tener que diseñarlo vos.
- Estás **aprendiendo** y querés un agente que te **enseñe** mientras te ayuda.

Si solo querés autocompletar más rápido, alcanza con el agente pelado. Si querés un **compañero de trabajo persistente**, ahí entra Gentle-AI.

---

## ¿Y técnicamente, qué es?

Para quien le interese el detalle: Gentle-AI es un **CLI escrito en Go** (binario único, multiplataforma) que:

1. **Detecta** qué agente(s) tenés y dónde guardan su configuración.
2. **Inyecta** archivos de configuración, skills, prompts de persona y servidores MCP en las rutas correctas de cada agente (sin pisar lo que ya tenías: usa *merge* con marcadores).
3. **Snapshotea** tus configs antes de tocarlas (backups comprimidos, deduplicados, auto-pruneados).
4. **Mantiene** todo actualizado con `gentle-ai update`.

Vive en `~/.atl/`, `~/.engram/` y en las carpetas de configuración de cada agente (`~/.claude/`, `~/.config/opencode/`, `~/.cursor/`, etc.). Si algo te asusta, podés volver atrás con un *rollback* desde la TUI.

No hace falta saber Go ni los detalles internos para usarlo. Esto está acá solo para que sepas que **no hay magia oculta**: todos los archivos que toca quedan visibles en tu disco.

---

## Resumen

- Tu agente es el **motor**. Gentle-AI es la **configuración** que lo vuelve usable a largo plazo.
- Te suma **memoria, skills, SDD, MCP, persona y delegación** sin que tengas que armarlo vos.
- No reemplaza al agente, no toca tus modelos, no te obliga a aprender nuevos comandos.
- Lo instalás una vez y, idealmente, **te olvidás de él**.

---

## Siguiente paso

➡️ **[2 · Glosario rápido](02-glosario.md)** — definiciones cortas de los términos que vas a ver una y otra vez: *agente*, *sub-agente*, *skill*, *preset*, *componente*, *MCP*, *SDD*, *Engram*, *persona*, *orquestador*, *delegación*.

---

← [Volver al índice](index.md)
