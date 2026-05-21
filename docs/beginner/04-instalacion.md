# 4 · Instalación paso a paso

← [Volver al índice](index.md) · ← [Anterior: Agentes soportados](03-agentes-soportados.md)

---

Esta página te lleva de **no tener nada** a **tener `gentle-ai` instalado y funcionando**. No instala todavía componentes ni configura agentes (eso es la página 5): solo el binario base.

> Estimación: **2 a 5 minutos** en macOS/Linux, **3 a 7 minutos** en Windows.

---

## Antes de empezar

Necesitás:

- **Un agente de IA ya instalado** en tu máquina (Claude Code, OpenCode, Cursor, etc.). Si todavía no instalaste ninguno, andá a la página 3 y elegí uno. Gentle-AI **configura** agentes; no los descarga.
- **Una terminal**. En macOS y Linux ya la tenés. En Windows: PowerShell o Git Bash funcionan; el camino recomendado es **PowerShell + Scoop**.
- **Conexión a internet** para bajar el binario.

No necesitás:

- Cuenta de GitHub.
- Privilegios de root/sudo en el caso recomendado (Homebrew/Scoop).
- Conocer Go (aunque hay un método opcional con `go install`).

---

## macOS / Linux — método recomendado (Homebrew)

Homebrew es el más simple y mantiene la actualización limpia:

```bash
brew tap Gentleman-Programming/homebrew-tap
brew install gentle-ai
```

Verificá que quedó:

```bash
gentle-ai version
```

Si aparece una línea con la versión, listo.

> ¿No tenés Homebrew? Instalalo desde [brew.sh](https://brew.sh) y volvé a este paso. O usá el método de una línea más abajo.

---

## macOS / Linux — método universal (script `install.sh`)

Esto auto-detecta el mejor método disponible (Homebrew → `go install` → binario precompilado):

```bash
curl -fsSL https://raw.githubusercontent.com/Gentleman-Programming/gentle-ai/main/scripts/install.sh | bash
```

> ⚠️ **Buena práctica de seguridad**: revisar siempre los scripts que ejecutás con `curl | bash`. Podés descargarlo primero y leerlo:
>
> ```bash
> curl -fsSLO https://raw.githubusercontent.com/Gentleman-Programming/gentle-ai/main/scripts/install.sh
> less install.sh        # leer
> chmod +x install.sh
> ./install.sh
> ```

### Opciones útiles del script

| Flag | Para qué sirve |
| --- | --- |
| `--method brew` | Fuerza Homebrew. |
| `--method go` | Fuerza `go install` (requiere Go 1.24+). |
| `--method binary` | Descarga un binario precompilado desde GitHub Releases. |
| `--dir $HOME/.local/bin` | Elegís dónde dejar el binario (solo `binary`). |
| `--insecure` | Salta la verificación de checksum. **No recomendado.** |
| `-h` / `--help` | Ayuda. |

Ejemplo: instalar el binario en `~/.local/bin` sin tocar el sistema:

```bash
./install.sh --method binary --dir $HOME/.local/bin
```

Después agregá `~/.local/bin` a tu `PATH` si todavía no está:

```bash
# bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
# zsh
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

---

## macOS / Linux — método para devs (Go)

Si ya tenés Go 1.24 o superior:

```bash
go install github.com/gentleman-programming/gentle-ai/cmd/gentle-ai@latest
```

El binario queda en `$(go env GOPATH)/bin`. Asegurate de que esa carpeta esté en tu `PATH`.

---

## Windows — método recomendado (Scoop)

[Scoop](https://scoop.sh) es el camino soportado en Windows; mantiene actualizaciones limpias y no necesita admin:

```powershell
scoop bucket add gentleman https://github.com/Gentleman-Programming/scoop-bucket
scoop install gentle-ai
```

Verificá:

```powershell
gentle-ai version
```

> ¿No tenés Scoop? Instalalo desde [scoop.sh](https://scoop.sh) (una sola línea de PowerShell) y volvé a este paso.

### Notas Windows

- **PowerShell o Git Bash**: ambos funcionan. Para usuarios nuevos: PowerShell.
- **No necesitás `sudo`**: Scoop instala en tu carpeta de usuario.
- **`curl` ya viene** en Windows 10/11; no hace falta instalarlo aparte.
- **GGA** funciona desde PowerShell y Git Bash. Gentle-AI instala un shim `gga.ps1` que delega a Git Bash automáticamente.

---

## Plataformas y gestores de paquetes soportados

| SO | Gestor | Estado |
| --- | --- | --- |
| macOS (Apple Silicon e Intel) | Homebrew | ✅ Soportado |
| Linux Ubuntu/Debian (y derivados como Mint, Pop!_OS) | apt | ✅ Soportado |
| Linux Arch (y derivados como Manjaro, EndeavourOS) | pacman | ✅ Soportado |
| Linux Fedora/RHEL (y CentOS Stream, Rocky, AlmaLinux) | dnf | ✅ Soportado |
| Windows 10/11 | Scoop | ✅ Soportado |

Las distros derivadas se detectan automáticamente vía `ID_LIKE` en `/etc/os-release`.

---

## Verificar que la instalación está bien

Después de instalar, corré estos checks:

```bash
# 1) ¿Está en el PATH?
which gentle-ai          # macOS/Linux
Get-Command gentle-ai    # Windows PowerShell

# 2) ¿Qué versión tengo?
gentle-ai version

# 3) ¿Responde el help?
gentle-ai --help
```

Si las tres responden sin errores, **estás listo**. La página 5 se ocupa de configurar tu primer agente con la TUI.

---

## Solución de problemas comunes

### "command not found: gentle-ai"

El binario quedó instalado, pero su carpeta no está en tu `PATH`. Buscalo:

```bash
# macOS/Linux
ls -l /usr/local/bin/gentle-ai 2>/dev/null
ls -l $HOME/.local/bin/gentle-ai 2>/dev/null
ls -l $(go env GOPATH 2>/dev/null)/bin/gentle-ai 2>/dev/null
```

Cuando lo encuentres, agregá esa carpeta al `PATH` (`~/.bashrc`, `~/.zshrc`, perfil de PowerShell, etc.) y abrí una terminal nueva.

### "permission denied" en macOS al ejecutar

macOS puede bloquear binarios sin firma. Soluciones:

- Instalar vía **Homebrew** (recomendado, evita el bloqueo).
- Si bajaste el binario directo: `xattr -d com.apple.quarantine /ruta/gentle-ai`.

### El script `install.sh` falló a mitad

Re-corré con `--method binary` para forzar el binario precompilado (no necesita Homebrew ni Go):

```bash
./install.sh --method binary
```

Si sospechás de un problema de checksum, **no uses `--insecure`** a menos que entiendas el riesgo. Reportá el problema en [issues](https://github.com/Gentleman-Programming/gentle-ai/issues).

### Windows: "no se puede ejecutar script" en PowerShell

Tu política de ejecución está bloqueando scripts. Para el perfil de usuario:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Después volvé a instalar.

### Scoop: "bucket already exists"

Si ya tenías un bucket `gentleman` apuntando a otra URL, removelo y volvé a agregarlo:

```powershell
scoop bucket rm gentleman
scoop bucket add gentleman https://github.com/Gentleman-Programming/scoop-bucket
```

### Linux: error "GLIBC not found" al ejecutar el binario

Tu distro es muy vieja o muy minimal. Probá `--method go` (compila localmente) en lugar de bajar el binario precompilado.

---

## Actualizar después

Cuando ya estés con `gentle-ai` instalado, las actualizaciones son fáciles:

```bash
# Homebrew (macOS/Linux)
brew upgrade gentle-ai

# Scoop (Windows)
scoop update gentle-ai

# Cualquier método: el self-updater integrado
gentle-ai update
```

`gentle-ai update` también actualiza componentes instalados (skills, persona, MCPs) y crea un **backup** automático antes de tocar nada.

---

## Desinstalar

Si querés sacar Gentle-AI:

```bash
# Homebrew
brew uninstall gentle-ai
brew untap Gentleman-Programming/homebrew-tap

# Scoop
scoop uninstall gentle-ai
scoop bucket rm gentleman

# Go install
rm "$(go env GOPATH)/bin/gentle-ai"
```

Esto saca el **binario**. Tus **configs de agentes** quedan intactas (`~/.claude/`, `~/.config/opencode/`, etc.). Si querés limpiarlas también:

```bash
# Cuidado: revisá qué hay adentro antes de borrar.
rm -rf ~/.atl ~/.engram
```

Antes de borrar `~/.engram` pensalo dos veces: es donde vive tu memoria persistente. Exportala con `engram sync` o `engram tui` si querés conservarla.

---

## Resumen

| Plataforma | Recomendado | Verificación |
| --- | --- | --- |
| macOS / Linux | `brew install gentle-ai` (tras `brew tap`) | `gentle-ai version` |
| Windows | `scoop install gentle-ai` (tras `scoop bucket add`) | `gentle-ai version` |
| Cualquiera con Go 1.24+ | `go install github.com/gentleman-programming/gentle-ai/cmd/gentle-ai@latest` | `gentle-ai version` |
| Una línea universal | `curl -fsSL .../install.sh \| bash` | `gentle-ai version` |

---

## Siguiente paso

➡️ **[5 · Tu primera configuración](05-primera-configuracion.md)** — abrir la TUI con `gentle-ai`, elegir agente(s), elegir preset, ver qué se instala y verificar que todo quedó en su lugar.

📚 Doc de referencia (avanzado): **[docs/platforms.md](../platforms.md)** — paths de config por agente en Windows, verificación de firma, notas de instalación más profundas.

---

← [Volver al índice](index.md) · ← [Anterior: Agentes soportados](03-agentes-soportados.md)
