# 20 · Cómo contribuir siendo principiante

← [Volver al índice](index.md) · ← [Anterior: ¿Qué leer ahora?](19-siguientes-pasos.md)

---

> **Idea central**: Gentle-AI tiene un workflow estricto pero **claro**. Si seguís los pasos en orden, contribuir es **una experiencia agradable**, incluso si es tu primer PR open source. Esta página te lleva de "tengo una idea" a "mergeado" sin saltarte ningún paso silencioso.

---

## Por qué contribuir vale la pena

Tres razones:

1. **Aprendés** mirando cómo otros estructuran código, tests y reviews.
2. **Mejorás algo que usás**: el bug que te molesta lo arreglás de raíz.
3. **Sumás tu nombre** a un proyecto que tiene comunidad activa. En tu próximo trabajo, "contribuí a Gentle-AI" es un line item real.

No tenés que escribir Go ni saber de TUIs para ayudar. Hay tres formas de contribuir:

- **Código** — features, fixes, refactors.
- **Documentación** — como esta guía. Vale tanto como el código.
- **Skills** — agregar skills nuevas al ecosistema (Gentleman-Skills repo).

---

## La regla más importante: **issue-first**

Antes de empezar a escribir código, **abrí un issue**. **Sin excepciones**.

> Los PRs sin issue aprobado **se rechazan automáticamente por CI**. No hay forma de saltearse esto.

El flujo es:

```text
1. Abrís un issue (bug o feature)
   │
   ▼
2. Un maintainer agrega el label "status:approved"
   │
   ▼
3. Comentás en el issue: "lo agarro"
   │
   ▼
4. Recién ahí empezás a codear
   │
   ▼
5. Abrís el PR linkeado al issue (Closes #N)
```

Razones de esta regla:

- **Evita duplicar trabajo**: alguien podría estar haciendo lo mismo.
- **Valida la idea antes que el código**: te ahorrás reescribir si el approach no es el que esperaban.
- **Tracking de scope**: el issue es el contrato; el PR cumple ese contrato.

---

## Tu primer issue: dónde encontrarlo

### Si ya tenés una idea propia

Abrí uno nuevo con el template correcto:

- 🐛 **[Bug report](https://github.com/Gentleman-Programming/gentle-ai/issues/new?template=bug_report.yml)** — para algo roto.
- ✨ **[Feature request](https://github.com/Gentleman-Programming/gentle-ai/issues/new?template=feature_request.yml)** — para algo nuevo.

Esperá que un maintainer le ponga `status:approved`.

### Si querés agarrar algo que ya existe

Filtros útiles en GitHub:

```
# Issues fáciles para empezar
is:issue is:open label:"good first issue"

# Issues aprobados (listos para agarrar)
is:issue is:open label:"status:approved"

# Bugs aprobados
is:issue is:open label:"status:approved" label:"type:bug"

# Documentación
is:issue is:open label:"type:docs"
```

[Link directo a issues abiertos](https://github.com/Gentleman-Programming/gentle-ai/issues).

> **Tip**: empezá con un `type:docs` o `type:chore`. Te familiarizás con el flujo de PR sin la presión de un cambio crítico.

---

## Sistema de labels (qué vas a ver y qué significan)

### Type labels (van en PRs)

| Label | Cuándo aplica |
| --- | --- |
| `type:bug` | Arreglo de bug. |
| `type:feature` | Feature nueva o mejora. |
| `type:docs` | Solo documentación. |
| `type:refactor` | Refactor sin cambio de comportamiento. |
| `type:chore` | Build, CI, tooling. |
| `type:breaking-change` | Cambio que rompe compatibilidad. |

Exactamente **un** `type:*` por PR. CI lo verifica.

### Status labels (van en issues)

| Label | Significado |
| --- | --- |
| `status:needs-review` | Recién abierto, esperando triage. |
| `status:approved` | **Listo para que alguien lo agarre.** |
| `status:in-progress` | Hay alguien trabajándolo. |
| `status:blocked` | Bloqueado por otro issue. |
| `status:wont-fix` | Fuera de scope. |

### Priority labels

`priority:critical` / `high` / `medium` / `low`. Solo informativo: el maintainer prioriza, vos no tenés que pelearla.

### Size labels (van en PRs grandes)

`size:exception` — solo si un maintainer aprueba un PR por encima del límite de 400 líneas.

---

## Setup local para contribuir código

Si vas a tocar código (no solo docs), necesitás:

```bash
# Prerrequisitos
go version       # 1.24 o superior
docker --version # para E2E tests
git --version

# Clonar
git clone https://github.com/Gentleman-Programming/gentle-ai.git
cd gentle-ai

# Compilar
go build -o gga .

# Correr local
./gga
```

> El binario local se llama `gga` por la convención del proyecto.

---

## Correr los tests

### Unit tests (rápidos)

```bash
go test ./...                  # todo
go test ./internal/tui/...     # solo un paquete
go test -v ./...               # con output detallado
```

Tienen que pasar **antes** de abrir el PR. CI los corre igual, pero te ahorrás el ida y vuelta.

### E2E tests (lentos, requieren Docker)

```bash
cd e2e
chmod +x docker-test.sh
./docker-test.sh
```

Levanta contenedores Ubuntu y Arch para simular instalaciones reales. Lleva unos minutos.

### Sobre Windows

Algunos tests requieren `SeCreateSymbolicLinkPrivilege` (crear symlinks). En Windows se **skipean automáticamente** si no tenés el permiso. Para correrlos completos:

- **Activá Developer Mode** (Settings → System → For developers).
- **O abrí terminal como Administrator**.

En Linux/macOS no hace falta nada extra.

---

## Convenciones de commits (Conventional Commits)

Tus commits **tienen que** seguir esta pinta:

```text
<tipo>(<scope opcional>)!: <descripción>

[cuerpo opcional]

[footer opcional]
```

### Tipos permitidos

| Tipo | Para qué |
| --- | --- |
| `feat` | Feature nueva. |
| `fix` | Arreglar bug. |
| `docs` | Solo docs. |
| `refactor` | Cambio sin alterar comportamiento. |
| `chore` | Mantenimiento / deps / tooling. |
| `style` | Formato / linters. |
| `perf` | Mejora de performance. |
| `test` | Tests nuevos o actualizados. |
| `build` | Sistema de build / deps externas. |
| `ci` | Config de CI. |
| `revert` | Revertir un commit anterior. |

### Ejemplos válidos

```text
feat(tui): add progress bar to installation steps
fix(agent): correct Claude Code detection on macOS
docs: update contributing guide
chore(deps): bump bubbletea to v0.26
test(installer): add coverage for catalog step execution
```

### Breaking changes

Agregás `!` después del tipo/scope **y** un footer `BREAKING CHANGE:`:

```text
feat(cli)!: rename --config flag to --config-file

BREAKING CHANGE: the --config flag has been renamed to --config-file.
Update your scripts and aliases accordingly.
```

Esto mapea al label `type:breaking-change`.

---

## Naming de ramas

Tu rama también tiene formato obligatorio:

```text
<tipo>/<descripcion-corta-con-guiones>
```

**Reglas**:

- Todo en minúsculas.
- Separadores: guiones, puntos, o guiones bajos. **No espacios, no mayúsculas.**
- Descripción corta y descriptiva.

**Ejemplos válidos**:

```
feat/user-login
fix/crash-on-startup
docs/api-reference
ci/add-e2e-job
docs/beginner-page-20
```

---

## El presupuesto de 400 líneas

Cada PR tiene un **límite de 400 líneas cambiadas** (`additions + deletions`). Lo verifica un check automático.

**Por qué**: un PR de 400 líneas se revisa en **~60 minutos** sin fatiga del reviewer. Más que eso, la calidad de review baja.

### Si tu cambio se pasa

Tres caminos:

1. **Partir en stacked PRs** — varios PRs encadenados, cada uno revisable por sí solo (la skill `chained-pr` te ayuda — página 9).
2. **Work-unit commits** — agrupá commits por unidad de trabajo (no por tipo de archivo). La skill `work-unit-commits` también te ayuda.
3. **Pedir `size:exception`** — solo cuando un maintainer acepta que el diff grande es inevitable (vendor code, migraciones, generated files).

---

## Antes de abrir el PR (checklist)

Antes de apretar "Create pull request":

- [ ] El issue está abierto y con `status:approved`.
- [ ] La rama sigue la convención (`feat/...`, `fix/...`, etc.).
- [ ] Los commits siguen Conventional Commits.
- [ ] El PR está **al o por debajo de 400 líneas** (o tenés `size:exception` aprobado).
- [ ] Los commits están organizados por unidad de trabajo.
- [ ] `go test ./...` pasa local.
- [ ] `./e2e/docker-test.sh` pasa local (si tocaste algo que lo amerite).
- [ ] Te leíste el diff completo una vez (self-review).

---

## El PR en sí

### Título

Mismo formato que un commit Conventional:

```text
feat(tui): add keyboard shortcut help overlay
fix(agent): handle missing HOME env var gracefully
```

### Body

Tiene que incluir el link al issue. Una de estas frases:

```text
Closes #42
Fixes #42
Resolves #42
```

Sin eso, CI rechaza el PR.

Ejemplo de body completo:

```markdown
## Summary

Adds a `Ctrl+H` overlay that lists keyboard shortcuts in the TUI.

## Why

Closes #42 — new users struggle to discover shortcuts after install.

## How

- New screen `screens/help_overlay.go` rendered on `Ctrl+H` from any screen.
- Toggle handled in `app/app.go`.
- Tests in `screens/help_overlay_test.go` cover open/close and content.

## Test plan

- [x] go test ./internal/tui/...
- [x] manual: open TUI, press Ctrl+H, navigate with j/k, close with esc.
```

> Sí: incluí un *test plan*. Ahorra ida y vuelta con el reviewer.

---

## Checks automáticos del CI

Cuando abrís el PR, GitHub Actions corre:

| Check | Verifica |
| --- | --- |
| **Check PR Cognitive Load** | ≤ 400 líneas cambiadas (o `size:exception`). |
| **Check Issue Reference** | Body contiene `Closes/Fixes/Resolves #N`. |
| **Check Issue Has status:approved** | El issue linkeado fue aprobado. |
| **Check PR Has type:* Label** | Exactamente un `type:*` aplicado. |
| **Unit Tests** | `go test ./...` pasa. |
| **E2E Tests** | `./e2e/docker-test.sh` pasa. |

**Todos** tienen que pasar para mergear. Si uno falla, **leé el log** y corregí: re-pushear a la misma rama re-corre el CI.

---

## Cómo se ve un review típico

El maintainer va a:

1. **Leer el código** del PR.
2. **Dejar comentarios** — usando la skill `comment-writer` (página 9), suelen ser **directos y cálidos**: "esto rompe X porque Y; sugiero Z".
3. **Pedir cambios** si hace falta — vos hacés commits adicionales, no `--force` push.
4. **Aprobar** y mergear.

Tu trabajo durante el review:

- Respondé los comentarios con el mismo tono (directo y respetuoso).
- Si discordás con una sugerencia, **explicá por qué** con argumento técnico. No tenés que aceptar todo.
- Si la sugerencia es razonable, aplicala y comentá "done in <commit-sha>".
- Si el reviewer no responde por días, podés mencionarlo gentilmente.

---

## Contribuir docs (esta es una opción real)

Si Go no es lo tuyo o no te animás todavía, **la documentación necesita ayuda siempre**:

- Páginas confusas que se pueden reescribir.
- Ejemplos que se pueden agregar.
- Errores ortográficos o de traducción.
- Diagramas que ayudarían.

El flujo es **idéntico** al de código: issue-first, label `type:docs`, PR con el cambio, los checks corren (sin tests de Go, claro).

**Ejemplo de issue para docs**:

> *"La página `docs/beginner/04-instalacion.md` no menciona qué hacer si Homebrew falla con un error de `Cellar permissions`. Propongo agregar un caso al troubleshooting."*

Eso es un issue válido. Probablemente lo aprueben rápido.

---

## Contribuir skills

Si querés agregar skills (no solo usar las que vienen), el repo principal de skills "de codear" es:

🔗 **[Gentleman-Programming/Gentleman-Skills](https://github.com/Gentleman-Programming/Gentleman-Skills)**

Allá vive React, Angular, TypeScript, Tailwind, Zod, Playwright, etc. Si querés sumar una skill para tu framework favorito, ese es el repo.

Para escribir una skill bien:

- Usá la skill **`skill-creator`** dentro de tu agente.
- Seguí **[docs/skill-style-guide.md](../skill-style-guide.md)**.

---

## Anti-patterns: cosas que NO hacer

### ❌ Abrir un PR sin issue aprobado

CI lo rechaza. **Sin excepciones.**

### ❌ Hacer un PR de 800 líneas porque "es todo necesario"

Lo van a rechazar. Aprendé a partir en stacked PRs. Es **una habilidad**, no una traba.

### ❌ Force-pushear durante el review

Borra el historial de cambios que el reviewer estaba siguiendo. Hacé commits nuevos. Si necesitás reescribir historia por algún motivo (ej. eliminar un secreto pegado por error), avisalo en el PR.

### ❌ Editar `package.json`, `go.mod`, lockfiles "porque sí"

Cambios de deps van con `chore(deps)` y suelen necesitar discusión en el issue. No bumpees a la última versión solo porque podés.

### ❌ "Hagamos un refactor mientras estoy acá"

Un PR, una idea. Si ves un refactor que vale la pena, abrí **otro issue** para él. La regla de los 400 líneas existe por algo.

### ❌ Empezar a codear "para ver" sin esperar el `status:approved`

Si el maintainer no aprueba la dirección, perdiste tu tarde. Esperá la aprobación. Suele tardar poco.

---

## Code of Conduct (corto y directo)

- **Crítica al código, no a la persona.** "Esta función tiene un bug" ≠ "vos tenés un bug".
- **Constructivo en reviews.** "Sugiero" suena distinto a "esto está mal".
- **Bienvenida a los newcomers.** Si alguien hace su primer PR, ayudalo a llegar al merge.

Faltas serias pueden costar la participación en el proyecto.

---

## Preguntas, ideas, charla general

Para **preguntas y discusiones generales** (no bugs ni features):

🔗 **[GitHub Discussions](https://github.com/Gentleman-Programming/gentle-ai/discussions)**

No abras issues para preguntar. Los issues son para trabajo concreto. Las Discussions son para conversar.

---

## Tu primer PR ideal (un guión)

Si querés un camino seguro de "0 a primer PR mergeado":

1. **Encontrá un `good first issue`** o un `type:docs` open.
2. **Comentá** en él: *"Hola, soy nuevo. ¿Lo puedo agarrar?"*. Un maintainer responde.
3. **Cloná el repo**, abrí una rama `docs/<descripcion>` o `fix/<descripcion>`.
4. **Hacé el cambio chico**.
5. **Corré `go test ./...`** localmente.
6. **Commiteá** con conventional commits.
7. **Pusheá** y abrí el PR con `Closes #<numero>`.
8. **Esperá CI** — si algo falla, leé el log y arreglá.
9. **Responde a los comentarios** del review.
10. **Mergeado**. 🎉

Después de eso, el resto fluye. Cada contribución es más fácil que la anterior.

---

## Resumen

- **Issue-first, sin excepciones.** Sin issue aprobado, CI rechaza.
- Buscá **`good first issue`** y **`type:docs`** para empezar suave.
- PRs ≤ **400 líneas**. Más que eso, partilo.
- **Conventional Commits + branch naming** son obligatorios.
- Todos los checks de CI tienen que pasar.
- **Documentación cuenta tanto como código**. No es contribución de segunda.
- Code of Conduct: critica al código, no a la persona.
- Para charla general: **GitHub Discussions**, no issues.

---

## 🎉 Llegaste al final de la guía para principiantes

20 páginas. De *"acabo de oír de Gentle-AI"* a *"sé cómo contribuir al proyecto"*. 

Lo que aprendiste:

- **Parte 1** — qué es Gentle-AI, glosario, agentes.
- **Parte 2** — instalación, configuración, primera sesión.
- **Parte 3** — Engram, SDD, skills, sub-agentes, persona, MCP.
- **Parte 4** — proyecto nuevo, repo existente, multi-modelo, equipo.
- **Parte 5** — actualizar, backups, troubleshooting, qué leer ahora.
- **Parte 6** — contribuir.

A partir de acá, el camino es **práctica**. Usá Gentle-AI en proyectos reales, fijate cuándo el agente sorprende (para bien o para mal), volvé a las páginas relevantes cuando necesites refrescar algo, y considerá contribuir algo de vuelta — aunque sea un fix de typo en estos docs.

**Gracias por leer.**

---

← [Volver al índice](index.md) · ← [Anterior: ¿Qué leer ahora?](19-siguientes-pasos.md)
