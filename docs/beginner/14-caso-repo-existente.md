# 14 · Caso: sumarse a un repo existente

← [Volver al índice](index.md) · ← [Anterior: Caso: arrancar un proyecto nuevo](13-caso-proyecto-nuevo.md)

---

> **Objetivo del caso**: clonás un repo de un equipo o de un proyecto que **vos no escribiste**, y necesitás **ponerlo a producir rápido sin romper nada**. Vas a usar el agente para entender el código antes de tocarlo, descubrir las convenciones del equipo, y abrir tu primer PR con seguridad.
>
> **Diferencia clave con la página 13**: acá **no diseñás un sistema**, **lo descifrás**. El protagonista es `sdd-explore`, no `sdd-propose`.
>
> **Tiempo estimado**: 30-60 minutos.

---

## Lo que vas a tener al final

- Una **mente cargada** con el stack, la estructura, las convenciones y los flujos del repo.
- **Engram poblado** con esa información, lista para acompañarte mañana.
- Una **rama hecha**, un cambio chico aplicado, **tests pasando**, y un **PR abierto** siguiendo las convenciones del equipo.

Sin haber tenido que leer 50 archivos a mano.

---

## Paso 0 · Pre-requisitos

- Gentle-AI instalado (página 4) y agente configurado (página 5).
- Acceso al repo (ya clonado en tu máquina).
- Una task chiquita asignada, o una idea concreta de qué cambio querés hacer. Si todavía no tenés ni eso, igual el flujo arranca con la **exploración guiada** y al final aterriza en una task.

---

## Paso 1 · Clonar y mirar lo básico (vos, no el agente)

Antes de abrir el agente, hacé **3 cosas a mano**. Te ahorran tiempo después.

```bash
git clone <repo-url>
cd <repo>
```

Después:

```bash
# ¿Qué archivos clave hay en la raíz?
ls -la

# ¿Hay memoria compartida?
ls .engram/ 2>/dev/null && echo "✓ tiene memoria compartida" \
                       || echo "✗ no hay .engram, vamos a construirlo"

# ¿Hay archivos de contexto para agentes?
ls AGENTS.md CLAUDE.md .cursorrules openspec/ 2>/dev/null
```

Estas tres lecturas te dicen si **el equipo ya pensó en esto** o si sos el primero. Las dos situaciones son válidas; cambian un poco lo que sigue.

---

## Paso 2 · Si hay `.engram/`, importala primero

Si el `ls .engram/` dio resultado, hay memoria compartida del equipo. **Importala antes de abrir el agente**:

```bash
engram sync --import
```

Esto carga en tu base local todas las decisiones, bugs resueltos, convenciones y arquitectura que el equipo fue acumulando. Cuando abras el agente, va a arrancar con **ese contexto cargado**.

Verificá:

```bash
engram projects list
# tu repo debería aparecer con N observaciones
```

Si no aparece, mirá si el `git remote -v` está bien (Engram detecta el proyecto por ahí — página 7).

---

## Paso 3 · Abrir el agente

```bash
claude     # o el que uses
```

Si **importaste memoria** en el paso 2, el agente puede arrancar reconociendo el proyecto:

> *"Veo que es el repo `frontend-store`. Tengo memoria previa: stack Next 14 + TS, tests con Vitest, ESLint estricto. ¿En qué te puedo ayudar?"*

Si **no había memoria**, va a arrancar más neutro. No pasa nada, lo armamos juntos.

---

## Paso 4 · `sdd-init` adaptado a "no escribí esto"

Pedido:

```
hacé sdd-init. quiero entender bien este proyecto antes de tocar nada.
```

> Con slash commands: `/sdd-init`.

A diferencia de la página 13 (donde el repo estaba vacío y el agente te preguntaba), acá el agente **escanea** lo que ya hay:

- `package.json` / `pyproject.toml` / `go.mod` / etc. → stack y dependencias.
- `tsconfig.json`, `eslint.config.js`, `prettier.config.js` → linters y reglas.
- `vitest.config.ts`, `jest.config.js`, `pytest.ini` → framework de tests.
- `README.md`, `CONTRIBUTING.md` → instrucciones del equipo.
- `AGENTS.md`, `CLAUDE.md`, `.cursorrules` → convenciones específicas para agentes.
- `openspec/` o `.engram/` → trabajo previo SDD.

El agente te devuelve un resumen estructurado y **guarda todo en Engram** con `topic_key: sdd-init/<proyecto>`.

Si activaste Strict TDD durante la instalación (página 5), va a confirmar si lo activa para este repo.

---

## Paso 5 · Exploración guiada del código

Acá es donde brilla `sdd-explore`. Pedido:

```
quiero entender la arquitectura. usá sdd-explore para mapear:
- cómo está organizado el código (src/, app/, lib/, etc.)
- por dónde entra el flujo principal
- dónde están los tests y cómo se estructuran
- qué convenciones de naming y commits sigue el equipo
```

El orquestador delega a un sub-agente `sdd-explore` (página 10) con contexto limpio. El sub-agente:

1. Recorre la estructura.
2. Lee los archivos críticos (entrypoints, configs, ejemplos representativos).
3. Identifica patrones (¿hexagonal? ¿feature-first? ¿layered?).
4. Te devuelve un resumen estructurado.

Ejemplo de salida que podés ver:

```
status: ok
executive_summary:
  - Arquitectura: Next 14 App Router, feature-first
  - Entrada: src/app/[locale]/(routes)
  - State: Zustand, stores en src/stores/
  - Data: tRPC en src/server/api/routers/
  - Tests: Vitest + Testing Library, co-locados (*.test.tsx)
  - Convenciones: feat:/fix:/chore:/docs:, branches feat/xxx
  - PRs: requieren issue link, descripción, checklist
artifacts:
  - engram: sdd-explore/<proyecto>/overview
next_recommended:
  - lectura puntual de un router tRPC representativo
  - lectura del setup de auth (NextAuth + sesiones)
```

**Aprobalo**. Eso queda guardado en Engram para siempre.

> **Tip**: hacé el resumen **mientras tomás café**. Es lectura, no escritura. El componente `permissions` ni te molesta porque es todo `read-only`.

---

## Paso 6 · Profundizar en lo que vas a tocar

Ya sabés el panorama general. Ahora pedí exploración **focalizada** en lo que vas a cambiar. Ejemplo: te asignaron mejorar el endpoint de checkout.

```
usá sdd-explore otra vez, pero focalizado en el flujo de checkout.
- archivos involucrados
- estado actual
- tests existentes
- decisiones que el equipo ya tomó al respecto (revisá engram)
```

El agente:

1. Mira lo que ya guardó (`mem_search "checkout"`).
2. Lee solo los archivos relevantes.
3. Te devuelve un mapa más detallado.

> Notá que el segundo `sdd-explore` **ya partió de la base del primero**. No re-lee todo el proyecto: usa la memoria guardada.

---

## Paso 7 · Una tarea chiquita primero (no la que te asignaron)

**Antes de meter mano en la feature grande**, hacé un cambio trivial. Te sirve para:

- Validar que tus tests pasan localmente.
- Validar que el lint pasa.
- Validar que sabés cómo hacer un PR para este repo.
- Calibrar al agente con las convenciones reales.

Ejemplo:

```
sin SDD, hagamos un cambio chiquito para validar mi setup:
- arreglá un typo o agregá un comentario en algún lado obvio
- asegurate de que tests, lint y build pasen
- preparalo para hacerle PR
```

El agente:

1. Hace el cambio chico.
2. Corre `npm test`, `npm run lint`, `npm run build` (o lo que corresponda).
3. Te confirma que pasa todo.

---

## Paso 8 · Tu primer PR — `branch-pr` en acción

Pedido:

```
abrime un PR usando branch-pr. seguí las convenciones del equipo que
ya tenés en memoria.
```

El agente reconoce la skill `branch-pr` (página 9) y aplica las convenciones que detectó en el paso 5:

```
✓ branch: docs/fix-typo-readme (feat-style branch naming)
✓ commit: docs: fix typo in README quick start section
✓ PR title: docs: fix typo in README quick start section
✓ PR body: usando el template del repo (.github/PULL_REQUEST_TEMPLATE.md)
  - Issue link: N/A (cambio trivial)
  - Description: ...
  - Checklist: tests pasan ✓ lint pasa ✓ build pasa ✓
```

Si el componente `permissions` está activo, te pide confirmación antes del `git push` (página 11). Confirmás.

**Felicitaciones, tu primer PR al repo está abierto.** Es chico, es seguro, y validaste todo el pipeline.

---

## Paso 9 · La feature real (SDD completo)

Recién ahora, con el setup probado y la arquitectura clara, atacás la tarea de verdad. Pedido:

```
use sdd. mejorar el flujo de checkout para soportar cupones de descuento.
```

A diferencia de la página 13 (proyecto nuevo), acá `sdd-explore` ya tiene la mitad del trabajo hecho (paso 6). El flujo va a ser más rápido:

- **Explore**: enfocado en lo que falta saber.
- **Propose**: tomando en cuenta los patrones que ya conoce.
- **Spec**: con escenarios basados en cómo el equipo escribe specs.
- **Design**: respetando arquitectura existente.
- **Tasks → Apply → Verify**: como siempre.

> Aprobá cada gate como en la página 13. Sigue siendo **tu** decisión qué dirección tomar; el agente solo te muestra las opciones con tradeoffs.

---

## Paso 10 · Sincronizar lo que aprendió

Si el repo no traía `.engram/` y vos sos el primero que carga memoria, **commiteala** para el equipo:

```bash
engram sync
git add .engram/
git commit -m "chore: bootstrap engram memory for the repo"
```

Si ya traía `.engram/`, igual conviene sincronizar lo nuevo:

```bash
engram sync
git add .engram/
git commit -m "chore: sync engram memory with checkout discoveries"
```

Tu equipo te lo va a agradecer (literalmente: la próxima persona arranca con todo cargado).

---

## Mini-checklist al sumarte a un repo

Esta es la versión condensada que podés guardar:

```
[ ] git clone
[ ] ls .engram/ → si existe: engram sync --import
[ ] abrir agente
[ ] /sdd-init o "hacé sdd-init, quiero entender el proyecto"
[ ] /sdd-explore overview (arquitectura, convenciones)
[ ] /sdd-explore focalizado en lo que vas a tocar
[ ] tarea chiquita de validación + PR con branch-pr
[ ] feature real con use sdd
[ ] engram sync + commit de .engram/
```

---

## Diferencias clave con el caso 13

| | Repo nuevo (página 13) | Repo existente (página 14) |
| --- | --- | --- |
| Hay código previo | No. | Sí, ése es **el punto**. |
| Skill estrella | `sdd-propose` (diseñás) | `sdd-explore` (descifrás) |
| Convenciones | Vos las definís en `sdd-init`. | El agente las **detecta** en `sdd-init`. |
| Primera acción | Bootstrap. | Cambio chiquito de validación. |
| Engram al inicio | Empieza vacío. | Importás si hay `.engram/`. |
| Riesgo principal | "Sobre-diseñar". | "Romper algo que no entendés". |

---

## Errores comunes en este flujo

### "El agente arranca a editar código antes de entender el repo"

Cortalo:

```
pará. antes de tocar nada, corré sdd-explore para mapear la arquitectura.
```

Y/o reforzá con la **fresh review rule** (página 10):

```
no asumas nada. leé el código relevante antes de editar.
```

### "El agente usa patrones de otro proyecto, no de este"

Si confunde convenciones (por ejemplo, te propone un patrón React 18 cuando el repo es React 19), forzá la lectura de docs vivas:

```
mirá en Context7 cómo se hace esto en la versión exacta que usa el proyecto.
```

Y si tenés Engram cargada con las convenciones, recordáselo:

```
revisá la memoria del proyecto antes de proponer. ya hay decisiones tomadas.
```

### "Importé `.engram/` pero `engram projects list` no lo muestra"

Tres causas frecuentes:

1. **Sin `git remote`**: agregalo. Si el repo tiene varios, Engram usa el `origin`.
2. **Nombre con drift** (`my-app` vs `My-App`): `engram projects consolidate` (página 7).
3. **Carpeta `.engram/` malformada**: pedile a quien la generó que la regenere con `engram sync`.

### "El PR me lo arma pero no respeta el template del repo"

Probablemente el agente no leyó `.github/PULL_REQUEST_TEMPLATE.md`. Decile:

```
leé .github/PULL_REQUEST_TEMPLATE.md y rearmá el PR siguiendo ese formato.
```

Una vez que lo guarde en Engram, no debería volver a fallar.

### "Tengo miedo de mergear algo que no entiendo"

Bien, eso significa que prestás atención. Estrategias:

- Activá la **fresh review rule** explícitamente: *"hacé judgment-day sobre este diff antes de mergear"*.
- Pedile que **explique el cambio en términos del flujo del repo**: *"explicame en 5 líneas qué efecto tiene este cambio en el flujo de checkout, asumiendo que no leí los otros archivos"*.
- Si todavía dudás: **no mergees**. Pedile review a alguien del equipo. El agente no reemplaza al review humano cuando importa la confianza.

---

## Resumen

- Antes de abrir el agente: `ls` rápido para ver si hay **`.engram/`**, **`AGENTS.md`**, **`openspec/`**.
- Si hay `.engram/`: **`engram sync --import`** primero.
- **`sdd-init`** detecta stack y convenciones automáticamente (no las inventás).
- **`sdd-explore`** general → **`sdd-explore`** focalizado en lo que vas a tocar.
- **Primera tarea trivial** + **PR con `branch-pr`** para validar tu pipeline.
- Recién después, la **feature real con SDD completo**.
- Al final: **`engram sync`** + commit de `.engram/` para tu equipo.
- Cuando dudás: **no mergees**, pedí review humano.

---

## Siguiente paso

➡️ **[15 · Caso: OpenCode con múltiples modelos](15-caso-opencode-profiles.md)** — asignar un modelo barato a exploración y uno potente a diseño. Cómo crear y cambiar perfiles SDD (`gentle-orchestrator` y derivados).

---

← [Volver al índice](index.md) · ← [Anterior: Caso: arrancar un proyecto nuevo](13-caso-proyecto-nuevo.md)
