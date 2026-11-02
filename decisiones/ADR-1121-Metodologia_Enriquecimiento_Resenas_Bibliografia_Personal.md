# ADR-1121: Metodología de Enriquecimiento de Reseñas de Bibliografía Personal con Fuentes Externas

Status: APPROVED

Date: 2026-08-31

Scope:
Knowledge

Category:
METHODOLOGY

Domain (Optional):
Documentation, Research, History

Tags:
resena-bibliografia, enriquecimiento-fuentes, agent-browser, verificacion-cruzada, afirmaciones-a-contrastar

Related ADRs:
ADR-1101 (nota 2026-08-31 — ubicación y convención de nombres de `fuentes/260830_bibliografia_propia/{md,docx}/`; esta Decision es metodología de contenido, no de nombrado, deliberadamente separada por el mismo criterio que separó `ADR-1116` de `ADR-1115`), ADR-1115 (agent-browser como motor de acceso, reutilizado aquí sin la lista de fuentes vetadas — ver punto 3), ADR-1116 (precedente directo: metodología formalizada en ADR tras aplicación práctica repetida, no diseñada de antemano), ADR-1119 (verificación obligatoria por lectura directa antes de citar — precondición de toda esta Decision), ADR-1120 (verificación de denominaciones y enlaces antes de redactar Knowledge Assets — mismo espíritu aplicado aquí a afirmaciones de una fuente externa, no solo a nombres/enlaces internos)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

`fuentes/260830_bibliografia_propia/{md,docx}/` reúne 64 reseñas de lectura personal (citas literales transcritas por el usuario, con ficha bibliográfica mínima: Autor, Libro, Editorial, Biblioteca, Signatura). El usuario planteó enriquecer estos archivos con contexto externo fiable para construir Knowledge Assets — no limitarse a las citas ya transcritas.

Un piloto sobre dos casos (`000924_SUN-TZU_arte-guerra` y `970918_HEMINGWAY_enviado-especial`) reveló tres problemas que una aplicación ad hoc, archivo a archivo, no resolvería de forma consistente en los 64 casos:

1. **Fuentes externas no siempre concuerdan entre sí.** Al enriquecer la reseña de Hemingway con *Hemingway on the China Front* (Peter Moreira), se presentó inicialmente su tesis de que Hemingway actuó como espía del Tesoro de EE. UU. como si fuera un hecho asentado. El usuario señaló la desproporción; una segunda búsqueda localizó una reseña independiente del mismo libro (The Washington Times, 2006) que la matiza como basada en "evidencia documental escasa". Sin un procedimiento explícito de contraste, una afirmación llamativa de una sola fuente externa puede colarse en un KA con más autoridad de la que realmente tiene.
2. **Una cita del propio archivo original puede ser igual de arriesgada.** El usuario destacó la frase "Salvo el español, ningún pueblo tiene más amor propio que el chino" (pág. 240) como especialmente útil para el proyecto. Al proponer un KA, se le dio protagonismo de título — el usuario corrigió: es una afirmación de fuente única, en traducción española, sin contrastar, y no debería pesar más que eso hasta corroborarse. Tres búsquedas y dos fuentes externas revisadas no lograron localizar el pasaje en el inglés original (*By-Line*) ni citado por terceros — se documenta como "no verificable con el esfuerzo razonable disponible", ni verdadera ni falsa.
3. **Sin una estructura fija, cada archivo reinventaba el formato.** Las primeras versiones de la reseña de Hemingway mezclaban en prosa libre metadatos de edición, síntesis externa y contraste de afirmaciones, cambiando de forma en cada revisión. El usuario pidió explícitamente una composición uniforme extrapolable a los 62 archivos restantes.

Un cuarto ajuste, menor pero con impacto en el patrón de archivos de `ADR-1101`: la reseña enriquecida termina absorbiendo también las citas textuales originales, por lo que el archivo de reseña sustituye al archivo de citas bajo el mismo nombre — un fichero por libro, no dos.

---

## Decision

1. **Estructura canónica de documento único**, aplicada al mismo archivo `AAMMDD_AUTOR_titulo-slug.md`/`.docx` que ya fija `ADR-1101` (nota 2026-08-31) — no se crea un archivo `_reseña` aparte; el enriquecimiento sustituye al archivo de citas original bajo el mismo nombre. Plantilla: `ai/templates/04_resena_bibliografia.md`. Secciones fijas, en este orden:
   1. **Ficha bibliográfica** (tabla): fusiona los campos originales del usuario (Autor, Libro, Editorial, Biblioteca, Signatura) con los verificados externamente (Año, Páginas, Obra original, Traductor). Un dato no verificado por lectura directa se marca como tal, nunca se omite silenciosamente ni se asume (`ADR-1119`).
   2. **Fuentes externas consultadas** (bloque de cita): lista numerada con URL, fecha de consulta y herramienta usada; incluye cualquier advertencia de verificación relevante (falsos positivos descartados, homonimias — ver punto 3).
   3. **Citas textuales**: el contenido original del usuario, verbatim, con página. Antes de esta Decision vivía en un archivo separado; ahora es una sección de este mismo documento.
   4. **Contexto y recepción de la obra**: síntesis de las fuentes externas, con inferencias propias marcadas explícitamente como tales cuando la fuente no las confirma directamente.
   5. **Notas del usuario sobre pasajes destacados**: sección reservada. El asistente de IA no genera contenido aquí — es space explícitamente dejado para el criterio del usuario, distinto tanto de las citas transcritas como de la síntesis externa.
   6. **Afirmaciones a contrastar** (tabla): cualquier afirmación llamativa, sea del archivo original o de una fuente externa, con columnas `Afirmación / Fuente que la sostiene / Estado / Nota`. Tres estados posibles: **Verificada** (≥2 fuentes independientes concuerdan), **Contestada** (una fuente la afirma, otra la matiza o refuta — se documentan ambas, no se elige una), **No verificable** (esfuerzo razonable sin resultado — nunca se traduce en "falsa", solo en "sin respaldo localizado con las herramientas disponibles ahora").

2. **Selección de fuente externa — sin lista cerrada, a diferencia de `ADR-1115`/`ADR-1116`.** Esas Decisions vetan fuentes para investigación temática en chino sobre China; esta bibliografía cubre libros generales en varios idiomas y épocas, un universo de fuentes distinto. Criterio de prioridad: (a) una reseña específica de la obra concreta, frente a (b) contexto genérico del autor como fallback cuando no existe reseña específica. Antes de citar cualquier fallback genérico (p. ej. Wikipedia), verificar que no hay colisión de título por homonimia — caso real detectado: "Enviado especial" en Wikipedia ES es, por homonimia, la película *Foreign Correspondent* de Hitchcock (1940), no el libro de Hemingway.

3. **Procedimiento de contraste obligatorio para afirmaciones llamativas**: antes de dar peso destacado a cualquier afirmación (comparativa cultural, dato biográfico inusual, tesis historiográfica) — venga del archivo original del usuario o de una fuente externa —, intentar corroborarla con al menos una fuente independiente. El resultado del intento (éxito, matización, o fracaso tras esfuerzo razonable) se documenta siempre en la tabla del punto 1.6, nunca se omite. Este procedimiento aplica con el mismo rigor a una cita que el propio usuario destaque como especialmente valiosa — el origen de la afirmación (lectura propia vs. fuente externa) no exime del contraste.

4. **Proporcionalidad del título de un Knowledge Asset derivado**: cuando una reseña enriquecida alimenta un Knowledge Asset en `conocimiento/`, una afirmación en estado Contestada o No verificable no debe ocupar el título del KA ni presentarse como su tesis central — el título refleja el contenido verificado (el reportaje, el evento, el testimonio), y la afirmación dudosa se documenta dentro, con su estado explícito.

5. **Alcance**: aplica a los 64 archivos de `fuentes/260830_bibliografia_propia/{md_new,docx_new}/` que son reseñas de lectura genuinas según `ADR-1101` (nota 2026-08-31) — no a los 8 excluidos por no ser reseñas de libro (película, documental, conferencia, exposición, notas personales sin obra real).

---

## Consequences

### Positive

* Una estructura fija, extrapolable a los 62 archivos restantes sin reinventar el formato cada vez — motivo explícito de esta Decision, pedido por el usuario.
* El procedimiento de contraste (punto 3) queda documentado como regla permanente, no como corrección puntual de un caso — futuros enriquecimientos heredan la lección del caso Hemingway/Moreira automáticamente.
* Un fichero por libro, no dos — reduce la carga de mantenimiento y el riesgo de que las citas originales y la reseña enriquecida diverjan silenciosamente.

### Negative

* Cada afirmación llamativa exige al menos un intento de corroboración externa antes de poder documentarse con peso — más lento que citar la primera fuente encontrada, deliberadamente, para evitar el tipo de sobre-peso ya detectado en el caso Hemingway.
* La consolidación de archivos (punto 1) es un cambio respecto al patrón inicial de esta misma sesión de trabajo (que sí mantenía `_reseña.md` separado) — los dos primeros archivos tocados tuvieron que reestructurarse dos veces.

### Risks Mitigated

* Que una tesis historiográfica de una sola fuente externa (o una cita llamativa del propio archivo del usuario) se presente en un Knowledge Asset con más autoridad de la que realmente tiene — mitigado por el procedimiento de contraste obligatorio y la regla de proporcionalidad de título (puntos 3 y 4).
* Que el enriquecimiento de las 64 reseñas derive en 64 formatos distintos, dificultando su uso homogéneo al construir Knowledge Assets — mitigado por la estructura canónica del punto 1.
* Que aplicar sin más el criterio de fuentes vetadas de `ADR-1115`/`ADR-1116` (pensado para fuentes chinas) descarte fuentes legítimas para literatura general, o que se acepte sin más una fuente genérica cuando existe una específica mejor — mitigado por el criterio de prioridad del punto 2.

---

## References

* `docs/decisiones/ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md` (nota 2026-08-31 — ubicación y nombrado de los mismos archivos)
* `docs/decisiones/ADR-1115-Adopcion_Agent_Browser_Web_Monitoring.md`
* `docs/decisiones/ADR-1116-Metodologia_Evaluacion_Comparativa_Fuentes.md`
* `docs/decisiones/ADR-1119` (verificación obligatoria por lectura directa)
* `docs/decisiones/ADR-1120` (verificación de denominaciones y enlaces antes de redactar Knowledge Assets)
* `ai/templates/04_resena_bibliografia.md` (plantilla que implementa esta Decision)
* Primera aplicación real: `fuentes/260830_bibliografia_propia/md_new/970918_HEMINGWAY_enviado-especial.md`, y el Knowledge Asset derivado `conocimiento/260831-history-hemingway-china-1941.md`
