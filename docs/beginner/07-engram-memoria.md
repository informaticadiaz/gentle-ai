# 7 · Engram: memoria persistente

← [Volver al índice](index.md) · ← [Anterior: Tu primera sesión real](06-primera-sesion.md)

---

> **Idea central**: Engram es lo que convierte a tu agente de "amnésico carismático" en "compañero de trabajo que recuerda lo que hablaron ayer". Funciona **solo**; vos no tenés que guardar nada a mano. Esta página te explica **qué guarda, cómo verlo y cómo compartirlo**.

---

## El problema que resuelve

Sin Engram, una sesión típica es así:

> **Vos** (martes): "Decidimos que el endpoint `/users/me` no devuelva el campo `password_hash` ni en admin. Acordate."
> **Agente**: "Perfecto."
>
> **Vos** (miércoles): "Agregá un endpoint similar para `/users/:id`."
> **Agente** (sin Engram): "Listo, te devuelve todos los campos del usuario, incluido `password_hash`."

Estás de nuevo explicando lo mismo. Con Engram, el agente **lee la decisión de ayer** antes de escribir código, y arranca con:

> "Veo en memoria que decidimos no exponer `password_hash` ni en admin. Aplico la misma regla acá."

Esa es toda la magia. **Memoria que persiste entre sesiones, días, máquinas y agentes.**

---

## Qué guarda Engram (y qué no)

Engram guarda **observaciones**: pedacitos de información que valen la pena para el futuro. Cada observación tiene un **tipo**:

| Tipo | Ejemplo |
| --- | --- |
| `architecture` | "El proyecto usa el patrón hexagonal con `internal/domain` y `internal/adapter`." |
| `decision` | "Decidimos no exponer `password_hash` nunca, ni a admins." |
| `bug` | "El test E2E fallaba por timezone; se arregló forzando UTC en `setup.ts`." |
| `discovery` | "El router de Next 14 maneja `params` como Promise; hay que `await`." |
| `convention` | "Conventional commits con scope. PRs siempre desde una rama `feat/...`." |
| `artifact` | "Salida estructurada de la fase `sdd-design` para el feature X." |

**No guarda**:

- Todo el chat literal (sería ruido).
- Contenido sensible que no le marques. **Cuidado**: si le pegás un `.env` o una API key en el chat, Engram puede capturarlo. Es tu responsabilidad no compartir secretos.
- Archivos binarios, capturas, etc.

---

## Cómo guarda (sin que vos hagas nada)

Tu agente tiene acceso a Engram vía **MCP** (Model Context Protocol). Las herramientas que usa por debajo:

| Herramienta MCP | Para qué la usa el agente |
| --- | --- |
| `mem_save` | Guardar una decisión, bug, descubrimiento, convención. |
| `mem_search` | Buscar memorias relacionadas con la tarea actual. |
| `mem_context` | Cargar contexto reciente al arrancar la sesión. |
| `mem_session_summary` | Guardar un resumen al cerrar la sesión. |
| `mem_get_observation` | Traer el contenido completo de una memoria. |
| `mem_save_prompt` | Atar tu prompt al artefacto que generó el agente. |

**Vos casi nunca llamás estas herramientas.** El agente las dispara solo cuando detecta que vale la pena guardar/buscar algo.

> Si querés forzar un guardado, podés pedírselo: *"guardá esta decisión en memoria"*. El agente respeta la intención y llama a `mem_save`.

---

## Día a día: solo tres comandos

Esto es **todo** lo que la mayoría de la gente usa:

```bash
engram tui                    # browse visual: el camino más rápido
engram search "auth refactor" # búsqueda rápida desde la terminal
engram sync                   # exportar al repo (para compartir o respaldar)
```

### `engram tui` — la ventana a la memoria

Abrí la TUI:

```bash
engram tui
```

Vas a ver tres niveles de navegación:

1. **Proyectos** — todos los proyectos donde el agente guardó algo.
2. **Memorias** — observaciones dentro del proyecto seleccionado.
3. **Detalle** — el contenido completo de una observación, con fecha, tipo, prompt asociado.

Movete con flechas/`j`/`k`, `enter` para entrar, `esc` para volver, `q` para salir. La interfaz es bastante intuitiva.

> **Tip**: si no sabés qué tiene tu agente guardado, **abrí la TUI antes de tu próxima sesión**. Ver lo que ya sabe te da una idea más clara de qué se acuerda y qué no.

### `engram search` — búsqueda desde la terminal

Si ya sabés qué buscás:

```bash
engram search "endpoint users/me"
engram search "decisión password_hash"
engram search "bug timezone"
```

Te devuelve las observaciones que matchean, ordenadas por relevancia. Útil cuando estás depurando algo y querés ver si el agente ya tropezó con un problema parecido antes.

---

## Cómo Engram identifica tu proyecto

Desde v1.11.0, Engram **lee el `git remote`** del proyecto al arrancar, lo **normaliza a minúsculas**, y eso es tu "project name". Si está fuera de git, usa el nombre de la carpeta como fallback.

Esto resuelve el problema #1 históricamente: que el mismo repo terminara con memorias bajo nombres distintos (`my-app`, `My-App`, `my-app-frontend`).

### Si ves nombres duplicados

A veces, especialmente si arrancaste antes de v1.11.0 o cambiaste de carpeta, vas a ver algo así:

```bash
engram projects list
```

```
my-app           42 observations
My-App           17 observations
my-app-frontend   8 observations
```

Eso es **drift de nombre**. Solucionalo con:

```bash
engram projects consolidate
```

Te guía interactivamente para fusionar los duplicados en un solo proyecto. **No pierde datos**: solo unifica.

> El agente puede detectar el drift por su cuenta y llamar a `mem_merge_projects` vía MCP. Pero es lindo saber que vos también podés.

---

## Compartir memoria con tu equipo (`engram sync`)

Por defecto, Engram vive **solo en tu máquina** (`~/.engram/`). Si querés que las decisiones de tu proyecto las vea tu equipo:

### 1) Exportar al repo

```bash
engram sync
```

Esto crea (o actualiza) una carpeta `.engram/` con la memoria del proyecto exportada como archivos.

### 2) Versionarla

```bash
git add .engram/
git commit -m "chore: sync engram memory"
git push
```

### 3) En otra máquina, después del clone

```bash
git clone <repo>
cd <repo>
engram sync --import
```

Y arrancan con todo el contexto del proyecto cargado. **Ideal para onboarding**: el nuevo dev no parte de cero, parte del conocimiento acumulado del equipo.

### ¿Cuándo correr `engram sync`?

- Después de **sesiones largas** o cierres de feature.
- Antes de **subir cambios importantes**.
- Antes de **cerrar el día** si vas a seguir mañana en otra máquina.

No tenés que correrlo todo el tiempo: la memoria local sigue funcionando para vos sin sync.

---

## ¿Dónde vive Engram en mi máquina?

Dos lugares:

- `~/.engram/` — **base local** (SQLite + archivos). Todo lo que tu agente guardó vive acá.
- `<repo>/.engram/` — **exportable** (solo si corriste `engram sync` en ese repo). Es lo que se commitea.

Podés inspeccionar la carpeta local, pero **evitá tocar archivos a mano**: arruinás los índices. Usá siempre los comandos `engram`.

---

## Comandos del día a día (resumen)

| Comando | Cuándo usarlo |
| --- | --- |
| `engram tui` | Ver lo que el agente guardó (visual). |
| `engram search "..."` | Búsqueda rápida desde la terminal. |
| `engram projects list` | Ver todos los proyectos con sus conteos. |
| `engram projects consolidate` | Fusionar duplicados de nombre. |
| `engram sync` | Exportar al repo para compartir/respaldar. |
| `engram sync --import` | Importar `.engram/` después de clonar un repo. |

---

## Comandos avanzados (rara vez los necesitás)

Engram tiene más herramientas MCP que el agente puede usar por debajo: `mem_update`, `mem_delete`, `mem_timeline`, `mem_stats`, `mem_capture_passive`, etc. La doc completa está en el repo: [github.com/Gentleman-Programming/engram](https://github.com/Gentleman-Programming/engram).

A vos te alcanza con los seis comandos de arriba.

---

## Problemas comunes

### "El agente no encuentra lo de ayer"

1. Revisá que estés en el mismo proyecto (mismo `git remote`).
2. Probá `engram projects list` y verificá que no haya duplicados.
3. Si los hay, `engram projects consolidate`.

### "Cloné el repo en otra máquina y no veo nada"

¿El repo tiene `.engram/`? Si sí:

```bash
engram sync --import
```

Si no, no hay nada compartido. Pedile a quien tenga la memoria que corra `engram sync` y commitee.

### "Le pegué una API key al agente, ahora está en memoria"

Buscala y borrala:

```bash
engram search "sk-"      # o el prefijo que sea
# anotá el ID de la observación
```

Después, desde la TUI o vía el agente:

> *"Borrá la observación con id X."*

El agente puede llamar a `mem_delete` por vos.

### "Engram no detecta mi proyecto"

Verificá que la carpeta tenga `git remote`:

```bash
git remote -v
```

Si no tiene, agregá uno o trabajá conscientemente con el fallback al nombre de carpeta.

---

## Resumen

- Engram convierte tu agente en **un compañero con memoria**.
- Guarda **observaciones tipadas** (decisión, arquitectura, bug, descubrimiento, convención, artifact).
- Lo hace **solo**, vía MCP, sin que vos toques nada.
- Vos lo inspeccionás con `engram tui` y `engram search`.
- Lo compartís con tu equipo con `engram sync` + commit de `.engram/`.
- Si ves nombres duplicados, `engram projects consolidate`.
- **Cuidado con pegar secretos en el chat**: Engram puede capturarlos.

---

## Siguiente paso

➡️ **[8 · SDD: Spec-Driven Development](08-sdd-spec-driven.md)** — qué es SDD sin jerga, qué hace cada fase, cuándo el agente lo activa solo y cuándo conviene pedirlo explícitamente.

📚 Doc de referencia (avanzado): **[docs/engram.md](../engram.md)** — referencia completa de comandos, lista de herramientas MCP y comportamiento de detección.

---

← [Volver al índice](index.md) · ← [Anterior: Tu primera sesión real](06-primera-sesion.md)
