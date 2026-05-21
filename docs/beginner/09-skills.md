# 9 · Skills: capacidades curadas

← [Volver al índice](index.md) · ← [Anterior: SDD: Spec-Driven Development](08-sdd-spec-driven.md)

---

> **Idea central**: una **skill** es un archivo `SKILL.md` con instrucciones detalladas para que el agente haga **una cosa específica bien**. Gentle-AI viene con un set de skills curadas, el agente las **descubre solas** vía el *skill registry*, y las carga **completas** cuando hace falta (no resúmenes generados, el archivo original).

---

## El problema que resuelve

Sin skills, cada vez que querés que el agente haga una tarea no trivial (crear un PR, escribir un buen issue, hacer un commit con scope, redactar docs), tenés tres opciones malas:

1. **Pegarle un prompt largo** copiado de internet. Funciona una vez. Al rato lo perdés.
2. **Confiar en el modelo crudo**. Te genera commits estilo `update stuff` o issues sin pasos para reproducir.
3. **Esperar que el agente "se acuerde"** de tus convenciones. Spoiler: no se acuerda.

Las skills resuelven esto: **una vez instaladas**, el agente sabe **cuándo aplicarlas** y **cómo seguirlas al pie de la letra**, en cada proyecto, en cada sesión.

---

## Qué es una skill, técnicamente

Una skill es una **carpeta** con un archivo `SKILL.md` adentro (puede tener archivos auxiliares como ejemplos, plantillas, etc.):

```
skills/
└── branch-pr/
    ├── SKILL.md
    ├── examples/
    └── templates/
```

El `SKILL.md` tiene un **frontmatter YAML** y un **cuerpo en markdown**:

```yaml
---
name: branch-pr
description: "Create Gentle AI pull requests with issue-first checks.
              Trigger: creating, opening, or preparing PRs for review."
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "2.0"
---

## When to Use

(instrucciones detalladas para el agente)
```

- **`name`**: el identificador.
- **`description`**: incluye el **trigger** (cuándo aplicarla). El orquestador la elige leyendo esto.
- **Cuerpo**: la guía concreta. El agente la lee **entera** antes de ejecutarla.

---

## Dos grupos de skills

### A) SDD skills

Las cubrimos en la **página 8**: `sdd-init`, `sdd-explore`, `sdd-propose`, `sdd-spec`, `sdd-design`, `sdd-tasks`, `sdd-apply`, `sdd-verify`, `sdd-archive`, `sdd-onboard`. Más `judgment-day` (revisión adversaria paralela).

No las invocás vos: forman parte del flujo SDD.

### B) Foundation skills (las del día a día)

Estas son las que vas a notar en tareas comunes:

| Skill | Trigger / cuándo aparece |
| --- | --- |
| **`branch-pr`** | Crear, abrir o preparar **PRs**. Aplica conventional commits, naming de rama (`feat/...`), enforcement de "issue primero". |
| **`chained-pr`** | Planificar **stacks de PRs** (cambios grandes partidos en PRs encadenados revisables). |
| **`comment-writer`** | Redactar **comentarios de review o de PR** con tono directo y cálido. |
| **`issue-creation`** | Abrir **issues** (bugs con steps-to-reproduce + features con user value y acceptance criteria). |
| **`work-unit-commits`** | Partir una implementación en **unidades de commit revisables**, con buenas descripciones. |
| **`cognitive-doc-design`** | Escribir **docs que bajan carga cognitiva** al que las lee (esta misma guía sigue esos principios). |
| **`go-testing`** | Patrones de testing para **Go** (incluye Bubbletea TUI). |
| **`skill-creator`** | Crear **skills nuevas** siguiendo la spec. |
| **`skill-improver`** | Mejorar skills existentes. |
| **`skill-registry`** | Construir el índice de skills (lo usa el orquestador). |

Casos típicos donde las vas a "sentir":

- Le pedís *"abrime un PR de este cambio"* → arranca `branch-pr`.
- Le pedís *"agregá tests para esto"* en Go → arranca `go-testing`.
- Le pedís *"redactame un issue para reportar este bug"* → arranca `issue-creation`.
- Le pedís *"escribime un README para principiantes"* → puede aplicar `cognitive-doc-design`.

---

## El Skill Registry: cómo el agente las descubre

El **skill registry** es un **índice local del proyecto** que lista todas las skills disponibles + sus triggers + sus rutas. Vive en:

```
<tu-proyecto>/.atl/skill-registry.md
<tu-proyecto>/.atl/.skill-registry.cache.json
```

### Flujo en tiempo real

```text
Vos hacés un pedido
   │
   ▼
Orquestador lee .atl/skill-registry.md (1 vez por sesión, cacheado)
   │
   ▼
Hace match: ¿qué skill aplica para este pedido y este contexto?
   │
   ▼
Pasa la ruta exacta del SKILL.md al sub-agente
   │
   ▼
Sub-agente lee el SKILL.md COMPLETO (no un resumen)
   │
   ▼
Ejecuta siguiendo la skill al pie de la letra
```

> **Detalle importante**: el orquestador **no le pasa al sub-agente un resumen** de la skill. Le pasa la **ruta del archivo** y el sub-agente lo lee íntegro. Esto preserva la intención del autor de la skill y evita que la summarización rompa instrucciones sutiles.

### Refresh del registry

Cuando agregás, removés, renombrás o movés skills:

```bash
gentle-ai skill-registry refresh
gentle-ai skill-registry refresh --force      # ignora la caché
gentle-ai skill-registry refresh --quiet      # sin output
gentle-ai skill-registry refresh --cwd /ruta  # apuntar a otro proyecto
```

> **Buena noticia**: en agentes que soportan **startup hooks** (Claude Code, OpenCode, Pi), esto ya **se dispara solo** al arrancar la sesión. Vos solo lo corrés cuando el agente no soporta hooks o cuando querés forzarlo (`--force`).

### ¿De dónde saca las skills al hacer refresh?

El orden de escaneo es:

1. **Skill roots del proyecto** primero — `skills/`, `.opencode/skills/`, `.claude/skills/`, etc.
2. **Skill roots globales del agente** después — `~/.config/opencode/skills/`, `~/.claude/skills/`, etc.
3. **Dedupe por nombre** — si una skill existe en los dos lugares, **gana la del proyecto** (te permite "pisar" una skill global con una versión específica del repo).

### El contrato del registry

| Campo | Significado |
| --- | --- |
| `Skill` | El `name` del frontmatter (o el nombre de carpeta como fallback). |
| `Trigger / description` | La `description` completa del frontmatter (multi-línea incluida). |
| `Scope` | `project` o `user`. |
| `Path` | Ruta exacta al `SKILL.md`. |

---

## Skills incluidas vs skills "de codear" (otro repo)

Gentle-AI viene con las **foundation + SDD skills** (~20 skills). Lo que **no** trae son las **skills de framework/lenguaje específicas** (React 19, Angular, TypeScript, Tailwind 4, Zod 4, Playwright, etc.).

Esas viven en un repo aparte mantenido por la comunidad:

🔗 [github.com/Gentleman-Programming/Gentleman-Skills](https://github.com/Gentleman-Programming/Gentleman-Skills)

Para sumarlas a tu agente (ejemplo con Claude Code):

```bash
git clone https://github.com/Gentleman-Programming/Gentleman-Skills.git
cp -r Gentleman-Skills/curated/react-19 ~/.claude/skills/
cp -r Gentleman-Skills/curated/typescript ~/.claude/skills/
# o copiá todo el curated/ si querés todas
```

Después:

```bash
gentle-ai skill-registry refresh    # detecta las nuevas
```

Y ya. El agente las descubre y las carga **solo** cuando aplican.

---

## Crear tu propia skill

Si querés agregar una skill propia (interna del equipo, o pública):

1. Dentro del agente: *"corré skill-creator para crear una skill llamada X con propósito Y"*.
2. La skill `skill-creator` te guía paso a paso siguiendo la spec oficial (nombre, descripción, trigger, body).
3. La carpeta queda creada con `SKILL.md` válido.
4. `gentle-ai skill-registry refresh` y el orquestador la incorpora.

Para mejorar una skill existente (por ejemplo, una que querés afinar para tu proyecto):

> *"corré skill-improver sobre la skill X y mejorala para este proyecto"*

---

## ¿Skill "user" vs skill "project"? ¿Cuál usar?

- **User scope** (global, `~/.claude/skills/`, `~/.config/opencode/skills/`, etc.): la usás en **todos** los proyectos. Buena para skills genéricas tuyas.
- **Project scope** (`<repo>/skills/` o equivalente): específica de **este repo**. Buena para convenciones de equipo que no querés exportar a otros proyectos.

Recordá: **project gana sobre user** si tienen el mismo nombre. Eso te deja "sobreescribir" una skill genérica con una versión más estricta por repo.

---

## Problemas comunes

### "Le pedí algo y no aplicó la skill que esperaba"

Tres causas frecuentes:

1. **El registry está desactualizado**. Corré:
   ```bash
   gentle-ai skill-registry refresh --force
   ```
2. **El trigger no matchea tu pedido**. Reformulá: en lugar de *"hacé un PR"*, decí *"abrime un pull request siguiendo branch-pr"* para forzar.
3. **La skill no está instalada**. Verificá:
   ```bash
   cat .atl/skill-registry.md   # ¿aparece la skill?
   ```

### "El agente aplica la skill pero ignora detalles del SKILL.md"

Probablemente está usando un resumen (no debería). Mirá los logs/transcript: ¿el agente leyó el archivo completo? Si no, forzá:

> *"leé el SKILL.md completo de branch-pr antes de actuar"*

### "Quiero ver qué skills tengo disponibles"

```bash
cat .atl/skill-registry.md
```

O desde el agente:

> *"listame todas las skills disponibles en este proyecto"*

### "Agregué una skill nueva y no la encuentra"

Necesitás refresh:

```bash
gentle-ai skill-registry refresh
```

Y reabrí la sesión del agente.

---

## Resumen

- Una **skill** = carpeta con `SKILL.md` + frontmatter + cuerpo de instrucciones.
- Gentle-AI trae ~20 skills (SDD + foundation).
- El **skill registry** indexa todas las skills del proyecto y globales en `.atl/skill-registry.md`.
- El orquestador **descubre y carga skills solo**, leyendo el archivo **completo** (no resúmenes).
- **Project > user** cuando hay conflicto de nombre.
- Skills de framework (React, Angular, etc.) viven en [Gentleman-Skills](https://github.com/Gentleman-Programming/Gentleman-Skills) y se instalan copiándolas.
- `skill-creator` y `skill-improver` te ayudan a hacer las tuyas.

---

## Siguiente paso

➡️ **[10 · Sub-agentes y delegación](10-subagentes-delegacion.md)** — por qué un solo hilo monolítico es mala idea, las reglas concretas que disparan delegación, y cómo se traducen en mejor calidad y menos errores.

📚 Doc de referencia (avanzado): **[docs/skill-registry.md](../skill-registry.md)** — flujo de runtime/refresh, contrato del registry, y guía de autoría. **[docs/components.md](../components.md)** — catálogo completo de skills incluidas.

---

← [Volver al índice](index.md) · ← [Anterior: SDD: Spec-Driven Development](08-sdd-spec-driven.md)
