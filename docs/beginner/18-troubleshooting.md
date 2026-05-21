# 18 · Resolución de problemas comunes

← [Volver al índice](index.md) · ← [Anterior: Actualizar y respaldos](17-update-backups.md)

---

> **Cómo usar esta página**: organizada **por síntoma**, no por componente. Buscá la frase que se parece a tu problema, leé el diagnóstico, aplicá la solución. Si no encontrás tu caso, ojo a la sección de **diagnóstico general** al final.

---

## 🛠 Instalación

### "El comando `gentle-ai` no se encuentra después de instalar"

El binario quedó instalado pero la carpeta no está en tu `PATH`. Probá:

```bash
which gentle-ai                                            # macOS/Linux
Get-Command gentle-ai                                      # Windows PS

ls -l /usr/local/bin/gentle-ai 2>/dev/null
ls -l $HOME/.local/bin/gentle-ai 2>/dev/null
ls -l "$(go env GOPATH 2>/dev/null)/bin/gentle-ai" 2>/dev/null
```

Cuando lo encuentres, agregá su carpeta al `PATH` (`~/.bashrc`, `~/.zshrc`, perfil PowerShell) y abrí una terminal nueva.

📘 Más detalle: [página 4 · instalación](04-instalacion.md).

### "El script `install.sh` falla a mitad"

Re-corré forzando el método "binary" (no requiere Homebrew ni Go):

```bash
./install.sh --method binary
```

Si te sigue fallando el checksum, **no uses `--insecure`**: reportá el problema en [issues](https://github.com/Gentleman-Programming/gentle-ai/issues).

### "En Windows: 'no se puede ejecutar script' en PowerShell"

Política de ejecución bloqueando scripts. Una sola vez:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

### "Scoop dice 'bucket already exists'"

```powershell
scoop bucket rm gentleman
scoop bucket add gentleman https://github.com/Gentleman-Programming/scoop-bucket
```

### "Linux: 'GLIBC not found' al ejecutar el binario"

Tu distro es muy vieja. Compilá localmente:

```bash
go install github.com/gentleman-programming/gentle-ai/cmd/gentle-ai@latest
```

(Requiere Go 1.24+).

---

## 🤖 El agente / la persona

### "El agente responde como chatbot pelado, sin la persona"

La persona no se inyectó (o el agente cacheó la versión vieja).

```bash
gentle-ai sync                       # re-inyecta
# después:
gentle-ai sync --component persona   # si seguís sin verla
```

Cerrá y reabrí el agente.

### "Quiero la persona Gentleman pero estoy viendo respuestas frías"

Probablemente quedó en `custom`. Re-corré:

```bash
gentle-ai install --agent <tu-agente> --persona gentleman
```

### "El agente está escribiendo identificadores en español ('crearUsuario')"

La persona afecta **chat**, no **código**. Si te lo está mezclando es porque entendió mal. Decile explícitamente:

> *"código, identificadores y UI en inglés siempre. La persona es solo para el chat."*

📘 Detalle: [página 11 · personas y permisos](11-personas-permisos.md), sección "scope".

### "El agente quiere borrar `.env` o hacer `rm -rf` algo"

Los permisos están bien configurados — el sistema bloquea esas operaciones. Si **necesitás** ese acceso, ajustá tu config (`~/.claude/settings.json` o `~/.config/opencode/opencode.json`).

📘 Detalle: [página 11 · personas y permisos](11-personas-permisos.md).

### "Cada `git push` me pregunta y me cansa"

Cambiá en tu config:

```json
"bash": {
  "git push *": "allow"   // antes era "ask"
}
```

Pero dejá `"git push --force *": "ask"` por seguridad.

---

## 🔧 Skills

### "Le pedí algo y no aplicó la skill que esperaba"

Tres causas, en orden de frecuencia:

1. **Registry desactualizado**:
   ```bash
   gentle-ai skill-registry refresh --force
   ```
   Cerrá y reabrí el agente.

2. **Trigger no matchea**. Forzá:
   > *"aplicá la skill `branch-pr` para esto."*

3. **Skill no instalada**:
   ```bash
   cat .atl/skill-registry.md   # ¿aparece tu skill?
   ```

📘 Detalle: [página 9 · skills](09-skills.md).

### "Agregué una skill nueva (manualmente) y el agente no la ve"

Necesitás refresh:

```bash
gentle-ai skill-registry refresh
```

Si igual no aparece, verificá el frontmatter del `SKILL.md` (debe tener `name` y `description` válidos).

### "Quiero ver qué skills tengo disponibles ahora"

```bash
cat .atl/skill-registry.md
```

O dentro del agente:

> *"listame todas las skills disponibles en este proyecto."*

### "Una skill global y una skill del proyecto tienen el mismo nombre"

**Gana la del proyecto** (página 9). Si querés la global, renombrá la del proyecto o borrala.

---

## 🧠 Engram

### "El agente no recuerda lo que hablamos ayer"

```bash
engram projects list   # ¿está mi proyecto?
```

Si aparece con N observaciones: la memoria existe, pero el agente no la lee. Probá:

> *"buscá en engram qué hicimos antes en este proyecto."*

Si **no aparece**, hay drift de nombre. Verificá:

```bash
git remote -v
engram projects list
```

Si ves variantes (`my-app`, `My-App`):

```bash
engram projects consolidate
```

📘 Detalle: [página 7 · engram memoria](07-engram-memoria.md).

### "Cloné un repo con `.engram/` y `--import` no carga nada"

```bash
ls -la .engram/   # ¿existe?
git log .engram/  # ¿tiene historia?
```

Si la carpeta está, la versión local de `engram` puede ser vieja:

```bash
gentle-ai update
engram sync --import
```

### "Hago `engram sync` pero no aparece nada como modificado en git"

Engram dedupe: si no hubo cambios desde el último sync, no escribe nada. Forzá una observación nueva pidiéndole al agente algo concreto que valga la pena guardar, después sync.

### "Le pegué una API key al agente sin querer y quedó en memoria"

Buscala y borrala:

```bash
engram search "sk-"     # ajustá el prefijo según el tipo de clave
```

Anotá el ID y desde el agente:

> *"borrá la observación con id X."*

### "Tengo conflict de merge en `.engram/observations.json`"

**No edites el JSON a mano** (rompe los índices).

```bash
git checkout --theirs .engram/
engram sync --import
engram sync
git add .engram/
git commit -m "chore: merge engram memory"
```

📘 Detalle: [página 16 · engram en equipo](16-caso-engram-team.md).

---

## 🔌 MCP

### "El agente no ve a Engram (tools `mem_*` ausentes)"

```bash
which engram
engram mcp --tools=agent   # ¿arranca?
```

Si ambos andan pero el agente no conecta:

```bash
gentle-ai sync   # restaura la ruta absoluta del comando engram
```

Cerrá y reabrí el agente.

### "Context7 no responde"

Probá manualmente:

```bash
npx -y @upstash/context7-mcp --help
```

Si necesita red y estás detrás de un firewall corporativo, verificá acceso a `mcp.context7.com`.

### "Agregué un MCP custom (GitHub, Postgres) y no aparece"

1. **JSON válido**: `cat tu-mcp.json | jq` (sin error).
2. **Reiniciá el agente** (no alcanza con recargar).
3. **Revisá logs** del agente al arrancar.
4. **El agente lo soporta**: algunos solo aceptan stdio, otros HTTP — verificá la doc del agente.

📘 Detalle: [página 12 · MCP](12-mcp.md).

### "Quiero saber qué tools llamó el agente en este turno"

- **Claude Code**: `claude --verbose`.
- **OpenCode**: vista de tool calls en la TUI.
- **Cualquiera**: preguntale: *"¿Qué tools llamaste en este turno?"*.

---

## 🔀 SDD y delegación

### "El agente no propone SDD y arranca a codear directo en una feature grande"

1. **¿Está SDD instalado?**
   ```bash
   gentle-ai install --component sdd
   ```
2. **Skill registry desactualizado**:
   ```bash
   gentle-ai skill-registry refresh --force
   ```
3. **Forzalo**:
   > *"use sdd para esto."*

### "El agente arranca SDD para tareas triviales"

Decile desde el primer turno:

> *"sin sdd, hacelo directo."*

O reformulá el prompt para que no parezca una feature grande.

### "Mi agente es solo-agent (Codex / Windsurf / Antigravity / OpenClaw) y no delega"

Es esperable. En solo-agent **no hay delegación a sub-agentes**: SDD corre inline en la misma sesión. Engram sigue funcionando como "memoria entre fases".

📘 Detalle: [página 10 · sub-agentes y delegación](10-subagentes-delegacion.md).

### "Cerré la sesión a la mitad de SDD y no sé cómo retomar"

```
/sdd-continue
```

O: *"retomá el último cambio SDD donde lo dejamos"*. El agente lo lee de Engram.

### "Quedó esperando aprobación en cada fase y me cansa"

Pasá a fast-forward:

```
/sdd-ff
```

O: *"hacé fast-forward, no me consultes hasta verify."*

---

## 🎛 OpenCode SDD Profiles

### "Creé un perfil pero `Tab` no lo muestra"

```bash
gentle-ai sync --agent opencode
```

Cerrá y reabrí OpenCode.

### "Mi modelo custom no aparece en el picker de Gentle-AI"

En tu `opencode.json`, el modelo necesita:

```json
"tool_call": true
```

Sin eso, el picker lo descarta (SDD requiere tools). Editá y volvé a `gentle-ai sync`.

### "Mi config tiene `sdd-orchestrator` sin el prefijo `gentle-`"

Config legacy. Corré:

```bash
gentle-ai sync --agent opencode
```

La migración a `gentle-orchestrator` es automática. Tus perfiles con nombre se preservan.

### "Tengo perfiles externos (`~/.config/opencode/profiles/*.json`) y Gentle-AI los pisa"

Forzá la estrategia compatible:

```bash
gentle-ai sync --agent opencode --sdd-profile-strategy external-single-active
```

📘 Detalle: [página 15 · OpenCode profiles](15-caso-opencode-profiles.md).

---

## 💾 Backups y restore

### "`gentle-ai update` dice que estoy al día pero hay cambios visibles en el repo de assets"

El binario está al día, pero los assets locales no se re-escribieron:

```bash
gentle-ai sync
```

### "Hice `restore latest` y el agente sigue raro"

1. Reiniciá el agente (puede tener cache en runtime).
2. Probá un backup más viejo desde la TUI.
3. Limpiá caches locales:
   ```bash
   gentle-ai skill-registry refresh --force
   ```

### "Quiero ver qué hay en un snapshot antes de restaurar"

```bash
ls ~/.atl/backups/
cat ~/.atl/backups/<timestamp>/manifest.json | jq
tar -tzf ~/.atl/backups/<timestamp>/snapshot.tar.gz | head -40
```

**Solo lectura**: no modifiques nada a mano.

### "Borré sin querer un backup que necesitaba"

**No hay papelera**. Antes de borrar a fondo, **pineá y renombrá** los importantes (`p` + `r` en la TUI).

📘 Detalle: [página 17 · update y backups](17-update-backups.md).

---

## 🤖 Modo no-interactivo (CI / scripts)

### "Quiero usar Gentle-AI desde CI o un script"

```bash
gentle-ai install \
  --agent claude-code,opencode \
  --preset full-gentleman \
  --persona gentleman \
  --dry-run
```

Sin `--dry-run` para aplicar. Todos los flags son **idénticos** entre plataformas; el package manager se detecta solo.

📘 Detalle: [docs/non-interactive.md](../non-interactive.md).

### "El CI falla porque pregunta cosas"

El install **completo** con `--preset` y `--persona` **no pregunta nada**. Si te pide algo, te falta un flag. Probá con `--dry-run` primero para ver el plan.

### "Quiero ver qué se va a hacer sin aplicarlo"

```bash
gentle-ai install --dry-run [tus-flags]
gentle-ai sync --dry-run
```

`--dry-run` también imprime la línea **Platform decision** (OS, distro, package manager, status).

---

## 🔬 Diagnóstico general (cuando nada de lo anterior aplica)

Si tu problema no encaja en ninguna sección, **antes de abrir un issue**, recolectá:

### 1) Versión y entorno

```bash
gentle-ai version
uname -a            # macOS/Linux
$PSVersionTable     # Windows PowerShell
```

### 2) Estado del setup

```bash
ls ~/.gentle-ai/state.json     # ¿qué agentes/componentes marcó como instalados?
cat ~/.gentle-ai/state.json | jq
```

### 3) Logs del agente

Cada agente tiene su forma:

- **Claude Code**: `claude --verbose`
- **OpenCode**: TUI muestra logs; o `~/.config/opencode/logs/`
- **Cursor / VS Code**: panel de Output → seleccioná el canal del agente
- **Pi**: `~/.pi/logs/`

### 4) Reproducción mínima

Hacé un dry-run del install y del sync para tu setup:

```bash
gentle-ai install --dry-run --agent <agente> --preset <preset>
gentle-ai sync --dry-run
```

Pegá la salida en el issue.

### 5) Abrí el issue

[github.com/Gentleman-Programming/gentle-ai/issues](https://github.com/Gentleman-Programming/gentle-ai/issues)

Con: versión, OS, comando exacto, qué esperabas, qué pasó, logs relevantes. **Más detalle = respuesta más rápida.**

---

## Cosas que rara vez son "bugs" (pero confunden)

### "El agente me hizo una pregunta y no avanza"

Esperable. La persona Gentleman hace **una pregunta a la vez** y **se calla a esperar**. Respondé.

### "El skill registry tarda 2 segundos en arrancar"

Primera vez. Después está cacheado por `.atl/.skill-registry.cache.json` y arranca casi instantáneo.

### "Los snapshots ocupan espacio"

5 snapshots ronda los 0.5-10 MB. Es negligible. No los borres por temor al disco.

### "Mi commit tiene 1000 líneas de `.engram/` modificado"

Normal en repos con `.engram/` versionado. No es código que revisar línea por línea, es estado. Si te molesta el ruido, considerá la Filosofía B de la página 16 (PRs separados de memoria).

---

## Resumen

- Buscá por **síntoma** en esta página antes de abrir issue.
- Comandos que arreglan el **80% de los problemas**:
  - `gentle-ai sync` — re-inyecta assets.
  - `gentle-ai skill-registry refresh --force` — refresca el index.
  - `gentle-ai restore latest` — volvé al estado anterior.
- Cuando hay duda: **`--dry-run`** primero, aplicar después.
- Si nada funciona: recolectá versión + estado + logs + dry-run y abrí un issue.

---

## Siguiente paso

➡️ **[19 · ¿Qué leer ahora?](19-siguientes-pasos.md)** — mapa de los docs "avanzados" del repo: cuándo ir a `intended-usage.md`, `architecture.md`, `CODEBASE-GUIDE.md`, `PRD.md`, y otros.

---

← [Volver al índice](index.md) · ← [Anterior: Actualizar y respaldos](17-update-backups.md)
