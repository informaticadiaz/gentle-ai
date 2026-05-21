# 16 · Caso: compartir memoria con tu equipo

← [Volver al índice](index.md) · ← [Anterior: Caso: OpenCode con múltiples modelos](15-caso-opencode-profiles.md)

---

> **Objetivo del caso**: convertir la memoria de Engram (que por defecto vive solo en tu máquina) en un **bien compartido del equipo**. El que llega nuevo arranca con todo el contexto cargado; el que cambia de máquina no pierde nada; las decisiones quedan trazadas como cualquier otro artefacto del repo.
>
> **Tiempo estimado para setup inicial del equipo**: 20 minutos.

---

## La premisa

Vimos en la **página 7** que Engram guarda decisiones, bugs, descubrimientos y artefactos SDD en `~/.engram/` (local de tu máquina). Eso es genial **para vos**, pero cuando trabajás en equipo aparecen tres problemas:

1. **Mi compañera no ve lo que mi agente aprendió.** Cada quien tiene su silo.
2. **Cambié de máquina y arranqué de cero.** Toda la memoria quedó en la otra.
3. **Onboardeo a alguien nuevo y le toma semanas entender el repo.** Aunque las decisiones estén documentadas en algún lado, no están dentro del flujo del agente.

**Solución**: versionar `.engram/` en el repo. La memoria pasa a ser un artefacto **versionado, mergeable y revisable**, como cualquier otro archivo del proyecto.

---

## Cuándo hacer esto

Versionar `.engram/` tiene sentido cuando:

- ✅ El proyecto tiene **más de una persona** trabajando.
- ✅ Las decisiones técnicas son **valiosas a largo plazo** (no prototipo descartable).
- ✅ Hay **rotación** o entrada de gente nueva al equipo.
- ✅ El equipo entero usa Gentle-AI (o al menos Engram).

**No tiene tanto sentido** cuando:

- ✗ Es un repo **personal** sin colaboradores.
- ✗ El repo es **público** y la memoria contiene cosas privadas/internas del equipo (cuidado con esto).
- ✗ El equipo no usa Engram → no van a ver beneficio.

---

## Setup inicial (una sola vez por repo)

### 1) La persona que arranca: poblar la memoria

Si recién empieza el proyecto, seguí los pasos de la página 13 normalmente. Si ya tiene historia, la página 14.

A medida que trabajás, Engram va guardando. Después de una sesión productiva:

```bash
engram sync
```

Eso crea (o actualiza) la carpeta `.engram/` en la raíz del repo, con los datos exportados como archivos.

### 2) Commitearla por primera vez

```bash
git add .engram/
git commit -m "chore: bootstrap engram memory for the team"
git push
```

### 3) Documentar el flujo en el README del repo

Agregá una sección corta al README del proyecto para que el equipo sepa:

```markdown
## Engram (memoria compartida del proyecto)

Este repo incluye memoria SDD versionada en `.engram/`. Si usás
Gentle-AI:

- Tras clonar: `engram sync --import` para cargar el contexto.
- Tras sesiones largas: `engram sync` + commit de los cambios.

Si no usás Gentle-AI, ignorá la carpeta tranquilamente.
```

Eso es todo. **El resto fluye solo** a partir de acá.

---

## El loop diario del equipo

### Para cada persona, en su máquina

```text
1. git pull
2. engram sync --import   # solo si hubo cambios en .engram/ del repo
3. trabajar con el agente
4. engram sync             # al final de una sesión productiva
5. git add .engram/
6. git commit -m "chore: sync engram with <tema>"
7. git push
```

Eso es todo el flujo.

> **Tip**: si en tu equipo hay disciplina de PRs, **los cambios de `.engram/` van en el PR de la feature**. Así la decisión y su trazabilidad viajan juntas.

---

## ¿Cuándo correr `engram sync`?

No tenés que correrlo todo el tiempo. Buenos disparadores:

| Disparador | Por qué |
| --- | --- |
| **Cierre de feature** | Las decisiones más importantes pasaron en esa sesión. |
| **Antes de cambiar de máquina** | Para no perder el contexto del día. |
| **Antes de salir de vacaciones** | Tu reemplazo arranca con todo. |
| **Después de resolver un bug "ouch"** | Para que nadie tropiece con el mismo de nuevo. |
| **Después de un `judgment-day` o revisión adversaria** | Las observaciones críticas valen oro. |

No tiene sentido correrlo después de un cambio trivial. Usá tu criterio: ¿esto va a ser útil para el yo del futuro o para mi equipo? Si sí, sync.

---

## Engram y los PRs

Hay dos filosofías sobre cómo manejarlo. Elegí la que se acomode a tu equipo.

### Filosofía A · "Engram va en el PR de la feature"

Cuando trabajás una feature con SDD, la memoria que se generó queda **dentro del mismo PR**:

```
PR: feat(checkout): add discount coupon support

Files:
  src/server/api/routers/checkout.ts   (modified)
  src/lib/coupons.ts                   (new)
  tests/coupons.test.ts                (new)
  .engram/observations.json            (modified)
  .engram/index.json                   (modified)
```

**Ventajas**: trazabilidad perfecta. La decisión vive con el código.
**Desventaja**: más conflictos de merge en `.engram/` cuando hay muchos PRs paralelos.

### Filosofía B · "Engram en PRs separados"

Las features se mergean primero, y después una persona "cura" la memoria con un PR aparte:

```
PR: chore: sync engram with checkout coupons feature

Files:
  .engram/observations.json   (modified)
  .engram/index.json          (modified)
```

**Ventajas**: menos conflictos. Curaduría humana antes del push.
**Desventaja**: la memoria queda "atrasada" respecto al código.

> **Recomendación principiante**: arrancá con **Filosofía A**. Es más simple y más fiel al flujo "todo viaja junto".

---

## Resolver conflictos de merge en `.engram/`

A medida que el equipo crezca, vas a tener merge conflicts en `.engram/`. **No entres en pánico**: hay una herramienta para esto.

### El conflicto típico

```bash
git pull
# CONFLICT (content): Merge conflict in .engram/observations.json
```

### No edites el JSON a mano

Engram tiene índices internos. Si los rompés, perdés búsqueda. **Usá los comandos**:

```bash
# 1) Aceptar lo de la otra rama tal cual (tu local lo pierde temporalmente)
git checkout --theirs .engram/

# 2) Cargá lo de la rama remota en tu base local
engram sync --import

# 3) Volvé a exportar fusionando con tu memoria local actual
engram sync

# 4) Commiteá la fusión
git add .engram/
git commit -m "chore: merge engram memory"
```

Engram dedupliza por `topic_key`, así que **memorias del mismo "tema" no se duplican**, se actualizan. Lo único que podés perder es **el orden** de algunas observaciones, no su contenido.

> **Tip**: si los conflictos son frecuentes en tu equipo, considerá **Filosofía B** (PRs separados de memoria) o programá un sync semanal "oficial" por una persona designada.

---

## Onboarding de alguien nuevo

Este es el escenario donde más se nota el valor.

### El flujo del nuevo

```bash
# 1) Clonar
git clone <repo>
cd <repo>

# 2) Instalar Gentle-AI y configurar agente (página 4-5)
brew install gentle-ai
gentle-ai install --agent claude-code --preset ecosystem-only

# 3) Importar la memoria del equipo
engram sync --import

# 4) Abrir el agente y pedir el tour
claude
```

Dentro del agente:

```
soy nuevo en este repo. dame un tour usando lo que ya está en engram.
contame stack, arquitectura, convenciones, y decisiones importantes
que el equipo tomó en los últimos meses.
```

El agente lee Engram y arma un resumen **basado en datos reales**, no inventado. Cosas que normalmente toman semanas de absorber, en 10 minutos las tenés digeridas.

### Lo que el nuevo va a poder hacer enseguida

- Hablar el lenguaje del equipo (conventions, naming, patrones).
- Saber qué decisiones **ya están tomadas** (no proponer cosas que ya se descartaron).
- Reconocer bugs históricos que pueden volver a aparecer.
- Abrir un PR siguiendo el template del repo desde el día uno.

---

## Drift entre máquinas / nombres de proyecto

Cuando varias personas trabajan en el mismo repo, Engram **debería** detectar el proyecto igual en todas las máquinas (por `git remote -v`, página 7). Pero a veces aparece drift:

```bash
engram projects list
```

```
my-app           102 observations
My-App            18 observations
my-app-frontend    5 observations
```

Causas frecuentes:

- Alguien renombró el `origin` remote.
- Alguien clonó sin remote y trabajó con nombre de carpeta.
- Versiones viejas de Engram (pre v1.11.0) que no normalizaban a minúsculas.

**Solución**:

```bash
engram projects consolidate
```

Interactivo. Te muestra los duplicados y te deja fusionarlos. **No pierde datos**: junta todas las observaciones bajo un solo nombre canónico.

Hacelo **una persona del equipo** y commiteá el resultado:

```bash
engram sync
git add .engram/
git commit -m "chore: consolidate engram project name drift"
git push
```

Después todos hacen `git pull` + `engram sync --import` y quedan alineados.

---

## Seguridad: qué NO commitear nunca

Engram guarda lo que el agente le pide guardar. Si en el chat alguien pegó:

- 🚨 Una API key real.
- 🚨 Un connection string con password.
- 🚨 Información del cliente que no debería salir del equipo.
- 🚨 Una URL interna sensible.

…es posible que **eso quede en la memoria**. Antes de commitear `.engram/`, revisá:

```bash
engram search "sk-"           # claves OpenAI
engram search "ghp_"          # GitHub PAT
engram search "AKIA"          # AWS access key
engram search "postgres://"   # connection strings
```

Si encontrás algo:

```bash
# desde el agente
"borrá la observación con id X"
```

O abrí `engram tui`, navegá a la observación y borrala desde ahí.

> **Regla general**: si el repo es **público**, doblá el cuidado. La memoria es código: una vez pushed, está en el historial.

---

## Engram en `.gitignore`: cuándo sí, cuándo no

Algunos casos donde querés **no** commitear `.engram/`:

- Repo público y la memoria tiene cosas internas.
- Cada dev quiere su propia memoria privada (preferencias personales, atajos, exploración casual).
- El equipo no usa Engram → no aporta nada commitearlo.

En esos casos:

```bash
echo ".engram/" >> .gitignore
git add .gitignore
git commit -m "chore: keep engram memory local per-developer"
```

Cada quien mantiene su `~/.engram/` local sin compartir. **Engram sigue funcionando**, solo que sin la capa de team-sharing.

---

## ¿Y si parte del equipo no usa Gentle-AI?

Está bien. **`.engram/` no rompe nada** si lo ignorás. Para los devs que no usan Gentle-AI:

- La carpeta `.engram/` es simplemente un par de JSONs/SQLite que no afectan el código.
- Pueden agregarla a su `~/.gitignore_global` si les molesta verla.
- Pueden commitearla normalmente (les agradecerá el resto del equipo).

**No hay efectos colaterales** en el build, los tests o el deploy.

---

## Errores comunes

### "Cloné el repo y `engram sync --import` no importó nada"

Verificá:

```bash
ls -la .engram/   # ¿existe?
git log .engram/  # ¿hay historia?
```

Si el `.engram/` está pero `--import` no ve nada, probablemente la versión de Engram local es vieja:

```bash
gentle-ai update   # actualiza también engram
```

### "Hago `engram sync` pero `.engram/` no aparece como modificado en git"

Probablemente no hubo cambios desde el último sync. Engram dedupe: si nada nuevo, no escribe nada nuevo.

Forzá con una observación nueva (pedile al agente algo concreto que guarde) y volvé a probar.

### "El sync de `.engram/` genera muchísimas líneas en cada commit"

Es normal. Los JSONs internos cambian con cualquier observación nueva. Si te molesta el ruido en el diff:

- Usá **Filosofía B** (PRs separados de memoria).
- O simplemente skipeá `.engram/` en code review (no es código que revisar línea por línea, es estado).

### "Mi compañero está viendo cosas viejas que el agente ya 'sabe' que están desactualizadas"

Engram **upserts por topic_key**. Las cosas viejas se actualizan, no se duplican. Pero si una observación **legítimamente** ya no aplica:

```
# desde el agente
"borrá la observación X de engram, ya no es válida"
```

Después `engram sync` + commit.

### "Llegué a un repo con `.engram/` pero la memoria está mal/incompleta/sucia"

No te quedes con lo malo. Algunos approaches:

- **Curaduría manual**: abrí `engram tui`, identificá observaciones erradas o obsoletas, borralas.
- **Re-bootstrap**: con `sdd-init` y `sdd-explore` general (página 14) generás memoria fresca, complementaria.
- **Si está muy contaminada**: el equipo decide hacer un "reset" — borrar `.engram/`, empezar de cero, y curar mejor en el futuro.

---

## Resumen

- Versionar `.engram/` convierte la memoria de Engram en **un bien de equipo**.
- Loop diario simple: `git pull` → `engram sync --import` → trabajar → `engram sync` → commit + push.
- Decidí entre **Filosofía A** (memoria en el PR de la feature) o **B** (PRs separados). Para empezar: A.
- Conflictos: nunca editar el JSON a mano. Usar `git checkout --theirs` + `engram sync --import` + `engram sync`.
- **Onboarding** es donde más se siente el valor: el nuevo arranca con contexto cargado en minutos.
- Si hay drift de nombres: `engram projects consolidate` por una persona, commit y todos alineados.
- **Cuidado con secretos**: revisar antes de commitear (`engram search "sk-"`, etc.).
- Si el equipo no usa Engram o el repo es público con datos sensibles: agregar `.engram/` a `.gitignore`.

---

## Siguiente paso

Con esto cerramos la **Parte 4 · Casos de uso**. Seguimos con mantenimiento.

➡️ **[17 · Actualizar y respaldos](17-update-backups.md)** — `gentle-ai update`, dónde quedan los backups (tar.gz, deduplicados, auto-pruned), cómo *pinear* uno, cómo restaurar.

📚 Doc de referencia (avanzado): **[docs/engram.md](../engram.md)** — referencia completa de comandos Engram.

---

← [Volver al índice](index.md) · ← [Anterior: Caso: OpenCode con múltiples modelos](15-caso-opencode-profiles.md)
