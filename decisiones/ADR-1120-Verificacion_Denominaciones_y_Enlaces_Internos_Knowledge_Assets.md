# ADR-1120: Verificación de Denominaciones y Enlaces Internos al Redactar Knowledge Assets

Status: APPROVED

Date: 2026-08-29

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Documentation, Research

Tags:
knowledge-asset, alucinacion, verificacion-fuentes, indice-corpus, taxonomy, enlaces-markdown, navegabilidad, autores-figuras-mencionados, plantilla

Related ADRs:
ADR-1100 (ubicación de `conocimiento/` y convención de nombres — plantilla `ai/templates/30_knowledge_asset.md` que esta Decision amplía), ADR-1108 (patrón de índice derivado de corpus, incluidos los códigos de bloque `^YYMMDD-xxx-NNNNN` cuya atribución indebida motiva el punto 2 de esta Decision), ADR-1110 (precedente directo: convención de plantilla respaldada por un ADR propio, mismo patrón aplicado aquí), ADR-1119 (principio hermano: verificar antes de citar — allí aplicado a `WebSearch`/`WebFetch`/`agent-browser`, aquí a la integridad interna del propio repositorio: denominaciones y enlaces que no dependen de ninguna fuente externa para poder comprobarse)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

El 2026-08-23, al crear `conocimiento/260823-technology-economics-cxmt-debuto-bolsa-shanghai.md` a partir de la transcripción de un vídeo de YouTube sobre el debut bursátil de CXMT, el asistente de IA introdujo tres anomalías que el usuario detectó y señaló en la revisión posterior:

1. **Nombre inventado**: la ficha atribuía a CXMT la denominación china "华微电子" — nombre real de una empresa china distinta (China Resources Microelectronics), no de CXMT (长鑫存储, ChangXin Memory Technologies). El vídeo fuente no mencionaba ningún nombre en chino; el asistente lo añadió por iniciativa propia, presentándolo como si formara parte del hecho verificado.
2. **Enlace a un índice inexistente**: la sección `Related Assets` citaba `conocimiento_index/260802-index-technology-semiconductores.md` como si fuera una entrada real del corpus. Ese fichero nunca existió — el asistente lo dio por hecho a partir de un patrón de nombrado plausible, sin comprobar el directorio real.
3. **Código de bloque atribuido incorrectamente**: la misma sección enlazaba `^260806-FNT-00001` como si correspondiera a "Negocios TV" (el canal del vídeo fuente); ese bloque corresponde en realidad a China Book International, una fuente completamente distinta ya registrada en `conocimiento_index/260806-index-documentation-fuentes-curadas.md`. Además, el propio vídeo de YouTube nunca había sido incorporado a ese índice — no existía ninguna fila a la que enlazar.

Ninguna de las tres anomalías depende de una fuente externa dinámica (a diferencia del caso que motivó `ADR-1119`, un resumen fabricado por `WebSearch` sobre una página real): las tres eran comprobables de inmediato contra el propio repositorio local, con un simple `ls`/`grep`, sin necesidad de ninguna herramienta de red. `ADR-1119` no cubre este caso — su alcance está explícitamente acotado a "Verificación Obligatoria del Contenido Real de la Fuente (**WebFetch**) antes de Citar un Resumen de **WebSearch**"; forzar esta lección dentro de esa Decision desalinearía su título con su contenido.

Tras corregir la ficha, el usuario pidió una propuesta de actualización de normas para evitar la repetición del problema. La respuesta inicial del asistente editó directamente `ai/templates/30_knowledge_asset.md` con tres reglas nuevas y permanentes, sin el respaldo de ningún ADR — rompiendo el propio patrón que el proyecto ya sigue para convenciones de plantilla duraderas: la sección `## Glossary` de esa misma plantilla está explícitamente respaldada por `ADR-1110` (*"This section is a project-level convention (ADR-1110)"*), y `ADR-1100` documenta que la plantilla es anterior a cualquier ADR, no un sustituto de ellos. El `Proportionality Principle` de `AI_ENTRYPOINT.md` clasifica como de impacto medio cualquier cambio que "establish a lasting rule", exigiendo entonces un ADR — las tres reglas introducidas encajan en esa categoría.

El 2026-10-08, el usuario reportó que los enlaces de las 16 fichas `conocimiento/261006-history-sesion-*.md` (Introducción a la Historia) "no estaban funcionando": Ctrl+click/Ctrl+Enter en VS Code no navegaba a los documentos fuente. La revisión encontró dos anomalías distintas:

1. **Ruta relativa incorrecta**: la referencia al índice de bibliografía usaba `conocimiento_index/261004-....md`, sin el `../` necesario — `conocimiento_index/` es hermana de `conocimiento/`, no está anidada en ella. El punto 2 de esta Decision (verificar existencia antes de citar), de haberse aplicado con rigor en su momento, habría detectado el fallo; el incidente es en parte una reincidencia de proceso, no solo un hueco nuevo de gobernanza.
2. **Sintaxis no navegable**: tanto esa referencia como la de "Fuente principal" (PDF + transcripción) estaban escritas entre comillas de código (`` `ruta` ``) en vez de con sintaxis de enlace Markdown (`[texto](ruta)`). Una ruta correcta escrita como código inline sigue sin ser clicable para el editor — el punto 2 exige verificar que el destino *existe*, pero no exige que la referencia esté *escrita como enlace*. Este segundo punto no estaba cubierto por ninguna Decision existente.

---

## Decision

1. **Denominación de entidades**: al redactar la sección `Description`/`Key Characteristics`/`Evidence and Sources` de un Knowledge Asset, la denominación de una entidad (nombre propio, traducción, transliteración, identificador) se toma tal como la proporciona la fuente citada. No se infiere, traduce ni completa un nombre en otro idioma, escritura o forma cuando la fuente citada no lo proporciona — hacerlo, aunque parezca un enriquecimiento razonable, introduce una afirmación no verificable presentada con la misma confianza que el resto del hecho. Cuando el nombre en otro idioma/escritura es conocimiento general del asistente y no consta en la fuente, se marca explícitamente como asunción en `Evidence and Sources` (mismo mecanismo ya exigido por la plantilla para evidencia vs. asunción), nunca se integra sin marca en el cuerpo de la ficha.

2. **Enlaces y códigos de bloque hacia recursos del propio repositorio**: antes de citar en `Related Assets` (o en cualquier otra sección) un enlace a un índice de `conocimiento_index/`, otra Knowledge Asset, un ADR, o un código de bloque específico (`^YYMMDD-xxx-NNNNN`, `ADR-1108`), se verifica su existencia real mediante listado de directorio o búsqueda de texto en el repositorio — nunca se asume a partir de un patrón de nombrado recordado de otras fichas o de un título plausible. Un código de bloque solo se cita cuando la fuente en cuestión está efectivamente registrada en esa fila del índice referenciado; si la fuente no ha sido incorporada a ningún índice, se documenta ese hecho explícitamente en vez de enlazar igualmente. Si tras verificar persiste una duda genuina sobre la existencia de un recurso, se omite el enlace en vez de incluirlo sin confirmar.

3. **Checklist de calidad previo al cambio de `Status`**: `ai/templates/30_knowledge_asset.md` incorpora una sección `## Quality Checklist`, a completar antes de que el `Status` de una ficha deje de ser `DRAFT`. Cubre, como mínimo: verificación de hechos concretos contra la fuente citada, distinción explícita evidencia/asunción, verificación de existencia de cada enlace interno citado, ausencia de denominaciones inventadas, ubicación del enriquecimiento contextual (solo en `Purpose` o marcado como asunción), y cumplimiento de la convención de nombrado aplicable (`ADR-1100`/`ADR-1101`). El checklist permanece visible en la ficha terminada — documenta que la verificación ocurrió, no solo que era posible realizarla.

4. **Relación con `ADR-1119`**: esta Decision y `ADR-1119` comparten el mismo principio general ("verificar antes de afirmar como hecho"), aplicado a dos superficies distintas y complementarias — `ADR-1119` a fuentes externas dinámicas accedidas vía `WebSearch`/`agent-browser`; esta Decision a la integridad interna del propio repositorio (denominaciones y enlaces verificables localmente, sin depender de ninguna herramienta de red). No se fusionan en una sola Decision para mantener cada título alineado con su alcance real, mismo criterio ya aplicado en el proyecto para separar decisiones de herramienta concreta (`ADR-1109`, `ADR-1114`, `ADR-1115`) de las de gobernanza general que las motivan.

5. **Aplicación retroactiva limitada**: esta Decision no exige una revisión retroactiva de todas las Knowledge Assets ya existentes. Se aplica hacia adelante a partir de su aprobación; una ficha antigua se corrige bajo este criterio solo si se audita o edita por otro motivo, mismo criterio de aplicación no retroactiva ya usado por el punto 3 de `ADR-1100` para los slugs con transliteración de `ñ` pendientes de refactorización.

6. **Cobertura de entidades nombradas en la fuente**: toda persona, autor, obra o institución nombrada en la transcripción de la fuente que un Knowledge Asset trate debe aparecer en la ficha, con su denominación tal como la proporciona la fuente, o quedar excluida de forma explícita con su motivo. Se incluyen las menciones que aparecen solo en notas al pie o en la sección de fuentes. Antes de que el `Status` deje de ser `DRAFT`, la comprobación se hace contra la transcripción mediante búsqueda de nombres, no de memoria. Las obras que una entidad aporta a la ficha se trasladan al índice de bibliografía de la asignatura cuando sean obras citadas o recomendadas en la fuente; una entidad sin obra citada no genera fila en el índice.

7. **Sintaxis de enlace para referencias navegables**: toda referencia a un documento del propio repositorio que se presente como destino a consultar (fuente principal/transcripción en `Evidence and Sources`, entradas de `Related Assets`, índices citados) se escribe con sintaxis de enlace Markdown, `[texto](ruta_relativa)`, nunca solo entre comillas de código. Las comillas de código se reservan para nombres de fichero o rutas mencionados sin intención de que se naveguen (p. ej. citar el nombre de un PDF de forma incidental dentro de una frase). La ruta relativa se calcula y verifica desde la ubicación real del fichero que la contiene — no se asume por analogía con otra ficha — y la verificación de existencia exigida por el punto 2 se hace sobre esa ruta ya resuelta, para no repetir el fallo de 2026-10-08 (ruta sin el `../` correcto).

8. **Formato estructural de la cobertura del punto 6 (añadido 2026-10-18)**: cuando una Knowledge Asset documenta una fuente narrativa con varias entidades nombradas (persona, autor, obra, institución, o figura mitológica/literaria tratada como parte sustantiva del contenido de la fuente), la cobertura exigida por el punto 6 se materializa en una sección `## Autores y figuras mencionados`, situada entre `Scope` y `Evidence and Sources`, como tabla Markdown de cuatro columnas: `| Nombre | Fechas (según la fuente) | Papel en el texto | Obras citadas |`. Cada fila usa la denominación y las fechas tal como las da la fuente (punto 1); los campos ausentes se marcan explícitamente como tales ("no constan en la fuente"), nunca se dejan en blanco ni se completan por conocimiento general sin marcarlo. La fila introductoria de la sección declara literalmente este criterio: *"Las denominaciones y fechas son las que da la fuente; los campos ausentes se indican como tales."*
   - **Criterio de inclusión**: toda persona, autor, obra o institución nombrada con sustancia narrativa propia en la fuente — incluidas figuras mitológicas o legendarias cuando forman parte del contenido histórico/cultural que la fuente describe (p. ej. las deidades nórdicas citadas al describir las creencias de un pueblo), y autores de obras recomendadas por la fuente para ampliar un episodio (no solo autores de obras citadas textualmente).
   - **Criterio de exclusión, documentado explícitamente en la ficha**: comparaciones puramente decorativas o retóricas sin contenido histórico propio en el texto (un símil con una celebridad moderna, una cita literaria usada solo como recurso estilístico y ajena al periodo tratado). La exclusión de una entidad nombrada bajo este criterio se documenta con una frase explícita al inicio de la sección (p. ej. "Se excluyen las comparaciones puramente decorativas de la fuente... [lista]"), nunca por omisión silenciosa — distinto de los campos de fecha ausentes, que sí pueden quedar implícitos fila a fila sin una nota aparte.
   - **Alcance deliberadamente distinto del índice de bibliografía (aclarado 2026-10-18)**: el criterio de exclusión de esta sección (sustancia narrativa/histórica propia) es más estricto que el de los índices de bibliografía recomendada (`260822-index-documentation-west-bibliografia-recomendada.md` y análogos), que incluyen **cualquier título nombrado por la fuente**, sin excepción de peso narrativo (criterio ya establecido en esos índices: "título nombrado → se incluye"). Que una obra o referencia quede excluida de `Autores y figuras mencionados` por ser decorativa no implica que deba excluirse del índice de bibliografía, ni viceversa — son dos inventarios con propósito distinto (uno cataloga personas/figuras con papel narrativo; el otro, cualquier título citado, tenga o no sustancia narrativa). Caso real que motivó esta aclaración: `conocimiento_west/260904-history-julio-cesar-ascenso-asesinato.md` excluye de su tabla la cita de *Don Juan Tenorio* (recurso estilístico sin relación con la Roma del artículo) y la serie *Roma*, pese a que ambas sí tienen fila propia en el índice de bibliografía (`^260822-RVR-00024`, `^260822-RVR-00031`) por el criterio más amplio de ese índice — no es una inconsistencia a corregir, es el resultado esperado de dos alcances distintos.
   - **Obras citadas**: cuando la fuente cita o recomienda una obra de una entidad, esa obra se traslada al índice de bibliografía correspondiente al corpus de la ficha (índice de la asignatura si aplica el punto 6 original; para `conocimiento_west/`, `conocimiento_west_mundo/` o `conocimiento_west_espana/`, su respectivo índice de bibliografía recomendada), mismo mecanismo ya exigido por el punto 6 — esta sección documenta la mención en el cuerpo de la ficha, no sustituye esa alta en el índice.
   - **Enlace a la fila del índice bibliográfico (añadido 2026-10-18)**: cuando una obra listada en la columna `Obras citadas` ya tiene fila en el índice de bibliografía correspondiente (verificado por búsqueda de texto, punto 2), esa celda se escribe como enlace Markdown al bloque exacto de esa fila: `[*Título*](ruta_relativa#^YYMMDD-xxx-NNNNN)`, nunca solo como texto plano o entre comillas de código (mismo criterio del punto 7). Si la obra todavía no tiene fila en el índice, se documenta el nombre sin enlace y se añade la fila nueva en el mismo lote de edición cuando sea razonable; si no se añade en el momento, se deja constancia explícita en `Notes` de que la obra está pendiente de alta. No se enlaza a una fila que no se ha verificado existente.
   - **Alcance no retroactivo**: mismo criterio del punto 5 — no se exige revisar retroactivamente las Knowledge Assets existentes que no tengan esta sección; se añade hacia adelante o al auditar/editar una ficha por otro motivo.
   - Precedentes: las 16 fichas `asignaturas/011_17617_Introduccion_Historia/conocimiento/261006-history-sesion-*.md` (origen del patrón, previo a esta ampliación formal) y, como primera extensión fuera de esa asignatura, `conocimiento_west/260904-history-europa-barbara-celtas-germanos-cimbrios-teutones.md` y `conocimiento_west/260904-history-julio-cesar-ascenso-asesinato.md` (2026-10-18) — esta última con las dos primeras celdas `Obras citadas` enlazadas a fila existente del índice West-Reverte (`^260822-RVR-00022`, `^260822-RVR-00023`), primera aplicación del enlace a índice bibliográfico del mismo punto.

---

## Consequences

### Positive

* Cierra una vía de alucinación distinta de la que ya cubre `ADR-1119`: la integridad interna del repositorio (nombres, enlaces, códigos de bloque) verificable sin ninguna herramienta externa, con el mismo nivel de daño potencial (afirmaciones falsas presentadas como hechos).
* Alinea la plantilla `ai/templates/30_knowledge_asset.md` con el patrón ya establecido por `ADR-1100`/`ADR-1110`: toda convención estructural duradera de la plantilla queda respaldada por un ADR propio, citado inline, en vez de vivir solo como texto de una respuesta de sesión.
* El `Quality Checklist` da un mecanismo mecánico y verificable (marcar cada ítem) en vez de depender de la disciplina puntual del asistente en cada ficha — mismo patrón de mitigación ya usado por `AI_ENTRYPOINT.md` para las verificaciones de `14_ADR_INDEX.md` y por `ADR-1119` punto 2 para `WebFetch`.

### Negative

* Añade un paso de verificación adicional (listar/grep el repositorio) antes de citar cualquier enlace interno, con el coste de tiempo correspondiente.
* El `Quality Checklist` visible en la ficha terminada añade longitud a cada Knowledge Asset — aceptado como coste menor frente al riesgo que mitiga.
* No elimina el riesgo de que una denominación *sí* provista por la fuente citada sea, a su vez, incorrecta en origen — esta Decision solo garantiza fidelidad a lo que la fuente dice, no la exactitud última de la fuente misma (mismo límite que `ADR-1119` ya reconoce para su propio alcance).

### Risks Mitigated

* Incorporación a una Knowledge Asset de un nombre propio, traducción o identificador de entidad inventado y presentado con la misma confianza que un hecho verificado.
* Cita de un enlace o código de bloque hacia un recurso del repositorio que no existe, o que existe pero no corresponde a la entidad citada.
* Introducción silenciosa de convenciones estructurales duraderas en `ai/templates/30_knowledge_asset.md` sin el respaldo de gobernanza que el propio proyecto exige para ese tipo de cambio (`AI_ENTRYPOINT.md`, Proportionality Principle).
* Omisión silenciosa de autores, figuras u obras nombradas en la fuente que una ficha trata (punto 6).
* Referencia a un documento del repositorio escrita de forma que, aunque la ruta sea correcta, no resulte navegable desde el editor (comillas de código en vez de enlace Markdown), o que lo sea con una ruta relativa mal resuelta (punto 7).
* Formato de cobertura del punto 6 inconsistente entre fichas (columnas distintas, campos ausentes dejados en blanco, exclusiones no documentadas) al no existir, hasta esta ampliación, una convención estructural única — mismo riesgo que ya motivó respaldar `## Glossary` con `ADR-1110` (punto 8).

### Positive (ampliación 2026-10-18, punto 8)

* Normaliza un patrón que ya se seguía de facto en 16 fichas sin respaldo formal, cerrando la misma brecha de gobernanza de plantilla que motivó originalmente esta Decision (ver Context, párrafo 2026-10-08): una convención estructural duradera documentada solo en texto de sesión, no en un ADR.
* Extiende el patrón fuera de `asignaturas/011_17617_Introduccion_Historia/` a `conocimiento_west/`, confirmando que es aplicable a cualquier Knowledge Asset narrativa del corpus, no solo a material docente.

---

## References

* ai/templates/30_knowledge_asset.md (plantilla ampliada por esta Decision — secciones `Evidence and Sources`, `Related Assets`, y nueva sección `Quality Checklist`)
* docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md (Context: origen de la plantilla, anterior a cualquier ADR)
* docs/decisiones/ADR-1110 (precedente directo: convención de plantilla — `## Glossary` — respaldada por un ADR propio y citada inline)
* docs/decisiones/ADR-1119-Verificacion_Obligatoria_WebFetch_sobre_Resumenes_WebSearch.md (principio hermano, alcance distinto y complementario)
* docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md (patrón de código de bloque `^YYMMDD-xxx-NNNNN` cuya atribución indebida motiva el punto 2)
* conocimiento/260823-technology-economics-cxmt-debuto-bolsa-shanghai.md (ficha que motivó el incidente — las tres anomalías fueron corregidas en ella antes de esta Decision)
* conocimiento_west/260904-history-europa-barbara-celtas-germanos-cimbrios-teutones.md, conocimiento_west/260904-history-julio-cesar-ascenso-asesinato.md (primera extensión del punto 8 fuera de la asignatura de origen, 2026-10-18)
* AI_ENTRYPOINT.md § Proportionality Principle (criterio que clasifica esta Decision como de impacto medio, exigiendo ADR)
* asignaturas/011_17617_Introduccion_Historia/conocimiento/261006-history-sesion-*.md (16 fichas que motivaron el punto 7 — enlaces corregidos a `[texto](ruta)` y ruta del índice de bibliografía corregida con `../`)
