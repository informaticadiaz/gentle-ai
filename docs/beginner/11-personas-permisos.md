# 11 · Personas y permisos

← [Volver al índice](index.md) · ← [Anterior: Sub-agentes y delegación](10-subagentes-delegacion.md)

---

> **Idea central**: la **persona** es **cómo te habla** el agente (tono, idioma, filosofía). Los **permisos** son **qué puede tocar** (qué archivos lee, qué comandos ejecuta sin preguntar). Gentle-AI inyecta una persona "teaching-first" con permisos *security-first* por defecto, pero todo es configurable.

---

## Persona ≠ permisos. No mezclar.

Estos dos conceptos viven juntos y se confunden seguido. Vale la pena separarlos:

| | Persona | Permisos |
| --- | --- | --- |
| Pregunta que responde | "¿Cómo te habla el agente?" | "¿Qué puede tocar el agente sin pedirte permiso?" |
| Afecta a | El **chat** y las decisiones de comunicación. | Los **comandos** que ejecuta y los **archivos** que toca. |
| Ejemplo | Te explica el porqué de una decisión. | No lee `.env` aunque se lo pidas. |
| Componente | `persona` | `permissions` |

Las dos cosas se instalan por separado (`--component persona`, `--component permissions`), aunque el preset `full-gentleman` y `ecosystem-only` traen ambas.

---

## Las 3 opciones de persona

Las repasamos del glosario (página 2):

### `gentleman` — la persona por defecto (didáctica + directa)

Inyecta un perfil de **Senior Architect, 15+ años, GDE & MVP, profesor apasionado**. Algunas reglas que aplica:

- **Te empuja cuando podés hacerlo mejor**. No por enojo: porque le importa que aprendas.
- **No te miente con un "sí" rápido**: verifica antes de confirmar tus claims.
- **Si te equivocás, te dice por qué con evidencia**. Si se equivocó él, lo reconoce.
- **Una pregunta por vez** y se calla a esperar.
- **Respuestas cortas por defecto**. Si querés más, le pedís más.
- **Sin menús de opciones inútiles** ni "exhaustive lists": solo hay alternativas si hay un fork real con tradeoffs.

Tono: directo, cálido, con énfasis (CAPS cuando importa). En español usa **voseo rioplatense**, en inglés mantiene la misma energía.

### `neutral` — misma filosofía, sin regionalismos

**El mismo profesor**, las mismas convicciones, pero **sin slang ni voseo**. Para quien prefiere un tono más universal, o trabaja en equipos donde el rioplatense distrae.

Mismas reglas técnicas: respuestas cortas, una pregunta por vez, verificación antes de confirmar, etc.

### `custom` — Gentle-AI no toca tu persona

Si ya tenés tu propia persona definida (en `CLAUDE.md`, `agents.md`, lo que sea) y querés que Gentle-AI **la respete**, elegí `custom`. Esto **no es un editor**: simplemente le dice a Gentle-AI "ya tengo lo mío, no lo pises".

> Útil cuando estás migrando desde otro setup, o cuando trabajás en un equipo con persona propia documentada.

---

## La regla más importante de la persona: scope

Esto es **crítico** y suele confundir:

> **La persona gobierna lo que el agente DICE en el chat. NO gobierna lo que el agente CONSTRUYE.**

Es decir:

| Donde aplica la persona | Donde NO aplica |
| --- | --- |
| Las respuestas que te escribe en el chat. | Los nombres de variables y funciones. |
| El tono al explicar una decisión. | El texto de UI (botones, labels, errores). |
| El idioma del mensaje al usuario. | Los comentarios del código. |
| Si usa CAPS para enfatizar al hablar contigo. | Los mensajes de commit / descripciones de PR. |
| | Los identificadores en general. |

**Default técnico**: el agente escribe **código, identificadores, UI, docs y commits en inglés**, salvo que:

- Vos le pidas explícitamente otro idioma para ese artefacto.
- El proyecto **ya** usa otro idioma claramente.

Esto evita el clásico bug de pedir "voseo" y terminar con `function obtenerUsuariosLogueados()` y botones con "Mandalo che" en producción.

---

## Cómo cambiar de persona

### Durante la instalación

En la TUI: elegís en la pantalla "Persona".

Por CLI:

```bash
gentle-ai install --agent claude-code --persona gentleman
gentle-ai install --agent claude-code --persona neutral
gentle-ai install --agent claude-code --persona custom
```

### Más tarde

Re-corré la instalación con el nuevo flag:

```bash
gentle-ai install --agent claude-code --persona neutral
```

Gentle-AI hace **backup** de tu persona actual antes de pisar.

### En Pi específicamente

Pi tiene su propio comando para cambiar persona en runtime:

```
/gentleman:persona
```

Switchea entre `gentleman` y `neutral`, guarda la elección en `.pi/gentle-ai/persona.json`, y puede requerir `/reload` o una sesión nueva para que tome efecto.

---

## Permisos: el componente `permissions`

Acá entramos a "qué puede hacer el agente sin pedirte confirmación". Gentle-AI tiene defaults **security-first**: el agente puede trabajar fluido para lo cotidiano, pero **se traba en seco** ante operaciones peligrosas o secretos.

### Defaults en Claude Code

Modo `bypassPermissions` (auto-aceptar) **excepto**:

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions",
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo rm -rf /)",
      "Bash(rm -rf ~)",
      "Bash(sudo rm -rf ~)",
      "Read(.env)",
      "Read(.env.*)",
      "Edit(.env)",
      "Edit(.env.*)"
    ]
  }
}
```

Es decir: el agente **no puede** hacer `rm -rf` en raíz/home, ni leer/editar `.env*`. Punto.

### Defaults en OpenCode

Granularidad por tipo de operación:

```json
{
  "permission": {
    "bash": {
      "*": "allow",
      "git commit *": "ask",
      "git push *": "ask",
      "git push --force *": "ask",
      "git rebase *": "ask",
      "git reset --hard *": "ask"
    },
    "read": {
      "*": "allow",
      "*.env": "deny",
      "**/.env": "deny",
      "**/secrets/**": "deny",
      "**/credentials.json": "deny"
    }
  }
}
```

Es decir:

- **Bash**: permite casi todo, pero **pregunta antes** de git commit, push, rebase, o reset --hard. Esas son las operaciones que pueden borrar trabajo.
- **Read**: lee todo, pero **niega** acceso a cualquier `.env*`, `secrets/`, o `credentials.json`.

---

## La lógica detrás de los defaults

Tres categorías:

### 1) Bloqueado siempre — operaciones destructivas

`rm -rf /`, `rm -rf ~`, etc. **No hay caso de uso legítimo** para que un agente AI las corra. Bloqueadas, fin.

### 2) Bloqueado siempre — secretos

`.env`, `.env.*`, `secrets/`, `credentials.json`. Aunque vos le digas "leelo", se niega. Mejor un agente paranoico que filtrar tu API key a Engram, logs o, peor, al chat.

> Si necesitás que el agente sepa de configuración sensible, **inyectala como variables de entorno en tu shell** y referenciala como `process.env.X` en código. El agente no necesita ver el valor.

### 3) Pregunta antes — operaciones git con riesgo

`git commit`, `git push`, `git push --force`, `git rebase`, `git reset --hard`. Todo esto **modifica historia compartida** o **publica código**. Vale la pena que vos confirmes cada vez (al menos por defecto).

> ¿Te molesta confirmar todo el tiempo? Podés cambiar `"ask"` por `"allow"` para tu caso. Pero pensalo: el costo de una pregunta es **1 segundo**; el costo de un push accidental a `main` es bastante más.

---

## Personalizar permisos para tu flujo

Si los defaults te quedan muy estrictos o muy permisivos, podés:

### Editar el archivo de config del agente directamente

Cada agente tiene su lugar:

- **Claude Code**: `~/.claude/settings.json` (sección `permissions`).
- **OpenCode**: `~/.config/opencode/opencode.json` (sección `permission`).

Editá, guardá, reabrí la sesión.

### O usar la TUI / CLI

Re-correr `gentle-ai sync` después de editar mantiene los marcadores y no pisa lo tuyo.

> **Importante**: Gentle-AI hace **merge con marcadores**, no overwrite. Si vos editás dentro de los marcadores que controla Gentle-AI, en el próximo sync se pisa. Pero **fuera de los marcadores tu config queda intacta**.

---

## Casos típicos: permisos según contexto

### "Estoy en mi máquina personal, quiero menos fricción"

Cambiá los `"ask"` de git por `"allow"`. **Mantené los `"deny"` de secretos**.

### "Estoy en una máquina de cliente / producción, quiero más fricción"

Cambiá los `"allow"` peligrosos por `"ask"`. Por ejemplo:

```json
"bash": {
  "rm *": "ask",
  "mv *": "ask",
  "*": "allow"
}
```

### "Quiero leer un .env por una razón legítima"

No saques el deny global. Hacelo **una vez** desactivando temporalmente en la session, o:

```bash
cat .env    # vos lo leés, le pegás lo necesario al agente sin la API key
```

### "Quiero permitir un script específico que el agente necesita"

Agregá entradas más específicas con `"allow"`:

```json
"bash": {
  "*": "ask",
  "npm test": "allow",
  "npm run build": "allow"
}
```

Las más específicas ganan sobre el catch-all.

---

## ¿Qué hace `permissions` que no hace `persona`?

Son **complementarias**. Mirá la diferencia con un ejemplo:

> Le pedís: *"borrá toda la carpeta `node_modules` y volvé a instalar"*.

| | Sin `permissions` | Con `permissions` |
| --- | --- | --- |
| Sin `persona` | Lo hace en silencio. | Te pregunta antes de `rm -rf`. |
| Con `persona` | Te explica que va a hacer `rm -rf node_modules` y por qué (y lo hace). | Te explica + te pregunta antes. |

La persona te **acompaña**. Los permisos te **protegen**.

---

## Errores comunes

### "El agente no me deja leer `.env` y yo necesito mirarlo"

Está bien que no te deje. Cambiá las reglas a mano si tu caso es legítimo:

```json
"read": {
  "*.env": "ask"   // en vez de "deny"
}
```

Después podés aprobar caso por caso. **No pongas `"allow"`** si tu `.env` tiene secretos reales.

### "Cada git push me pregunta y me cansa"

Cambiá `"git push *": "ask"` por `"git push *": "allow"`. Tu llamada. Solo dejá en `"ask"` el `--force`.

### "Quiero forzar la persona Gentleman pero el agente responde frío"

Probablemente quedó `custom`. Re-corré:

```bash
gentle-ai install --agent <tu-agente> --persona gentleman
```

Y reabrí la sesión. Si seguís viendo frío, mirá que tu archivo de persona no esté siendo pisado por otro (algunos agentes tienen prompts viejos cargados).

### "Quiero la persona pero quiero que el código quede en español"

Decíselo explícitamente al inicio: *"este proyecto usa identificadores y UI en español, mantenelo así"*. La persona respeta tu instrucción **para el proyecto**. La regla por defecto es solo para evitar accidentes.

---

## Resumen

- **Persona** = cómo te habla. **Permisos** = qué puede tocar. **No son lo mismo.**
- Tres personas: `gentleman` (default, didáctica, voseo en chat), `neutral` (igual de didáctica, sin regionalismos), `custom` (no tocar la tuya).
- **Regla de oro**: la persona afecta solo el **chat**. **Código, UI, docs y commits siempre en inglés por default** (salvo que pidas otro idioma).
- Permisos: **security-first** por defecto — bloquea `rm -rf` raíz/home, lee/edita `.env*` y secrets, y pregunta antes de operaciones git destructivas.
- Todo configurable. Gentle-AI usa **merge con marcadores**: tu config fuera de los marcadores queda intacta.

---

## Siguiente paso

➡️ **[12 · MCP en 5 minutos](12-mcp.md)** — qué es el Model Context Protocol, qué servidores MCP instala Gentle-AI (filesystem, github, engram, etc.) y para qué sirven.

---

← [Volver al índice](index.md) · ← [Anterior: Sub-agentes y delegación](10-subagentes-delegacion.md)
