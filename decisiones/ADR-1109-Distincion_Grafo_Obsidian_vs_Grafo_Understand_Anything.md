# ADR-1109: Distincion entre el Grafo de Obsidian y el Grafo de Understand-Anything — Como Declarar Relaciones Verificadas

Status: APPROVED

Date: 2026-08-02

Scope:
Process

Category:
PROCESS

Domain: (Optional)
Documentation, Governance

Tags:
knowledge-repository, methodology, obsidian

Related ADRs:
ADR-1106 (retirada del Relationship Record — las relaciones verificadas pasaron a declararse como wikilinks directos entre Knowledge Assets), ADR-1108 (identificadores de bloque de Obsidian)

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Revisión (2026-08-03)

Al enlazar el índice de ubicaciones geográficas (`260802-geography-indice-ubicaciones-conocimiento.md`) desde 60 fichas de origen, se detectó que el wikilink literal `[[...]]` exigido por el punto 2 de esta Decision se renderiza como texto en bruto (ruta completa e identificador de bloque visibles) en cualquier visor Markdown que no sea Obsidian (VSCode, GitHub).

Se probó primero la alternativa `[alias](fichero.md#^bloque)` (ruta relativa simple, sin prefijo): confirmado en Obsidian que **no navega al bloque exacto y no se registra como Linked Mention** — solo abre el fichero. Un primer intento de adoptar este formato en todo el corpus se revirtió a wikilink por este motivo.

El propio usuario encontró después la causa exacta: el problema no era el formato Markdown en sí, sino la ausencia del prefijo de ruta relativa explícita. Con **`[alias](./fichero.md#^bloque)`** (con `./` y con la extensión `.md`), confirmado en Obsidian:

* Navega correctamente al bloque exacto (igual que el wikilink).
* Aparece como "Linked Mention" (backlink) en el documento de destino (igual que el wikilink).
* Además, a diferencia del wikilink, se renderiza como un enlace legible normal en visores Markdown ajenos a Obsidian.

Con esta evidencia, `[alias](./fichero.md#^bloque)` pasa a ser el formato recomendado para todo enlace interno en `conocimiento/` — incluidas las relaciones verificadas del punto 2 de esta Decision — sin que suponga ninguna renuncia de funcionalidad respecto al wikilink. Se documenta aquí en detalle, con los dos intentos previos (uno fallido, uno exitoso), porque la diferencia entre ambos (un simple prefijo `./`) no es obvia y ya causó una reversión completa de 305 enlaces en la sesión que motivó este hallazgo.

**Caso adicional probado el mismo día — enlaces hacia `fuentes/` (carpeta excluida de la bóveda)**: al convertir 89 citas de fichero primario (`Evidence and Sources`) a `[fuentes/ruta.md](../fuentes/ruta.md)`, confirmado en Obsidian que el enlace **sí navega correctamente** (tanto en Obsidian como en vista Markdown plana), pero el fichero de destino **no registra Linked Mention ni aparece en la bóveda** — a diferencia del caso `conocimiento/` ↔ `conocimiento/`, donde ambas cosas funcionan. Esto es coherente con la exclusión de `fuentes/` del índice de Obsidian (punto 1 de Context) y no se considera una limitación relevante: una fuente primaria no es un Knowledge Asset par en el grafo, solo necesita ser navegable para consulta. El punto 3 de la Decision (más abajo), que hasta ahora asumía que un enlace hacia una carpeta excluida es "un enlace roto sin ningún beneficio", queda matizado por este hallazgo: el enlace no está roto (navega), simplemente no participa del grafo — se actualiza en consecuencia.

**Extensión del hallazgo — exclusión individual de Knowledge Assets recopilatorios dentro de `conocimiento/` (2026-08-03)**: partiendo de que un enlace hacia un fichero excluido de la bóveda sigue navegando con normalidad (hallazgo anterior), el usuario añadió a `userIgnoreFilters` dos fichas concretas que, pese a residir en `conocimiento/`, actúan como recopilatorios agregados (`260801-language-hsk1-xindongfang-vocabulario.md`, `260802-geography-indice-ubicaciones-conocimiento.md`): al recibir enlaces desde decenas de fichas sin relación temática entre sí (cada una citando un término o lugar puntual), estos ficheros se comportan como *hubs* que distorsionan el grafo de Obsidian, ocultando las relaciones genuinas 1:1 entre Knowledge Assets temáticos bajo un volumen desproporcionado de aristas hacia el mismo nodo.

Primer criterio propuesto — "cualquier Knowledge Asset que use el formato tabla de ADR-1108" — se descartó por ser demasiado amplio: los glosarios de las fichas Shi Zhilong (`260802-society-shi-zhilong-*`, etc.) también usan tabla, pero son una sección de apoyo dentro de un documento temático sobre un referente real único, no el objeto entero de la ficha, y no reciben enlaces de decenas de Knowledge Assets sin relación entre sí. El criterio correcto no es el **formato** de una sección sino el **rol estructural de la ficha completa** (ver punto 5 de la Decision).

---

## Context

El proyecto mantiene dos sistemas de grafo completamente independientes, que no se sincronizan entre si de ninguna forma:

1. **El grafo de Obsidian** (vista "Graph View" de la aplicacion Obsidian): se construye exclusivamente a partir de wikilinks `[[...]]` literales presentes en el texto de los archivos. Ademas, la boveda de este proyecto excluye explicitamente las carpetas `ai/`, `docs/` y `fuentes/` de su indice (`.obsidian/app.json` -> `userIgnoreFilters`), por lo que un wikilink que apunte a un archivo dentro de esas carpetas no se resuelve ni aparece como enlace navegable en el grafo de Obsidian, aunque el archivo exista en el repositorio.
2. **El grafo de `understand-anything`** (`docs/arquitectura/.ua/knowledge-graph.json`): se genera mediante analisis semantico por subagentes de IA (tipos de arista `related`, `documents`, `depends_on`, etc.), cubre todo el repositorio sin exclusiones de carpeta, y no tiene ninguna via de sincronizacion hacia el grafo de Obsidian.

Esta distincion causo confusion repetida durante la sesion del 2026-08-01/02: en dos ocasiones distintas, el usuario senalo que aristas reportadas por el asistente de IA (fruto de una actualizacion de `understand-anything`) no aparecian en el grafo de Obsidian, esperando razonablemente que ambos sistemas reflejaran la misma informacion. Adicionalmente, ilustrar el formato de un wikilink en prosa usando corchetes dobles literales con texto de ejemplo (en vez de describirlo sin corchetes) provoco que Obsidian intentara resolver ese texto como un enlace real, roto por definicion (ADR-1108, seccion 5, y la ficha `260801-language-hsk1-mandarin-corner-150-oraciones.md`).

Al aplicar por primera vez el punto 2 de esta misma decision (declarar relaciones verificadas como wikilinks), el asistente de IA declaro como "relacion verificada" un vinculo entre `260801-language-hsk1-mandarin-corner-150-oraciones.md` y `260801-language-hsk1-xindongfang-vocabulario.md` basado unicamente en que la segunda ficha resuelve un punto "Pendiente" anotado por la primera — una coincidencia de procedencia/proceso (ambas implementan ADR-1108), no de contenido. No existia solapamiento de vocabulario verificado entre ambos mazos. El usuario senalo que un enlace en `Related Assets` sin justificacion didactica (util para navegar conocimiento genuinamente conectado) no deberia declararse como relacion verificada solo por compartir procedencia arquitectonica.

---

## Decision

1. **Los dos grafos son y seguiran siendo independientes.** Ninguna actualizacion de `understand-anything` debe describirse ante el usuario como "visible en Obsidian", ni viceversa. Al reportar cambios en el grafo de `understand-anything`, el asistente de IA debe evitar dar por hecho que el usuario vera el resultado reflejado en su boveda de Obsidian.

2. **Toda relacion verificada entre Knowledge Assets (Paso 1 de ADR-1106, o equivalente) debe declararse ademas como enlace interno navegable** en las secciones `Related Knowledge Assets` (cabecera) y `Related Assets` (cuerpo) de ambas fichas implicadas, bidireccionalmente — no basta con crear una arista `related`/`documents` en el JSON de `understand-anything`. Solo un enlace interno real (wikilink o el formato Markdown equivalente, ver Nota de Revision 2026-08-03) hace la relacion visible en el grafo de Obsidian.

   **Formato recomendado (revisado 2026-08-03)**: `[alias](./fichero.md#^bloque)` — ruta relativa explicita (prefijo `./` obligatorio) mas extension `.md`. Verificado que preserva navegacion exacta a bloque y registro de Linked Mention igual que el wikilink `[[fichero#^bloque|alias]]`, con la ventaja adicional de renderizarse legible en visores Markdown ajenos a Obsidian (VSCode, GitHub). El wikilink literal sigue siendo valido como alternativa equivalente; ambos formatos cumplen igual esta regla. **No es valido** `[alias](fichero.md#^bloque)` sin el prefijo `./` — confirmado que no navega al bloque ni registra Linked Mention (ver Nota de Revision).

   **Criterio de "verificada"**: exige solapamiento de contenido comprobable (mismos terminos, mismos hechos, misma fuente primaria, cita textual explicita de una ficha dentro de la otra referida al tema tratado) — no basta con que ambas fichas compartan procedencia arquitectonica (p. ej. "ambas implementan el mismo ADR") ni con que una mencione haber resuelto un punto pendiente dejado por la otra si ese punto es de proceso y no de contenido. Una coincidencia de procedencia/proceso sin solapamiento de contenido se documenta como nota en prosa dentro de `Notes` (de cada ficha implicada), nunca como entrada en `Related Knowledge Assets` o `Related Assets`.

3. **Excepcion por exclusion de boveda, matizada (revisado 2026-08-03)**: si una de las dos partes de una **relacion verificada entre Knowledge Assets** (punto 2) reside en `ai/`, `docs/` o `fuentes/` (carpetas excluidas de la boveda), no se declara como enlace navegable — se referencia como texto plano (p. ej. "ADR-1108" o el nombre del archivo, sin corchetes ni parentesis de enlace), igual que ya se hace con las citas a ADRs en las cabeceras de Knowledge Asset. Un enlace hacia una carpeta excluida nunca aparecera en el grafo ni en Linked Mentions, asi que no cumple el proposito del punto 2 (relacion descubrible en el grafo) por mucho que navegue.

   Este punto no aplica a otros usos de enlace que no persiguen presencia en el grafo — p. ej. citar el fichero fuente primario en `Evidence and Sources`. Confirmado que `[fuentes/ruta.md](../fuentes/ruta.md)` navega correctamente (Obsidian y Markdown plano) aunque el destino no registre Linked Mention (ver Nota de Revision 2026-08-03) — en ese caso el enlace aporta valor real (navegacion a la fuente citada) y su uso es recomendable, con el entendimiento explicito de que no construye una relacion visible en el grafo.

   **Tercer caso — mención incidental sin relación verificada (añadido 2026-08-05)**: cuando la prosa de una ficha nombra otro Knowledge Asset solo a título informativo (p. ej. "se comprobó contra estos mazos ya elaborados: X, Y" en una sección `Glosario`), sin que exista solapamiento de contenido que justifique una relación verificada (punto 2) ni se trate de citar la fuente primaria propia (punto 3, primer párrafo), no se usa wikilink ni el formato `[alias](./fichero.md)` — se referencia entre backticks con la **ruta relativa completa, incluida la carpeta** (`` `conocimiento/260801-language-hsk1-xindongfang-vocabulario.md` ``), nunca el nombre de fichero suelto. Verificado por inspección de las 25 apariciones ya existentes de este patrón en el corpus antes de esta nota: el 100% incluye el prefijo de carpeta; la omisión detectada en `260805-language-calendario-lunar-enero-2026.md` (fichero sin prefijo) era una desviación aislada, corregida en la rama de contenido correspondiente (`oushinet`, commit `dda8283`). La diferencia con el punto 2 es deliberada: un enlace real añadiría una arista al grafo de Obsidian sin que exista relación de contenido verificada, contradiciendo el criterio de "verificada" ya fijado en este mismo punto.

4. **Al ilustrar el formato de un wikilink o de un identificador de bloque en prosa** (documentacion, ADRs, Notes), nunca se usan corchetes dobles literales con la sintaxis real — se describe en palabras o se usa un formato visualmente distinto (comillas, backticks) que no sea interpretado por Obsidian como un enlace.

5. **Exclusion de Knowledge Assets recopilatorios en `userIgnoreFilters` — por carpeta desde ADR-1100 (anadido 2026-08-03, revisado 2026-08-06)**: un Knowledge Asset cumple el **criterio de rol estructural**, no de formato, cuando:

   > La ficha **entera** es una agregacion (vocabulario, glosario global, indice derivado de corpus, segun ADR-1108) que actua como destino de enlaces desde un numero elevado de Knowledge Assets sin relacion tematica entre si — cada uno cita una entrada puntual (un termino, un lugar), no la ficha como un todo.

   Esto **no aplica** a un Knowledge Asset tematico (Purpose/Description/Scope centrados en un referente real unico) que simplemente contenga una tabla de apoyo en formato ADR-1108 (p. ej. un glosario dentro de una tesis) — ese tipo de tabla es una seccion del documento, no el objeto de la ficha, y no genera el efecto *hub* que motiva esta exclusion. El formato de tabla por si solo nunca es motivo de exclusion.

   **Revision 2026-08-06**: desde que `ADR-1100` (Nota de Ampliacion 2026-08-06) autoriza `conocimiento_index/` como ubicacion fisica de todo Knowledge Asset `Knowledge Type: Index`, la exclusion se aplica **a la carpeta completa** (`conocimiento_index/` anadida a `userIgnoreFilters`, junto a `ai/`, `docs/`, `fuentes/`), no ficha a ficha. Un Knowledge Asset que cumpla el criterio de rol estructural pero aun no se haya trasladado a esa carpeta sigue necesitando su entrada individual en `userIgnoreFilters` hasta que se traslade.

   Consecuencia practica ya verificada (ver Nota de Revision): los enlaces hacia una ficha excluida por este motivo siguen navegando con normalidad (formato `[alias](./fichero.md#^bloque)` o, tras el traslado, `[alias](../conocimiento_index/fichero.md#^bloque)`); solo se pierde su presencia como nodo en Graph View y el registro de Linked Mention en esa ficha — perdida aceptada, no un enlace roto.

   **Mantenimiento**: `.obsidian/app.json` no esta versionado (`.gitignore`), por lo que `userIgnoreFilters` no tiene rastro en el repositorio. La seccion "Recopilatorios excluidos de la boveda" (mas abajo en este ADR) sigue siendo la fuente de verdad versionada — ahora para los dos casos: la regla de carpeta (`conocimiento_index/`) y cualquier ficha individual pendiente de traslado.

6. **Adopción formal de Obsidian como capa de navegación humana (añadido 2026-08-08, formalizando una decisión ya implementada el 2026-07-29)**: Obsidian se adopta como interfaz de navegación humana sobre el grafo ya indexado por `understand-anything` — nunca como una segunda fuente de verdad ni como una taxonomía paralela a la ya existente en el corpus (riesgo identificado explícitamente desde el diseño de esta adopción, ver Anexo). El despliegue se ejecutó por etapas reversibles, cada una revertible con una sola acción:

   1. Abrir el repositorio directamente como Vault en la aplicación Obsidian (huella en disco: solo `.obsidian/`, generada por la propia app; reversible borrando la carpeta).
   2. Configurar `userIgnoreFilters` para excluir del índice las carpetas que no aportan valor a la navegación humana (`ai/`, `docs/`, `fuentes/`, y desde el 2026-08-06 también `conocimiento_index/` — ver punto 5).
   3. Frontmatter YAML mínimo en una muestra pequeña de fichas, para validar que Obsidian lee correctamente `Domain`/`Tags`/`Status` ya existentes, antes de extenderlo al resto del corpus.
   4. Fusión de la rama de despliegue (`obsidian`) en `develop` — ejecutada el 2026-07-29 (commit `da9f90c`, "obsidian: configurar vault y exclusiones de estado local").

---

## Anexo: Comparativa y fuentes de inspiración (Obsidian)

No se evaluaron alternativas formales a Obsidian con tabla comparativa: el proyecto ya usaba Markdown plano como formato canónico de `conocimiento/`, por lo que Obsidian — un visor/organizador de bóvedas Markdown que no exige cambiar el formato de los ficheros — era la opción de menor fricción, no una entre varias comparadas por pros/contras. Se documentan aquí las fuentes que motivaron la decisión — no como parte de la Decision en sí, sino como el camino que condujo a ella —, trasladadas desde el registro original de esta adopción. El contenido íntegro de esa sección, incluidas las fuentes de Understand-Anything, sub-agentes y otras herramientas evaluadas, se conserva en el Anexo de `docs/decisiones/ADR-1114-Adopcion_Understand_Anything_Knowledge_Provider.md`:

* https://obsidian.md/
* https://www.youtube.com/shorts/Avvn90oqAcY
* https://www.youtube.com/shorts/8XRJfTk7Jgc
* https://www.youtube.com/shorts/73yz8gTrj1A
* https://graphify.net/

---

## Recopilatorios excluidos de la bóveda (checklist versionado)

Fuente de verdad para reconstruir `userIgnoreFilters` (`.obsidian/app.json`, no versionado). Las exclusiones de carpeta completa (`ai/`, `docs/`, `fuentes/`) se mantienen documentadas en el punto 1 de Context y no se repiten aquí.

**Regla de carpeta (desde 2026-08-06)**: `conocimiento_index/` — cualquier Knowledge Asset `Knowledge Type: Index` alojado ahí queda excluido de la bóveda sin necesidad de entrada individual (`ADR-1100`, Nota de Ampliación 2026-08-06).

| Fichero individual pendiente de traslado a `conocimiento_index/` | Motivo | Añadido |
|---|---|---|
| *(ninguno a fecha de esta revisión — los 3 recopilatorios existentes ya están en `conocimiento_index/`)* | — | — |

Histórico (ya cubierto por la regla de carpeta, se conserva por trazabilidad): `260801-index-language-hsk1-xindongfang-vocabulario.md` y `260802-index-geography-ubicaciones-conocimiento.md` se añadieron individualmente el 2026-08-03; `260801-index-language-hsk1-mandarin-corner-150-oraciones.md` (Vocabulario, `Knowledge Type: Index` desde el 2026-08-06) nunca necesitó entrada individual — se creó ya con Domain `Language` similar a los otros dos, pero su exclusión quedó cubierta directamente por la regla de carpeta al trasladarse el mismo día que se formalizó `Knowledge Type: Index`.

Al crear un nuevo recopilatorio (p. ej. futuros índices de dominio: ingeniería, arte, turismo, montañismo, cocina), ubicarlo directamente en `conocimiento_index/` — la regla de carpeta lo cubre sin pasos adicionales.

---

## Consequences

### Positive

* Evita que se repita la confusion sobre por que una arista reportada por el asistente de IA no aparece en Obsidian.
* Da una regla mecanica y verificable ("visible en Obsidian" = "tiene wikilink literal Y ambas partes estan dentro de la boveda") en vez de una expectativa implicita.
* Refuerza la practica ya establecida en el Paso 1 de ADR-1106 (wikilinks bidireccionales para relaciones verificadas), en vez de dejarla como convencion no escrita.
* (2026-08-03) El formato `[alias](./fichero.md#^bloque)` consigue legibilidad fuera de Obsidian sin sacrificar ninguna funcionalidad del grafo — deja de existir la tension entre ambos objetivos que parecia real tras el primer intento fallido (sin `./`).
* (2026-08-03) El criterio de rol estructural (punto 5) permite excluir recopilatorios del grafo de Obsidian sin perder navegacion, manteniendo el Graph View util para relaciones tematicas genuinas incluso a medida que crezca el numero de indices/vocabularios de dominio.

### Negative

* Anade un paso manual (escribir el enlace en ambas fichas) cada vez que se verifica una relacion, ademas de la arista semantica en `understand-anything`.
* (2026-08-03) Coexisten en el corpus dos formatos validos (wikilink y Markdown con `./`) para el mismo concepto, sin una regla que obligue a unificar hacia uno solo — riesgo menor de inconsistencia visual, no de perdida de funcionalidad.
* (2026-08-03) `userIgnoreFilters` no esta versionado, asi que la exclusion individual de un recopilatorio depende de mantener sincronizado el checklist de este ADR con la configuracion local — un paso manual adicional, mitigado por dejarlo como fuente de verdad explicita en vez de una convencion tacita.

### Risks Mitigated

* Expectativa incorrecta de que el grafo de `understand-anything` y el de Obsidian son la misma cosa o se sincronizan.
* Wikilinks rotos por apuntar a carpetas excluidas de la boveda (`ai/`, `docs/`, `fuentes/`).
* Corchetes literales de ejemplo interpretados por Obsidian como enlaces reales rotos.
* Relaciones declaradas como "verificadas" por coincidencia de procedencia/proceso (mismo ADR, resolucion de un pendiente) sin solapamiento de contenido real, restando utilidad didactica a `Related Assets`.
* (2026-08-03) Repetir la asuncion de que cualquier enlace Markdown estandar hacia un fichero interno se comporta igual en Obsidian — verificado que la ausencia del prefijo `./` rompe la navegacion a bloque y el registro de Linked Mention, mientras que incluirlo preserva ambos.
* (2026-08-03) Recopilatorios (vocabularios, indices) actuando como *hub* que satura el Graph View con enlaces entrantes sin relacion tematica entre si, ocultando las relaciones 1:1 genuinas entre Knowledge Assets — mitigado por el criterio de exclusion del punto 5, acotado por rol estructural y no por formato (evita excluir por error Knowledge Assets tematicos que solo contienen una tabla de apoyo, como los glosarios Shi Zhilong).

---

## References

* docs/decisiones/ADR-1106-Retirada_Relationship_Record.md
* docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md
* docs/decisiones/ADR-1114-Adopcion_Understand_Anything_Knowledge_Provider.md (Anexo con el registro histórico íntegro de las fuentes de esta adopción)
* .obsidian/app.json (`userIgnoreFilters`)
* conocimiento/260802-geography-indice-ubicaciones-conocimiento.md (caso que motivo el hallazgo del formato `./fichero.md#^bloque`)
* conocimiento/260726-politics-china-conquista-del-mundo.md, conocimiento/260726-politics-nuevos-sennores-africa-china.md (relacion verificada, caso probado por el usuario que confirmo el formato con `./`)
* conocimiento/260726-anthropology-mongolia-hombres-reno-tsaatan.md (primer caso probado de enlace hacia `fuentes/`, confirmado navegable sin Linked Mention)
* conocimiento/260802-society-shi-zhilong-01-antecedentes-migratorios.md (caso que descarto "formato tabla ADR-1108" como criterio de exclusion: contiene tabla-glosario pero no es recopilatorio)
