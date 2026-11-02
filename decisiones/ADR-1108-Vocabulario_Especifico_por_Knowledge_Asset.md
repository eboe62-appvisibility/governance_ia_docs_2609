# ADR-1108: Vocabulario Específico por Knowledge Asset

Status: APPROVED

Date: 2026-08-01

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Language

Tags:
knowledge-asset, language, linguistics

Related ADRs:
ADR-1100 (estructura de Knowledge Asset), ADR-0033 (vocabulario Domain), ADR-1110 (Glosario — comparte la convención de formato de entrada y de identificador de bloque revisada aquí)

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Aprobación (2026-08-01)

Este Architectural Decision Record se propuso inicialmente como `PROPOSED`, con la aprobación formal condicionada a la aparición de un caso real de gestión de vocabulario chino en un Knowledge Asset. Ese caso apareció el mismo día: dos mazos de vocabulario del propio usuario (`vocabulary/260718_anno00_MandarinCorner_HSK1.xml`, `vocabulary/260718_anno00_XinDongFang_HSK1.xml`) que requieren wikilinks de Obsidian a nivel de entrada individual. Con el formato concreto ya decidido y validado (sección Decision), este Architectural Decision Record pasa a `APPROVED`.

---

## Context

`vocabulary/vocabulario.csv` es el vocabulario maestro del proyecto, alimentado exclusivamente por el progreso real del curso académico (palabras y frases que se van aprendiendo a lo largo del itinerario). Durante la elaboración del primer Knowledge Asset derivado de un vídeo (`conocimiento/260801-language-hsk-1-cambios-v2-vs-v3.md`), surgió la pregunta de si un Knowledge Asset que verifica términos chinos contra el vocabulario maestro debería, a la inversa, poder alimentar `vocabulario.csv` con términos nuevos que descubra.

La respuesta es no: `vocabulary/` refleja el progreso curricular del propio usuario, no un agregado automático de todo término chino que aparezca en cualquier fuente procesada. Mezclar ambos flujos rompería el propósito de `vocabulary/` como registro de aprendizaje real.

Sin embargo, existe una necesidad distinta y genuina: comprobar el uso de palabras y frases ya conocidas dentro de un contexto nuevo y familiar (un Knowledge Asset), en lugar de solo memorizarlas de forma aislada.

### Triple uso consolidado (2026-08-03)

Este documento nació para un único caso (un mazo de vocabulario ligado a un vídeo o curso concreto) y, con el tiempo, ha pasado a cubrir tres patrones distintos de "listado de entradas enlazable por bloque". Se documentan aquí de forma explícita, en vez de dispersos en notas de revisión sucesivas, precisamente para que el conjunto sea legible de una sola vez:

1. **Vocabulario** — un mazo de vocabulario completo sustituye a la ficha de catalogación habitual de su fuente externa (un vídeo, un mazo XML/Pleco). Casos reales: `260801-language-hsk1-mandarin-corner-150-oraciones.md`, `260801-language-hsk1-xindongfang-vocabulario.md`.
2. **Glosario** — dentro de una ficha narrativa que ya cita muchos términos chinos en su `Description`, una sección `## Glosario` (ADR-1110) los consolida y enlaza selectivamente, cuando existe coincidencia verificada, a la entrada exacta de un mazo de vocabulario ya elaborado. Casos reales: las 6 fichas de la tesis de Zhilong Shi.
3. **Ubicación / índice derivado de corpus** — un índice construido agregando información ya presente en múltiples Knowledge Assets del propio `conocimiento/`, no en una única fuente externa. Caso real: `260802-geography-indice-ubicaciones-conocimiento.md`, con columnas propias del caso (tipo, región, país, coordenadas) además de la estructura común.

Los tres comparten la misma mecánica de fondo (formato tabla, identificador de bloque `^YYMMDD-xxx-NNNNN` por entrada) pero difieren en dos ejes que la sección `Decision` distingue explícitamente: de dónde procede el contenido (fuente externa única vs. agregación del propio corpus) y en qué dirección apunta el enlace cuando hay coincidencia con otro mazo (nunca se mantiene una columna de referencia inversa — ver punto 4 de `Decision` y el precedente de `ADR-1110`).

El primer caso (Vocabulario) originalmente se documentó en formato de lista numerada; la tabla se adoptó primero para el tercer caso (Ubicación), que necesitaba admitir columnas de coordenadas futuras, y se generalizó de inmediato como formato estándar para los tres usos — los dos mazos de Vocabulario ya se migraron de lista a tabla el mismo día, sin alterar contenido, orden ni códigos de bloque (verificado mecánicamente).

---

## Decision

Se autoriza, para cuando se apruebe este Architectural Decision Record, la construcción de un vocabulario específico por Knowledge Asset — un listado de términos acotado al contenido de esa ficha concreta, siguiendo el mismo formato y las mismas reglas ya establecidas para `vocabulary/vocabulario.csv` (columnas: forma simplificada/tradicional, pinyin, traducción al español).

Reglas:

1. El vocabulario específico de un Knowledge Asset es un artefacto derivado, no una ampliación de `vocabulary/vocabulario.csv` ni de ningún mazo existente en `vocabulary/`. Ambos permanecen completamente separados; no hay flujo automático ni manual en ninguna dirección entre ellos.
2. Su propósito es pedagógico: permitir comprobar el uso de palabras y frases ya conocidas (o nuevas, propias del texto) dentro de un contexto real y familiar para el usuario, no sustituir al vocabulario maestro del curso.
3. **Formato**: se implementa como una instancia normal de `ai/templates/30_knowledge_asset.md`, ubicada en `conocimiento/` y sujeta a la misma convención de nombres de ADR-1100 (`YYMMDD-domain_principal-slug.md`, donde `YYMMDD` es la fecha de curación de esta ficha, no la fecha del mazo o fuente original que representa). No se crea una plantilla nueva ni una ubicación especial. Cada entrada se representa como **fila de una tabla Markdown**, con una estructura básica común a todo vocabulario/glosario/índice bajo este documento (columna de término o nombre, transliteración cuando aplique, traducción/significado) más las columnas adicionales que requiera el caso concreto (p. ej. tipo, región, país, coordenadas geográficas). El formato de lista numerada usado originalmente queda superseded por este formato tabla (ver Context § Triple uso consolidado).
4. Cuando el Knowledge Asset cataloga un mazo de vocabulario derivado de una fuente ya identificable (p. ej. un vídeo), esta ficha **sustituye** a la ficha de catalogación habitual de esa fuente — no se crean dos Knowledge Assets separados (uno para "el vídeo" y otro para "su vocabulario"). La sección `Evidence and Sources` cataloga la fuente original con normalidad; la sección `Description` contiene las entradas de vocabulario en lugar de (o además de) prosa descriptiva.
   - **Fuente derivada del propio corpus** (además de la fuente externa única del punto anterior): el vocabulario/índice específico también puede derivar de información ya presente en Knowledge Assets existentes de `conocimiento/`, en vez de un mazo o vídeo externo preexistente. En ese caso, `Evidence and Sources` cita como fuente primaria el propio corpus de Knowledge Assets de origen — no declara `Related Knowledge Assets` individual con cada uno de ellos (evita relaciones no verificadas por simple pertenencia a un corpus de agregación, conforme al criterio de relación verificada de ADR-1109).
     **Artefacto de consolidación, opcional e instrumental (revisado 2026-08-04)**: durante la elaboración puede resultar útil construir un artefacto intermedio de consolidación (p. ej. un CSV en `fuentes/<YYMMDD>_<slug>/`) para extraer, deduplicar y clasificar las menciones antes de escribir la tabla final. Este artefacto **no es de retención obligatoria**: una vez verificada mecánicamente su equivalencia con la tabla del Knowledge Asset, puede eliminarse sin que la ficha pierda fuente primaria — el corpus de `conocimiento/` que le dio origen sigue siendo la fuente primaria real y permanece disponible para re-derivar la extracción si hiciera falta. Retenerlo indefinidamente introduce una reiteración de los mismos datos en dos sitios (el CSV y la tabla) con riesgo real de divergencia en ediciones futuras — riesgo observado en la práctica en `260802-geography-indice-ubicaciones-conocimiento.md`, cuyos CSV intermedios se retiraron por este motivo tras verificar su equivalencia (ver Notes de esa ficha).
   - **Sin columna de referencia inversa**: una ficha construida así no mantiene un campo o columna que liste qué fichas de origen citan cada entrada — ese mantenimiento crecería indefinidamente según se añadan fichas nuevas al corpus. Cuando una ficha de origen necesite referenciar una entrada concreta, enlaza hacia ella mediante wikilink de Obsidian dirigido a su bloque exacto (mismo mecanismo que la sección Glosario de ADR-1110); el descubrimiento inverso queda disponible vía "Linked Mentions" de Obsidian, sin mantenimiento manual.
5. **Enlazabilidad a nivel de entrada**: cada entrada (fila de tabla) dentro de `Description` se marca con un identificador de bloque de Obsidian con formato fijo `^YYMMDD-xxx-NNNNN`. Requisito técnico de Obsidian, no una elección de diseño: este marcador debe ir como último token de la línea de la fila (tras la última celda) — Obsidian solo reconoce como ancla de bloque el `^` situado al final de la línea, no en una columna intermedia. En modo lectura, Obsidian oculta este marcador.
   - `YYMMDD`: fecha de curación de la ficha (la misma que el nombre de archivo, por ADR-1100 punto 3) — permite identificar de un vistazo cuándo se formalizó la entrada.
   - `xxx`: acrónimo de exactamente 3 caracteres, preferentemente consonantes, derivado mecánicamente de las 3 primeras consonantes del nombre de la fuente tal como aparece en su propio nombre de archivo (p. ej. `MandarinCorner` → `MND`, `XinDongFang` → `XND`). Existe precisamente porque `YYMMDD` por sí solo no distingue entre dos mazos de vocabulario curados el mismo día — caso ya observado en la práctica, no hipotético.
   - `NNNNN`: número secuencial de 5 dígitos con ceros a la izquierda, único dentro de la ficha (00001–99999). Cinco dígitos se eligen como margen holgado para el mazo individual más grande previsible (el CSV maestro del proyecto tiene 857 líneas), evitando tanto quedarse corto (3 dígitos) como el exceso de un identificador de 8+ dígitos sin beneficio real.

   Cualquier otro Knowledge Asset enlaza directamente a una entrada mediante wikilink de Obsidian (`YYMMDD-domain-slug` seguido de `#^YYMMDD-xxx-NNNNN`) o mediante el formato Markdown equivalente `[alias](./YYMMDD-domain-slug.md#^YYMMDD-xxx-NNNNN)` — ambos preservan navegación exacta a bloque y registro de Linked Mention en Obsidian; el segundo, además, se renderiza legible fuera de Obsidian (ver ADR-1109, Nota de Revisión 2026-08-03, que fija el prefijo `./` como imprescindible en la variante Markdown). No se escribe aquí como wikilink literal para evitar que Obsidian intente resolverlo como un enlace roto a un archivo llamado literalmente "YYMMDD-domain-slug". Primeros casos reales, ambos migrados a formato tabla el 2026-08-02: `conocimiento/260801-language-hsk1-mandarin-corner-150-oraciones.md`, con IDs `^260801-MND-00001` a `^260801-MND-00149`; y `conocimiento/260801-language-hsk1-xindongfang-vocabulario.md`, con IDs `^260801-XND-00001` a `^260801-XND-00674`. Primer caso real construido directamente en tabla desde su creación: `conocimiento/260802-geography-indice-ubicaciones-conocimiento.md`, con IDs `^260802-BCC-00001` a `^260802-BCC-00201`.
6. **Duplicidad de datos, aceptada como limitación conocida**: cuando el vocabulario específico deriva de un mazo ya existente en `vocabulary/` (formato XML/Pleco, no enlazable a nivel de entrada por Obsidian), el nuevo Knowledge Asset contiene una copia derivada de esas entradas. El archivo de `vocabulary/` permanece como fuente autoritativa de los datos; el Knowledge Asset es una representación secundaria, creada específicamente para permitir el enlazado, y sin garantía de sincronización automática si el archivo de `vocabulary/` se corrige más adelante — cualquier corrección debe propagarse manualmente.
7. **Nivel de confianza**: cuando el vocabulario específico procede de una transcripción personal cuidada del propio usuario (no de una transcripción automática de reconocimiento de voz), se documenta con un Confidence Level más alto que el de fichas basadas en ASR, y se señala explícitamente el contraste en `Notes`.

8. **Inserción ordenada y numeración de bloque al ampliar un índice existente (añadido 2026-08-06)**: cuando un índice declara un orden explícito en su `Description` (alfabético, cronológico u otro), una entrada nueva se inserta en la posición que le corresponde según ese orden — nunca se añade al final de la tabla sin más, salvo que el orden declarado sea precisamente "orden de llegada".

   El identificador de bloque de la entrada nueva es siempre el siguiente número secuencial disponible (`NNNNN` más alto existente + 1), independientemente de en qué posición de la tabla quede insertada la fila. No se renumeran los bloques ya existentes para que la numeración "coincida" con la posición visual — hacerlo rompería cualquier enlace ya existente hacia esos bloques. Esto significa que, con el tiempo, el orden numérico de los bloques y el orden visual de las filas divergen — es el comportamiento esperado, no un defecto. Precedente ya documentado individualmente por `260802-index-geography-ubicaciones-conocimiento.md` (huecos en la numeración tras una revisión que retiró filas); este punto lo generaliza como regla para cualquier índice, en ambos sentidos (insertar o retirar filas).

9. **Verificación global de orden tras cualquier inserción (añadido 2026-10-07)**: insertar la fila nueva en su posición correcta (punto anterior) no basta cuando la misma operación amplía la celda de Fuentes/Notas de una fila **ya existente** citando una pieza o capítulo más bajo que el que tenía hasta ese momento — esa ampliación cambia su clave de orden sin mover su posición física, y nada vuelve a comprobarlo si solo se valida la fila nueva. Caso real que motiva este punto: `260822-index-documentation-west-mundo-bibliografia-recomendada.md`, donde ampliar la fila de *Historias* (Herodoto) y la de *Anábasis* (Jenofonte) con citas de fascículos posteriores, o con el historial de citas que traía de `Authors`, dejó ambas filas fuera de orden sin que la inserción lo detectara.

   Por ello, tras cualquier inserción o ampliación en un índice sujeto a este punto, se ejecuta una comprobación mecánica sobre **la tabla completa** (no solo sobre la fila nueva o ampliada): para cada fila, extraer la pieza/capítulo más bajo citado en su celda de Fuentes/Notas y verificar que la secuencia resultante es no decreciente. Toda fila que quede fuera de orden se reposiciona en el mismo commit; el `block-id` permanece fijo, conforme al punto anterior.

10. **Etiqueta de enlace autodescriptiva cuando el identificador de pieza no es global (añadido 2026-10-07)**: los puntos 8 y 9 asumen que ordenar por "pieza/capítulo más bajo" tiene sentido por sí solo. Eso solo es cierto cuando ese número es único en todo el corpus (p. ej. el capítulo de `fuentes/260828_abc_reverte/` y `fuentes/260910_abc_reverte/`, numerado de forma continua 001-NNN sin reiniciarse). Cuando el corpus de origen agrupa sus piezas en unidades mayores que reinician la numeración en cada una (p. ej. `fuentes/260824_mundo_west/`, donde la pieza "02" existe de forma independiente en el fascículo 1, en el 5, en el 7...), dos consecuencias:

    - El orden de fila (puntos 8-9) debe tomar como clave la tupla (unidad mayor, pieza), nunca la pieza sola — ordenar solo por pieza entremezclaría unidades distintas sin ningún criterio coherente.
    - La **etiqueta visible** del enlace en `Notas`/`Fuentes` debe incluir el identificador de la unidad mayor junto con la pieza (p. ej. `[02-08]`, nunca `[08]` a secas), porque una etiqueta con solo la pieza es ambigua a simple vista sin seguir el enlace — el lector no puede saber, sin abrirlo, a qué unidad mayor pertenece.

    Caso real que motiva este punto: `260822-index-documentation-west-mundo-bibliografia-recomendada.md` citaba sus fuentes como `[08]`, `[20]`, etc. (solo pieza), heredado sin adaptar de la redacción del índice hermano de Reverte (donde sí basta, por ser capítulo global); corregido a formato `fascículo-pieza` en las ~170 referencias del índice. Revisado el resto de índices del proyecto (cronologías de Europa, España y Este; índice bibliográfico del corpus chino) y ninguno más presenta este problema, bien porque su identificador es global, bien porque su etiqueta ya es el nombre de archivo completo, bien porque no citan piezas por número.

---

## Consequences

### Positive

* Deja trazada la decisión de separación entre vocabulario curricular y vocabulario contextual por ficha, evitando decidirlo de forma improvisada cuando la necesidad aparezca.
* Sienta las bases para un caso de uso pedagógico reconocido como importante por el usuario, sin comprometerse todavía a un formato concreto no probado.
* (2026-08-02) Fija la tabla Markdown como formato estándar de entrada y admite fuente derivada del propio corpus (además de fuente externa única) a partir de un segundo caso real, sin fragmentar la decisión en un ADR adicional. La sección Glosario de ADR-1110 hereda la generalización sin necesidad de editar ese documento.
* (2026-08-04) Aclara que el artefacto de consolidación intermedio (CSV en `fuentes/`) es opcional e instrumental, no un componente permanente de la ficha — evita mantener indefinidamente una copia redundante de los mismos datos en dos sitios una vez la tabla final está verificada.

### Negative

* Introduce una copia derivada de los datos de vocabulario cuando la fuente es un mazo existente en `vocabulary/` (punto 6). Ver sección Verification para el check que detecta desincronización.

### Risks Mitigated

* Riesgo de contaminar `vocabulary/vocabulario.csv` (registro de progreso curricular real) con términos extraídos automáticamente de fuentes procesadas.
* Riesgo de definir un formato de vocabulario por ficha de forma especulativa, sin casos reales de Knowledge Assets derivados de texto chino delante.
* (2026-08-02) Riesgo de repetir, para el índice de ubicaciones, el mismo problema de mantenimiento de referencias inversas que ADR-1110 ya identificó y resolvió para el Glosario — mitigado adoptando aquí la misma direccionalidad (enlace desde la ficha de origen hacia la entrada, nunca al revés).
* (2026-08-04) Riesgo de divergencia entre un artefacto de consolidación retenido indefinidamente (CSV) y la tabla final del Knowledge Asset, si futuras ediciones actualizan una sin la otra — mitigado aclarando que el artefacto es opcional/instrumental y puede retirarse tras verificar su equivalencia, en vez de mantenerse como componente permanente a sincronizar.
* Riesgo de desincronización entre el mazo original en `vocabulary/` y su copia derivada en un Knowledge Asset, si el mazo se corrige más adelante y la copia no se actualiza — mitigado mediante el check de la sección Verification, no solo documentado como limitación aceptada.

---

## Verification (run before closing any task touching a vocabulary-derived Knowledge Asset or its source `vocabulary/*.xml`)

Este proyecto no tiene pipeline de CI — el asistente de IA (o el usuario) es la puerta de validación (`AI_ENTRYPOINT.md` § Self-Review Gate). Este check, paramétrico y reutilizable para cualquier par mazo/ficha, hace esa puerta concreta en vez de solo aspiracional:

```bash
# Uso: verificar_sync_vocabulario <xml-origen> <knowledge-asset.md>
verificar_sync_vocabulario() {
  python3 -c "
import sys, xml.etree.ElementTree as ET
cards = ET.parse(sys.argv[1]).getroot().findall('.//card')
for i, card in enumerate(cards, 1):
    e = card.find('entry')
    hw = e.find(\"headword[@charset='sc']\").text
    pron = e.find('pron').text
    defn = e.find('defn').text.strip()
    print(f'{i}. {hw} ({pron}) — {defn}')
" "$1" > /tmp/xml_actual.txt

  python3 -c "
import sys, re
def unpad(cell):
    # quita solo el espacio de relleno de la tabla (uno en cada borde), no toda whitespace:
    # algunas entradas tienen un tab o un espacio real como parte de los datos originales
    # (p. ej. XinDongFang ^260801-XND-00083/00214/00298) que .strip() borraria de mas.
    if cell.startswith(' '): cell = cell[1:]
    if cell.endswith(' '): cell = cell[:-1]
    return cell
with open(sys.argv[1], encoding='utf-8') as f:
    content = f.read()
rows = re.findall(r'^\|([^\n|]+)\|([^\n|]+)\|([^\n|]+)\|\s*\^[0-9]{6}-[A-Z]{3}-[0-9]{5}\s*\$', content, re.M)
for i, (hw, pron, defn) in enumerate(rows, 1):
    print(f'{i}. {unpad(hw)} ({unpad(pron)}) — {unpad(defn)}')
" "$2" > /tmp/ka_actual.txt

  diff /tmp/xml_actual.txt /tmp/ka_actual.txt
}
```

(Extracción del lado del Knowledge Asset actualizada el 2026-08-02: lee filas de tabla Markdown en vez de ítems de lista numerada, conforme a la revisión de ese día que fija la tabla como formato estándar. Quita exactamente un espacio de relleno por borde de celda — no un `.strip()` genérico, que borraría tabuladores o espacios reales presentes en los datos de origen. La reconstrucción `i. hw (pron) — defn` para el diff no cambia.)

Salida vacía = el Knowledge Asset y su XML origen siguen sincronizados. Cualquier línea de diferencia identifica exactamente qué entrada cambió en el XML desde que se curó la ficha, sin necesidad de comparar a mano las 149 (o las que sean) entradas.

Coste: milisegundos por par mazo/ficha (parseo de XML con la librería estándar de Python, sin dependencias nuevas) — no supone un problema de rendimiento ni a decenas de mazos.

Validado contra el primer caso real (2026-08-01):

```bash
verificar_sync_vocabulario vocabulary/260718_anno00_MandarinCorner_HSK1.xml conocimiento/260801-language-hsk1-mandarin-corner-150-oraciones.md
```

Salida vacía confirmada en el momento de creación de la ficha.

Re-validado tras la migración de lista a tabla (2026-08-02) contra ambos mazos ya migrados:

```bash
verificar_sync_vocabulario vocabulary/260718_anno00_MandarinCorner_HSK1.xml conocimiento/260801-language-hsk1-mandarin-corner-150-oraciones.md
verificar_sync_vocabulario vocabulary/260718_anno00_XinDongFang_HSK1.xml conocimiento/260801-language-hsk1-xindongfang-vocabulario.md
```

Salida vacía confirmada en ambos casos con el script de extracción de tabla actualizado (149 y 674 entradas respectivamente). La primera versión del script de extracción de tabla (con `.strip()` genérico en vez de la función `unpad` descrita arriba) produjo 3 falsos positivos en XinDongFang (entradas `^260801-XND-00083`, `^260801-XND-00214`, `^260801-XND-00298`) por borrar un tabulador y un espacio que son datos originales del XML, no relleno de tabla — corregido antes de dar el check por bueno.

---

## References

* vocabulary/vocabulario.csv
* conocimiento/260801-language-hsk-1-cambios-v2-vs-v3.md (caso que motivó esta decisión)
* docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* docs/decisiones/ADR-0033-Taxonomia_ADRs.md
* docs/decisiones/ADR-1109-Distincion_Grafo_Obsidian_vs_Grafo_Understand_Anything.md (criterio de relación verificada)
* docs/decisiones/ADR-1110-Seccion_Glosario_en_Knowledge_Assets.md (precedente de direccionalidad sin referencia inversa; casos reales de Glosario en tabla: las 6 fichas de la tesis de Zhilong Shi)
* conocimiento/260802-geography-indice-ubicaciones-conocimiento.md (caso real de índice derivado de corpus, formato tabla desde su creación; sus CSV intermedios en `fuentes/260802_ubicaciones/` se retiraron el 2026-08-04 tras verificar su equivalencia — ver Notes de esa ficha y punto 4 de Decision)
