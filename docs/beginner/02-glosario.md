# 2 · Glosario rápido

← [Volver al índice](index.md) · ← [Anterior: ¿Qué es Gentle-AI?](01-que-es-gentle-ai.md)

---

Esta página es un **diccionario de bolsillo**. No hace falta leerla entera de un saque; volvé cuando te crucces con un término que no conocés.

Las definiciones están escritas para que las entiendas la primera vez, sin asumir que ya sabés cómo funcionan los agentes.

---

## Términos centrales

### Agente

El programa que **conversa con vos y escribe código por vos**. Por ejemplo: Claude Code, OpenCode, Cursor, Gemini CLI, Codex, Windsurf, Kiro. Vive en tu terminal o en tu editor y, por debajo, le habla a un modelo (Claude, GPT, Gemini, Qwen, etc.).

Gentle-AI no es un agente. **Configura** agentes.

### Modelo

La IA en sí (Claude Opus, Claude Sonnet, GPT-5, Gemini, etc.). Es lo que "piensa". El **agente** es la app; el **modelo** es el cerebro que la app consulta. Un mismo agente puede usar varios modelos.

### Orquestador

El hilo principal del agente: el que **lee tu pedido y decide qué hacer**. Si la tarea es chica, la hace él mismo. Si es grande, **delega** en sub-agentes especializados. La regla de Gentle-AI es **mantener el orquestador liviano** y no dejar que se vuelva un monstruo monolítico.

### Sub-agente

Un agente "hijo" con **su propia sesión, contexto y herramientas**. El orquestador lo lanza para una tarea concreta (por ejemplo, explorar el código, aplicar cambios, revisar un diff) y recibe el resultado. No es un script tonto: es un agente completo enfocado en una sola cosa.

### Delegación

El acto de **pasarle trabajo a un sub-agente** en vez de hacerlo todo en el mismo hilo. Gentle-AI tiene reglas claras sobre cuándo delegar:

- Leer **4+ archivos** para entender un flujo.
- Tocar **2+ archivos** no triviales.
- Antes de un **commit / push / PR** no trivial.
- Después de un **accidente** (cwd equivocado, problema de git, conflicto raro).
- Cuando la sesión se **alargó** y se está acumulando complejidad.

Delegar = menos errores y menos contexto sucio.

---

## Capas que instala Gentle-AI

### Componente

Una **pieza opcional** que Gentle-AI puede instalar y configurar en tu agente. Cada componente se identifica por un `id` corto:

| Componente | ID | Qué hace |
| --- | --- | --- |
| Engram | `engram` | Memoria persistente entre sesiones. |
| SDD | `sdd` | Flujo de Spec-Driven Development. |
| Skills | `skills` | Biblioteca de skills curadas. |
| Context7 | `context7` | MCP de documentación de frameworks. |
| Persona | `persona` | Persona "Gentleman" o neutral inyectada en el agente. |
| Permissions | `permissions` | Defaults de seguridad y guardrails. |
| GGA | `gga` | Gentleman Guardian Angel: switcher de proveedor de IA. |
| Theme | `theme` | Tema "Gentleman Kanagawa". |

### Preset

Un **paquete predefinido de componentes**. En lugar de elegir uno por uno, elegís un preset y listo:

| Preset | ID | Qué trae |
| --- | --- | --- |
| Full Gentleman | `full-gentleman` | **Todo**: todos los componentes + todas las skills + persona Gentleman. |
| Ecosystem Only | `ecosystem-only` | Núcleo: Engram + SDD + Skills + Context7 + GGA + skills + persona Gentleman. |
| Minimal | `minimal` | Lo más chico: solo Engram + SDD skills. |
| Custom | `custom` | Vos elegís componentes y skills a mano (no toca tu persona/settings actuales). |

> Si dudás, **`ecosystem-only`** es un buen punto medio para arrancar. Después podés re-correr el installer y cambiar el preset cuando quieras.

### Skill

Un **archivo de instrucciones reusable** (`SKILL.md`) que le dice al agente *cómo* hacer algo concreto: crear un PR, escribir un commit, redactar un comentario de review, diseñar docs con baja carga cognitiva, etc.

Las skills no son prompts sueltos: tienen **triggers** (cuándo aplicar), **scope** (a qué proyectos), y un cuerpo con la guía. El agente las **descubre solo** desde el *skill registry* y las **carga cuando hacen falta**.

Gentle-AI trae skills de **SDD** (`sdd-init`, `sdd-explore`, `sdd-propose`, …) y skills de **foundation** (`branch-pr`, `chained-pr`, `comment-writer`, `issue-creation`, `work-unit-commits`, `cognitive-doc-design`, `skill-registry`, `skill-creator`, etc.). Las skills de framework (React, Angular, Tailwind, Playwright…) viven en otro repo y se instalan aparte.

### Skill registry

El **índice de skills** disponibles en tu máquina + las convenciones específicas del proyecto (`CLAUDE.md`, `AGENTS.md`, `.cursorrules`…). El orquestador lo lee al arranque de sesión y le pasa a cada sub-agente la skill exacta que necesita, en vez de un resumen genérico.

Se refresca solo en los agentes que soportan hooks (Claude Code, OpenCode, Pi). En el resto, lo refrescás con `gentle-ai skill-registry refresh`.

### Engram

La **memoria persistente** del agente. Guarda decisiones, bugs resueltos, descubrimientos del código y contexto, y lo deja disponible **entre sesiones, días y máquinas** (vía `engram sync` con `.engram/` en git).

Tu agente guarda y consulta Engram **solo**, vía MCP. Vos casi nunca interactuás con él, salvo cuando querés:

- ver lo guardado (`engram tui`, `engram search`),
- exportarlo (`engram sync`) o importarlo en otra máquina (`engram sync --import`),
- consolidar nombres de proyecto (`engram projects consolidate`).

### SDD (Spec-Driven Development)

Un **flujo de trabajo estructurado** para features que no son triviales. Tiene fases:

1. **Explore** — entender el código antes de tocarlo.
2. **Propose** — proponer un enfoque (intención, alcance, approach).
3. **Spec** — requirements y escenarios.
4. **Design** — decisiones técnicas y arquitectura.
5. **Tasks** — descomponer en tareas concretas.
6. **Apply** — implementar siguiendo spec y diseño.
7. **Verify** — validar que la implementación cumple la spec.
8. **Archive** — sincronizar deltas con las specs principales.
9. **Onboard** — walkthrough guiado del flujo completo.

**No tenés que memorizar nada.** El agente decide si activa SDD según el tamaño de la tarea, o vos lo pedís diciendo "use sdd" / "hazlo con sdd".

### Persona

La **personalidad y estilo** del agente: cómo te habla, qué tan didáctico es, qué advertencias da. Gentle-AI inyecta una persona "teaching-first" (te explica los porqués) con **permisos `security-first`** (no toca cosas peligrosas sin pedir confirmación). También hay opción neutral, o "custom" si querés mantener la tuya.

### MCP (Model Context Protocol)

El **estándar** por el cual un agente puede hablar con **herramientas externas**: un servidor de archivos, GitHub, un buscador, una DB, Engram… El agente "carga" un servidor MCP y a partir de ahí puede invocar sus herramientas como si fueran propias.

Gentle-AI configura MCPs útiles por vos (Engram, Context7, etc.) en cada agente que los soporte. No tenés que editar JSONs a mano.

### GGA (Gentleman Guardian Angel)

Un **switcher de proveedores de IA** que se instala como binario aparte (`gga`). Sirve para cambiar entre Anthropic, OpenAI, OpenRouter, etc. **a nivel de proyecto**, sin tocar las claves globales. Se activa por repo con `gga init` y `gga install`.

---

## Términos de OpenCode

Estos solo aplican si usás **OpenCode** como agente principal.

### Perfil SDD

Una **configuración de modelos por fase de SDD**. Por ejemplo: usar `claude-sonnet` para `sdd-design` y `qwen3-30b` (gratis vía OpenRouter) para `sdd-explore`. Permite **ahorrar plata** y elegir el modelo más adecuado por tipo de trabajo.

Los perfiles se crean desde la TUI o con `gentle-ai sync --profile nombre:provider/modelo`.

### `gentle-orchestrator`

El **orquestador SDD por defecto** en OpenCode. Es el "agente padre" que vas a ver si abrís OpenCode y presionás Tab. Cada perfil que crees aparece como `sdd-orchestrator-{nombre}` y podés alternar entre ellos también con Tab.

---

## Términos del CLI y la app

### TUI

**Text User Interface**: la interfaz visual que se ve en la terminal. Gentle-AI tiene una TUI (con tema Rose Pine) para configurar todo sin tener que recordar flags. La abrís corriendo `gentle-ai` sin argumentos.

### Hook (de startup)

Un **script que el agente ejecuta solo** al iniciar una sesión. Gentle-AI instala un hook que refresca el skill registry, así no tenés que correrlo a mano. Si tu agente no soporta hooks, lo corrés vos cuando haga falta.

### `/sdd-init`

Un **comando que le pedís al agente** dentro del proyecto para que detecte stack, framework de tests, convenciones, etc. Es lo primero que conviene correr en un repo nuevo. Si no lo corrés vos, el orquestador SDD lo dispara solo cuando lo necesita.

### Backup

Un **snapshot comprimido** (`tar.gz`) de tus archivos de configuración. Gentle-AI los crea **antes** de cada install, sync o upgrade. Quedan **deduplicados** (no se repiten si no cambió nada) y **auto-pruneados** (se guardan los 5 más recientes). Podés "pinear" uno desde la TUI para protegerlo del prune.

### Rollback

**Volver atrás** a un backup anterior. Útil si una actualización te rompió algo. Se hace desde la TUI.

---

## Cuándo usar cada cosa (resumen ultra corto)

| Querés… | Mirá… |
| --- | --- |
| Instalar / cambiar agentes o componentes | **TUI** (`gentle-ai`) o flags del **CLI** |
| Que el agente recuerde decisiones | **Engram** (automático) |
| Planificar una feature seria | **SDD** (le decís "use sdd") |
| Que el agente use una habilidad puntual bien | **Skills** (se cargan solas) |
| Conectar al agente con GitHub / docs / archivos | **MCP** (Gentle-AI lo configura) |
| Cambiar de proveedor de IA por proyecto | **GGA** (`gga init` + `gga install`) |
| Volver atrás un cambio de config | **Backup + Rollback** desde la TUI |
| Asignar modelos distintos por fase (OpenCode) | **Perfiles SDD** |

---

## Siguiente paso

➡️ **[3 · Agentes soportados](03-agentes-soportados.md)** — un recorrido por los 13+ agentes que Gentle-AI configura, cómo se diferencian entre "full delegation" y "solo-agent", y cuál te conviene según tu situación.

---

← [Volver al índice](index.md) · ← [Anterior: ¿Qué es Gentle-AI?](01-que-es-gentle-ai.md)
