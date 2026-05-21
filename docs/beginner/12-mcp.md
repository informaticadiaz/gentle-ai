# 12 · MCP en 5 minutos

← [Volver al índice](index.md) · ← [Anterior: Personas y permisos](11-personas-permisos.md)

---

> **Idea central**: **MCP (Model Context Protocol)** es el "USB" estándar que conecta tu agente a **herramientas externas**: memoria persistente, documentación de librerías, GitHub, archivos, búsquedas, lo que sea. Gentle-AI te configura **dos MCPs útiles desde el día uno** (Engram y Context7); el resto lo agregás vos si querés.

---

## La idea en una frase

Antes de MCP, un agente era una **caja sin enchufes**: solo podía leer/escribir archivos del proyecto y correr comandos de shell. Para darle acceso a algo más (tu base de datos, GitHub, tu sistema de tickets), había que hackear plugins propietarios por agente. Cada agente tenía su propio formato y su propia lista de integraciones.

**MCP estandariza el enchufe.** Un mismo servidor MCP funciona con Claude Code, OpenCode, Cursor, etc. Vos lo instalás una vez y todos los agentes lo ven.

---

## La analogía del USB

Pensalo así:

| MCP en el mundo de IA | USB en el mundo físico |
| --- | --- |
| El **agente** | La computadora |
| Un **servidor MCP** | Un dispositivo USB (impresora, disco, cámara) |
| El **protocolo MCP** | El estándar USB |
| La **config MCP** del agente | Los drivers cargados |

Un agente con MCP es a un agente sin MCP **lo que una computadora con USB es a una computadora sin enchufes**: la diferencia entre "solo lo que vino instalado" y "todo lo que quieras conectarle".

---

## Cómo lo usa el agente

Cuando un servidor MCP está conectado, expone **herramientas** (tools) que el agente puede invocar como si fueran propias. Por ejemplo, Engram expone:

```
mem_save, mem_search, mem_context, mem_session_summary, ...
```

El agente, cuando ve algo que vale la pena recordar, **llama** a `mem_save(...)`. Cuando empieza una sesión, llama a `mem_context()`. Vos **no escribís nada** de eso: es transparente.

Cada MCP define sus propias tools con su propio schema. El agente lo descubre al conectarse.

---

## Los MCPs que Gentle-AI configura por vos

Cuando elegís el preset `ecosystem-only` o `full-gentleman`, Gentle-AI conecta dos MCPs en todos los agentes que soporten MCP (es decir: todos los de la página 3, sin excepciones):

### 1) **Engram** — memoria persistente

Ya lo vimos en la página 7. Lo importante acá: Engram **se conecta vía MCP**. Lo que el agente "ve" son las herramientas `mem_save`, `mem_search`, etc.

**Comando que ejecuta el agente para hablar con él:**

```
engram mcp --tools=agent
```

Esto arranca el servidor MCP de Engram en stdio (el agente lo lanza como sub-proceso). Tu base de memoria sigue siendo `~/.engram/`.

### 2) **Context7** — documentación viva de librerías y frameworks

Context7 es un servidor MCP de [Upstash](https://upstash.com) que le da al agente acceso a **documentación actualizada** de miles de librerías y frameworks (React, Next.js, Tailwind, Zod, Postgres, etc.).

Lo que cambia: cuando le pedís al agente *"hacé esto con la API nueva de Next 14"*, en lugar de inventar (porque su training cutoff es de hace meses), **consulta la doc oficial vigente** vía Context7 y responde con la firma correcta.

**Cómo lo conecta cada agente** (Gentle-AI maneja los detalles):

| Agente | Estrategia |
| --- | --- |
| Claude Code | `npx --package=@upstash/context7-mcp` (stdio local) |
| OpenCode / Kilo | Endpoint remoto `https://mcp.context7.com/mcp` |
| VS Code Copilot / Antigravity / Kimi | Endpoint remoto HTTP |
| OpenClaw | `npx @upstash/context7-mcp` |

> Para vos es indistinto: el agente sabe llamar a la tool `get-library-docs` (o como se llame en tu agente) cuando le hace falta.

---

## Otros MCPs útiles (no instalados por defecto)

MCP tiene un ecosistema creciente. Algunos populares:

| MCP | Para qué sirve |
| --- | --- |
| **GitHub MCP** | Listar/crear/comentar issues y PRs, leer diffs, ver runs de CI. |
| **Filesystem MCP** | Lectura/escritura sandboxeada de carpetas específicas. |
| **Postgres / SQLite MCP** | Consultas SQL contra tus DBs. |
| **Slack MCP** | Leer canales, postear mensajes. |
| **Brave Search MCP** | Búsqueda web. |
| **Puppeteer / Playwright MCP** | Control de navegador para scraping o tests E2E. |
| **Memory MCP** (oficial Anthropic) | Memoria simple basada en grafos. *(Engram lo reemplaza con algo mejor para nuestro flujo.)* |

> Lista oficial / comunidad: [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers).

Gentle-AI **no los instala**, pero **no estorba** que los agregues. Cada agente tiene su propio formato de config MCP (eso lo veremos enseguida).

---

## Dónde vive la config MCP

Cada agente guarda la lista de servidores MCP en un lugar distinto:

| Agente | Archivo de config MCP |
| --- | --- |
| Claude Code | `~/.claude/mcp/*.json` (un archivo por servidor) o `~/.claude/settings.json` |
| OpenCode / Kilo | `~/.config/opencode/opencode.json` (sección `mcp`) |
| Cursor | `~/.cursor/mcp.json` |
| VS Code Copilot | `Code/User/mcp.json` (o `%APPDATA%\Code\User\mcp.json` en Windows) |
| Codex | `~/.codex/config.toml` (sección MCP) |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` |
| Gemini CLI | `~/.gemini/settings.json` |
| Antigravity | `~/.gemini/antigravity/mcp_config.json` |
| Kimi | `~/.kimi/` config con sección `mcpServers` |
| Kiro IDE | `~/.kiro/settings/mcp.json` *(siempre en `settings/`, distinto de GlobalConfigDir)* |
| OpenClaw | `~/.openclaw/openclaw.json` (sección `mcp.servers`) |
| Qwen Code | `~/.qwen/settings.json` (clave `mcpServers`) |
| Pi | `.pi/agent/settings.json` (vía `pi-mcp-adapter`) |

> **Punto importante**: cada agente tiene su propio dialecto JSON/TOML para definir MCPs. Las llaves cambian (`mcp` vs `mcpServers` vs `mcp.servers`), el shape también. Gentle-AI **te abstrae todo esto**: vos elegís componentes y servidores, y Gentle-AI escribe el formato correcto por agente.

---

## Agregar un MCP por vos mismo

Si querés sumar uno que Gentle-AI no instale (digamos, GitHub MCP), tenés dos caminos:

### A) Edición manual del archivo del agente

Ejemplo en Claude Code (`~/.claude/settings.json` o `~/.claude/mcp/github.json`):

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    }
  }
}
```

Reabrí el agente y debería conectar.

### B) Vía la TUI del propio agente

Algunos agentes (OpenCode, Cursor, VS Code Copilot) tienen pantallas para agregar MCPs interactivamente. Buscá "MCP servers" en sus settings.

> **Cuidado con tokens y secretos**: meterlos directo en el JSON es la forma fácil pero **peligrosa**. Mejor usar variables de entorno (`"env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "$GH_TOKEN" }`) y exportarlas en tu shell.

---

## ¿Cómo sé qué tools tengo disponibles?

Depende del agente:

- **Claude Code**: el comando `/mcp` lista los servidores conectados y sus tools.
- **OpenCode**: en la TUI hay una pantalla con MCPs detectados.
- **Otros**: revisá la doc del agente o probá: *"¿Qué herramientas MCP tenés disponibles ahora?"*. El agente las lista.

Si un MCP que configuraste **no aparece**, casi siempre es:

1. Mal formato JSON (faltó una coma).
2. Comando inexistente (`npx ...` que no encuentra el paquete).
3. Token faltante o mal puesto.
4. El agente no fue reiniciado después de editar.

---

## MCP y los componentes de Gentle-AI

Para que se entienda cómo encajan las piezas:

- Si instalás el componente **`engram`** → Gentle-AI configura el **Engram MCP server** en todos los agentes seleccionados.
- Si instalás el componente **`context7`** → Gentle-AI configura el **Context7 MCP server**.
- Los **demás componentes** (`sdd`, `skills`, `persona`, `permissions`, etc.) **no son MCPs**: son prompts, archivos de config y reglas.

Es decir: MCP es **solo una de las capas** que Gentle-AI instala. Convive con SDD, skills y persona.

---

## Limitaciones y precauciones

### 1) Los MCPs corren con los permisos del usuario

Un MCP que abre un puerto, accede a tu DB, o escribe en archivos, lo hace **como vos**. Si el binario es malicioso o tiene una vulnerabilidad, tiene tu mismo nivel de acceso. **Solo instales MCPs que confíes** (de fuentes oficiales o auditadas).

### 2) Los MCPs ven el contexto que el agente les pasa

Si un MCP "buscador" recibe tu prompt para buscar algo, puede registrarlo del lado servidor (si es remoto). **Tratá los MCPs remotos como servicios externos**: no les mandes secretos.

### 3) No todos los agentes soportan todos los transportes

MCP tiene dos transportes principales: **stdio** (local, comando + args) y **HTTP** (remoto). Algunos agentes solo soportan stdio (Codex), otros prefieren HTTP (VS Code Copilot, Antigravity). Gentle-AI elige el transporte correcto por agente cuando configura los MCPs propios; si agregás uno a mano, fijate qué soporta tu agente.

### 4) MCP es un estándar joven

El protocolo es de 2024 y todavía evoluciona. Algunos servidores son experimentales. Si uno se cuelga o devuelve cosas raras, **no es culpa de Gentle-AI**: revisá la versión y el repo del servidor.

---

## Errores comunes

### "El agente no ve a Engram"

```bash
which engram          # ¿está en el PATH?
engram mcp --tools=agent  # ¿arranca?
```

Si las dos cosas andan pero el agente no conecta, revisá el archivo MCP del agente y verificá que el `command` sea correcto (a veces queda con ruta relativa "engram" en lugar de la absoluta). Gentle-AI tiene lógica especial para preservar la ruta absoluta — si la perdiste, corré `gentle-ai sync` y se rearma.

### "Context7 no responde"

Probá manualmente:

```bash
npx -y @upstash/context7-mcp --help
```

Si requiere conexión a internet (sí), verificá tu red. Algunos firewalls corporativos bloquean `mcp.context7.com`.

### "Le agregué un MCP custom pero no aparece"

1. Reiniciá el agente (no alcanza con recargar).
2. Validá que el JSON sea válido (`cat tu-mcp.json | jq`).
3. Mirá los logs del agente al arrancar — los errores de MCP suelen quedar ahí.

### "Quiero saber qué tool llamó el agente"

Casi todos los agentes tienen un modo verbose / transcript. En Claude Code:

```bash
claude --verbose
```

En OpenCode: hay una vista de tool calls en la TUI. En general: si no encontrás la opción, preguntale al agente directamente: *"¿Qué tools llamaste en este turno?"*.

---

## Resumen

- **MCP** = estándar para conectar agentes a herramientas externas. Es **el USB de la era IA**.
- Gentle-AI configura **dos MCPs útiles por defecto**: **Engram** (memoria) y **Context7** (docs vivas).
- Cada agente guarda su config MCP en su propio formato/lugar. **Gentle-AI te abstrae eso.**
- Podés sumar más MCPs vos (GitHub, DB, Slack, Filesystem, etc.) editando el archivo del agente o vía su TUI.
- **Cuidados**: tokens en env vars (no en JSON), MCPs corren con tus permisos, MCPs remotos ven tu prompt.
- Si un MCP no aparece: revisá JSON, reiniciá el agente, validá comando/red.

---

## Siguiente paso

➡️ **[13 · Caso: arrancar un proyecto nuevo](13-caso-proyecto-nuevo.md)** — primer caso práctico: de `git init` a primer commit con el agente ayudándote, usando SDD para la primera feature.

📚 Spec oficial (opcional): **[modelcontextprotocol.io](https://modelcontextprotocol.io)** — si te entró curiosidad por el protocolo.

---

← [Volver al índice](index.md) · ← [Anterior: Personas y permisos](11-personas-permisos.md)
