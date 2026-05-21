# 13 · Caso: arrancar un proyecto nuevo

← [Volver al índice](index.md) · ← [Anterior: MCP en 5 minutos](12-mcp.md)

---

> **Objetivo del caso**: arrancar un proyecto **desde cero** con el agente acompañándote en cada paso — desde `git init` hasta el primer commit con una feature funcional. Vas a usar SDD por primera vez "en serio" y a ver Engram guardando contexto en tiempo real.
>
> **Stack del ejemplo**: Node.js + TypeScript + Vitest. Pero el flujo aplica a cualquier stack: cambiale los comandos, el patrón es el mismo.
>
> **Tiempo estimado**: 30-45 minutos.

---

## Lo que vas a tener al final

Una mini-CLI llamada `gemini-tasks` que:

- Lee/guarda tareas en un JSON local.
- Soporta `add`, `list`, `done`.
- Tiene tests unitarios.
- Tiene un README útil.
- Está versionada en git con 4-5 commits bien escritos.

Y, más importante: vas a tener **memoria del agente cargada** con las decisiones del proyecto, lista para acompañarte en la próxima sesión.

---

## Paso 0 · Pre-requisitos

Asumimos que ya hiciste:

- [x] Página 4 — Gentle-AI instalado.
- [x] Página 5 — Tu agente configurado (vamos a usar Claude Code en el ejemplo).
- [x] Tener `node`, `npm` y `git` en tu PATH.

Si te falta algo, volvé y completá. Esperamos.

---

## Paso 1 · Crear el repo

```bash
mkdir -p ~/code/gemini-tasks
cd ~/code/gemini-tasks
git init
```

**Importante**: agregale un `remote` aunque sea uno "fake" si todavía no tenés repo en GitHub. Por qué: Engram usa `git remote -v` para identificar el proyecto (página 7).

Si ya tenés un repo vacío en GitHub:

```bash
git remote add origin git@github.com:tu-usuario/gemini-tasks.git
```

Si todavía no querés crear el remoto: dale, Engram va a usar el nombre de carpeta como fallback. Lo arreglamos después.

---

## Paso 2 · Abrir el agente

Desde la carpeta del proyecto:

```bash
claude     # o opencode, o el que uses
```

Vas a ver el saludo de tu agente con la persona Gentleman/Neutral. Si no, revisá la página 5.

---

## Paso 3 · `sdd-init` — el agente conoce el terreno

Primer pedido literal:

```
hacé sdd-init en este proyecto
```

> En agentes con slash commands: `/sdd-init`.

El agente va a:

1. Mirar la carpeta (está vacía o casi).
2. **Preguntarte** qué stack vas a usar (porque no hay nada para detectar).
3. Guardar el contexto en Engram con `topic_key: sdd-init/gemini-tasks`.

**Respondé**:

```
Node.js con TypeScript. Tests con Vitest. Conventional commits.
Estructura src/ + tests/. Sin frameworks, es una CLI pequeña.
```

El agente va a confirmar y dejar el contexto guardado. **A partir de acá**, en cada futura sesión, ya sabe qué stack es esto.

---

## Paso 4 · Bootstrap del proyecto

Pedido:

```
inicializá el proyecto: package.json, tsconfig estricto, vitest config,
.gitignore, .editorconfig. Sin SDD, esto es bootstrap chico.
```

Notá el `sin SDD`: es un bootstrap mecánico, no amerita el flujo completo. Mirá la página 8 si dudás.

El agente arma:

- `package.json` con scripts (`dev`, `build`, `test`, `lint`).
- `tsconfig.json` estricto (`strict: true`, `noUncheckedIndexedAccess: true`).
- `vitest.config.ts`.
- `.gitignore` (Node + dist).
- `.editorconfig`.

> Si el componente `permissions` está activo, te va a **pedir confirmación** antes de escribir cada archivo. Aceptá. La página 11 explica por qué.

Una vez aplicado, podés instalar deps:

```bash
npm install
```

---

## Paso 5 · Primer commit (con `work-unit-commits` o `branch-pr`)

Pedido:

```
hagamos un commit inicial con el bootstrap. Seguí conventional commits.
```

El agente reconoce la skill `work-unit-commits` (página 9), te muestra un mensaje propuesto:

```
chore: bootstrap typescript cli project

- package.json with dev/build/test/lint scripts
- strict tsconfig
- vitest config
- .gitignore + .editorconfig
```

Si te parece bien, lo aplica:

```bash
git add .
git commit -m "..."
```

> **Recordá**: en el componente `permissions`, **`git commit` requiere `ask`** en OpenCode. El agente te muestra el mensaje y vos confirmás. En Claude Code el comando pasa pero los `rm -rf` están bloqueados (ver página 11).

---

## Paso 6 · Primera feature con SDD: `add`

Acá empieza lo bueno. Pedido:

```
use sdd. agregá un comando `add <titulo>` a la CLI que guarde una tarea
en .tasks/tasks.json con id, titulo, done=false, createdAt.
```

Notá `use sdd` explícito: aunque la tarea no sea enorme, **querés practicar el flujo** acá.

El agente arranca el orquestador:

### Fase 1 · `sdd-explore`

> *"Estoy en un proyecto vacío recién bootstrapeado. No hay código previo para explorar. ¿Avanzo directo a propose?"*

Decile que sí.

### Fase 2 · `sdd-propose`

El agente te muestra una propuesta corta:

```
intent: añadir comando 'add' a la CLI para crear tareas
scope:  src/commands/add.ts, src/storage/tasks.ts, tests/add.test.ts
approach:
  - parseo de args con minimist (o process.argv directo)
  - storage como repositorio simple (load/save JSON)
  - id incremental basado en timestamp
```

> *"¿Avanzamos con specs?"*

Vos:

```
ok, pero usá nanoid para el id en vez de timestamp.
```

El agente ajusta. **Esto es un gate de aprobación** (página 8): vos decidís el approach acá, no en medio del código.

### Fase 3 · `sdd-spec`

Te lista requirements y escenarios:

```
R1: `gemini-tasks add "comprar pan"` crea una tarea con id único, done=false
R2: el archivo .tasks/tasks.json se crea si no existe
R3: si existe, se preserva y se appendea
R4: salida en stdout: "✓ tarea {id} creada"

Escenarios:
- happy path: agregar una primera tarea
- happy path: agregar una segunda tarea (no pierde la primera)
- error: titulo vacío → exit code 1 + mensaje
```

Aprobás.

### Fase 4 · `sdd-design`

Decisiones técnicas: estructura de carpetas, naming, manejo de errores, atomicity de la escritura (write to .tmp + rename para evitar corrupción).

Aprobás.

### Fase 5 · `sdd-tasks`

Desglose:

```
1. crear src/storage/tasks.ts con load/save
2. test storage: load vacío, save+load, save 2 veces
3. crear src/commands/add.ts
4. test add: titulo válido, titulo vacío
5. wire en src/cli.ts
6. smoke test e2e
```

### Fase 6 · `sdd-apply`

El agente implementa **tarea por tarea**, te muestra el diff de cada una, y vos aprobás. Si el componente `permissions` está activo, te pregunta antes de cada `Write`/`Edit`.

Entre tarea y tarea corre los tests:

```bash
npm test
```

Si algo falla, **ahí mismo** se diagnostica, sin esperar al final.

### Fase 7 · `sdd-verify`

```
✓ npm test pasa (6/6)
✓ npm run build pasa
✓ npm run lint pasa
✓ smoke manual: agregué 2 tareas, ambas persisten
```

**Listo**. La feature está, los tests están, el build está.

### Fase 8 · `sdd-archive`

El agente guarda el resumen final en Engram (`sdd/add/archive`). Si elegiste `openspec` como artifact store, también deja archivos en `openspec/`.

---

## Paso 7 · Commit de la feature

Pedido:

```
hacelo con work-unit-commits: partí esto en commits revisables.
```

El agente propone:

```
1. feat(storage): add tasks repository with atomic write
   - src/storage/tasks.ts
   - tests/storage.test.ts

2. feat(cli): add 'add' command
   - src/commands/add.ts
   - tests/add.test.ts

3. feat(cli): wire commands into entry point
   - src/cli.ts
```

Aprobás. El agente hace 3 commits separados. Mucho más revisables que un mega-commit.

---

## Paso 8 · Repetir para `list` y `done` (sin SDD, ya sabe el patrón)

Para las features siguientes, el agente **ya tiene contexto cargado** (Engram + skill registry). Probá:

```
agregá `list` que muestre las tareas pendientes con id y título.
seguí el mismo patrón que `add`. sin sdd, es chico.
```

El agente:

1. Lee de Engram el design de `add`.
2. Aplica el mismo patrón a `list`.
3. Hace tests siguiendo el mismo estilo.
4. Te propone el commit.

**No tuviste que volver a explicar nada**. Ese es el momento "click".

Hacé lo mismo con `done <id>`.

---

## Paso 9 · README con `cognitive-doc-design`

Pedido:

```
escribime un README usando cognitive-doc-design. Audiencia: alguien que
acaba de clonar el repo y quiere usar la CLI.
```

El agente aplica la skill (página 9): README corto, con jerarquía visual clara, una sección "Quick start", una "Commands", una "How it works" minimalista. Sin paja.

Aprobalo y commit.

---

## Paso 10 · Sincronizar memoria con el repo

Para que la próxima persona (o vos en otra máquina) arranque con el contexto cargado:

```bash
engram sync
git add .engram/
git commit -m "chore: sync engram memory"
```

Ahora `.engram/` viaja con tu repo (página 7).

---

## Paso 11 · La prueba final

Salí del agente:

```
exit
```

Esperá un minuto. Volvé a entrar:

```bash
claude
```

Decile:

```
qué hicimos en este proyecto?
```

Si todo está bien configurado, te resume con detalles reales:

> *"Es una CLI en TS para gestionar tareas. Implementamos `add`, `list` y `done` siguiendo un patrón de repositorio con escritura atómica. Tests con Vitest, conventional commits, build a dist/. Decisiones: nanoid para IDs, write to .tmp + rename para atomicity. Quedó un README."*

**Ese resumen no lo inventó**: lo leyó de Engram.

---

## Lo que cambió en este flujo respecto al "chatbot pelado"

| Sin Gentle-AI | Con Gentle-AI |
| --- | --- |
| Le explicás stack en cada sesión. | `sdd-init` una vez y listo. |
| Tareas grandes = caos lineal. | SDD parte la tarea en pasos con gates. |
| Commits gigantes "wip". | `work-unit-commits` te propone commits revisables. |
| README inventado. | `cognitive-doc-design` lo arma con criterio. |
| El día siguiente, "¿qué hicimos?" → vacío. | El día siguiente, contexto cargado de Engram. |

---

## Variaciones del caso

### Si tu stack es otro

Cambiá Node/TS por lo que uses. El flujo es **idéntico**:

- Python + pytest → Cambiá `vitest` por `pytest`, `package.json` por `pyproject.toml`.
- Go + testing → Cambiá la skill auxiliar a `go-testing` (página 9).
- Rust + cargo test → idem.

`sdd-init` se adapta a cualquier stack que el agente reconozca.

### Si el proyecto es web (Next.js, etc.)

Después de `sdd-init`, va a sugerir agregarte el componente **Context7** si todavía no lo tenés (página 12), para tener docs vivas de React/Next. Aceptalo.

### Si no querés SDD para nada

Decile *"sin SDD nunca, hacé todo directo"* desde el primer pedido. Vas a perder los gates de aprobación, pero ganás velocidad. Útil para prototipos descartables.

---

## Errores comunes en este flujo

### "El agente arrancó a codear sin sdd-init"

Probablemente le pediste una feature antes de bootstrap. Decile:

```
pará. corré sdd-init primero.
```

Y empezá de nuevo.

### "Engram no me encuentra el proyecto al volver"

Verificá:

```bash
git remote -v          # ¿hay un remote?
engram projects list   # ¿aparece tu proyecto?
```

Si hay duplicados, `engram projects consolidate` (página 7).

### "El agente quiere instalar dependencias sin preguntar"

Si tenés `permissions` activo, debería pedirte. Si no te pide, revisá tu config (página 11) y subí la fricción a `"ask"` para `npm install`.

### "Los commits me quedan todos juntos en uno"

Pedile explícitamente:

```
usá work-unit-commits para partir esto en N commits revisables.
```

---

## Resumen

- **`sdd-init` primero**, antes de cualquier feature. Te ahorra explicar el stack 100 veces.
- Usá **SDD explícito (`use sdd`)** para la primera feature, aunque sea chica: practicás el flujo y dejás contexto cargado.
- A partir de ahí, el agente **reaplica patrones** sin que se los recuerdes.
- **`work-unit-commits` + `cognitive-doc-design`** elevan la calidad sin esfuerzo.
- **`engram sync` + commit** al final = memoria viaja con el repo.
- La prueba clave: cerrar la sesión, volver al rato y preguntar *"¿qué hicimos?"*. Si responde con detalles reales, ganaste.

---

## Siguiente paso

➡️ **[14 · Caso: sumarse a un repo existente](14-caso-repo-existente.md)** — el flujo opuesto: clonás un repo grande del equipo y tenés que ponerlo a producir rápido sin romper nada.

---

← [Volver al índice](index.md) · ← [Anterior: MCP en 5 minutos](12-mcp.md)
