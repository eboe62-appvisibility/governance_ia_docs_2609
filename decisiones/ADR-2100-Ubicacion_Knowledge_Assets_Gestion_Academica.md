# ADR-2100: Ubicación de Knowledge Assets de Gestión Académica Personal en asignaturas/000_Gestion/materiales/

Status: APPROVED

Date: 2026-09-23

Scope:
Documentation

Category:
DOCUMENTATION

Domain: (Optional)
Education, Governance

Tags:
knowledge-asset, repository-structure, gestion-academica, adr, standards

Related ADRs:
ADR-1100 (excepción parcial y acotada a su punto 1 — ubicación del "Knowledge Repository"); ADR-1101 (deslinde explícito — esa ADR regula naming de `fuentes/`, no la ubicación de Knowledge Assets, y corrige una cita errónea repetida en fichas previas); ADR-0033 (vocabulario `Domain`/`Category` que esta ADR sigue reutilizando sin cambios)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

`ADR-1100` establece `conocimiento/` como la única ubicación física del componente "Knowledge Repository" y descarta expresamente `asignaturas/` (árbol curricular oficial, carpetas 011-042) como lugar apropiado para alojar conocimiento curado por el proyecto: *"no son apropiados para alojar conocimiento curado por el propio proyecto"*.

Sin embargo, desde el 2026-09-15, por instrucción explícita del usuario —delegado de curso que usa este mismo repositorio para gestionar su propia vida académica (matrícula, horarios, normativa de permanencia y prácticas externas, microcredenciales, directorio de departamentos, guía de estudiantes UAM)— se han venido creando decenas de Knowledge Assets directamente en `asignaturas/000_Gestion/materiales/`, en lugar de `conocimiento/`. Esta práctica es coherente con la jerarquía de conflictos de `AI_ENTRYPOINT.md` (prioridad 1: instrucción explícita del usuario en la sesión), pero nunca se formalizó como decisión propia.

Como consecuencia de no existir un ADR dedicado, las fichas afectadas citan en su campo `Related ADRs` la referencia `ADR-1101 (ubicación excepcional en asignaturas/000_Gestion/materiales/)` — una cita **incorrecta**: `ADR-1101` regula en realidad la migración y convención de nombres de `fuentes/` (tildes, transliteración, códigos de canal), no la ubicación de Knowledge Assets. El error se ha repetido en, al menos, 15 fichas nuevas.

El contenido de estos Knowledge Assets no es "conocimiento curado" en el sentido de `ADR-1100` (saber transferible sobre lengua, historia o sociedad china/asiática) sino gestión administrativa y burocrática de la propia trayectoria universitaria del usuario dentro del Grado en Estudios de Asia y África — un dominio operativamente distinto ("Gestión") que, no obstante, comparte el mismo formato de Knowledge Asset (`ai/templates/30_knowledge_asset.md`) y el mismo estándar de evidencia trazable y `Confidence Level`.

---

## Decision

1. Se reconoce **"Gestión"** como un dominio de contenido distinto del "conocimiento curado" regulado por `ADR-1100`: trazabilidad administrativa y académica personal del usuario dentro de su propio Grado (matrícula, horarios, normativa, prácticas, formación complementaria, directorio de contactos).

2. Los Knowledge Assets de dominio Gestión se ubican en `asignaturas/000_Gestion/materiales/` — excepción explícita y acotada al punto 1 de `ADR-1100`, que sigue rigiendo sin cambios para el resto del repositorio.

3. Mantienen sin modificación: la plantilla `ai/templates/30_knowledge_asset.md`, la convención de nombres `YYMMDD-domain_principal-slug.md` de `ADR-1100` punto 3 (incluida la transliteración de `ñ` → `nn`), y el vocabulario `Domain`/`Category` de `ADR-0033`. La excepción es exclusivamente de **ubicación física**, no de forma ni de rigor evidencial.

4. Se inaugura la serie **ADR-2nnn** para decisiones de gobernanza propias del dominio Gestión, distinta de la serie ADR-1nnn (Knowledge Repository / conocimiento curado) — mismo criterio de reserva de rangos numéricos por área que ya separa ADR-0nnn (gobernanza transversal de los propios ADRs, `ADR-0033`) de ADR-1nnn.

5. Se corrige retroactivamente la cita a `ADR-1101` que aparece en el campo `Related ADRs` de los Knowledge Assets de `asignaturas/000_Gestion/materiales/` creados entre 2026-09-15 y esta fecha: debe sustituirse por `ADR-2100` en la próxima revisión de referencias cruzadas de esa carpeta (pendiente, no ejecutada por esta ADR).

---

## Consequences

### Positive

* Corrige una cita normativa errónea que se venía repitiendo en, al menos, 15 fichas.
* Formaliza una práctica ya consolidada durante más de una semana, dejando de depender de una referencia implícita a `AI_ENTRYPOINT.md` sin registro propio.
* Abre una serie numérica propia (2nnn) que evita seguir mezclando decisiones de Gestión personal dentro de la serie 1nnn, pensada para conocimiento curado transferible.

### Negative

* Introduce una segunda ubicación válida para Knowledge Assets (`conocimiento/` y `asignaturas/000_Gestion/materiales/`), lo que exige mantener claro el criterio de pertenencia (dominio Gestión vs. dominio conocimiento) para no confundir dónde debe ir cada ficha nueva.
* Requiere una pasada de corrección pendiente en las fichas ya existentes que citan `ADR-1101` incorrectamente (punto 5 de la Decision, no ejecutada aquí).

### Risks Mitigated

* Ambigüedad futura sobre por qué existen Knowledge Assets fuera de `conocimiento/`.
* Repetición del error de citar `ADR-1101` en fichas nuevas de Gestión.

---

## References

* ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md (deslinde)
* ADR-0033-Taxonomia_ADRs.md
* AI_ENTRYPOINT.md (jerarquía de conflictos, prioridad 1)
* ai/templates/30_knowledge_asset.md
