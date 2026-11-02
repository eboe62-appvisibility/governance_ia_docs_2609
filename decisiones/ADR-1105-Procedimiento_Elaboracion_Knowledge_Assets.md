# ADR-1105: Procedimiento de Elaboración y Verificación de Knowledge Assets por Lotes

Status: APPROVED

Date: 2026-07-26

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Documentation, Research, Governance

Tags:
knowledge-asset, knowledge-repository, methodology, taxonomy, batch-processing

Related ADRs:
ADR-1100, ADR-1101, ADR-1102, ADR-1103, ADR-1104, ADR-1106 (enmienda parcial — retirada del Relationship Record), ADR-1108 (formato de Knowledge Asset recopilatorio), ADR-1109 (exclusión de recopilatorios de la bóveda)

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Enmienda Parcial (2026-07-30)

Fase 3 y checklist de la Fase 4: revocadas en lo relativo a Relationship Record por ADR-1106. Resto del procedimiento vigente sin cambios.

---

## Nota de Revisión (2026-08-03)

Se añade a la Fase 4 un quinto punto de checklist: revisión obligatoria de si el contenido de la ficha (o lote) recién elaborada aporta entradas a un Knowledge Asset recopilatorio existente (vocabulario, glosario, índice derivado de corpus, ADR-1108) o justifica crear uno nuevo. Motivo: tras construir el índice de ubicaciones geográficas (`260802-geography-indice-ubicaciones-conocimiento.md`) a partir de referencias ya presentes en decenas de fichas previas, quedó claro que sin esta revisión los recopilatorios se quedan desactualizados respecto al corpus creciente — cada nueva ficha es una fuente potencial de entradas que de otro modo solo se detectarían de forma ad hoc. Se documenta como obligación explícita, no como buena práctica implícita, siguiendo el mismo motivo que originó este ADR (evitar que un procedimiento de valor duradero dependa de que cada sesión lo re-derive por su cuenta).

---

## Nota de Revisión (2026-08-22) — Criterio DRAFT → VERIFIED

El campo `Status` de `ai/templates/30_knowledge_asset.md` (`[DRAFT | VERIFIED | DEPRECATED | SUPERSEDED]`) no tenía, hasta esta fecha, ningún criterio documentado para la transición `DRAFT` → `VERIFIED` — consecuencia: el 100% de las Knowledge Assets del corpus (111 ficheros a esta fecha, 98 en `conocimiento/` y 13 en `conocimiento_index/`) permanecían en `DRAFT` indefinidamente, sin que el campo aportara ninguna señal real.

**Criterio adoptado**: `VERIFIED` significa que **el documento en sí está construido y estable** — ha completado su proceso de elaboración (Fase 3), su checklist de verificación (Fase 4) y, si aplica, tiene un mecanismo de mantenimiento activo que lo mantiene alineado con el corpus (`ai/skills/documentation/verificar_indices_derivados.md`). **No significa que no quede ningún dato pendiente de verificar** — mismo criterio ya vigente para `Status: APPROVED` en los ADR de este proyecto, que siguen acumulando secciones y notas fechadas sin dejar de estar aprobados. Los mecanismos de matiz a nivel de dato (marcador `†` de inferencia no verificada, `Confidence Level` de `05_EVIDENCE_CONFIDENCE.md`, secciones "Pregunta sin resolver") son independientes del campo `Status` y siguen operando sin cambios tras la promoción — `VERIFIED` no los sustituye ni los oculta.

**Aplicación (2026-08-22)**: los 13 índices de `conocimiento_index/` (`Knowledge Type: Index`) pasan a `VERIFIED` — justificado por su nivel de escrutinio activo (auditorías repetidas, `verificar_indices_derivados` vigilándolos hacia adelante) frente al resto del corpus. Los 98 Knowledge Assets de contenido en `conocimiento/` **permanecen deliberadamente en `DRAFT`** — decisión explícita del usuario de no extender la promoción a ese conjunto todavía, sin que ello implique un juicio sobre su calidad.

---

## Context

Durante la elaboración de 54 Knowledge Assets y 54 Relationship Records a partir del corpus `fuentes/260726_ABC/` (54 artículos de prensa depurados), se aplicó un procedimiento completo — ingesta, depuración, chequeo de cobertura `Domain`/`Tags`/Ontology, elaboración en lotes paralelos, verificación post-elaboración y secuencia de ramas — que **no está documentado en ningún ADR existente**. `ADR-1103.md` regula el *merge* entre ramas, pero no cómo se elabora contenido dentro de una rama de contenido; `ADR-1100.md` y `ADR-1101.md` regulan ubicación y naming, pero no el proceso de curación en sí.

Este procedimiento tiene valor duradero: se repetirá con cada nuevo corpus incorporado al proyecto (ya ocurrió antes, de forma menos sistemática, con `fuentes/260726_revista_CBI/`). Dejarlo sin documentar implica que cada sesión futura tendría que re-derivarlo, con riesgo de inconsistencia (criterios de verificación distintos, tamaños de lote arbitrarios, o —el riesgo más importante observado en la práctica— fichas que declaran relaciones no verificadas entre sí por simple coincidencia temática).

**Nota de gobernanza**: este ADR fue aprobado explícitamente por el usuario el 2026-07-26, junto con la activación del skill `ai/skills/documentation/elaborar_knowledge_assets.md` que lo operacionaliza.

Evidencia concreta de por qué el procedimiento es necesario, observada durante su aplicación:

* Sin un chequeo de cobertura previo (`ADR-1104.md`), 3 de los 54 documentos no habrían tenido un `Domain` adecuado (`Society`, `Education`, `Ecology`).
* Sin una regla explícita de lectura íntegra de la fuente antes de escribir, dos lotes paralelos habrían podido duplicar la misma ficha bajo nombres distintos (ocurrió un caso límite: los ficheros `130201_NYT_Economia.md` y `130202_NYT_Economia.md` recibieron sugerencias de `Domain`/slug cruzadas por error del orquestador; dos lotes distintos lo detectaron y corrigieron de forma independiente durante la lectura, sin necesidad de intervención humana).
* Sin una regla explícita de "no declarar relación sin verificación cruzada", el hallazgo de temática solapada entre varias fichas (dos perfiles de Mao Zedong, tres artículos sobre inmigración china en España, dos artículos sobre Zheng He) se documentó como nota en `Notes`/`Related Assets`, no como una relación `related_to` fabricada — este es precisamente el comportamiento que se quiere formalizar como regla, no como buena suerte.

---

## Decision

Se adopta el siguiente procedimiento como Knowledge Asset Elaboration Procedure, obligatorio para cualquier corpus de fuentes ya ingerido conforme a `ADR-1101.md`.

### Fase 1 — Ingesta y depuración de fuentes

* Ubicación conforme a `ADR-1101.md` (`fuentes/<corpus>/`).
* Depuración de contenido ajeno al tema principal del proyecto (estudios chinos) cuando el escaneo original mezclaba artículos no relacionados.
* Corrección del nombre de fichero al día real de publicación cuando el propio documento lo revele (patrón `YYMMDD`, sin inventar el dato si no está disponible).

### Fase 2 — Chequeo de cobertura Domain/Tags/Ontology

* Antes de iniciar la elaboración de fichas, contrastar el tema principal de cada documento del corpus contra el vocabulario canónico vigente (`ADR-0033-Taxonomia_ADRs.md` § Domain) y las familias de `docs/Project_Definition/03_ONTOLOGY.md`.
* Si existen huecos de cobertura reales (no forzables con el vocabulario vigente), proponer su ampliación mediante un ADR específico (siguiendo el precedente de `ADR-1104.md`) **en la rama `develop`**, antes de continuar a la Fase 3.
* No proceder a la Fase 3 usando términos de `Domain` fuera del vocabulario canónico vigente.

### Fase 3 — Elaboración de fichas en lotes paralelos

* Tamaño de lote recomendado: **~9 documentos por lote** (equilibrio observado entre paralelismo y carga de contexto por agente).
* Cada lote se asigna a un agente independiente con: la lista exacta de ficheros fuente asignados, el vocabulario `Domain` vigente completo, los niveles de confianza de `05_EVIDENCE_CONFIDENCE.md`, y una sugerencia inicial de `Domain`/slug por documento (ajustable tras la lectura).
* **Regla obligatoria**: cada agente debe leer el fichero fuente completo antes de escribir su ficha — no se infiere contenido a partir del nombre de fichero o de metadatos.
* **Regla obligatoria**: toda desviación respecto al `Domain`/slug sugerido debe justificarse explícitamente en el campo `Purpose` de la ficha resultante.
* **Regla obligatoria**: ninguna ficha declara una relación (`Related Knowledge Assets`, `related_to`) con otra ficha salvo que se haya verificado explícitamente por lectura cruzada — el solapamiento temático detectado sin verificación cruzada se documenta como nota informativa, nunca como relación formal.
* **Regla obligatoria (añadida 2026-08-23)**: verificar (`ls`/`grep`) la existencia de cualquier recurso citado (índice, Knowledge Asset, ADR) antes de enlazarlo; si no se confirma, omitir el enlace. Todo dato no verificable contra la fuente (nombres traducidos, hanzi/pinyin de conocimiento general) se marca `†`.
* Knowledge Type, Status y Confidence Level se asignan conforme a `ai/templates/30_knowledge_asset.md` y `05_EVIDENCE_CONFIDENCE.md` respectivamente. Criterio de promoción `DRAFT` → `VERIFIED`: ver "Nota de Revisión (2026-08-22)" al inicio de este ADR.

### Fase 4 — Verificación post-elaboración (checklist obligatorio)

Antes de considerar completa la elaboración de un lote o corpus:

1. **Cobertura**: cada fichero fuente del corpus tiene exactamente una ficha Knowledge Asset (ni huérfanos ni duplicados).
2. **Integridad estructural**: ninguna ficha está truncada (verificar que termina en el bloque `Prohibited Actions` de la plantilla).
3. **Sin colisiones**: no hay nombres de fichero duplicados entre lotes paralelos.
4. **Solapamientos documentados, no inventados**: los casos de temática solapada detectados se listan aparte, sin relaciones formales no verificadas.
5. **Revisión de índices implicados (añadido 2026-08-03, actualizado 2026-08-06)**: para cada ficha o lote elaborado, revisar si su contenido aporta entradas verificables a algún Knowledge Asset `Knowledge Type: Index` ya existente en `conocimiento_index/` (vocabulario, glosario, índice derivado de corpus — ADR-1108), o si el volumen aportado justifica crear uno nuevo ahí.
   - **Regla anti-fabricación (idéntica a la de Fase 3)**: toda entrada incorporada a un índice, existente o nuevo, debe ser literal y verificable en la fuente citada de la ficha de origen — nunca se completa con conocimiento general para "engordar" el índice.
   - **Incrementar un índice ya existente**: no exige umbral mínimo; cualquier entrada verificada se añade, respetando el orden declarado en su propia sección `Description` (alfabético, cronológico u otro) — no se añade al final sin más. Ver `ADR-1108` punto 8 para la regla de inserción ordenada y numeración de bloque.
   - **Crear un índice nuevo**: exige una masa crítica que justifique el esfuerzo (evitar índices triviales de una o dos entradas); la justificación se documenta explícitamente en el `Purpose` de la ficha resultante, ubicada en `conocimiento_index/` conforme a `ADR-1100`.
   - Todo índice nuevo sigue el formato tabla de `ADR-1108` y se evalúa contra el criterio de rol estructural de `ADR-1109` (punto 5) — desde 2026-08-06, la exclusión de la bóveda de Obsidian ya la cubre la regla de carpeta `conocimiento_index/`, sin paso adicional.
   - Como con cualquier contenido nuevo, la creación o el incremento de un índice requiere confirmación explícita del usuario antes de comitear (Fase 5) — no se comitea de forma automática por derivarse de esta revisión.
6. **Enlaces citados verificados y datos inferidos marcados (añadido 2026-08-23)**: todo enlace a un recurso del proyecto se ha verificado existente; todo dato no verificable literalmente contra la fuente está marcado `†`.

### Fase 5 — Secuencia de ramas y commits

* La elaboración de fichas ocurre en la rama de contenido (nunca se commitea automáticamente: se espera confirmación explícita del usuario, conforme al principio general de separación análisis/ejecución).
* Tras la verificación de la Fase 4 y la confirmación del usuario, se commitea el lote completo en la rama de contenido.
* El resto de la secuencia (merge a `develop`, regeneración del grafo de conocimiento, merge a `master`) sigue `ADR-1102.md` y `ADR-1103.md` sin modificación.

### Diagrama del pipeline completo

```
 fuentes/<corpus>/ (escaneo bruto, ADR-1101)
        │
        ▼
 Fase 1 — Depuración
   • quitar contenido ajeno
   • corregir YYMMDD real
        │
        ▼
 Fase 2 — Chequeo Domain/Tags/Ontology
   • contrastar contra ADR-0033 §Domain + 03_ONTOLOGY.md
   • ¿hueco real de cobertura?
        │           │
        │ no        │ sí
        │           ▼
        │     ADR de ampliación (ver ADR-1104)
        │     — se redacta y aprueba en `develop` —
        │           │
        └─────┬─────┘
              ▼
 Fase 3 — Elaboración por lotes paralelos (~9 docs/lote)
   • leer fuente completa antes de escribir
   • Domain/slug sugerido → ajustar y justificar en Purpose
   • no declarar relaciones sin verificación cruzada
        │
        ▼
 Fase 4 — Verificación (checklist)
   1. cobertura 1:1 fuente→Knowledge Asset
   2. integridad estructural (sin truncados)
   3. sin colisiones de nombre
   4. solapamientos documentados, no inventados
   5. revisión de recopilatorios implicados (ADR-1108/1109)
        │
        ▼
 Fase 5 — Ramas y commits
   rama de contenido (sin commit automático)
        │  ← confirmación explícita del usuario
        ▼
   commit en rama de contenido
        │  ← usuario decide cerrar la rama (ADR-1103)
        ▼
   merge → develop
        │
        ▼
   regenerar knowledge graph (/understand)
        │  ← develop por delante de master + grafo fresco (ADR-1103)
        │  ← confirmación explícita del usuario
        ▼
   merge → master
```

---

## Consequences

### Positive

* El procedimiento aplicado ad hoc en la sesión que originó este ADR queda reproducible para futuros corpus (p. ej. futuras incorporaciones a `fuentes/`).
* Reduce el riesgo de relaciones fabricadas entre Knowledge Assets — riesgo real observado (solapamientos temáticos genuinos que podrían haberse convertido en relaciones no verificadas).
* Establece un tamaño de lote de referencia (9 documentos) basado en experiencia real, evitando improvisación futura.
* El checklist de verificación de la Fase 4 es mecánico y auditable (recuento de fuentes citadas, comprobación de sufijo `Prohibited Actions`, diff de nombres de fichero).
* (2026-08-03) Los Knowledge Assets recopilatorios (vocabulario, glosario, índices) se mantienen alineados con el corpus creciente de forma sistemática, en vez de depender de que alguien repare ad hoc en que una ficha nueva aporta entradas relevantes.

### Negative

* Añade una fase de gobernanza (Fase 2) antes de poder empezar a elaborar fichas, lo que puede percibirse como sobrecarga en corpus pequeños o de un solo documento.
* El tamaño de lote recomendado (9) es una observación empírica de un único corpus (54 documentos) y puede no ser óptimo para corpus de tamaño muy distinto.
* (2026-08-03) Añade un quinto paso de revisión a la Fase 4 para cada ficha o lote — sobrecarga adicional, mitigada por no exigir umbral al incrementar un recopilatorio ya existente (solo al crear uno nuevo) y por seguir apoyándose en el mismo agente/lote que ya está leyendo la fuente completa.

### Risks Mitigated

* Fichas Knowledge Asset con `Domain` forzado por ausencia de una opción adecuada.
* Relaciones `related_to` fabricadas a partir de similitud temática superficial, sin verificación cruzada real.
* Fichas truncadas o incompletas pasando desapercibidas tras interrupciones de sesión (observado en la práctica: un lote se cortó por límite de sesión y la verificación posterior confirmó que, pese al corte, ningún fichero quedó incompleto).
* Commits prematuros de contenido de conocimiento antes de completar la verificación.
* (2026-08-03) Recopilatorios (vocabularios, índices) quedando desactualizados respecto al corpus creciente por falta de una revisión sistemática — mitigado por el punto 5 del checklist de Fase 4.
* (2026-08-03) Proliferación de recopilatorios triviales (uno por dominio, con una o dos entradas) — mitigado por exigir masa crítica y justificación explícita en `Purpose` solo para la creación de un recopilatorio nuevo, no para incrementar uno existente.

---

## References

* ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md
* ADR-1102-Modelo_de_Ramas_Gobernanza_vs_Contenido.md
* ADR-1103-Procedimiento_Validado_de_Integracion_de_Ramas.md
* ADR-1104-Ampliacion_Domain_Tags_Corpus_ABC.md
* ADR-1106-Retirada_Relationship_Record.md
* ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md (formato de Knowledge Asset recopilatorio)
* ADR-1109-Distincion_Grafo_Obsidian_vs_Grafo_Understand_Anything.md (criterio de exclusión de recopilatorios de la bóveda)
* ai/templates/30_knowledge_asset.md
* docs/Project_Definition/05_EVIDENCE_CONFIDENCE.md
* ai/skills/documentation/elaborar_knowledge_assets.md (skill que operacionaliza este procedimiento)
* fuentes/260726_ABC/ y conocimiento/ (54+54 fichas elaboradas como caso de referencia de este procedimiento)
