# ADR-1100: Knowledge Repository — Ubicación Física y Convención de Nombres de Knowledge Assets

Status: APPROVED

Date: 2026-07-26

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Research, Documentation

Tags:
taxonomy, classification, knowledge-asset, knowledge-repository

Related ADRs:
ADR-0033, ADR-1106 (enmienda parcial — retirada del Relationship Record)

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Enmienda Parcial (2026-07-30)

Puntos 2 y 4: revocados por ADR-1106.

---

## Nota de Revisión (2026-08-05) — Alcance de la transliteración de `ñ`

Al curar `conocimiento/260805-language-calendario-lunar-enero-2026.md`, se detectó que la regla de transliteración `ñ → nn` del punto 3 (más abajo) se había extendido, sin decisión explícita, a dos contextos que esta ADR nunca cubrió:

* **Campo `Tags:`** de varios Knowledge Assets ya existentes (`espanna`, `sennores`, `montannas`, `ensennanza`...), a veces incluso de forma inconsistente con el propio slug del nombre de archivo del mismo fichero (p. ej. `260726-economics-ruta-seda-ferroviaria-china-espanna.md` con tag `comercio-china-espana`, sin `ñ` ni `nn`).
* **Mensajes de commit** (`anadir` en vez de `añadir`, repetido en al menos 9 commits de sesiones distintas).

Se aclara: el punto 3 de esta Decision aplica **exclusivamente al `slug` del nombre de fichero de un Knowledge Asset** (necesidad real de seguridad de sistema de archivos/URL — un nombre de fichero debe ser portable entre sistemas operativos). No aplica a:

* `Tags:` — sin restricción de sistema de archivos; `ADR-0033-Taxonomia_ADRs.md` § Política de Tags no exige ni sugiere transliteración de acentos o `ñ`. Los tags preservan la ortografía real del español (`españa`, `enseñanza`, `año`, `muñoz`...).
* Mensajes de commit — Git admite UTF-8 sin ninguna restricción técnica; no hay justificación para omitir `ñ` o tildes.
* Cualquier otro texto en prosa dentro de un Knowledge Asset (`Purpose`, `Description`, `Notes`, etc.), donde ya se escribe en español correcto sin que nadie lo haya cuestionado.

Los Knowledge Assets ya afectados en `Tags:` se normalizaron en la rama de contenido correspondiente (`oushinet`, commit `dda8283`), conforme a la separación de rama gobernanza/contenido de ADR-1102. No se reescribe el historial de Git para corregir mensajes de commit ya publicados — se corrige el hábito hacia adelante.

---

## Nota de Ampliación (2026-08-06) — Carpeta `conocimiento_index/`

El punto 2 fija archivos planos, sin subcarpetas por dominio, dentro de `conocimiento/`. Se añade una excepción acotada: los Knowledge Assets marcados `Knowledge Type: Index` (recopilatorios que agregan información ya presente en otros Knowledge Assets — vocabularios, glosarios consolidados, índices derivados de corpus) se alojan en `conocimiento_index/` — carpeta de primer nivel, hermana de `conocimiento/`, no una subcarpeta de esta (corregido 2026-08-07: se creó originalmente como subcarpeta `conocimiento/index/` y se trasladó a este nivel ese mismo día, con reescritura verificada mecánicamente de los enlaces afectados). La separación de `conocimiento/` sigue respondiendo al mismo criterio que motivó la excepción original — rol estructural, no dominio —, el mismo criterio que `ADR-1109 §5` ya usa para excluir estos ficheros de la bóveda de Obsidian.

La convención de nombres del punto 3 se especializa para esta carpeta: un fichero en `conocimiento_index/` se llama `YYMMDD-index-domain_principal-slug.md` — se inserta el segmento literal `index` entre la fecha y el dominio, señalando el rol de la ficha en el propio nombre de archivo, no solo en `Knowledge Type` y en la ubicación. El resto del punto 3 (fecha de curación, dominio del vocabulario canónico, transliteración de `ñ`) no cambia.

Primer caso real: `260801-index-language-hsk1-mandarin-corner-150-oraciones.md`, `260801-index-language-hsk1-xindongfang-vocabulario.md` y `260802-index-geography-ubicaciones-conocimiento.md` se trasladaron a `conocimiento/index/` y renombraron el 2026-08-06 (ver `ADR-1109 §5` para el efecto sobre la exclusión de la bóveda de Obsidian). El slug de la tercera ficha se simplificó de paso (`indice-ubicaciones-conocimiento` → `ubicaciones-conocimiento`) al quedar la palabra "índice" redundante con el nuevo segmento `index` del nombre. El 2026-08-07 esta carpeta, junto con las restantes fichas Index creadas en el ínterin, se trasladó de `conocimiento/index/` a `conocimiento_index/` conforme a la corrección de este mismo punto.

---

## Context

`docs/Project_Definition/02_MASTER_ARCHITECTURE.md` (componente 4) define un componente arquitectónico llamado "Knowledge Repository": *"Contains the project's human-readable Knowledge Assets and supporting documentation"*. Este componente está definido conceptualmente desde el origen del proyecto, pero nunca se le asignó una ubicación física en el árbol de directorios.

De forma paralela, `ai/templates/30_knowledge_asset.md` y `ai/templates/31_relationship_record.md` existen como plantillas desde antes de este ADR, pero nunca habían sido instanciadas — no existe en el repositorio ningún ejemplo real de Knowledge Asset ni de Relationship Record.

Esta carencia se hizo evidente al catalogar tres artículos de la revista China Book International (Revista_CBI_2606) como Knowledge Assets: no había ni ubicación designada ni convención de nombres para los archivos resultantes. La solución ad hoc adoptada inicialmente (prefijar el nombre de archivo con el identificador de escaneo OCR del documento fuente, p. ej. `S45C-0i26072306440_slug.md`) resultó frágil: acopla el nombre de la ficha a un identificador mutable y ajeno al propio conocimiento curado, y duplica una trazabilidad que ya proporciona el Relationship Record — violando el principio arquitectónico "Relationships have priority over hierarchy" (`02_MASTER_ARCHITECTURE.md`).

`asignaturas/` (árbol curricular oficial, carpetas 011-042) y `fuentes/` (material de referencia externo/primario) ya tienen roles definidos y no son apropiados para alojar conocimiento curado por el propio proyecto.

---

## Decision

1. `conocimiento/` (raíz del repositorio, hermana de `asignaturas/`, `fuentes/`, `docs/`, `ai/`) se adopta como la implementación física del componente "Knowledge Repository" definido en `02_MASTER_ARCHITECTURE.md` §4.

2. `conocimiento/*.md` — instancias de `30_knowledge_asset.md` (archivos planos, sin subcarpetas por dominio mientras el volumen no lo justifique). *(La subcarpeta `conocimiento/relaciones/` descrita originalmente en este punto fue revocada por ADR-1106.)*

3. Convención de nombres para archivos de Knowledge Asset:

   ```
   YYMMDD-domain_principal-slug.md
   ```

   - `YYMMDD`: fecha en que se curó/redactó el Knowledge Asset (no la fecha de publicación de la fuente original, que puede no conocerse con precisión de día — evita fabricar datos, conforme al principio de veracidad de `00_PROJECT_VISION_Chino_Definicion_Proyecto.md`).
   - `domain_principal`: el primer término listado en el campo `Domain` de la ficha, en minúsculas, tomado del vocabulario canónico de `ADR-0033-Taxonomia_ADRs.md` § Domain.
   - `slug`: identificador breve en kebab-case del tema.
   - **Transliteración de `ñ`**: se sustituye por `nn` (nunca por `n` simple). Regla añadida el 2026-08-01 tras detectar un uso inconsistente previo (`n` simple) que genera colisiones ortográficas reales del español (p. ej. `año` → `ano` es una palabra española distinta y no relacionada; `año` → `anno` evita la colisión). Los slugs ya existentes con `n` simple (`espana`, `senores`, `montanas`, etc.) quedan pendientes de una refactorización planificada aparte; esta regla aplica de forma obligatoria a partir de esta fecha para cualquier slug nuevo.

4. **[REVOCADO POR ADR-1106]** La procedencia de cada Knowledge Asset se documenta en su propia sección `Evidence and Sources`.

5. El campo `Domain` de `30_knowledge_asset.md` reutiliza el vocabulario canónico de `ADR-0033-Taxonomia_ADRs.md` § Domain como fuente única — no se mantienen listas de ejemplos divergentes entre plantillas.

---

## Consequences

### Positive

* El componente "Knowledge Repository", definido desde el origen del proyecto pero sin ubicación, queda materializado.
* Nombres de archivo ordenables cronológicamente, legibles por humanos y desacoplados de identificadores externos mutables.
* Vocabulario de `Domain` unificado entre ADRs y Knowledge Assets (una sola fuente canónica).

### Negative

* Requiere renombrar los 3 Knowledge Assets y 3 Relationship Records creados antes de este ADR para conformarlos a la convención.
* Introduce una nueva carpeta raíz que debe recordarse al auditar la estructura del repositorio.

### Risks Mitigated

* Acoplamiento de nombres de archivo a identificadores externos mutables (códigos de escaneo OCR).
* Ambigüedad sobre dónde debe residir el conocimiento curado por el proyecto.
* Divergencia futura y silenciosa de convenciones de nombrado entre distintas fichas.

---

## References

* ADR-0033-Taxonomia_ADRs.md
* docs/Project_Definition/02_MASTER_ARCHITECTURE.md
* ai/templates/30_knowledge_asset.md
