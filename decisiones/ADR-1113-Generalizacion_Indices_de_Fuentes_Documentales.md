# ADR-1113: Generalización de Índices de Fuentes de "Periódicas" a "Documentales" y Política de Renombrado en Revisiones Mayores

Status: APPROVED

Date: 2026-08-07

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Research, Documentation

Tags:
knowledge-asset, knowledge-repository, indice-corpus, fuentes-documentales, taxonomy

Related ADRs:
ADR-1100 (convención de nombres de Knowledge Asset — se añade aquí la política de renombrado en revisiones mayores, no cubierta por ese documento), ADR-1101 (ubicación física de `fuentes/`), ADR-1108 (mecánica de tabla + identificador de bloque por entrada — se generaliza aquí a índices que agregan tipos de fuente heterogéneos), ADR-1111 (criterios de interés para la curación de conocimiento — confirma que las fuentes documentales incorporadas por este Architectural Decision Record cumplen el Criterio B), ADR-1112 (prohibición de siglas propias del proyecto en prosa — no aplica a siglas de entidades externas citadas por su propio nombre, como se aclara en Context), ADR-1118 (no proliferación de índices sobre el mismo tipo de entidad — motivó mantener la sexta tabla dentro de este mismo fichero en vez de un índice nuevo, ampliación 2026-08-27), ADR-1119 (contraste externo — los tres casos reales que motivaron la sexta tabla, ampliación 2026-08-27)

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Ampliación (2026-08-27) — Tabla de fuentes de contraste no centradas en estudios chinos

Tras tres contrastes externos sucesivos (`ADR-1119`: CXMT, Evergrande, misiles nucleares chinos), el mismo grupo de fuentes de prensa/análisis occidentales generalistas (CNBC, Bloomberg, CNN, The Diplomat, USNI News, CSIS, entre otras) se descartó repetidamente del índice por no encajar en su `Scope` ("fuentes... sobre estudios chinos"). El usuario señaló que esa exclusión, aunque correcta, estaba desperdiciando información real y reutilizable — en particular, qué dominios bloquean `WebFetch` (CNN, The Diplomat, CNBC y USNI News devolvieron 403/451 en intentos reales durante el contraste de misiles) y qué fuentes occidentales especializadas se repiten entre contrastes (CNBC y CNN en dos contrastes distintos, Bloomberg en dos).

Se evaluó explícitamente crear un índice nuevo separado para esto, y se descartó por el mismo motivo que ya rechazó esa vía en el Context original de este Architectural Decision Record: fragmentar el conocimiento sobre fuentes documentales en varios índices estrechos, y por el criterio de `ADR-1118` de evitar proliferación de índices sobre el mismo tipo de entidad sin una razón de mantenimiento genuinamente distinta.

Se añade como ampliación del punto 1 de esta Decision (alcance versátil por defecto): el `Scope` de `260806-index-documentation-fuentes-curadas.md` se amplía con una **sexta tabla explícitamente fuera del alcance "sobre estudios chinos"** — *"Fuentes de contraste no centradas en estudios chinos"* —, con un criterio de entrada propio y distinto del resto del índice, para no convertirla en un registro sin filtro de cualquier URL citada una vez:

* Entra una fuente cuando **(a)** se ha reutilizado en más de un contraste `ADR-1119` distinto, **o (b)** es especializada/autoritativa en un dominio técnico relevante para el corpus (p. ej. una base de datos de referencia sobre armamento, no un diario generalista) — no por haber sido citada una sola vez para un dato puntual.
* Cada fila documenta, cuando se conoce, su accesibilidad real vía `WebFetch` (bloqueada/accesible, con código de estado si se probó) — el dato con más valor de reutilización futura, evita reintentar fuentes ya confirmadas como bloqueadas.
* Ver el detalle completo de columnas y primeras filas en `Key Characteristics`/`Description` de `260806-index-documentation-fuentes-curadas.md`.

---

## Context

El 2026-08-06 se creó `conocimiento_index/260806-index-documentation-fuentes-periodicas-curadas.md`, un Knowledge Asset de tipo Index que cataloga los medios de prensa periódica (diarios, suplementos, revistas) ya incorporados al corpus mediante su propia carpeta transcrita en `fuentes/`.

El 2026-08-07, al investigar fuentes documentales para el estudio de China (enciclopedias colaborativas, bases de datos académicas, medios de la diáspora china, instituciones culturales y académicas en Madrid — conversación recogida en `fuentes/260807_Geminy/260807_Geminy_Enciclopedias.md`, contrastada y ampliada en `fuentes/260807_Geminy/260807_Geminy_Enciclopedias_Contraste.md`), se planteó inicialmente crear un segundo índice nuevo para estas fuentes. El usuario rechazó esa vía explícitamente: prefiere generalizar el índice ya existente, aunque suponga renombrarlo, redefinir su propósito y ampliar su esquema de columnas, en lugar de fragmentar el conocimiento sobre fuentes documentales en varios índices estrechos.

El usuario señaló además que ya había pedido, al crear el primer índice, que los índices se diseñaran con vistas a su evolución futura — indicación que no quedó reflejada en aquel momento. El proyecto está en una fase temprana; el usuario quiere establecer desde ahora una estructura de índice de fuentes documentales lo bastante general como para no repetir este mismo ciclo de renombrado cada vez que aparezca un nuevo tipo de fuente (bases de datos académicas, redes sociales, instituciones, etc.).

Esto plantea dos preguntas que ningún Architectural Decision Record previo resuelve:

1. ¿Cómo debe generalizarse un índice ya existente para admitir tipos de fuente heterogéneos (algunos incorporados como transcripción propia en `fuentes/`, otros solo documentados mediante investigación) sin perder la trazabilidad de lo ya construido?
2. Cuando una revisión mayor de un Knowledge Asset canónico implica cambiar su nombre de archivo, ¿qué ocurre con el `YYMMDD` de la convención de nombres de `ADR-1100`? Ese Architectural Decision Record define la fecha de creación, no la política de renombrado en una revisión posterior.

Sobre las siglas de entidades citadas en el índice resultante (p. ej. `CNKI`, `NSSD`, `GISEC`, `SISU`): `ADR-1112` prohíbe siglas que abrevian un término **propio del proyecto** (`Knowledge Asset` → "KA"), no el nombre propio con el que una entidad externa se autodenomina. Este Architectural Decision Record no altera ni necesita reinterpretar esa distinción; se cita aquí solo para dejar constancia expresa de que no hay conflicto.

---

## Decision

1. **Principio general — alcance versátil por defecto**: al definir el `Purpose` y `Scope` de un nuevo Knowledge Asset de tipo Index, se adopta el alcance razonablemente más amplio de la categoría que representa, no el alcance implícito en el primer lote de datos disponible en el momento de crearlo. Un índice se restringe a un subtipo concreto solo cuando existe una razón de fondo para mantenerlo separado (propósito, audiencia o mecánica de mantenimiento genuinamente distintos), documentada explícitamente — nunca por omisión.

2. **Caso concreto — generalización del índice de fuentes**: `conocimiento_index/260806-index-documentation-fuentes-periodicas-curadas.md` se renombra a `conocimiento_index/260806-index-documentation-fuentes-curadas.md` y su `Purpose`/`Scope` se redefinen de "medios periódicos incorporados al corpus" a "fuentes documentales sobre estudios chinos — incorporadas o simplemente referenciadas/investigadas". El fichero conserva su identidad (no se trata de un Knowledge Asset nuevo con `Supersedes`): mismo `YYMMDD` de archivo, mismas filas y mismos identificadores de bloque ya existentes, sin renumerar.

3. **Columna `Estado`, mecanismo general para índices de fuentes mixtas**: se añade una columna `Estado` con dos valores posibles — `Incorporada` (existe una carpeta propia transcrita en `fuentes/`) o `Referenciada` (documentada mediante investigación, sin transcripción propia todavía). Este mecanismo es reutilizable por cualquier índice futuro que combine ambos grados de incorporación, no exclusivo de este caso.

4. **El identificador de bloque no cambia de acrónimo**: conforme al criterio ya fijado por `ADR-1108` punto 5 para índices derivados de agregación de corpus (acrónimo derivado de la raíz de agregación, no de cada fuente individual), el acrónimo `FNT` (de "fuentes") se mantiene para todas las filas del índice generalizado, incluidas las que documentan fuentes solo "Referenciadas" — su columna `Fuente` sigue apuntando siempre dentro del árbol `fuentes/` (a la carpeta transcrita, o al fichero de investigación curada), por lo que la raíz de agregación no cambia.

5. **Política de renombrado en revisiones mayores de un Knowledge Asset canónico (nueva — no cubierta por `ADR-1100`)**: cuando una revisión mayor redefine el propósito/alcance de un Knowledge Asset ya existente hasta el punto de requerir un nuevo `slug` de archivo, el segmento `YYMMDD` **se mantiene igual al de creación original** — no se actualiza a la fecha de la revisión. Justificación: `YYMMDD` identifica la identidad/fecha de origen del Knowledge Asset, no la de cada edición; actualizarlo en cada revisión mayor rompería esa semántica y la distinción con el mecanismo real para "esto ya no es la misma ficha" (`Supersedes`/`Superseded By`, reservado para cuando se crea un Knowledge Asset genuinamente nuevo que reemplaza al anterior, no cuando el mismo fichero evoluciona). La fecha de la revisión mayor, si se considera valiosa, se documenta en la sección `Notes` del propio Knowledge Asset, no en el nombre de archivo.

6. **Fuentes documentales cubiertas por esta generalización cumplen `ADR-1111`**: las fuentes añadidas (enciclopedias chinas, bases de datos académicas chinas, medios de la diáspora, instituciones culturales/académicas sobre China en Madrid) tratan directamente sobre China o la comunidad china — Criterio B de `ADR-1111` — sin necesidad de invocar ninguna excepción.

---

## Consequences

### Positive

* Evita repetir, para cada tipo nuevo de fuente documental, el mismo ciclo de "crear índice estrecho → descubrir que hace falta uno más general → renombrar y refactorizar" ya vivido en este caso.
* Fija por primera vez una política de renombrado para revisiones mayores de Knowledge Assets canónicos, ausente en `ADR-1100`.
* La columna `Estado` (punto 3) es un mecanismo reutilizable, no ad hoc — cualquier índice futuro que mezcle fuentes incorporadas y solo referenciadas puede adoptarlo sin un Architectural Decision Record adicional.
* Dado que ningún otro fichero del repositorio enlazaba a `260806-index-documentation-fuentes-periodicas-curadas.md` (verificado por búsqueda literal antes de esta decisión) y que los Knowledge Assets de tipo Index quedan excluidos de la bóveda de Obsidian por `ADR-1109` §5, el renombrado no rompe ningún enlace interno ni grafo de Obsidian existente.

### Negative

* El índice generalizado pierde la precisión semántica de "solo periódicas" en su propio nombre — mitigado por la columna `Tipo` (ya existente) y la nueva `Estado`, que permiten filtrar por subtipo sin necesidad de un nombre de archivo restrictivo.
* Introduce una distinción (`Incorporada` / `Referenciada`) que debe mantenerse coherente manualmente conforme una fuente "Referenciada" pase a tener su propia carpeta transcrita — no hay verificación automática (no hay pipeline de CI en este proyecto).

### Risks Mitigated

* Proliferación futura de índices estrechos y solapados en `conocimiento_index/` (p. ej. uno por cada tipo de fuente documental) que habría sido difícil de mantener coherente.
* Confusión futura sobre si una revisión mayor de un Knowledge Asset debe tratarse como el mismo fichero renombrado o como un fichero nuevo con `Supersedes` — el punto 5 fija el criterio de forma explícita antes de que aparezca una divergencia real entre sesiones.

---

## References

* conocimiento_index/260806-index-documentation-fuentes-curadas.md (Knowledge Asset generalizado por esta decisión; antes `260806-index-documentation-fuentes-periodicas-curadas.md`)
* fuentes/260807_Geminy/260807_Geminy_Enciclopedias.md, fuentes/260807_Geminy/260807_Geminy_Enciclopedias_Contraste.md (fuentes documentales que motivaron esta generalización)
* docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* docs/decisiones/ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md
* docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md
* docs/decisiones/ADR-1109-Distincion_Grafo_Obsidian_vs_Grafo_Understand_Anything.md
* docs/decisiones/ADR-1111-Criterios_Interes_Curacion_Conocimiento.md
* docs/decisiones/ADR-1112-Prohibicion_Siglas_en_Documentos_del_Repositorio.md
