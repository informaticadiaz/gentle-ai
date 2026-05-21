# 5 · Tu primera configuración

← [Volver al índice](index.md) · ← [Anterior: Instalación paso a paso](04-instalacion.md)

---

Ya tenés `gentle-ai` instalado (página 4) y al menos un agente en tu máquina (página 3). Ahora viene lo divertido: **configurar el agente** para que tenga memoria, skills, SDD, persona y MCP.

Hay dos caminos:

- **TUI (recomendado)**: lanzás `gentle-ai` sin argumentos y te guía paso a paso.
- **CLI (no interactivo)**: una sola línea con flags. Útil para automatizar o repetir en otra máquina.

Vamos a recorrer los dos.

---

## Camino A · La TUI guiada

Abrí una terminal y corré:

```bash
gentle-ai
```

Vas a ver un menú estilo "wizard" con tema Rose Pine. Las teclas son siempre las mismas:

| Tecla | Acción |
| --- | --- |
| `j` / `k` o ↑ / ↓ | Mover el cursor |
| `space` | Marcar/desmarcar en pantallas multi-selección |
| `enter` | Confirmar / avanzar |
| `esc` | Volver atrás |
| `q` | Salir |

### Paso 1 · Menú principal

La pantalla de bienvenida muestra estas opciones (algunas aparecen solo si tenés OpenCode):

- **Start installation** ← arrancá acá la primera vez
- Upgrade tools
- Sync configs
- Upgrade + Sync
- Configure models
- Create your own Agent
- OpenCode Community Plugins
- OpenCode SDD Profiles *(solo si tenés OpenCode)*
- Manage backups
- Managed uninstall
- Quit

Movete con `j/k`, pará en **Start installation**, `enter`.

### Paso 2 · Detección de agentes

Gentle-AI escanea tu máquina y muestra qué agentes encontró. Por ejemplo:

```
Detected:
  ✓ claude-code      ~/.claude
  ✓ opencode         ~/.config/opencode
  ✗ cursor           (no encontrado)
```

Solo los detectados son seleccionables. Si esperabas ver alguno y no aparece, salí (`q`), instalalo, y volvé a empezar.

### Paso 3 · Seleccionar agentes

Pantalla **"Select AI Agents"**. Marcá con `space` los que querés configurar, y `enter` para seguir.

> **Tip de principiante**: arrancá con **uno solo**. Es más fácil entender qué cambia. Después podés re-correr `gentle-ai install` y agregar más.

### Paso 4 · Persona

Tres opciones (revisalas en la [página 2 · glosario](02-glosario.md#persona) si dudás):

- `gentleman` — mentor *teaching-oriented*, con tono Gentleman.
- `neutral` — misma filosofía, sin regionalismos.
- `custom` — Gentle-AI **no toca** tu persona actual.

Si recién empezás, `gentleman` es la elección clásica.

### Paso 5 · Preset

Elegí qué tan completo querés el setup:

| Preset | ID | Para quién |
| --- | --- | --- |
| Full Gentleman | `full-gentleman` | Querés **todo** de una. |
| Ecosystem Only | `ecosystem-only` | Núcleo sano sin temas/extras visuales. |
| Minimal | `minimal` | Solo Engram + skills de SDD. |
| Custom | `custom` | Vos elegís componente por componente. |

> **Recomendación principiante**: `ecosystem-only`. Tiene Engram + SDD + Skills + Context7 + GGA + persona, sin extras que puedan confundir.

### Paso 6 · Componentes y skills (solo si elegiste `custom`)

Si elegiste un preset, **saltea este paso**. Gentle-AI ya sabe qué instalar.

Si elegiste `custom`, vas a ver dos pantallas más:

1. **Component picker** — marcá con `space` los componentes (engram, sdd, skills, context7, persona, permissions, gga, theme).
2. **Skill picker** — marcá las skills puntuales que querés (puede saltearse si elegiste el componente `skills` completo).

### Paso 7 · Strict TDD (si tu stack lo soporta)

Si Gentle-AI detecta que tu agente puede correr tests, pregunta si querés activar **Strict TDD Mode**. Esto le indica al agente que escriba el test **antes** que el código, y verifique que falla antes de implementar.

Es opcional. Si dudás, decí que sí: podés revertirlo más tarde re-corriendo el installer.

### Paso 8 · Review and Confirm

Pantalla resumen, parecida a esto:

```
Review and Confirm

  Agents     claude-code
  Persona    gentleman
  Preset     ecosystem-only

  Components
    engram        selected
    sdd           selected
    skills        selected
    context7      selected
    persona       selected
    gga           selected
    permissions   auto-dependency

  Skills
    sdd-init, sdd-explore, sdd-design, ..., branch-pr, ...

  Strict TDD   enabled

  enter: install • esc: back
```

Revisá. `esc` si querés cambiar algo. `enter` para aplicar.

### Paso 9 · Instalación

Gentle-AI:

1. **Hace un backup** de tus configs actuales (tar.gz, en `~/.atl/backups/`).
2. **Inyecta** archivos en las rutas correctas del agente (sin pisar lo tuyo: usa merge con marcadores).
3. **Verifica** el resultado.

Verás logs tipo `[info]`, `[ok]`, etc., y al final una pantalla **"Complete"** con un resumen.

¡Listo! Tu agente quedó configurado.

---

## Camino B · CLI no interactivo

Si querés evitar la TUI (porque ya sabés qué querés, o porque vas a script-earlo), usá `gentle-ai install` con flags.

### Ejemplo: preset completo en Claude Code

```bash
gentle-ai install \
  --agent claude-code \
  --preset full-gentleman \
  --persona gentleman
```

### Ejemplo: setup mínimo en Cursor

```bash
gentle-ai install \
  --agent cursor \
  --preset minimal
```

### Ejemplo: varios agentes a la vez

```bash
gentle-ai install \
  --agent claude-code,opencode,gemini-cli \
  --preset full-gentleman
```

### Ejemplo: elegir componentes y skills a mano

```bash
gentle-ai install \
  --agent claude-code \
  --component engram,sdd,skills,context7,persona,permissions \
  --skill go-testing,skill-creator,branch-pr,issue-creation \
  --persona gentleman
```

### Ensayar sin aplicar (`--dry-run`)

Antes de tocar nada, podés ver el plan:

```bash
gentle-ai install --dry-run \
  --agent claude-code,opencode \
  --preset full-gentleman
```

Esto imprime qué archivos se crearían/modificarían **sin escribir nada**. Útil para revisar antes de la instalación real.

> **Tip**: el primer `--dry-run` te muestra el plan completo sin riesgo. Si te convence, sacá el flag y volvé a correr.

---

## ¿Qué cambió en mi máquina?

Dependiendo del agente, Gentle-AI escribió en algunas de estas rutas (sin pisar lo que ya tenías):

| Agente | Carpeta |
| --- | --- |
| Claude Code | `~/.claude/` |
| OpenCode | `~/.config/opencode/` |
| Cursor | `~/.cursor/` |
| Codex | `~/.codex/` |
| Windsurf | `~/.codeium/windsurf/` |
| Kiro IDE | `~/.kiro/` + `~/.kiro/settings/mcp.json` |
| Pi | `~/.pi/` |

Además se creó (o actualizó):

- `~/.engram/` — base de memoria persistente.
- `~/.atl/backups/` — snapshots de tus configs anteriores.

Podés inspeccionar todo a mano si te da curiosidad. **Nada está oculto.**

---

## Verificar que quedó bien

### 1) El agente arranca y ve la persona

Abrí tu agente y pedile algo simple:

```
¿Quién sos y qué podés hacer?
```

Si respondió con tono "teaching-first" (te explica el por qué de las cosas) y mencionó SDD/skills/memoria, la persona quedó inyectada.

### 2) El skill registry está armado

En tu proyecto:

```bash
gentle-ai skill-registry refresh
```

La primera vez tarda un poco (escanea skills + convenciones del proyecto). Las siguientes son casi instantáneas porque cachea.

### 3) Engram responde

```bash
engram projects list
```

Debería listar al menos los proyectos donde ya lo usaste (si es la primera vez puede estar vacío, eso es normal).

### 4) Backups existen

```bash
ls -lh ~/.atl/backups/
```

Debe haber al menos un `.tar.gz` reciente con la marca de tiempo de cuando corriste el install.

---

## ¿Y si me equivoqué?

Tres caminos seguros:

1. **Volver a correr `gentle-ai`** y elegir otro preset/agente. Lo nuevo se merge con lo viejo, sin perder lo bueno.
2. **Rollback desde la TUI**: menú principal → *Manage backups* → elegí un snapshot → restaurar.
3. **Managed uninstall**: menú principal → *Managed uninstall* → seleccionás qué agentes / componentes sacar. También hace backup antes.

> Conclusión: **no le tengas miedo a probar**. Cada cambio queda respaldado.

---

## Sincronizar / actualizar más tarde

Cuando salga una versión nueva, o cuando agregues skills a tu repo:

```bash
gentle-ai update              # actualiza el binario
gentle-ai sync                # re-inyecta lo último en los agentes
```

Desde la TUI: opciones **Upgrade tools**, **Sync configs**, o **Upgrade + Sync** (que hace ambas).

---

## Resumen

- **TUI** (`gentle-ai`) → te guía paso a paso. Lo recomendado la primera vez.
- **CLI** (`gentle-ai install --agent X --preset Y`) → para repetir/scriptar.
- `--dry-run` antes de aplicar = cero riesgo.
- Todo lo que se toca queda **respaldado** en `~/.atl/backups/`.
- Si te equivocás: re-corré el installer, o restaurá un backup, o usá *Managed uninstall*.

---

## Siguiente paso

➡️ **[6 · Tu primera sesión real](06-primera-sesion.md)** — abrir el agente en un proyecto, correr `/sdd-init`, hacer una tarea chica y otra más grande para ver la diferencia.

📚 Doc de referencia (avanzado): **[docs/usage.md](../usage.md)** — todos los flags del CLI, todas las pantallas de la TUI, persona modes, gestión de dependencias.

---

← [Volver al índice](index.md) · ← [Anterior: Instalación paso a paso](04-instalacion.md)
