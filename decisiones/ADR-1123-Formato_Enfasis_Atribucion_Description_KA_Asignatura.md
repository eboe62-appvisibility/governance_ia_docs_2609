# ADR-1123: Formato de Énfasis, Atribución y Glosa en la Sección Description de Knowledge Assets de Sesión de Asignatura

Status: APPROVED

Date: 2026-10-08

Scope:
Knowledge

Category:
METHODOLOGY

Domain: (Optional)
Documentation, History, Research

Tags:
knowledge-asset, asignaturas, formato, atribucion, corriente-de-pensamiento, fidelidad-fuente, methodology

Related ADRs:
ADR-1120 (principio de fidelidad a la fuente que esta Decision extiende a un nuevo tipo de afirmación — la corriente de pensamiento — y a las glosas de términos no definidos; punto 1 de esa Decision ya exigía marcar como asunción cualquier dato de conocimiento general no provisto por la fuente), ADR-1100 (distinción entre `conocimiento/`, `conocimiento_west*/` y `asignaturas/*/conocimiento/` en la que se apoya el alcance de esta Decision), ADR-1105 (procedimiento general de elaboración de Knowledge Assets, al que esta Decision añade una regla de formato aplicable solo al subconjunto de fichas de sesión de asignatura), ADR-1110 (precedente de convención de plantilla respaldada por un ADR propio)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

El 2026-10-08, al revisar la sección `Description` de `asignaturas/011_17617_Introduccion_Historia/conocimiento/261006-history-sesion-01-introduccion.md` y `...-sesion-02-modernidad-occidental.md`, el usuario señaló tres carencias: (1) las ideas fuerza del documento original no quedaban enfatizadas visualmente; (2) no se avalaba cada idea, entre paréntesis, con el autor/figura en que se fundamenta y la corriente de pensamiento o filosófica en la que se sitúa (al menos esta última, en negrita); y (3) los conceptos que cada sesión va definiendo tampoco se remarcaban en negrita.

Una primera prueba sobre `sesion-01` resultó incompleta: se marcaron en negrita algunas ideas y conceptos pero no otros igual de centrales en el mismo párrafo (p. ej. «subjetividades constitutivas/proyectivas» sí, «escepticismo» y «pensamiento fuerte» no) — el usuario lo señaló directamente sobre el texto. Correlativamente, al extender la prueba a `sesion-02`, dos preguntas adicionales emergieron: qué hacer cuando un autor citado no tiene una corriente de pensamiento nombrada por la propia fuente (riesgo de inventarla, lo que ya prohíbe `ADR-1120` punto 1), y qué hacer con términos técnicos que la fuente usa sin definir (p. ej. «inmanencia», «epistemicida») que el usuario también pidió glosar.

La solución aplicada en la prueba — una nota de método explicativa (`> Nota de método (prueba): ...`) insertada al final de la sección `Description` de cada ficha — resolvió el problema puntualmente pero el usuario objetó dos cosas: (a) repetir la misma explicación de criterio en cada una de las 16 fichas de sesión es un patrón pobre, frente a fijar la regla una sola vez en un ADR; y (b) este nivel de densidad editorial (negrita de ideas fuerza, atribución de autor/corriente, glosas de conceptos) solo tiene sentido en las fichas de sesión de una asignatura concreta, ligadas 1:1 a una fuente escaneada y a un temario — no en los Knowledge Assets de conocimiento genérico del repositorio (`conocimiento/`, `conocimiento_west*/`, etc.), donde ese nivel de detalle sería desproporcionado frente al propósito recopilatorio de ese corpus.

---

## Decision

1. **Alcance**: esta Decision se aplica únicamente a los Knowledge Assets situados bajo `asignaturas/*/conocimiento/` cuyo `Knowledge Type` sea de resumen de sesión/clase (p. ej. "Session summary (learning resource)"), ligados 1:1 a una fuente concreta de la asignatura. No se aplica a Knowledge Assets de conocimiento genérico del repositorio (`conocimiento/`, `conocimiento_west*/`, `conocimiento_index/` u otro corpus transversal no ligado a una asignatura) — esos conjuntos mantienen el estilo ya definido por `ai/templates/30_knowledge_asset.md` sin la densidad editorial adicional de esta Decision.

2. **Énfasis de ideas fuerza**: en la sección `Description`, el enunciado central de cada párrafo o punto (la idea fuerza que el propio texto fuente defiende) se marca en negrita en su totalidad o en su núcleo argumental — no solo la primera palabra o una etiqueta. El criterio de qué marcar se aplica de forma exhaustiva dentro de un mismo párrafo: si un término pertenece a la misma familia conceptual que otro ya marcado en negrita en esa frase (p. ej. «pensamiento fuerte» junto a «subjetividades proyectivas»), también se marca — no se deja a mitad de camino.

3. **Atribución de autor y corriente de pensamiento**: inmediatamente después de la idea (entre paréntesis o en el cuerpo de la frase), se cita el autor/figura en que se fundamenta, cuando la fuente atribuye la idea a alguien. La corriente de pensamiento o escuela filosófica se añade en negrita **solo cuando la propia fuente la nombra explícitamente** para ese autor en esa sesión (p. ej. «crítico poscolonial» para Stuart Hall en `sesion-01`, «historia positivista» para Ranke) — nunca se infiere ni se completa con conocimiento general del asistente, por la misma razón de fondo que `ADR-1120` punto 1 prohíbe inventar denominaciones: presentar una clasificación no verificable con la misma confianza que un hecho de la fuente. Si la fuente no nombra una corriente para un autor citado, no se añade ninguna.

4. **Conceptos definidos en el capítulo**: los términos que la sesión introduce, define o usa como categoría técnica propia (p. ej. «afuera constitutivo», «subalternidad», «posmemoria», «oeconomia») se marcan en negrita en su primera aparición relevante dentro de `Description`.

5. **Glosa de términos no definidos por la fuente**: cuando un término técnico que la fuente usa sin definir (p. ej. «inmanencia», «epistemicida») dificulta la comprensión, se permite añadir una glosa breve entre paréntesis inmediatamente después del término en negrita, señalada explícitamente como adición editorial (p. ej. "nota editorial, no definida así por la fuente") — mismo mecanismo de marca de asunción que `ADR-1120` punto 1 exige para denominaciones de conocimiento general no provistas por la fuente, aplicado aquí a definiciones de concepto.

6. **Sin nota de método por ficha**: al quedar fijado el criterio de los puntos 2-5 en este ADR, ninguna ficha individual necesita repetir una nota explicativa del criterio aplicado (del tipo "Nota de método (prueba)" usado en la fase de prueba de esta Decision) — esta Decision es la referencia única. Una ficha puede citar `ADR-1123` en su campo `Related ADRs` si se quiere trazabilidad, pero no debe reexplicar la regla en el cuerpo de `Description`.

7. **Aplicación retroactiva limitada**: esta Decision no exige revisar retroactivamente todas las fichas de sesión ya existentes. Se aplica hacia adelante; una ficha de sesión ya elaborada se actualiza a este formato solo cuando se edita por otro motivo o en un lote explícitamente aprobado por el usuario — mismo criterio no retroactivo que `ADR-1120` punto 5 y `ADR-1100` punto 3.

---

## Consequences

### Positive

* Fija un criterio único y reutilizable para las 16+ fichas de sesión de cada asignatura, evitando que cada ficha reexplique (o aplique de forma inconsistente) el mismo criterio de énfasis y atribución.
* Mantiene los Knowledge Assets de conocimiento genérico del repositorio libres de una densidad editorial que no les corresponde, respetando la distinción de propósito ya establecida por `ADR-1100` entre `asignaturas/` y `conocimiento/`.
* Extiende el principio de fidelidad de `ADR-1120` (no inventar lo que la fuente no dice) a dos superficies nuevas — corriente de pensamiento y glosa de concepto — que no estaban explícitamente cubiertas por esa Decision.
* La prueba aplicada en `sesion-01` y `sesion-02` deja un precedente concreto y auditable de cómo se distingue una atribución sourced (Hall, Ranke) de una omitida por falta de respaldo en la fuente (Said, Bauman, Hegel, Nietzsche en esas sesiones concretas).

### Negative

* Añade trabajo editorial por ficha de sesión (identificar ideas fuerza, verificar si la fuente nombra una corriente, redactar glosas) frente al formato previo, más plano.
* El criterio "se marca en negrita todo lo que pertenece a la misma familia conceptual en la frase" (punto 2) exige juicio humano/del asistente en cada caso — no es puramente mecánico, y una revisión posterior puede encontrar inconsistencias puntuales igual que ocurrió en la prueba inicial.
* No elimina el riesgo de que una corriente de pensamiento *sí* nombrada por la fuente sea, a su vez, una simplificación o etiqueta cuestionable en origen — esta Decision solo garantiza fidelidad a lo que la fuente dice, no la exactitud última de esa clasificación (mismo límite que `ADR-1120` reconoce para su propio alcance).

### Risks Mitigated

* Corrientes de pensamiento o escuelas filosóficas atribuidas a un autor por iniciativa del asistente, sin respaldo en la fuente, presentadas con la misma confianza que un hecho verificado.
* Definiciones de concepto fabricadas y presentadas como si procedieran de la fuente, cuando en realidad son conocimiento general del asistente.
* Aplicación inconsistente del criterio de énfasis dentro de una misma ficha (ideas de la misma familia conceptual marcadas de forma desigual).
* Repetición de la misma explicación de criterio en cada una de las fichas de sesión de una asignatura, en vez de una única referencia de gobernanza.
* Extensión indebida de esta densidad editorial a Knowledge Assets de conocimiento genérico del repositorio, donde no está justificada por su propósito recopilatorio.

---

## References

* ADR-1120-Verificacion_Denominaciones_y_Enlaces_Internos_Knowledge_Assets.md (principio de fidelidad que esta Decision extiende)
* ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md (distinción `asignaturas/` vs. `conocimiento/`/`conocimiento_west*/` en la que se apoya el alcance, punto 1)
* ADR-1105-Procedimiento_Elaboracion_Knowledge_Assets.md (procedimiento general de elaboración al que esta Decision añade una regla de formato acotada)
* ai/templates/30_knowledge_asset.md (plantilla base; esta Decision no la modifica, añade una convención de redacción para un subconjunto de fichas)
* asignaturas/011_17617_Introduccion_Historia/conocimiento/261006-history-sesion-01-introduccion.md (primera prueba — corrección de inconsistencia de énfasis señalada por el usuario)
* asignaturas/011_17617_Introduccion_Historia/conocimiento/261006-history-sesion-02-modernidad-occidental.md (segunda prueba — caso de glosa de conceptos y de ausencia de corriente sourced)
