# 17 · Actualizar y respaldos

← [Volver al índice](index.md) · ← [Anterior: Caso: compartir memoria con tu equipo](16-caso-engram-team.md)

---

> **Idea central**: Gentle-AI hace **backup automático antes de cada cambio** (install, sync, upgrade). Si algo sale mal, restaurás en 10 segundos desde la TUI. Y mantener todo al día es **un solo comando**: `gentle-ai update`. Esta página explica las dos cosas y cómo se relacionan.

---

## Las tres operaciones que actualizan tu setup

Hay tres comandos que pueden modificar tu configuración. Vale la pena saber qué hace cada uno:

| Comando | Qué hace | Cuándo usarlo |
| --- | --- | --- |
| **`gentle-ai install`** | Configura agentes nuevos o cambia preset/componentes. | Primera vez, o cuando agregás agente/componente. |
| **`gentle-ai sync`** | Re-inyecta los assets actuales (prompts, skills, MCP, SDD) en los agentes ya configurados. | Después de cambiar versión, o si querés re-aplicar la config canónica. |
| **`gentle-ai update`** (alias `upgrade`) | Actualiza el binario y dependencias (`engram`, `gga`, etc.). | Cuando salió una nueva versión. |

Los tres **disparan un backup** antes de tocar nada. Los tres son **reversibles**.

---

## Actualizar el binario

### El camino recomendado

Si instalaste con Homebrew/Scoop:

```bash
# macOS / Linux
brew upgrade gentle-ai

# Windows
scoop update gentle-ai
```

### El self-updater integrado

Funciona para todos los métodos de instalación:

```bash
gentle-ai update
```

> `update` y `upgrade` son **alias** — el mismo comando con dos nombres. Usá el que te suene mejor.

### Después de actualizar: re-sync

El binario nuevo trae **assets nuevos** (skills mejoradas, MCPs actualizados, prompts evolucionados). Para que tus agentes los reciban:

```bash
gentle-ai sync
```

> **Atajo**: la opción **"Upgrade + Sync"** del menú principal de la TUI hace las dos cosas en orden.

### Verificá que quedó

```bash
gentle-ai version
```

---

## El sync, en detalle

`gentle-ai sync` es la operación más usada después de actualizar. Importante saber:

### Qué hace

- **Actualiza** el contenido de los prompts del sistema (`CLAUDE.md`, `AGENTS.md`, `QWEN.md`, etc.).
- **Refresca** las skills instaladas en `~/.claude/skills/`, `~/.config/opencode/skills/`, etc.
- **Reescribe** los configs MCP (Engram, Context7) con las versiones nuevas.
- **Regenera** los archivos de sub-agentes SDD por agente.

### Qué NO hace

- **No reinstala** binarios externos (`engram`, `gga`). Para eso `gentle-ai update`.
- **No toca** agentes que **no marcaste** como gestionados. Gentle-AI guarda tu selección en `~/.gentle-ai/state.json` y respeta ese scope.

### Preview antes de aplicar

Antes de un sync grande, mirá qué va a cambiar:

```bash
gentle-ai sync --dry-run
```

Lista los archivos que tocaría sin escribir nada.

### Sync selectivo

Si solo querés actualizar **un componente** (por ejemplo, las skills):

```bash
gentle-ai sync --component skills
gentle-ai sync --component sdd
gentle-ai sync --component engram
```

O un agente puntual:

```bash
gentle-ai sync --agent claude-code
gentle-ai sync --agent opencode
```

Combinable:

```bash
gentle-ai sync --agent claude-code --component skills
```

---

## Los backups: el seguro silencioso

### Cuándo se crean

**Cada vez** que corrés `install`, `sync` o `update/upgrade`, Gentle-AI:

1. **Calcula un checksum** de los archivos que va a tocar.
2. **Si es idéntico** al backup más reciente → **lo saltea** (dedup). No spamea backups inútiles.
3. **Si cambió algo** → crea un nuevo snapshot.
4. **Prunea** los viejos: mantiene los **5 más recientes** no pineados.

### Qué contiene un backup

Cada snapshot vive en `~/.atl/backups/<timestamp>/` con esta estructura:

```
~/.atl/backups/2026-05-21T12-34-56/
├── manifest.json      # metadata (origen, timestamp, archivos, checksum, pin)
└── snapshot.tar.gz    # archive comprimido con todos los archivos respaldados
```

- **`snapshot.tar.gz`** comprime ~75% más chico que la versión sin comprimir.
- **`manifest.json`** marca con `existed=false` los archivos que **no existían antes**: al restaurar, esos se **borran** (volvés al estado anterior).
- **Backups legacy** (pre-v1.16) usan una carpeta `files/` con copias sin comprimir. Siguen siendo restaurables.

### Política de retención

| Setting | Default | Comportamiento |
| --- | --- | --- |
| Cantidad mantenida | **5** | Los 5 más recientes no pineados quedan. |
| Pineados | **nunca borrados** | Sobreviven al prune sin importar la cantidad. |
| Duplicados | **saltados** | Si el config no cambió, no hay nuevo backup. |
| Compresión | **siempre** | Nuevos backups usan tar.gz. |

---

## Manejar backups desde la TUI

```bash
gentle-ai
```

Menú principal → **"Manage backups"**.

Teclas en la pantalla de backups:

| Tecla | Acción |
| --- | --- |
| `j` / `k` | Navegar arriba / abajo. |
| `Enter` | **Restaurar** el backup seleccionado. |
| `p` | **Pinear / despinear**. Pineados sobreviven al prune. |
| `r` | **Renombrar** (agregarle una descripción). |
| `d` | **Borrar**. |
| `Esc` | Volver al menú. |

> **Tip**: pineá los backups antes de cambios grandes. Por ejemplo, antes de cambiar de preset, pineá el último — así podés volver siempre, sin importar cuántos sync corras después.

---

## Restaurar: cómo funciona

### Por archivo, no por carpeta

El restore es **archivo por archivo**:

- Si el archivo **existía antes** (`existed=true` en el manifest) → restaura su contenido a la ruta original.
- Si el archivo **no existía antes** (`existed=false`) → lo **borra** (revierte la creación que hizo Gentle-AI).
- **Atómico**: cada archivo se escribe completo o no se toca. **No hay restores a medias**.
- Funciona con backups **comprimidos** (nuevos) y **legacy** (sin comprimir).

### Restaurar el último, sin TUI

```bash
gentle-ai restore latest
```

Útil cuando algo se rompió justo después de un cambio y querés volver atrás rápido.

### Caso típico

Acabás de hacer un `gentle-ai install` con un preset nuevo, abrís el agente y el comportamiento te resulta raro. Tres opciones:

1. **Restaurar el backup previo**:
   ```bash
   gentle-ai restore latest
   ```
2. **Cambiar el preset** sin restaurar:
   ```bash
   gentle-ai install --agent <tu-agente> --preset <otro-preset>
   ```
3. **Managed uninstall** para sacar componentes específicos (sin tocar otros):
   ```bash
   gentle-ai   # TUI → Managed uninstall
   ```

Cualquiera de las tres es segura.

---

## Lo que el rollback NO cubre

**Importante**: el sistema de backups maneja **archivos de configuración**, no paquetes del sistema:

- Paquetes instalados vía `brew install`, `apt-get install`, `pacman -S`, `npm install -g`, etc., **siguen instalados** después de un rollback.
- Si necesitás desinstalar un binario externo (por ejemplo, `engram`), usá tu package manager directamente:

  ```bash
  brew uninstall engram
  sudo apt remove engram
  sudo pacman -R engram
  ```

El backup te protege la **config**, no el **stack de binarios**.

---

## Si la verificación post-install falla

Cuando un install termina, Gentle-AI corre **checks de verificación**. Si alguno falla, vas a ver un reporte. Pasos sugeridos:

1. **Mirá** qué check falló (suelen ser claros: "engram binary not found in PATH", "context7 MCP not responding", etc.).
2. **Restaurá** el snapshot más reciente:
   ```bash
   gentle-ai restore latest
   ```
   O desde la TUI → Backups.
3. **Re-corré** con `--dry-run` para validar el plan sin aplicar:
   ```bash
   gentle-ai install --dry-run --agent <agente> --preset <preset>
   ```
4. **Arreglá** la dependencia externa que faltaba (instalar Node, Homebrew, etc.).
5. **Volvé** a correr el install real.

---

## Rutina de mantenimiento sugerida

No hay obligación, pero esta rutina mantiene todo prolijo:

### Cada 1-2 semanas

```bash
gentle-ai update   # actualizá el binario
gentle-ai sync     # propagá los assets nuevos a tus agentes
```

O usá **"Upgrade + Sync"** desde la TUI.

### Antes de cambios grandes (cambiar preset, agregar agentes nuevos)

1. Abrí la TUI, andá a Backups, **pineá** el último (`p`).
2. Renombralo con `r` y agregale un nombre tipo `pre-preset-change-2026-05`.
3. Recién después aplicá el cambio.

### Cada tanto

Revisá los backups (`gentle-ai` → Backups). Borrá los pineados que ya no necesitás con `d` para liberar disco.

### Si algo se rompe

`gentle-ai restore latest` y volvés al estado anterior. **Sin pensarlo**.

---

## Espacio en disco: ¿cuánto ocupa esto?

Aproximado:

- Un snapshot **comprimido** suele rondar **100 KB - 2 MB** según cuántos componentes y agentes manejes.
- Multiplicado por 5 (retención default) = **0.5 - 10 MB**.
- Es **insignificante** en cualquier disco moderno.

Si manejás muchos pineados con descripciones de hitos importantes, puede llegar a 50-100 MB. Sigue siendo nada.

---

## Errores comunes

### "`gentle-ai update` dice que ya estoy en la última, pero hay cambios visibles en el repo"

Tu binario está al día, pero los **assets** locales no se reescribieron. Corré:

```bash
gentle-ai sync
```

Eso re-inyecta el contenido canónico de la versión actual.

### "Hice `restore latest` y el agente sigue raro"

Tres causas posibles:

1. El agente cacheó algo en runtime — **reiniciá** el agente.
2. El backup elegido no era el correcto — usá la TUI y elegí uno más viejo.
3. Hay un cache fuera del scope de backups (por ejemplo, `.atl/skill-registry.cache.json` en un proyecto):
   ```bash
   gentle-ai skill-registry refresh --force
   ```

### "Quiero un backup ahora, antes de probar algo"

No hay un comando dedicado, pero podés forzarlo corriendo un sync que sepa "no hacer nada":

```bash
gentle-ai sync --component skills
```

Si las skills no cambiaron, no escribe nada **pero igual crea un snapshot** del estado actual. Pinealo desde la TUI.

> Alternativa más limpia: hacer una **copia manual** de las carpetas de config de tus agentes antes del experimento.

### "Tengo muchos backups viejos y quiero limpiarlos a fondo"

Desde la TUI → Backups → `d` en cada uno que no quieras. Los **pineados no se borran** automáticamente — los tenés que despinear (`p`) primero o borrar manualmente.

### "Borré sin querer un backup que necesitaba"

**No hay papelera**. Una vez borrado desde la TUI, está borrado. Por eso conviene **pinear y renombrar** los importantes antes de hacer limpieza.

### "Quiero saber qué hay adentro de un snapshot antes de restaurar"

```bash
ls ~/.atl/backups/
# elegí el timestamp que te interese
cat ~/.atl/backups/<timestamp>/manifest.json | jq
tar -tzf ~/.atl/backups/<timestamp>/snapshot.tar.gz | head -40
```

Vas a ver la lista completa de archivos que toca. **No los modifiques a mano** — solo mirá.

---

## Resumen

- Tres comandos pueden cambiar tu setup: **`install`**, **`sync`**, **`update`**. Los tres hacen backup automático antes de tocar nada.
- **`update`** actualiza el binario; **`sync`** propaga los assets nuevos a tus agentes. El atajo es **"Upgrade + Sync"** en la TUI.
- Backups: **tar.gz comprimidos**, **deduplicados**, **se mantienen los 5 más recientes** + los pineados.
- TUI de Backups: `j/k`, **Enter** (restore), `p` (pin), `r` (rename), `d` (delete), `Esc`.
- Restore es **atómico por archivo** y revierte también los archivos que Gentle-AI **creó**.
- `gentle-ai restore latest` es el botón de emergencia.
- **No cubre** paquetes de sistema (`brew`, `apt`, `npm -g`). Para eso, usá tu package manager.
- Rutina sana: `update` + `sync` cada 1-2 semanas, **pinear antes de cambios grandes**.

---

## Siguiente paso

➡️ **[18 · Resolución de problemas comunes](18-troubleshooting.md)** — cuando algo no anda: skills que no aparecen, Engram que no encuentra el proyecto, OpenCode que no ve tu perfil, modo no-interactivo, y más.

📚 Doc de referencia (avanzado): **[docs/rollback.md](../rollback.md)** — política de retención completa, formato de snapshots legacy, comportamiento de verify.

---

← [Volver al índice](index.md) · ← [Anterior: Caso: compartir memoria con tu equipo](16-caso-engram-team.md)
