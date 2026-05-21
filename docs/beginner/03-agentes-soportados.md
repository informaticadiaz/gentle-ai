# 3 · Agentes soportados

← [Volver al índice](index.md) · ← [Anterior: Glosario rápido](02-glosario.md)

---

Gentle-AI configura **14 agentes diferentes**. La buena noticia: **no tenés que probarlos todos**. Esta página te explica qué es cada uno, en qué se diferencian, y cuál te conviene según tu situación.

> Si ya tenés uno instalado y te gusta, **quedate con ese**. Gentle-AI lo va a configurar igual de bien que cualquier otro. Esta guía es para quien todavía está eligiendo, o para quien quiere entender por qué algunos agentes se "sienten" más capaces que otros con SDD.

---

## La idea grande: dos modelos de delegación

Antes de mirar nombres, entendé esta distinción. Es la única que importa de verdad:

### 1) Full delegation (con sub-agentes)

El agente puede **lanzar sub-agentes** con su propio contexto y herramientas. Cuando llega una tarea grande:

- El orquestador decide qué sub-agente lanzar (`sdd-explore`, `sdd-design`, `sdd-apply`, …).
- Cada sub-agente trabaja **aislado**, con su propio contexto limpio.
- Vuelve con un resultado y el orquestador sigue.

Resultado: **menos contexto sucio**, mejor calidad en features grandes, posibilidad de **paralelizar**.

Agentes "full delegation": **Claude Code, OpenCode, Kilo Code, Gemini CLI, Cursor, VS Code Copilot, Kimi Code, Kiro IDE, Qwen Code, Pi**.

### 2) Solo-agent

El agente **no puede crear sub-agentes**. Todo el flujo SDD corre **en la misma conversación**, con el mismo modelo y el mismo contexto. Engram sigue funcionando, así que la memoria persiste entre sesiones, pero **dentro** de una sesión no hay aislamiento.

Agentes "solo-agent": **Codex, Windsurf, Antigravity, OpenClaw**.

> ¿Es peor solo-agent? **No necesariamente.** Para tareas chicas y medianas, anda igual. La diferencia se nota cuando una feature requiere muchas fases o mucha exploración: ahí "full delegation" gana por aislamiento de contexto.

---

## Cuál elegir si recién empezás

Un mini árbol de decisión, sin trampa:

```
¿Tenés cuenta de Anthropic (Claude.ai paga o API)?
└── Sí → Claude Code  ← recomendación #1 para principiantes
└── No
    ├── ¿Querés modelos gratis o muy baratos vía OpenRouter?
    │   └── Sí → OpenCode  ← recomendación #2, mejor multi-modelo
    └── ¿Ya usás Cursor como editor?
        └── Sí → Cursor  ← recomendación #3, se integra con tu IDE
```

Las tres opciones son **full delegation**, así que aprovechás todo lo que Gentle-AI inyecta. Cualquiera de las tres es una elección sólida.

---

## Fichas rápidas (los 14 agentes)

Cada ficha trae: **qué es**, **delegación**, **dónde guarda su config**, y **cuándo te conviene**.

---

### Claude Code · `claude-code`

- **Qué es**: el CLI oficial de Anthropic para programar con Claude.
- **Delegación**: full (vía la *Task tool* nativa).
- **Config**: `~/.claude/`
- **Distintivo**: sub-agentes nativos, *output styles*, prompt del sistema en `CLAUDE.md`.
- **Conviene si**: ya tenés acceso a la API de Anthropic o suscripción a Claude. Es el camino más directo y, en general, **el más recomendado para empezar**.

### OpenCode · `opencode`

- **Qué es**: TUI open-source agnóstica de proveedor (Anthropic, OpenAI, OpenRouter, etc.).
- **Delegación**: full (overlay multi-agente con 11 agentes en `opencode.json`).
- **Config**: `~/.config/opencode/`
- **Distintivo**: **único agente con multi-mode real** — podés asignar **un modelo distinto por fase SDD**. Ideal para usar modelos baratos/gratis en exploración y caros solo en diseño.
- **Conviene si**: querés flexibilidad de proveedores, OpenRouter, o ahorrar plata mezclando modelos.

### Kilo Code · `kilocode`

- **Qué es**: fork/derivado compatible con OpenCode, también TUI.
- **Delegación**: full (mismo overlay multi-mode que OpenCode).
- **Config**: `~/.config/kilo/`
- **Distintivo**: comparte la lógica de OpenCode pero con su propio binario; instalable vía `npm install -g @kilocode/cli`.
- **Conviene si**: ya usás Kilo o querés la misma capacidad multi-modelo que OpenCode con otro binario.

### Gemini CLI · `gemini-cli`

- **Qué es**: CLI oficial de Google para Gemini.
- **Delegación**: full, pero **experimental** (requiere `experimental.enableAgents: true`).
- **Config**: `~/.gemini/`
- **Conviene si**: trabajás con Google Cloud / Vertex y querés mantener todo dentro del ecosistema Gemini.

### Cursor · `cursor`

- **Qué es**: editor (fork de VS Code) con IA integrada.
- **Delegación**: full (sub-agentes nativos en `~/.cursor/agents/`).
- **Config**: `~/.cursor/`
- **Distintivo**: 10 agentes SDD instalados como archivos `sdd-{fase}.md`. Cursor auto-delegar según el `description` en el frontmatter.
- **Conviene si**: querés un **IDE completo con IA** (no solo terminal) y ya estás cómodo con VS Code.

### VS Code Copilot · `vscode-copilot`

- **Qué es**: GitHub Copilot dentro de VS Code, con la herramienta `runSubagent`.
- **Delegación**: full, con soporte para **ejecución paralela**.
- **Config**: `~/.copilot/` + perfil de VS Code.
- **Conviene si**: ya pagás Copilot Business/Enterprise y usás VS Code como editor principal.

### Codex · `codex`

- **Qué es**: agente CLI de OpenAI, configurado con TOML.
- **Delegación**: **solo-agent**.
- **Config**: `~/.codex/`
- **Conviene si**: usás GPT como modelo principal y preferís CLI minimalista. Recordá que SDD va a correr **inline** (mismo contexto), así que Engram es clave acá.

### Windsurf · `windsurf`

- **Qué es**: editor de Codeium con IA y modos nativos.
- **Delegación**: solo-agent.
- **Config**: `~/.codeium/windsurf/`
- **Distintivo**: aprovecha *Plan Mode*, *Code Mode* y *Workflows* nativos. El orquestador clasifica tareas en Small / Medium / Large.
- **Conviene si**: ya usás Windsurf y querés sumar SDD + Engram sin cambiar de editor.

### Antigravity · `antigravity`

- **Qué es**: IDE agent-first de Google (basado en Gemini) con **Mission Control** y sub-agentes built-in (Browser, Terminal).
- **Delegación**: solo-agent + Mission Control (sin sub-agentes custom todavía).
- **Config**: `~/.gemini/antigravity/`
- **Conviene si**: ya estás probando Antigravity como IDE experimental. **Ojo**: comparte `~/.gemini/GEMINI.md` con Gemini CLI — Gentle-AI avisa si tenés ambos.

### Kimi Code · `kimi`

- **Qué es**: CLI con agentes custom de Moonshot AI (modelos Kimi).
- **Delegación**: full (custom agents nativos).
- **Config**: `~/.kimi/`
- **Distintivo**: arquitectura de prompts **modular** (`persona.md`, `output-style.md`, `engram-protocol.md`, `sdd-orchestrator.md`). Requiere `uv` para instalar (`uv tool install kimi-cli`).
- **Conviene si**: querés probar modelos Kimi o te gusta la idea de prompts modulares.

### Kiro IDE · `kiro-ide`

- **Qué es**: IDE con flujo de specs nativo (`requirements.md`, `design.md`, `tasks.md`) y gates de aprobación.
- **Delegación**: full (sub-agentes nativos).
- **Config**: `~/.kiro/` + MCP en `~/.kiro/settings/mcp.json`
- **Distintivo**: **soporta multi-modelo** vía `model:` en el frontmatter de cada fase (Opus/Sonnet/Haiku). Instalación manual desde [kiro.dev/downloads](https://kiro.dev/downloads).
- **Conviene si**: te gusta tener un IDE con **specs como ciudadano de primera clase**.
- **Más info**: [docs/kiro.md](../kiro.md)

### Qwen Code · `qwen-code`

- **Qué es**: CLI de Alibaba para modelos Qwen, con slash commands.
- **Delegación**: full (sub-agents nativos).
- **Config**: `~/.qwen/`
- **Distintivo**: slash commands con **namespaces** (`commands/sdd/init.md` → `/sdd:init`). Modo `auto_edit` (auto-aprueba ediciones, pide confirmación para shell). Instalable vía `npm install -g @qwen-code/qwen-code@latest`.
- **Conviene si**: usás modelos Qwen o querés un agente con slash commands organizados por namespace.

### OpenClaw · `openclaw`

- **Qué es**: agente con foco en **workspace** y configuración global de MCP.
- **Delegación**: solo-agent.
- **Config**: `~/.openclaw/` (global) + `<workspace>/.openclaw/` (por proyecto).
- **Distintivo**: instrucciones por workspace en `AGENTS.md` y `SOUL.md`. MCP global compartido entre todos los workspaces.
- **Conviene si**: trabajás con muchos repos y querés instrucciones específicas por workspace, con MCP centralizado.

### Pi · `pi`

- **Qué es**: harness extensible vía paquetes npm. Gentle-AI no le inyecta prompts directamente: **instala paquetes Pi** que cuidan todo.
- **Delegación**: full (sub-agents administrados por paquetes Pi).
- **Config**: `~/.pi/` + assets de proyecto en `.pi/agents/`, `.pi/chains/`
- **Distintivo**: instala un stack completo (`gentle-pi`, `gentle-engram`, `pi-mcp-adapter`, `pi-subagents`, `pi-intercom`, `pi-web-access`, `pi-lens`, `pi-btw`, etc.). Comandos `/gentleman:persona` y `/gentleman:models` propios.
- **Conviene si**: querés el flujo Gentle-AI más "todo-incluido" con manejo de paquetes prolijo.
- **Más info**: [docs/pi.md](../pi.md)

---

## Tabla comparativa

| Agente | Tipo | Delegación | Multi-modelo | Slash commands | Curva de entrada |
| --- | --- | --- | --- | --- | --- |
| **Claude Code** | CLI | Full | — | — | 🟢 Baja |
| **OpenCode** | TUI | Full | ✅ | ✅ | 🟡 Media |
| **Kilo Code** | TUI | Full | ✅ | ✅ | 🟡 Media |
| **Cursor** | IDE | Full | — | — | 🟢 Baja |
| **VS Code Copilot** | IDE | Full | — | — | 🟢 Baja |
| **Kiro IDE** | IDE | Full | ✅ | — | 🟡 Media |
| **Gemini CLI** | CLI | Full (exp.) | — | — | 🟡 Media |
| **Kimi Code** | CLI | Full | — | — | 🟡 Media |
| **Qwen Code** | CLI | Full | — | ✅ | 🟡 Media |
| **Pi** | CLI | Full | ✅\* | ✅ | 🟠 Alta |
| **Codex** | CLI | Solo | — | — | 🟢 Baja |
| **Windsurf** | IDE | Solo | — | — | 🟢 Baja |
| **Antigravity** | IDE | Solo + MC | — | — | 🟡 Media |
| **OpenClaw** | CLI | Solo | — | — | 🟡 Media |

\* Pi multi-modelo lo maneja `gentle-pi` vía `/gentleman:models`.

---

## Más de un agente al mismo tiempo

**Sí, se puede.** Gentle-AI te deja seleccionar varios agentes en una sola corrida y configura todos. Cada uno guarda su config en su propia carpeta, así que **no se pisan**.

Casos típicos:

- **Claude Code + OpenCode** → CLI principal + opción multi-modelo cuando hace falta.
- **Cursor + Claude Code** → IDE para edición visual + CLI para tareas pesadas en terminal.
- **Cualquiera + Pi** → tu agente habitual + Pi como sandbox de paquetes.

Engram comparte memoria **entre agentes** porque vive afuera de cada uno (`~/.engram/`). Es decir: una decisión guardada desde Claude Code aparece cuando consultás desde OpenCode.

---

## Resumen

- **Si dudás, arrancá con Claude Code.** Es el camino más corto y todo el ecosistema lo soporta.
- **Si querés multi-modelo o proveedores múltiples**, OpenCode (o Kilo).
- **Si trabajás en un IDE**, Cursor o VS Code Copilot.
- **Solo-agent ≠ peor**: Codex y Windsurf andan muy bien para tareas chicas/medianas.
- Podés instalar **varios agentes** sin conflictos; Engram comparte memoria entre ellos.

---

## Siguiente paso

➡️ **[4 · Instalación paso a paso](04-instalacion.md)** — instalar Gentle-AI en macOS, Linux y Windows. Verificar que quedó bien. Qué hacer si algo falla. Cómo desinstalar.

📚 Doc de referencia (avanzado): **[docs/agents.md](../agents.md)** — matriz completa con rutas de config, comportamiento de MCP por agente, y notas detalladas.

---

← [Volver al índice](index.md) · ← [Anterior: Glosario rápido](02-glosario.md)
