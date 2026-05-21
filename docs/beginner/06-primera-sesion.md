# 6 · Tu primera sesión real

← [Volver al índice](index.md) · ← [Anterior: Tu primera configuración](05-primera-configuracion.md)

---

Ya configuraste el agente (página 5). Ahora vamos a usarlo **dentro de un proyecto real** y ver, en vivo, las diferencias que aporta Gentle-AI: bootstrap de contexto, memoria persistente, delegación cuando la tarea crece, y SDD apareciendo solo cuando hace falta.

Esta página es **práctica**. La idea es que la sigas con un proyecto abierto y vayas viendo cada cosa.

> Tiempo estimado: **15-25 minutos** para hacer las dos tareas de ejemplo.

---

## Preparar el terreno

Elegí un proyecto **chico** para esta primera vuelta:

- Algo tuyo que ya conozcas (mejor).
- O cloná uno de prueba (por ejemplo, un repo de juguete que tengas).
- **Evitá** un repo de producción o un monorepo grande para la primera sesión.

Asegurate de que esté inicializado como git:

```bash
cd /ruta/a/tu-proyecto
git status   # si responde, está bien
```

> **Por qué importa**: Engram detecta el nombre del proyecto desde el `git remote` (y normaliza a minúsculas). Sin git, igual funciona, pero el nombre se infiere de la carpeta y puede haber "drift" entre máquinas.

---

## Paso 1 · Abrir el agente en el proyecto

Esto depende del agente que hayas configurado:

| Agente | Cómo abrirlo |
| --- | --- |
| **Claude Code** | `claude` desde la carpeta del proyecto |
| **OpenCode** | `opencode` desde la carpeta del proyecto |
| **Cursor** | Abrí el proyecto en Cursor y abrí el panel de Agent |
| **VS Code Copilot** | Abrí el proyecto en VS Code y abrí Copilot Chat |
| **Codex** | `codex` desde la carpeta del proyecto |
| **Pi** | `pi` desde la carpeta del proyecto |

Si tu agente abre y te saluda con tono "teaching-first" (te explica las cosas, no responde como chatbot pelado), la persona quedó bien inyectada. Si responde frío, salí, corré `gentle-ai sync` y volvé a entrar.

---

## Paso 2 · `/sdd-init` — bootstrap del contexto

Lo primero que se hace en un proyecto nuevo (para Gentle-AI):

```
/sdd-init
```

> En agentes sin slash commands (Cursor, Claude Code, Codex), simplemente decí: *"hacé sdd-init"* o *"inicializa el contexto SDD del proyecto"*. El agente sabe qué hacer.

Esto le pide al agente que:

1. **Detecte el stack** (lenguaje, framework, gestor de paquetes).
2. **Detecte las convenciones** del proyecto (linters, tests, estructura).
3. **Detecte el framework de testing** y, si corresponde, active *Strict TDD Mode*.
4. **Guarde el contexto** en Engram con `topic_key: "sdd-init/{proyecto}"` (de modo que re-correrlo actualiza, no duplica).

La salida típica se ve así (resumida):

```
status: ok
executive_summary:
  - Stack: Node.js + TypeScript + Next.js 14
  - Tests: Vitest detectado
  - Linters: ESLint + Prettier
  - Convenciones: ruta /src, alias '@/...', conventional commits
artifacts:
  - engram: sdd-init/mi-proyecto
next_recommended:
  - Sugerir SDD para features de scope >= medium
```

A partir de ahora, **el agente no te va a volver a preguntar** qué stack es, cómo correr los tests, ni cuál es la convención de commits. **Está todo en memoria.**

> Si tu agente soporta hooks (Claude Code, OpenCode, Pi), `/sdd-init` se dispara solo cuando hace falta. Igual no hace daño correrlo a mano la primera vez.

---

## Paso 3 · Tarea chica — para sentir el "antes vs después"

Pedile algo trivial:

```
Agregá un README chiquito con el nombre del proyecto, una descripción
de una línea, y las dos formas de correrlo (dev y prod).
```

Lo que vas a ver (y conviene observar):

1. **NO arranca SDD**. La tarea es chica → no hay ceremonia.
2. El agente **propone el cambio** (te muestra el contenido del archivo).
3. Si el componente `permissions` está activo (security-first), **te pide confirmación** antes de escribir.
4. Tras aplicar, **te explica** lo que hizo, no solo "listo".

Aceptá. Mirá el archivo. **Esa es la experiencia "chatbot bien configurado"**: el agente trabaja como un junior que te explica lo que hace.

---

## Paso 4 · Tarea más grande — para ver SDD entrar

Ahora pedile algo que **requiera planificación**. Algo así como:

```
Quiero agregar un endpoint `/api/users/me` que devuelva los datos del
usuario logueado (incluido permisos). Considerá cómo lo testeamos, qué
pasa si no hay sesión, y dónde encaja en el código actual.
```

Lo que conviene observar:

1. El agente **detecta que es grande** y propone **usar SDD**. Algo como:
   > *"Esta tarea toca routing, sesión, permisos y tests. Te propongo usar SDD: exploro el código, propongo enfoque, escribimos spec, diseño técnico, tareas, y recién después implementación. ¿Avanzamos?"*

2. Si aceptás, arranca la **fase Explore**. En agentes con full-delegation (Claude Code, OpenCode, Cursor, Kiro, etc.) **lanza un sub-agente** con contexto propio. Vas a ver en la TUI/log algo tipo:

   ```
   → delegating to sdd-explore
     skill: sdd-explore
     mode: read-only
   ```

3. El sub-agente vuelve con un **resumen** (qué encontró, qué archivos importan, qué patrones existentes seguir).

4. Sigue con **Propose** (intención, scope, approach), **Spec** (requirements y escenarios), **Design** (decisiones técnicas), **Tasks** (descomposición).

5. Recién en **Apply** empieza a tocar código, **tarea por tarea**, y cierra con **Verify** (corre tests, lint, lo que corresponda).

> No tenés que aprender los nombres de las fases. El orquestador te las muestra a medida que avanza, vos solo aprobás en los **puntos de decisión** importantes (al elegir approach, al ver el diseño, antes de aplicar).

### ¿Querés forzar SDD a propósito?

Decile:

```
hazlo con sdd
```

o en inglés:

```
use sdd
```

El agente arranca el flujo aunque la tarea sea chica.

### ¿Querés saltearlo?

Decile:

```
sin sdd, hacelo directo
```

El agente respeta tu pedido y va al grano.

---

## Paso 5 · Engram trabajando en segundo plano

Mientras pasaban los pasos 3 y 4, Engram fue **guardando cosas** sin que vos hicieras nada:

- El contexto del `/sdd-init`.
- Las decisiones de la fase Design (por qué eligió tal patrón).
- Los bugs que aparecieron y cómo se resolvieron.
- Los artefactos de cada fase SDD.

Probá ver qué guardó:

```bash
engram tui
```

Vas a ver una interfaz visual con: proyectos → memorias → observaciones. Buscá tu proyecto, abrí una memoria, y mirá.

Desde la terminal, también:

```bash
engram search "endpoint users/me"
```

Te lista los resultados con su tipo (`architecture`, `decision`, `bug`, etc.).

---

## Paso 6 · Cerrar y volver a entrar (la prueba clave)

Salí del agente. Esperá un minuto. Volvé a abrirlo en el mismo proyecto y decile:

```
¿Qué hicimos la última vez?
```

Si el setup quedó bien, el agente te resume **con detalles concretos**: qué endpoint armaste, qué decisiones tomaron, qué quedó pendiente. **No te inventa**: lee de Engram.

Eso es lo que diferencia un chatbot de un compañero de trabajo. **Esa es la razón por la que existe Gentle-AI.**

---

## Paso 7 · (Opcional) Sincronizar memoria al repo

Si querés que la memoria de este proyecto **viaje con el repo** (otras máquinas, otros colaboradores):

```bash
engram sync
```

Esto crea/actualiza una carpeta `.engram/` con los datos exportados. Hacé commit:

```bash
git add .engram/
git commit -m "chore: sync engram memory"
```

En otra máquina, al clonar el repo:

```bash
engram sync --import
```

Y arrancás con la memoria ya cargada. Lo vemos con más detalle en la página 16 (caso: compartir memoria).

---

## Qué deberías estar sintiendo

Al terminar esta sesión, esto es lo que cambia respecto a un chatbot pelado:

| Antes (chatbot) | Después (agente configurado) |
| --- | --- |
| "Hola, ¿en qué te ayudo?" | "Estoy en tu proyecto Next.js + Vitest, listo." |
| Cada sesión empezás de cero. | Recuerda decisiones de hace días. |
| Mete cualquier patrón. | Usa el patrón que ya existe en tu código. |
| Tareas grandes = caos. | Tareas grandes = SDD organiza el caos. |
| Confías y cruzás los dedos. | Te explica el porqué y pide confirmación cuando toca algo serio. |

---

## Si algo no anduvo

### El agente respondió como "chatbot pelado"

Probablemente la persona/skills no se aplicaron. Probá:

```bash
gentle-ai sync           # re-inyecta lo último
```

Después cerrá y volvé a abrir el agente.

### `/sdd-init` no existe como slash command

Tu agente no soporta slash commands (Cursor, Claude Code antiguo, Codex). Es esperable. Decile:

```
hacé sdd-init en este proyecto
```

Funciona igual.

### Engram no encuentra el proyecto

```bash
engram projects list
```

Si aparecen variantes (`mi-app`, `My-App`, `mi-app-frontend`):

```bash
engram projects consolidate
```

Te guía para unificarlas.

### El skill registry no se ve actualizado

```bash
gentle-ai skill-registry refresh --force
```

Forzá la regeneración. Cerrá y abrí el agente.

---

## Resumen

- `/sdd-init` (o "hacé sdd-init") **una vez por proyecto** para que el agente entienda tu stack.
- Tareas chicas → el agente las hace directo, sin ceremonia.
- Tareas grandes → el agente propone SDD; vos aceptás o lo forzás con "hazlo con sdd".
- **Engram guarda solo**. Al volver al día siguiente, el agente **recuerda**.
- `engram tui` y `engram search` son tus ventanas a la memoria del agente.
- `engram sync` exporta la memoria al repo para compartirla con tu equipo.

---

## Siguiente paso

➡️ **[7 · Engram: memoria persistente](07-engram-memoria.md)** — entender por qué tu agente "se olvida" cosas sin Engram, ver con qué precisión recuerda, cómo navegar la TUI, sincronizar entre máquinas, y resolver conflictos de nombre.

---

← [Volver al índice](index.md) · ← [Anterior: Tu primera configuración](05-primera-configuracion.md)
