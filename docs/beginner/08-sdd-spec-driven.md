# 8 · SDD: Spec-Driven Development

← [Volver al índice](index.md) · ← [Anterior: Engram: memoria persistente](07-engram-memoria.md)

---

> **Idea central**: SDD es un **flujo guiado** para tareas medianas/grandes. En lugar de pedirle al agente "implementame esto" y rezar, le pedís que **explore, proponga, especifique, diseñe, descomponga, implemente y verifique** — en ese orden. Cada paso queda guardado en Engram. El agente lo activa solo cuando hace falta; vos no aprendés comandos.

---

## El problema que resuelve

Cuando una tarea es **chica** ("agregá un README"), pedirle al agente que la haga directo está perfecto.

Cuando una tarea es **mediana o grande** ("agregá autenticación", "migremos a Postgres", "rehagamos el módulo de pagos"), pedirla directo es una receta para el caos:

- El agente arranca a codear sin entender el código existente.
- Mezcla decisiones que deberían discutirse **antes** con detalles de implementación.
- Toca cinco archivos a la vez, ninguno bien.
- Si algo sale mal, no hay manera de saber **en qué paso** se rompió.

SDD parte ese caos en **pasos verificables**, cada uno con un artefacto guardado en memoria. Si algo falla, sabés exactamente dónde retomar.

---

## Las 9 fases (en una imagen mental)

No las vas a tipear. Vienen una atrás de la otra. Pero conocerlas te ayuda a entender qué te está mostrando el agente:

```
sdd-init       → ¿qué hay en este proyecto? (stack, convenciones, tests)
   │
sdd-explore    → ¿qué hay alrededor del cambio que querés? (archivos clave)
   │
sdd-propose    → propuesta: intención + scope + approach (¿avanzamos?)
   │
sdd-spec       → requirements y escenarios (qué tiene que cumplir)
   │
sdd-design     → decisiones técnicas y arquitectura (cómo se va a hacer)
   │
sdd-tasks      → descomposición en tareas concretas (orden de ataque)
   │
sdd-apply      → implementación, tarea por tarea (recién acá toca código)
   │
sdd-verify    → validación: tests, lint, criterios de aceptación
   │
sdd-archive    → sincroniza specs/deltas y archiva el cambio
```

Bonus:

- **`sdd-onboard`** — un walkthrough guiado del flujo completo sobre tu propio código. Útil para entenderlo "en serio" la primera vez.

> **Vos no memorizás esto.** El agente muestra qué fase está corriendo. La tabla está acá para que cuando veas `sdd-design` en el log no te asustes.

---

## Cuándo se activa SDD

### Automático: cuando el agente detecta tarea grande

El orquestador de SDD aplica reglas internas (basadas en cuántos archivos hay que tocar, cuánto contexto leer, si hay decisiones de arquitectura). Si la tarea es chica, **no lo activa**. Si es mediana/grande, te propone usarlo:

> *"Esta tarea toca routing, sesión y tests. Te propongo usar SDD: arranco con `sdd-explore` para mapear el código. ¿Avanzamos?"*

Vos aceptás o decís "no, hacelo directo".

### Forzado: cuando vos lo pedís

Diciendo cualquiera de estas frases:

- `use sdd`
- `hazlo con sdd`
- `usá sdd para esto`

El agente arranca el flujo aunque la tarea sea chica. Útil cuando **vos** sabés que la cosa se va a complicar aunque parezca trivial.

### Evitado: cuando lo querés saltear

- `sin sdd, hacelo directo`
- `no uses sdd, mete el cambio nomás`

El agente respeta. Bueno para hot-fixes obvios.

---

## Los "gates" — puntos donde vos decidís

SDD no es un tren bala. Hay **paradas explícitas** donde el agente te muestra el artefacto y espera tu aprobación antes de seguir:

| Después de… | Vos decidís… |
| --- | --- |
| `sdd-explore` | Si la lectura del código fue suficiente. |
| `sdd-propose` | Si la **dirección** (intención + approach) es la correcta. |
| `sdd-spec` | Si los **requirements** capturan lo que querés. |
| `sdd-design` | Si las **decisiones técnicas** son aceptables. |
| `sdd-tasks` | Si el **orden** y la **granularidad** te cierran. |
| `sdd-apply` (por tarea) | Si la implementación de **esa tarea** se aplica. |
| `sdd-verify` | Si el cambio está **listo** para mergearse. |

En cualquier punto podés decir:

- *"cambiá esto, después seguimos"* — el agente ajusta y vuelve a mostrar.
- *"volvé a una fase anterior"* — el agente retrocede sin perder lo bueno.

---

## Slash commands disponibles

Si tu agente soporta slash commands (OpenCode, Pi, Qwen, etc.), tenés atajos:

| Comando | Para qué sirve |
| --- | --- |
| `/sdd-init` | Bootstrap del proyecto (stack, convenciones, tests). |
| `/sdd-new <nombre-del-cambio>` | Arranca un cambio nuevo: explora + propone. |
| `/sdd-explore` | Solo la exploración (sin propuesta). |
| `/sdd-apply` | Aplica las tareas pendientes del cambio actual. |
| `/sdd-verify` | Verifica el cambio (tests, lint, criterios). |
| `/sdd-archive` | Cierra y archiva el cambio. |
| `/sdd-continue` | Retoma un cambio donde quedó (cross-session). |
| `/sdd-ff` | **Fast-forward**: corre las fases siguientes en orden sin pausas. |
| `/sdd-onboard` | Walkthrough guiado del flujo completo sobre tu repo. |

En agentes sin slash commands (Claude Code, Codex, Cursor), decilo en lenguaje natural: *"corré sdd-new para agregar autenticación con JWT"*. El agente lo entiende.

> `/sdd-ff` es útil cuando ya pasaste el design y querés que termine **sin pararte** en cada gate. Lo opuesto al modo "una pregunta por paso".

---

## Dónde quedan los artefactos

Cada fase guarda su salida en Engram con un `topic_key` predecible:

```
sdd/<nombre-del-cambio>/exploration
sdd/<nombre-del-cambio>/proposal
sdd/<nombre-del-cambio>/spec
sdd/<nombre-del-cambio>/design
sdd/<nombre-del-cambio>/tasks
sdd/<nombre-del-cambio>/apply
sdd/<nombre-del-cambio>/verify
sdd/<nombre-del-cambio>/archive
```

Esto significa que:

- Si cerrás la sesión a la mitad, **podés volver mañana** y `sdd-continue` retoma exactamente donde quedó.
- Si cambiás de máquina (con `engram sync`), también.
- Si pasaron tres meses y querés saber **por qué** elegiste cierto approach, lo buscás con `engram search`.

### Otros backends

El `artifact store mode` por defecto es `engram`, pero también existen:

- `openspec` — guarda los artefactos como archivos markdown en `openspec/` del repo (versionables en git).
- `hybrid` — los dos.
- `none` — sin persistencia (no recomendado salvo pruebas).

Lo que se usa por defecto en Gentle-AI es **engram**. La página 17 entra en detalle si te interesa.

---

## Single-mode vs multi-mode

- **Single-mode**: todas las fases corren con el **mismo modelo**. Es lo que hacen casi todos los agentes (Claude Code, Cursor, Codex, Windsurf, etc.). **Funciona perfecto**.
- **Multi-mode**: cada fase puede usar **un modelo distinto**. Soportado en **OpenCode**, **Kilo Code** y **Kiro IDE**.

Ejemplo de multi-mode útil:

| Fase | Modelo | Por qué |
| --- | --- | --- |
| `sdd-explore` | qwen3-30b (gratis) | Es leer mucho código; un modelo más barato alcanza. |
| `sdd-design` | claude-sonnet | Decisiones de arquitectura: queremos calidad alta. |
| `sdd-apply` | claude-haiku | Implementación mecánica: rápido y barato. |
| `sdd-verify` | claude-sonnet | Validación crítica: vale la pena el modelo bueno. |

Esto es **opcional**. Si recién empezás, ignoralo. Lo vemos a fondo en la **página 15 (caso: OpenCode con múltiples modelos)**.

---

## SDD vs trabajar "a mano" — ejemplo concreto

**Sin SDD, pedido directo:**

> Vos: "Agregá login con email y password."
> Agente: *(20 minutos después)* "Listo, agregué `/login`, hasheado con bcrypt, sesión en cookie."
> Vos: "¿Y el rate limiting? ¿Y los tests? ¿Por qué cookie y no JWT?"
> Agente: "No los hicimos. ¿Querés que los agregue?"
> Vos: *😩*

**Con SDD:**

> Vos: "use sdd, agregá login con email y password."
> Agente *(sdd-explore)*: "Veo que ya hay `/api/*` con un middleware de auth. Tu test setup es Vitest. No hay rate limiting actualmente."
> Agente *(sdd-propose)*: "Propongo: ruta `/api/auth/login`, bcrypt + cookie httpOnly, validar con Zod, rate limit de 5/min por IP. ¿Vamos?"
> Vos: "Sí, pero JWT en vez de cookie."
> Agente *(sdd-spec)*: requirements + escenarios.
> Agente *(sdd-design)*: decisiones técnicas, alternativas consideradas.
> Agente *(sdd-tasks)*: 6 tareas atómicas con orden.
> Agente *(sdd-apply)*: tarea 1… (te muestra el diff, aprobás), tarea 2… etc.
> Agente *(sdd-verify)*: corre tests, lint, repasa criterios.
> Vos: *🎉*

La diferencia: **discutiste el approach una vez al principio**, no a los 20 minutos cuando ya estaba todo escrito.

---

## ¿Cuándo NO usar SDD?

- **Cambios triviales**: typos, formato, una línea de log, renombrar un símbolo.
- **Hot-fixes** que conocés al dedillo.
- **Exploración pura** donde solo querés que el agente lea código y te explique.
- **Prototipos descartables** donde la velocidad importa más que la calidad.

En todos esos casos, `sin sdd, hacelo directo` es la respuesta correcta.

---

## Errores comunes

### "El agente arrancó SDD para algo trivial"

Decile `sin sdd` desde el primer turno. O ajustá tu prompt para que se entienda que es chico: *"cambio rápido en tal archivo"*.

### "El agente no propone SDD y se mete en el barro"

Forzalo: `use sdd`. Si recurrentemente no lo activa cuando debería, puede ser que el proyecto no tenga contexto cargado. Probá:

```bash
gentle-ai skill-registry refresh
```

Y dentro del agente: *"corré sdd-init"*.

### "Quedó a la mitad y no sé cómo retomar"

```
/sdd-continue
```

O decile: *"retomá el último cambio SDD donde lo dejamos"*. El agente lee Engram y retoma.

### "Se atascó en una fase pidiendo aprobación tras aprobación"

Pasá a fast-forward:

```
/sdd-ff
```

O: *"hacé fast-forward, no me consultes hasta verify"*.

---

## Resumen

- SDD = **explorar → proponer → especificar → diseñar → tareas → aplicar → verificar → archivar**.
- El agente lo **activa solo** en tareas grandes. Vos lo forzás con `use sdd` o lo evitás con `sin sdd`.
- Hay **gates de aprobación** entre fases: vos decidís, vos cortás, vos reescribís.
- Cada fase guarda su artefacto en **Engram** con `topic_key` estable → podés retomar en cualquier momento.
- **No tenés que aprender comandos**. Los slash commands son atajos opcionales.
- Multi-mode (modelo distinto por fase) existe pero es opcional (OpenCode/Kilo/Kiro).

---

## Siguiente paso

➡️ **[9 · Skills: capacidades curadas](09-skills.md)** — qué es una skill, cómo se descubren mediante el *Skill Registry*, y recorrido por las skills incluidas (`branch-pr`, `chained-pr`, `comment-writer`, `issue-creation`, `work-unit-commits`, `cognitive-doc-design`).

📚 Doc de referencia (avanzado): **[docs/intended-usage.md](../intended-usage.md)** — comportamiento del orquestador SDD, reglas de delegación y modos single/multi.

---

← [Volver al índice](index.md) · ← [Anterior: Engram: memoria persistente](07-engram-memoria.md)
