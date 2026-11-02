# ADR-1110: Sección "Glosario" en Knowledge Assets, con Enlace a Mazos de Vocabulario Existentes

Status: APPROVED

Date: 2026-08-02

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Language, Documentation

Tags:
knowledge-asset, vocabulario, obsidian, block-id

Related ADRs:
ADR-1100 (plantilla y convención de nombres de Knowledge Assets), ADR-1108 (vocabulario específico por Knowledge Asset e identificadores de bloque de Obsidian), ADR-1109 (distinción grafo Obsidian vs. understand-anything, y criterio de relación verificada)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

Al elaborar las 6 Knowledge Assets derivadas de la tesis doctoral de Zhilong Shi (`fuentes/260718_UAM/transcripcion/260505_UAM_tesis_shi_zhilong_identidad_china.md`), cada ficha cita numerosos términos chinos (conceptos, topónimos, expresiones coloquiales) documentados en Hanzi + Pinyin dispersos a lo largo de la sección `Description`. El usuario señaló que sería útil consolidar esos términos en una sección propia, y — donde exista una coincidencia real, no solo temática — enlazarlos mediante wikilink a su entrada exacta en un mazo de vocabulario ya elaborado como Knowledge Asset (`ADR-1108`), aprovechando los identificadores de bloque de Obsidian que esos mazos ya tienen.

No existía ningún precedente de sección "Glosario" en ningún Knowledge Asset del proyecto — se verificó explícitamente antes de proceder (`grep` sin resultados sobre "glosario"/"glossary" en `conocimiento/` y `ai/templates/`).

Comprobación de solapamiento real antes de aprobar la decisión: de ~100 términos en Hanzi citados en las 6 fichas de la tesis, solo 6 coinciden textualmente con entradas de los mazos ya elaborados (`vocabulary/260718_anno00_XinDongFang_HSK1.xml` vía `conocimiento/260801-language-hsk1-xindongfang-vocabulario.md`); ninguno coincide con el mazo Mandarin Corner. Tres de esas seis coincidencias (华工/huagong, 华侨/huaqiao, 华人/huaren) no son casuales: son precisamente los términos centrales de migración que la ficha de antecedentes migratorios de la tesis define, y coinciden en hanzi, pinyin y significado exactos con las entradas `^260801-XND-00648/649/650` del mazo XinDongFang. Este hallazgo confirma valor real para la sección, aunque acotado — la mayoría del vocabulario académico/sociológico de una fuente como esta tesis no tendrá equivalente en mazos de vocabulario básico (HSK1).

---

## Decision

1. Se añade una sección opcional `## Glossary` (corregido 2026-08-06: el encabezado real de la plantilla y de las 6 fichas ya existentes siempre fue en inglés — esta Decision usaba "Glosario" solo en la prosa, lo que causó una búsqueda fallida al construir el glosario consolidado; ver Notes) a `ai/templates/30_knowledge_asset.md`, ubicada inmediatamente después de `## Description` y antes de `## Key Characteristics`.

2. **Uso**: no es obligatoria para todo Knowledge Asset — se usa cuando la sección `Description` de la ficha ya cita un número significativo de términos chinos que merece la pena consolidar para el lector, a criterio del redactor (Principio de Proporcionalidad).

3. **Formato de cada entrada** (corregido 2026-08-06): tabla Markdown, una entrada por fila, columnas `Hanzi | Pinyin | Traducción | Enlace vocabulario | Enlace glosario` (formato estándar fijado en ADR-1108 § Decision punto 3, revisado 2026-08-02; columna `Enlace glosario` añadida en esta revisión — ver punto 6), con identificador de bloque `^YYMMDD-xxx-NNNNN` como último token de cada fila.

4. **Enlazabilidad selectiva**: para cada término del glosario, se comprueba si existe una entrada equivalente — mismo hanzi, mismo pinyin, mismo significado, no solo coincidencia superficial de caracteres — en un Knowledge Asset de vocabulario específico ya elaborado conforme a `ADR-1108` (mazos con identificador de bloque `^YYMMDD-xxx-NNNNN`). Si existe una coincidencia genuina, se rellena la columna `Enlace vocabulario` con un enlace al bloque exacto: `[alias](../conocimiento_index/<archivo>.md#^YYMMDD-xxx-NNNNN)` (formato corregido 2026-08-06 para reflejar la ubicación real en `conocimiento_index/`, ver ADR-1100; el wikilink `[[...]]` sigue siendo válido como alternativa equivalente, ver ADR-1109). Si no existe coincidencia, el término se lista igualmente, con esa celda vacía — nunca se fuerza un enlace por proximidad temática, conforme al criterio de relación verificada de `ADR-1109`.

5. **Exclusión explícita de `vocabulary/vocabulario.csv`**: este glosario nunca enlaza contra el vocabulario curricular maestro, que no tiene identificadores de bloque a nivel de entrada y que `ADR-1108` mantiene deliberadamente aislado de cualquier flujo con Knowledge Assets, en cualquier dirección. La comprobación de coincidencia se hace únicamente contra mazos de vocabulario ya convertidos en Knowledge Asset enlazable.

6. **Columna `Enlace glosario` — reciprocidad acotada con el glosario consolidado (añadido 2026-08-06)**: cuando existe un Knowledge Asset consolidado que agrega las secciones `## Glossary` del corpus (`Knowledge Type: Index` en `conocimiento_index/`, p. ej. `260806-index-language-glosario-consolidado.md`), cada entrada de una sección `## Glossary` de origen enlaza de vuelta a su fila exacta en ese consolidado: `[glosario consolidado](../conocimiento_index/<archivo>.md#^YYMMDD-xxx-NNNNN)`. A diferencia de la prohibición de columna de referencia inversa de `ADR-1108` punto 4 (que evita que un agregador liste, sin límite, qué fichas lo citan), esta es una relación 1:1 acotada por fila entre exactamente dos fichas — no crece con el corpus — por lo que no incurre en ese problema.

---

## Consequences

### Positive

* Consolida en un único punto de lectura los términos chinos ya dispersos en `Description`, sin obligar a una relectura completa de la ficha para localizarlos.
* Aprovecha la infraestructura de identificadores de bloque ya construida por `ADR-1108` para un segundo caso de uso (enlazar *desde* una ficha narrativa *hacia* una entrada de vocabulario, en vez de que la ficha *sea* el vocabulario).
* Mantiene la disciplina de no fabricar relaciones: el enlace solo aparece cuando hay coincidencia real de contenido, verificada término a término.

### Negative

* Añade un paso de comprobación manual (contrastar cada término contra los mazos existentes) al elaborar cualquier Knowledge Asset con vocabulario denso.
* El valor práctico de la sección crece con el número de mazos de vocabulario ya elaborados como Knowledge Asset — con solo dos mazos existentes (Mandarin Corner, XinDongFang), la tasa de acierto es baja para fuentes de vocabulario no básico (6 de ~100 términos en el caso que motivó este ADR).

### Risks Mitigated

* Términos chinos relevantes de una ficha quedando dispersos y difíciles de localizar en fichas narrativas largas.
* Enlaces fabricados por coincidencia superficial de caracteres en vez de coincidencia real de hanzi+pinyin+significado.
* Enlaces rotos o conceptualmente incorrectos contra `vocabulary/vocabulario.csv` (que no tiene bloques direccionables).

---

## References

* ai/templates/30_knowledge_asset.md
* docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md
* docs/decisiones/ADR-1109-Distincion_Grafo_Obsidian_vs_Grafo_Understand_Anything.md
* conocimiento/260802-society-shi-zhilong-01-antecedentes-migratorios.md (caso que motivó esta decisión)
