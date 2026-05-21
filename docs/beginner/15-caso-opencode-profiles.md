# 15 · Caso: OpenCode con múltiples modelos

← [Volver al índice](index.md) · ← [Anterior: Caso: sumarse a un repo existente](14-caso-repo-existente.md)

---

> **Objetivo del caso**: usar **modelos diferentes por fase SDD** para ahorrar plata y/o mejorar la calidad. Por ejemplo: un modelo barato (o gratis) para explorar, uno potente para diseñar, uno rápido para implementar.
>
> **Aplica a**: **OpenCode** (full support) y **Kilo Code** (mismo overlay). Si usás otro agente, esta página es **opcional** — saltala. Kiro IDE tiene una variante propia que mencionamos al final.
>
> **Tiempo estimado**: 15-25 minutos.

---

## ¿Por qué querría esto?

Cada fase SDD (página 8) hace algo distinto:

- **`sdd-explore`** lee mucho código y devuelve un resumen.
- **`sdd-propose`** sintetiza opciones de approach.
- **`sdd-design`** toma decisiones técnicas.
- **`sdd-apply`** escribe código tarea por tarea.
- **`sdd-verify`** corre tests y valida criterios.

No todas necesitan el modelo más caro. **Asignar el modelo correcto por fase** te da:

| Beneficio | Cómo |
| --- | --- |
| **Ahorrar plata** | Usar modelos baratos/gratis en exploración y verificación. |
| **Ganar velocidad** | Usar modelos rápidos en tareas mecánicas (apply). |
| **Subir calidad** | Reservar el modelo top para diseño y review. |
| **Experimentar** | Probar modelos nuevos en fases acotadas sin riesgo. |

Combinación típica (la "cheap" recomendada):

| Fase | Modelo | Razón |
| --- | --- | --- |
| `sdd-explore` | `openrouter/qwen/qwen3-30b-a3b:free` | Lectura barata o gratis. |
| `sdd-propose` | `anthropic/claude-sonnet-4` | Necesita criterio. |
| `sdd-design` | `anthropic/claude-sonnet-4` | Decisiones técnicas. |
| `sdd-apply` | `anthropic/claude-haiku-3.5` | Rápido y barato para código mecánico. |
| `sdd-verify` | `anthropic/claude-sonnet-4` | Vale el modelo bueno en review. |

---

## Antes de empezar: los nombres clave

Cuando manejes perfiles vas a ver estos términos en `opencode.json` y en logs. Apuntalos:

| Nombre | Qué es |
| --- | --- |
| `gentle-orchestrator` | El **conductor SDD base** de OpenCode. Existe siempre. Todos los `/sdd-*` apuntan acá por defecto. |
| `sdd-orchestrator` | Nombre **legacy**. Si tu config viene de antes, sync lo migra a `gentle-orchestrator`. |
| `sdd-orchestrator-{nombre}` | Tu **perfil con nombre**. Ejemplo: `sdd-orchestrator-cheap`. |
| `sdd-{fase}` | Sub-agente por defecto de una fase. Ejemplo: `sdd-apply`. |
| `sdd-{fase}-{nombre}` | Sub-agente de un perfil con nombre. Ejemplo: `sdd-apply-cheap`. |

> **Regla**: estos nombres los maneja Gentle-AI. **No los edites a mano** en `opencode.json`. Usá la TUI o el CLI.

---

## Dos estrategias (vas a usar la primera)

Gentle-AI soporta dos formas de manejar perfiles:

### A) `generated-multi` (la que vas a usar)

Cada perfil con nombre que crees genera **11 entradas** en `opencode.json`: un orquestador + los 10 sub-agentes de fase, con el modelo que vos asignaste.

Vos cambiás entre perfiles **con `Tab`** dentro de OpenCode.

### B) `external-single-active` (para usuarios avanzados)

Si tenés una herramienta de la comunidad que guarda perfiles fuera de `opencode.json` (en `~/.config/opencode/profiles/*.json`) y los activa en runtime, Gentle-AI se pone en modo compatible: **no** regenera perfiles automáticos y **preserva** tu prompt actual de `gentle-orchestrator`.

> Si no entendiste el párrafo anterior, **no necesitás esta estrategia**. Quedate con la A.

Gentle-AI detecta automáticamente si tenés `profiles/*.json` y cambia de estrategia. También podés forzar:

```bash
gentle-ai sync --agent opencode --sdd-profile-strategy generated-multi
# o
gentle-ai sync --agent opencode --sdd-profile-strategy external-single-active
```

---

## Pre-requisitos

1. **OpenCode instalado** y configurado con Gentle-AI (página 5).
2. **Tus proveedores conectados** en OpenCode (Anthropic, OpenRouter, etc.). Para verlo:
   ```bash
   opencode auth list
   ```
3. **Refrescar lista de modelos** después de conectar nuevos proveedores:
   ```bash
   opencode models --refresh
   ```

Sin proveedores conectados, no vas a tener modelos para asignar.

---

## Camino A · Crear un perfil con la TUI

Este es el método recomendado para empezar. Te guía paso a paso.

### 1) Lanzá la TUI

```bash
gentle-ai
```

### 2) Elegí "OpenCode SDD Profiles"

En el menú principal aparece la opción (página 5). Si **no aparece**, es porque no tenés OpenCode instalado/detectado.

### 3) Creá un perfil nuevo

Apretá `n` o seleccioná "Create new profile".

### 4) Nombre del perfil

Usá **slug**: minúsculas, guiones permitidos. Ejemplos válidos:

- `cheap`
- `premium`
- `premium-v2`

Inválidos:

- `my profile` (espacios no)
- `default` (reservado)
- `LOUD` (se autoconvierte a `loud`)

### 5) Elegí modelo del orquestador

Pantalla de **provider → model**. Vas a ver listados los providers que conectaste. Elegí uno.

> **Tip**: para modelos con niveles de **reasoning effort** (como `gpt-5` con `low/medium/high/xhigh`), aparece un paso adicional **"Select reasoning effort level"**. Si no querés pensarlo, `default`.

### 6) Asigná modelos por fase

Tenés dos opciones:

- **"Set all phases"** — el mismo modelo para las 10 fases. Útil para "todo Haiku" o "todo Opus".
- **Setear cada fase individualmente** — para la combinación mixta de arriba.

### 7) Confirmar

Gentle-AI escribe el perfil en `opencode.json` y corre `sync` automáticamente.

---

## Camino B · Crear un perfil con el CLI

Para automatizar o si te gusta el terminal.

### Un perfil simple (mismo modelo para todas las fases)

```bash
gentle-ai sync --profile cheap:anthropic/claude-haiku-3.5-20241022
```

Crea `sdd-orchestrator-cheap` + 10 sub-agentes, todos con Haiku.

### Varios perfiles a la vez

```bash
gentle-ai sync \
  --profile cheap:anthropic/claude-haiku-3.5-20241022 \
  --profile premium:anthropic/claude-opus-4-20250514
```

Crea **dos** perfiles que después switcheás con Tab.

### Un perfil con una fase pisada

```bash
gentle-ai sync \
  --profile cheap:anthropic/claude-haiku-3.5-20241022 \
  --profile-phase cheap:sdd-apply:anthropic/claude-sonnet-4-20250514
```

Esto crea un perfil "cheap" con **Haiku en todo, salvo `sdd-apply` que usa Sonnet**.

### Una "cheap mixta" típica (con OpenRouter para exploración)

```bash
gentle-ai sync \
  --profile cheap:anthropic/claude-sonnet-4-20250514 \
  --profile-phase cheap:sdd-explore:openrouter/qwen/qwen3-30b-a3b:free \
  --profile-phase cheap:sdd-apply:anthropic/claude-haiku-3.5-20241022
```

---

## Usar perfiles dentro de OpenCode

Después de crear los perfiles, abrí OpenCode normal:

```bash
opencode
```

Apretá `Tab` y vas a ver tus orquestadores:

| En Tab | Qué corre |
| --- | --- |
| `gentle-orchestrator` | Tu config base (el default). |
| `sdd-orchestrator-cheap` | Tu perfil "cheap" — Haiku en todas las fases (o lo que hayas asignado). |
| `sdd-orchestrator-premium` | Tu perfil "premium" — Opus en todo. |

Switcheás libremente. Cada `/sdd-*` corre contra el orquestador **actualmente seleccionado**.

> **Aislamiento**: el orquestador `sdd-orchestrator-cheap` solo puede delegar a sus **propios sub-agentes** (`sdd-apply-cheap`, `sdd-design-cheap`, etc.). Los perfiles **no se cruzan** entre sí.

---

## Combinaciones útiles

### "Exploración gratis, diseño bueno, implementación rápida"

```bash
gentle-ai sync \
  --profile balanced:anthropic/claude-sonnet-4-20250514 \
  --profile-phase balanced:sdd-explore:openrouter/qwen/qwen3-30b-a3b:free \
  --profile-phase balanced:sdd-init:openrouter/qwen/qwen3-30b-a3b:free \
  --profile-phase balanced:sdd-apply:anthropic/claude-haiku-3.5-20241022
```

### "Premium total para incidentes / refactors críticos"

```bash
gentle-ai sync --profile premium:anthropic/claude-opus-4-20250514
```

### "Experimentación: probar gpt-5 solo en design"

```bash
gentle-ai sync \
  --profile experimental:anthropic/claude-sonnet-4-20250514 \
  --profile-phase experimental:sdd-design:openai/gpt-5-2025-08-07
```

> Si gpt-5 expone reasoning effort, durante la TUI vas a poder elegir `high` o `xhigh` para esa fase.

---

## Editar y borrar perfiles

Desde la TUI (`OpenCode SDD Profiles` → lista de perfiles):

| Acción | Tecla | Notas |
| --- | --- | --- |
| Editar | `Enter` | Cambiás modelos y sync corre solo. |
| Borrar | `d` | Quita el orquestador + sub-agentes del JSON. |
| Crear | `n` | Flujo de creación completo. |

> **`gentle-orchestrator` (default) no se puede borrar**. Solo editar. Es la fundación del SDD.

---

## ¿Cuánto se ahorra? (sentido común, no marketing)

Cuesta poco poner una "cheap" para tareas exploratorias o triviales. Ejemplo grueso (ordenes de magnitud, los precios cambian):

- 1 sesión SDD chica con todo Sonnet: **~$0.20-0.50**.
- Misma sesión con `explore` y `init` en Qwen3-free + resto en Haiku: **~$0.02-0.10**.

Multiplicado por decenas de sesiones por semana, los ahorros son reales. **Pero**: si la calidad cae demasiado en una fase, **subí el modelo de esa fase**. La idea no es ser tacaño, es **asignar el modelo que cada paso amerita**.

---

## Kiro IDE: una variante similar (multi-mode nativo)

Si usás **Kiro IDE** en vez de OpenCode, también tenés multi-modelo, pero implementado distinto:

- Asignás modelos por fase desde la TUI de Gentle-AI: **"Configure Models → Configure Kiro models"**.
- El alias (`opus|sonnet|haiku`) se resuelve a un model ID nativo de Kiro y se estampa en cada `~/.kiro/agents/sdd-{phase}.md` durante sync.
- No hay perfiles con nombre. Es una asignación única, no varias configurables con Tab.

---

## Errores comunes

### "Mi modelo custom de OpenCode no aparece en el picker"

Para que un modelo custom sea seleccionable como sub-agente SDD, en tu `opencode.json` tiene que tener:

```json
"tool_call": true
```

Sin eso, el picker lo descarta porque no puede usar tools (y SDD necesita tools). Editá tu provider config y volvé a `gentle-ai sync`.

### "Creé el perfil pero `Tab` no me lo muestra"

Verificá que el sync corrió:

```bash
gentle-ai sync --agent opencode
```

Cerrá y reabrí OpenCode. Los nuevos orquestadores aparecen al iniciar la sesión.

### "Quiero usar un perfil pero `/sdd-new` sigue corriendo con `gentle-orchestrator`"

`/sdd-*` corre contra el orquestador **actualmente seleccionado**. Apretá `Tab` antes para cambiar al perfil que querés, después corré el slash command.

### "Mi config tiene `sdd-orchestrator` (sin `gentle-`) y se ve viejo"

Es config legacy. Corré:

```bash
gentle-ai sync --agent opencode
```

Y la migración a `gentle-orchestrator` se hace sola. Tus perfiles con nombre no se tocan.

### "Tengo perfiles externos (en `~/.config/opencode/profiles/*.json`) y no quiero que Gentle-AI los pise"

Gentle-AI los detecta y entra en modo `external-single-active` automáticamente. Si no detecta, forzá:

```bash
gentle-ai sync --agent opencode --sdd-profile-strategy external-single-active
```

En ese modo, Gentle-AI mantiene los assets base de SDD pero no genera ni reescribe perfiles automáticos.

---

## Resumen

- Multi-modelo por fase SDD = **OpenCode** y **Kilo Code** (overlay generado), **Kiro IDE** (nativo, sin Tab).
- Empezá con un perfil **"cheap"** y vas viendo si la calidad baja.
- **`gentle-orchestrator`** es la base, los perfiles con nombre son **`sdd-orchestrator-{nombre}`**.
- Crearlos con la **TUI** (recomendado) o con `--profile`/`--profile-phase` (CLI).
- Cambiás entre perfiles con **`Tab`** dentro de OpenCode.
- Perfiles **aislados** entre sí — el `sdd-apply-cheap` no interfiere con `sdd-apply-premium`.
- **No edites manualmente** las entradas `sdd-orchestrator-*` en `opencode.json`. Usá TUI/CLI.

---

## Siguiente paso

➡️ **[16 · Caso: compartir memoria con tu equipo](16-caso-engram-team.md)** — versionar `.engram/` en git, cuándo correr `engram sync` después de sesiones importantes, cómo resolver drift de nombres de proyecto en equipo.

📚 Doc de referencia (avanzado): **[docs/opencode-profiles.md](../opencode-profiles.md)** — referencia completa de estrategias, key names, comportamiento de sync.

---

← [Volver al índice](index.md) · ← [Anterior: Caso: sumarse a un repo existente](14-caso-repo-existente.md)
