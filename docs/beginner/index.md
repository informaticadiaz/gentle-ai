# Gentle-AI para Principiantes

← [Volver al README](../../README.md)

---

Bienvenido. Esta sección es una **introducción amigable a Gentle-AI**, pensada para personas que recién empiezan con agentes de IA para programar (Claude Code, OpenCode, Cursor, etc.) y que quieren entender qué hace este proyecto y cómo sacarle provecho **sin tener que leer el PRD ni el código fuente**.

Si nunca usaste un agente de IA para codear, o si lo instalaste pero "siente que es solo un chatbot", este es tu punto de partida.

---

## ¿Para quién es esta guía?

- Personas que **acaban de instalar** un agente de IA y no saben por dónde seguir.
- Desarrolladores **junior o autodidactas** que quieren un flujo de trabajo serio con IA.
- Quienes **ya leyeron el README** pero quedaron con dudas sobre los conceptos (SDD, Engram, skills, sub-agentes, MCP).
- Quienes quieren contribuir al proyecto pero antes necesitan **entender el panorama general**.

No hace falta saber Go, ni nada sobre la arquitectura interna. Cada página explica un concepto a la vez, con ejemplos.

---

## Cómo leer esta guía

Las páginas están ordenadas de menor a mayor profundidad. Si seguís el orden, al final vas a poder:

1. Explicar qué es Gentle-AI y por qué existe.
2. Instalarlo y configurar tu primer agente.
3. Usar memoria persistente (Engram) y flujos SDD sin asustarte de la jerga.
4. Elegir el preset y los componentes correctos para tu caso.
5. Saber a qué documento "avanzado" ir cuando necesites más detalle.

---

## Temario sugerido

### Parte 1 · Fundamentos

1. **[¿Qué es Gentle-AI?](01-que-es-gentle-ai.md)**
   Qué problema resuelve, qué **no** es (no es un instalador de agentes), y la diferencia entre "tener un chatbot" y "tener un agente con memoria, skills y flujo de trabajo".

2. **[Glosario rápido](02-glosario.md)**
   Definiciones cortas de los términos que vas a encontrar: *agente*, *sub-agente*, *skill*, *preset*, *componente*, *MCP*, *SDD*, *Engram*, *persona*, *orquestador*, *delegación*.

3. **[Agentes soportados](03-agentes-soportados.md)**
   Qué es cada uno de los 13+ agentes (Claude Code, OpenCode, Cursor, Codex, Windsurf, etc.), en qué se diferencian (full-delegation vs. solo-agent) y cuál te conviene según tu situación.

### Parte 2 · Primeros pasos

4. **[Instalación paso a paso](04-instalacion.md)**
   Instalar en macOS, Linux y Windows. Verificar que quedó bien. Qué hacer si algo falla. Cómo desinstalar.

5. **[Tu primera configuración](05-primera-configuracion.md)**
   Recorrido por la TUI: elegir agente(s), elegir preset (`minimal` / `ecosystem-only` / `full-gentleman` / `custom`), elegir componentes. Qué se instala en cada caso.

6. **[Tu primera sesión real](06-primera-sesion.md)**
   Abrir el agente en un proyecto, correr `/sdd-init`, hacer una tarea chica y otra más grande para ver la diferencia. Qué esperar y qué **no** esperar.

### Parte 3 · Conceptos clave

7. **[Engram: memoria persistente](07-engram-memoria.md)**
   Por qué tu agente "se olvida" cosas y cómo Engram lo arregla. Cómo ver lo que recordó (`engram tui`, `engram search`), cómo sincronizar entre máquinas y cómo resolver conflictos de nombres de proyecto.

8. **[SDD: Spec-Driven Development](08-sdd-spec-driven.md)**
   Qué es SDD sin jerga: explorar, proponer, especificar, diseñar, implementar, verificar. Cuándo el agente lo activa solo y cuándo conviene pedirlo explícitamente.

9. **[Skills: capacidades curadas](09-skills.md)**
   Qué es una skill, cómo se descubren mediante el *Skill Registry*, y un recorrido por las skills incluidas (`branch-pr`, `chained-pr`, `comment-writer`, `issue-creation`, `work-unit-commits`, `cognitive-doc-design`).

10. **[Sub-agentes y delegación](10-subagentes-delegacion.md)**
    Por qué un solo hilo monolítico es mala idea. Cuándo el orquestador delega trabajo (leer 4+ archivos, tocar 2+ archivos, revisar antes de PR) y cómo eso se traduce en mejor calidad y menos errores.

11. **[Personas y permisos](11-personas-permisos.md)**
    La persona "teaching-oriented" con permisos *security-first*: qué puede y qué no puede hacer tu agente por defecto, y cómo cambiarlo.

12. **[MCP en 5 minutos](12-mcp.md)**
    Qué es el Model Context Protocol, qué servidores MCP instala Gentle-AI y para qué sirven (filesystem, github, engram, etc.).

### Parte 4 · Casos de uso

13. **[Caso: arrancar un proyecto nuevo](13-caso-proyecto-nuevo.md)**
    De `git init` a primer commit con el agente ayudándote, usando SDD para la primera feature.

14. **[Caso: sumarse a un repo existente](14-caso-repo-existente.md)**
    `/sdd-init` para que el agente entienda el stack, `engram sync --import` si el repo trae `.engram/`, primeras tareas seguras.

15. **[Caso: OpenCode con múltiples modelos](15-caso-opencode-profiles.md)**
    Asignar un modelo barato a exploración y uno potente a diseño. Cómo crear y cambiar perfiles SDD (`gentle-orchestrator` y derivados).

16. **[Caso: compartir memoria con tu equipo](16-caso-engram-team.md)**
    Versionar `.engram/` en git, `engram sync` después de sesiones importantes, cómo resolver "drift" de nombres de proyecto.

### Parte 5 · Mantenimiento y resolución de problemas

17. **[Actualizar y respaldos](17-update-backups.md)**
    `gentle-ai update`, dónde quedan los backups (tar.gz, deduplicados, auto-pruned), cómo *pinear* uno, cómo restaurar.

18. **[Resolución de problemas comunes](18-troubleshooting.md)**
    "Mi skill no aparece", "Engram no encuentra el proyecto", "OpenCode no ve mi perfil", "el agente no delega", `gentle-ai skill-registry refresh`, modo no-interactivo.

19. **[¿Qué leer ahora?](19-siguientes-pasos.md)**
    Mapa de los docs "avanzados" del repo: `intended-usage.md`, `architecture.md`, `CODEBASE-GUIDE.md`, `PRD.md`, y cuándo ir a cada uno.

### Parte 6 · Contribuir (opcional)

20. **[Cómo contribuir siendo principiante](20-contribuir.md)**
    Issues etiquetados como `good first issue`, cómo correr los tests, qué espera `CONTRIBUTING.md`, y cómo proponer una nueva skill o documento.

---

## Convenciones de esta guía

- **Negrita** para términos clave la primera vez que aparecen.
- `código` para comandos, nombres de archivo y banderas de CLI.
- Bloques de código con el lenguaje declarado para que copiar/pegar sea seguro.
- Cajas de aviso al principio de cada página cuando algo requiere una versión mínima, sistema operativo, o agente específico.
- Cada página termina con una sección **"Siguiente paso"** que enlaza a la próxima lectura recomendada.

---

## Estado

Este temario es una **propuesta inicial**. Las páginas se irán completando una a una. Si te falta alguna o querés sugerir cambios, abrí un issue con la etiqueta `docs` en el [repositorio](https://github.com/Gentleman-Programming/gentle-ai/issues).

---

← [Volver al README](../../README.md)
