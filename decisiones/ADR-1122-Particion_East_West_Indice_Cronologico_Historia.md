# ADR-1122: Partición East/West del Índice Cronológico de Historia

Status: APPROVED

Date: 2026-09-03

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
History

Tags:
knowledge-asset, indice-corpus, cronologia, adr-1108, adr-1118

Related ADRs:
ADR-1100 (ubicación `conocimiento_index/` y convención de nombres — punto 2, del que esta decisión amplía la excepción de carpeta de primer nivel a `conocimiento_west/`), ADR-1108 (patrón de índice derivado de corpus — punto 5, derivación mecánica de acrónimo de bloque, del que esta decisión introduce una excepción puntual), ADR-1118 (no solapamiento entre índices de corpus del mismo tipo de entidad — esta decisión introduce una excepción puntual, acotada, sin alterar la regla general), ADR-1115 (adopción de agent-browser — herramienta de apoyo, opcional, para estimar por aproximación la franja temporal de las Knowledge Assets de `conocimiento_west/`)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

`conocimiento_index/260806-index-history-cronologia.md` es, desde su creación (2026-08-06), un índice de corpus (patrón ADR-1108 punto 3) que agrega periodos/eventos históricos citados en Knowledge Assets de `Domain: History`. A fecha de esta decisión, sus 12 filas son en su totalidad de historia china, mongola o de Asia Central — el índice es, de hecho, íntegramente "oriental", sin que exista contraparte alguna sobre historia "occidental" en el dominio History del corpus.

El objetivo del proyecto se amplía para habilitar el análisis comparado de líneas temporales entre dos mitades del corpus: la historia de China/Asia Oriental ya curada, y la historia de Occidente que se empezará a curar a partir de `fuentes/260828_abc_reverte/` ("Una historia de Europa", Arturo Pérez-Reverte, 126 piezas) y de `fuentes/260824_mundo_west/` (100 piezas ya transcritas). Esto requiere dividir el índice cronológico único en dos índices paralelos — East y West — con el mismo esquema de columnas, para poder identificar paralelismos, divergencias y, sobre todo, convergencias (contacto directo documentado entre ambas esferas: viaje de Marco Polo, invasión mongola de Rusia/Europa, Tamerlán).

Este diseño choca, en dos puntos concretos, con reglas ya aprobadas:

1. **ADR-1108 punto 5** deriva el acrónimo de bloque de cada índice mecánicamente, de las 3 primeras consonantes de su dominio. Aplicado a dos índices que comparten el concepto "cronología", ambos derivarían al mismo acrónimo (`CRN`), disambiguados solo por la fecha de creación del archivo.
2. **ADR-1118** exige que, cuando dos índices de corpus cubren el mismo tipo de entidad, cada entidad figure en exactamente uno de ellos — precedente motivado por la duplicación observada de 4 personas entre `260806-index-history-personas.md` y `260825-index-society-personas-contemporaneas.md`. East y West de cronología son, estructuralmente, dos índices sobre el mismo tipo de entidad ("periodo/evento histórico"), el escenario que ADR-1118 regula.

La razón de fondo de la regla de ADR-1118, según su propia sección `Consequences`, es evitar mantener sincronizado a mano un dato que cambia con el tiempo — el caso real que la motivó fueron biografías y roles de personas vivas. Una entrada de cronología histórica no comparte ese riesgo con la misma intensidad: la fecha de un evento ya ocurrido no cambia, salvo corrección puntual de una transcripción, error ya cubierto por la disciplina de verificación general del proyecto. Este ADR documenta esa diferencia como justificación explícita de una excepción acotada, en vez de aplicar la regla general sin más o de modificarla para todo el corpus.

Existe además una asimetría real en cómo East y West obtienen la fecha de cada entrada. Las Knowledge Assets que alimentan East proceden de `fuentes/260824_mundo_east/`, cuyo apartado "Fuente" ya declara el periodo histórico exacto del fascículo de origen (p. ej. "El Mundo... nº 24 — La India (3000 a.C. - 200 a.C.)"); la fecha pasa al índice sin trabajo adicional. Las Knowledge Assets que alimentarán West, en cambio, proceden de `fuentes/260828_abc_reverte/` y `fuentes/260824_mundo_west/`, fuentes que no siempre declaran con esa misma precisión el periodo que cubre cada artículo — requieren una estimación por aproximación, apoyada cuando haga falta en fuentes externas. Esta asimetría motiva tanto el campo de franja temporal como la ubicación física separada que se deciden a continuación.

### Contexto de la ampliación (2026-09-06) — tercera partición para `fuentes/260824_mundo_west/`

El punto 8 original de esta Decision preveía fusionar en `conocimiento_west/` las Knowledge Assets de dos fuentes distintas: `fuentes/260828_abc_reverte/` (la columna "Una historia de Europa" de Pérez-Reverte, ya completada íntegramente en ese destino — 127 filas del índice West a fecha de esta ampliación) y `fuentes/260824_mundo_west/` (fascículos de la revista *Historia* de *El Mundo*, cuyo procesamiento hacia Knowledge Assets no había comenzado todavía cuando se aprobó este ADR). Al empezar ese segundo lote, el usuario decide explícitamente **no** fusionarlo con el primero: `fuentes/260824_mundo_west/` obtiene su propia carpeta (`conocimiento_west_mundo/`) y su propio índice cronológico, en vez de añadir filas a `conocimiento_west/` y a `260904-index-history-west-cronologia.md`.

Motivo estructural observado, además de la decisión explícita del usuario: `fuentes/260824_mundo_west/` es, a diferencia de la columna de opinión de Reverte (una única voz autorial, un artículo continuo por entrega), una revista de fascículos con muchas piezas cortas y heterogéneas por número (reportaje central, entrevista, tribuna, cronología, bibliografía, breves, editorial) — una granularidad y un formato de fuente distintos que justifican, más allá de compartir el rótulo genérico "Occidente", un tratamiento y una partición propios en vez de mezclarse en un mismo índice con la serie de Reverte.

### Contexto de la ampliación (2026-09-11) — cuarta partición para `fuentes/260910_abc_reverte/`

Con la transcripción completa de "Una historia de España" (Arturo Pérez-Reverte, *XL Semanal*, 92 piezas, I a "y XCII"), el usuario decide dedicar a esta serie una cuarta partición propia, `conocimiento_west_espana/`, en vez de fusionarla en `conocimiento_west/` (la partición ya existente para "Una historia de Europa", del mismo autor). Motivo estructural: aunque comparten autor, medio y columna ("Patente de corso"), son dos obras editoriales distintas y ya delimitadas de forma autónoma por el propio autor — "Una historia de España" se cerró y recopiló en libro (Alfaguara, 2019) antes de que "Una historia de Europa" comenzara —, con series numéricas independientes (I-XCII frente a I-CXXVI) y sin continuidad narrativa entre ambas. Mismo criterio ya aplicado a la separación West/West-Mundo: compartir el rótulo genérico "Occidente" (o, aquí, incluso el mismo autor) no basta por sí solo para fusionar dos corpus editorialmente distintos en un mismo índice.

---

## Decision

1. **Renombrar** `conocimiento_index/260806-index-history-cronologia.md` a `conocimiento_index/260806-index-history-east-cronologia.md`. El contenido y los block-ids existentes (`^260806-CRN-NNNNN`) no cambian — son estables por fecha de creación de la ficha, no del nombre de archivo. Se actualizan las 20 referencias entrantes desde `conocimiento/` (19 Knowledge Assets), la referencia en `ai/skills/documentation/verificar_indices_derivados.md`, y el `Knowledge Asset ID` de cabecera del propio archivo.

2. **Crear** `conocimiento_index/YYMMDD-index-history-west-cronologia.md` (fecha real de creación del archivo), como índice de corpus del mismo patrón que East: mismas columnas (Hanzi, Pinyin, Traducción, Época, Periodo), mismo orden cronológico descendente, misma mecánica de bloque por entrada.

3. **Excepción a ADR-1108 punto 5**: el acrónimo de bloque de West es `CRW`, no el `CRN` que resultaría de la derivación mecánica estándar aplicada por segunda vez al mismo dominio conceptual ("cronología"). Motivo: permitir identificar el índice de origen de un enlace por su acrónimo de bloque, no solo por la fecha del archivo. Esta excepción se limita a este par de índices; cualquier otro índice del corpus sigue la derivación mecánica estándar de ADR-1108 salvo decisión igualmente explícita.

4. **Excepción a ADR-1118**: las entradas que documentan contacto directo entre ambas esferas (convergencia East/West) llevan fila completa en ambos índices, apuntando a la(s) misma(s) Knowledge Asset(s), marcadas explícitamente como "convergencia East/West" en su fila o nota. Esta excepción se limita exclusivamente al par `history-east-cronologia`/`history-west-cronologia`; la regla general de ADR-1118 (una entidad, un único índice) sigue aplicando sin cambios a cualquier otro par de índices de corpus del mismo tipo de entidad, incluidos futuros índices especializados de personas, geografía u otro dominio.

5. El criterio de pertenencia a East no cambia respecto al vigente desde la creación del índice (2026-08-06): cada Knowledge Asset nueva de `Domain: History` sobre China/Asia Oriental añade su fila conforme se crea, sin lista cerrada de carpetas de origen ni criterio adicional.

6. El criterio de pertenencia a West es equivalente: cada Knowledge Asset nueva de `Domain: History` sobre Occidente derivada de `fuentes/260828_abc_reverte/` añade su fila conforme se crea. Las Knowledge Assets derivadas de esa fuente declaran, además, el segmento temporal exacto que cubre cada artículo, dado que no todas las piezas de esa fuente cubren un único periodo cerrado. **Modificado 2026-09-06**: `fuentes/260824_mundo_west/` ya no alimenta este índice — ver punto 12.

7. Esta partición se limita al índice cronológico de History. No constituye precedente automático de partición East/West para ningún otro índice de `conocimiento_index/` (personas, destinos, organizaciones, fuentes curadas, etc.); una partición equivalente en otro índice requeriría su propia decisión explícita.

8. **Excepción a ADR-1100 punto 2 (ubicación)**: las Knowledge Assets elaboradas a partir de `fuentes/260828_abc_reverte/` se alojan en `conocimiento_west/`, carpeta de primer nivel hermana de `conocimiento/` y `conocimiento_index/`. Sigue el mismo criterio que ya justificó la creación de `conocimiento_index/`: separación por rol/corpus de origen, no por dominio. La convención de nombres de archivo no cambia respecto a `conocimiento/` (`YYMMDD-domain_principal-slug.md`); solo cambia la carpeta. **Modificado 2026-09-06**: este punto ya no incluye `fuentes/260824_mundo_west/` — ver punto 12.

9. **Campo `tiempo_inicio`/`tiempo_final`**: toda Knowledge Asset de `conocimiento_west/` declara en su cabecera una franja temporal estimada por aproximación a partir del contenido del artículo, apoyándose cuando proceda en fuentes externas (agent-browser, ADR-1115). Esta franja es una aproximación declarada como tal, no un hecho verificado, y por tanto no requiere documentar el método de apoyo externo en `Evidence and Sources` — excepción acotada exclusivamente a este campo, sin relajar el nivel de evidencia exigido al resto del contenido de la ficha. Las Knowledge Assets de East no necesitan este campo: su fecha se obtiene, como siempre en este corpus, leyendo el contenido de la propia ficha y de su fuente transcrita.

10. **Una Knowledge Asset puede originar más de una fila de cronología** (East o West), cuando documenta de forma sustantiva —no de pasada— más de un periodo o evento históricamente distinto y fechable por separado. Confirmado en la muestra piloto del 2026-09-04: `conocimiento_west/260904-history-siglo-xv-albores-renacimiento-europeo.md` origina dos filas — su periodo central declarado (`^260904-CRW-00001`, siglo XV) y el viaje de Marco Polo que narra como telón de fondo sustancial (`^260904-CRW-00003`, 1271-1295, dos siglos antes) —, ambas con `Periodo` propio y distinto, en vez de fundirse en una sola fila que ocultaría la fecha del segundo evento. Este punto generaliza, en sentido inverso, el principio ya vigente en el índice East desde su creación (varias Knowledge Assets pueden originar una misma fila) — aquí es una misma Knowledge Asset la que puede originar varias filas.

11. **Convención de fechas anteriores a Cristo en `tiempo_inicio`/`tiempo_final`**, confirmada al abordar las 9 primeras entregas de "Una historia de Europa" (Mesopotamia, Egipto, Israel bíblico, mitología griega, Troya, mundo micénico, fenicios, democracia ateniense — todas total o parcialmente anteriores a Cristo, a diferencia de la muestra piloto de 2026-09-04, íntegramente d. C.):
    - Ambos campos se expresan como **enteros negativos** cuando la fecha es a. C. (p. ej. `-1700` para "1700 a. C."); la unidad ("a. C.") no se repite dentro del propio campo, se declara en prosa en `Evidence and Sources`.
    - **Ambos campos son obligatorios sin excepción**: nunca se deja `tiempo_inicio` o `tiempo_final` vacío, aunque la fuente no ofrezca una fecha exacta. En el peor de los casos, se redondea al siglo. Reglas de redondeo, en orden de preferencia:
      1. Si la fuente da una fecha o cifra exacta ("unos 1700 años antes de Cristo", "en el 996 a. C.", "en el año 334 a. C.", una duración sumada a un año de partida), se usa tal cual — igual criterio que ya aplica el índice East (aritmética directa sobre datos explícitos de la fuente, sin inventar).
      2. Si la fuente solo declara un siglo concreto ("siglo IV antes de Cristo"), se redondea al año en centena más próximo al límite narrativo que ese siglo representa: el borde de apertura del siglo (más alejado del presente) cuando marca el *inicio* del periodo cubierto por la ficha, o el borde de cierre (más próximo al presente) cuando marca su *fin*. Ejemplo: "siglo IV a. C." como fecha de cierre → `-300`; "siglo XI a. C." como fecha de apertura → `-1100`.
      3. Si la fuente no ofrece ninguna cifra ni siglo explícito para uno de los dos extremos, se completa por aproximación histórica general (conocimiento externo, no derivado del propio artículo), redondeada igualmente al siglo. Esta categoría de aproximación es más débil que las anteriores y **se documenta explícitamente como tal** en `Evidence and Sources` de la ficha afectada — nunca se presenta como si procediera de la fuente transcrita.
    - **Simetría del redondeo a siglo del lado d. C.**, confirmada al abordar los fascículos XX-XXIX (Roma imperial, cristianismo temprano, crisis del siglo III): la regla 11.2 se aplica igual en positivo. Ejemplo: "a finales de ese siglo II" (d. C.) como fecha de cierre → borde de cierre del siglo, el más alejado del año 1, `200`; "el Imperio del siglo I después de Cristo" como marco de apertura → borde de apertura, el más próximo al año 1, `1`.
    - **Franjas que cruzan la frontera a. C./d. C.**, confirmada con la misma muestra: no existe año 0 en esta convención (como tampoco en el calendario histórico habitual); el año `1` cubre tanto el cierre de una franja a. C. como la apertura de una franja d. C. cuando la fuente sitúa el hecho justo en esa transición (p. ej. "el día 1 antes de Cristo y el día 1 después de Cristo"). Una ficha puede legítimamente declarar `tiempo_inicio` negativo y `tiempo_final` positivo (o viceversa) cuando su contenido cubre ambos lados de la frontera.

12. **Tercera partición — `conocimiento_west_mundo/` (añadido 2026-09-06)**: las Knowledge Assets elaboradas a partir de `fuentes/260824_mundo_west/` (fascículos de la revista *Historia* de *El Mundo*) se alojan en `conocimiento_west_mundo/`, carpeta de primer nivel hermana de `conocimiento/`, `conocimiento_index/` y `conocimiento_west/` — misma excepción a ADR-1100 punto 2 que ya justificó `conocimiento_west/` en el punto 8, aplicada aquí a un tercer corpus de origen distinto. La convención de nombres de archivo no cambia (`YYMMDD-domain_principal-slug.md`).

13. **Índice cronológico propio para la tercera partición (añadido 2026-09-06)**: se crea `conocimiento_index/YYMMDD-index-history-west-mundo-cronologia.md` (fecha real de creación del archivo), mismo patrón de columnas y orden cronológico descendente que East y West. **Excepción a ADR-1108 punto 5**: su acrónimo de bloque es `CRM` (Cronología west-Mundo) — ni el `CRN` mecánico ni el `CRW` ya reservado a West/Reverte —, verificado sin colisión contra el resto de acrónimos de bloque ya en uso en el proyecto. El criterio de pertenencia es equivalente al de East/West: cada Knowledge Asset nueva de `Domain: History` derivada de `fuentes/260824_mundo_west/` añade su fila conforme se crea, con el mismo campo `tiempo_inicio`/`tiempo_final` estimado por aproximación que rige para West (punto 9), sujeto a las mismas convenciones de redondeo y notación a. C./d. C. (puntos 10-11).

14. **Convergencia con East y con West (añadido 2026-09-06)**: la excepción a ADR-1118 del punto 4 (fila completa en ambos índices para entradas de contacto directo documentado entre esferas) se extiende, en los mismos términos, a cualquier par que incluya el nuevo índice `west-mundo` — es decir, `east`/`west-mundo` y `west`/`west-mundo`, además del par `east`/`west` ya cubierto. Esta ampliación no crea automáticamente ninguna fila de convergencia; solo habilita el mecanismo para cuando una Knowledge Asset de `conocimiento_west_mundo/` documente contacto directo con una entrada ya presente en East o en West (p. ej., un fascículo de `fuentes/260824_mundo_west/` sobre un tema que también trate `conocimiento_west/` o `conocimiento/`).

15. **Cuarta partición — `conocimiento_west_espana/` (añadido 2026-09-11)**: las Knowledge Assets elaboradas a partir de `fuentes/260910_abc_reverte/` ("Una historia de España", Arturo Pérez-Reverte, 92 piezas) se alojan en `conocimiento_west_espana/`, carpeta de primer nivel hermana de `conocimiento/`, `conocimiento_index/`, `conocimiento_west/` y `conocimiento_west_mundo/` — misma excepción a ADR-1100 punto 2 ya aplicada a las otras tres particiones, extendida aquí a un cuarto corpus de origen distinto. La convención de nombres de archivo no cambia (`YYMMDD-domain_principal-slug.md`).

16. **Índice cronológico propio para la cuarta partición (añadido 2026-09-11)**: se crea `conocimiento_index/260911-index-history-west-espana-cronologia.md`, mismo patrón de columnas y orden cronológico descendente que East, West y West Mundo. **Excepción a ADR-1108 punto 5**: su acrónimo de bloque es `CRS` (Cronología west-eSpaña) — ni el `CRN` mecánico, ni los ya reservados `CRW` (West) y `CRM` (West Mundo) —, verificado sin colisión contra el resto de acrónimos de bloque ya en uso en el proyecto. El criterio de pertenencia es equivalente al de las otras particiones: cada Knowledge Asset nueva de `Domain: History` derivada de `fuentes/260910_abc_reverte/` añade su fila conforme se crea, con el mismo campo `tiempo_inicio`/`tiempo_final` estimado por aproximación que rige para West y West Mundo (punto 9), sujeto a las mismas convenciones de redondeo y notación a. C./d. C. (puntos 10-11).

17. **Convergencia con East, West y West Mundo (añadido 2026-09-11)**: la excepción a ADR-1118 de los puntos 4 y 14 (fila completa en cada índice para entradas de contacto directo documentado entre esferas) se extiende, en los mismos términos, a cualquier par que incluya el nuevo índice `west-espana` — es decir, `east`/`west-espana`, `west`/`west-espana` y `west-mundo`/`west-espana`, además de los pares ya cubiertos. Esta ampliación no crea automáticamente ninguna fila de convergencia; solo habilita el mecanismo para cuando una Knowledge Asset de `conocimiento_west_espana/` documente contacto directo con una entrada ya presente en cualquiera de los otros tres índices.

---

## Consequences

### Positive

* Habilita el análisis comparado de líneas temporales entre la historia de Asia Oriental y la de Occidente curadas en el proyecto, incluyendo la visibilidad directa de los puntos de contacto entre ambas.
* Preserva intactos los block-ids y enlaces existentes del índice East — el rename no invalida ningún enlace ya escrito, solo requiere actualizar la ruta en las fichas que lo citan.
* Dos excepciones puntuales, documentadas y acotadas, mantienen la regla general de ADR-1108 §5 y ADR-1118 vigente y sin ambigüedad para el resto del corpus.
* `conocimiento_west/` separa físicamente las Knowledge Assets de origen occidental sin necesidad de un campo de dominio nuevo ni de tocar la convención de nombres, siguiendo un precedente ya validado por `conocimiento_index/`.
* La franja `tiempo_inicio`/`tiempo_final` estimada por aproximación, sin exigir su propio aparato de evidencia, evita bloquear la elaboración de las Knowledge Assets de West en una investigación de precisión histórica que no es el objetivo de este corpus.

### Negative

* Introduce dos artefactos de mantenimiento no triviales: 20 referencias a actualizar en el rename inicial, y la disciplina de mantener sincronizadas ambas filas de cada entrada de convergencia si alguna vez se corrige un dato (riesgo aceptado explícitamente en el Context, por ser bajo y poco frecuente).
* El acrónimo `CRW` para West es una desviación de la regla mecánica de ADR-1108, que debe recordarse explícitamente al documentar la ficha (no se infiere automáticamente del dominio).
* La franja temporal de las Knowledge Assets de `conocimiento_west/` es una aproximación, no un dato verificado con el mismo rigor que el resto del corpus — riesgo aceptado explícitamente porque su función es comparativa (ubicar un evento en la línea de tiempo), no probatoria.
* La regla 11.3 (redondeo al siglo por conocimiento histórico externo cuando la fuente no da ninguna cifra) introduce, para ese subconjunto de fechas, un nivel de evidencia más débil que el resto del corpus — riesgo aceptado explícitamente para no dejar nunca un campo obligatorio vacío, mitigado por la exigencia de documentarlo como tal en cada ficha afectada.
* (2026-09-06) Tres particiones (East/West/West-Mundo) en vez de dos añaden un tercer conjunto de referencias a mantener y un tercer acrónimo de bloque (`CRM`) que recordar explícitamente al documentar cada ficha, mismo riesgo ya aceptado para `CRW` en el punto 3.
* (2026-09-11) Cuatro particiones (East/West/West-Mundo/West-España) añaden un cuarto conjunto de referencias a mantener y un cuarto acrónimo de bloque (`CRS`) que recordar explícitamente al documentar cada ficha, mismo riesgo ya aceptado para `CRW` y `CRM`.

### Risks Mitigated

* Ambigüedad futura sobre si esta partición aplica a otros índices del corpus — el punto 7 de la Decision la acota expresamente al índice cronológico de History.
* Duplicación de datos no documentada — al declarar la excepción a ADR-1118 explícitamente aquí, cualquier fila presente en ambos índices queda justificada por este ADR y no se confunde con el descuido que motivó ADR-1118 originalmente.
* (2026-09-06) Fusionar por defecto, sin decisión explícita, dos corpus de origen (Reverte y la revista *Historia* de *El Mundo*) estructuralmente distintos en un mismo índice solo por compartir el rótulo genérico "Occidente" — el punto 12 documenta la decisión explícita de mantenerlos separados y el motivo estructural observado.
* (2026-09-11) Fusionar por defecto, sin decisión explícita, dos obras del mismo autor ("Una historia de España" y "Una historia de Europa") en un mismo índice solo por compartir autor, medio y columna — el punto 15 documenta la decisión explícita de mantenerlas separadas y el motivo estructural observado (series numéricas independientes, obra ya cerrada y recopilada en libro antes del inicio de la siguiente).

---

## References

* `conocimiento_index/260806-index-history-east-cronologia.md` (antes `260806-index-history-cronologia.md`)
* `conocimiento_index/260904-index-history-west-cronologia.md` — completo, 127 filas, íntegro a partir de `fuentes/260828_abc_reverte/` (2026-09-06)
* `conocimiento_west/` — 127 Knowledge Assets, origen exclusivo `fuentes/260828_abc_reverte/` (2026-09-06)
* `conocimiento_index/260904-index-history-west-mundo-cronologia.md` — tercera partición, íntegra a partir de `fuentes/260824_mundo_west/`
* `conocimiento_west_mundo/` — carpeta de primer nivel para las Knowledge Assets de `fuentes/260824_mundo_west/`
* `conocimiento_index/260911-index-history-west-espana-cronologia.md` (creado 2026-09-11, punto 16, esqueleto vacío a la espera de las primeras Knowledge Assets de `conocimiento_west_espana/`)
* `conocimiento_west_espana/` (a crear, punto 15, carpeta de primer nivel para las Knowledge Assets de `fuentes/260910_abc_reverte/`)
* `docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md`
* `docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md`
* `docs/decisiones/ADR-1115-Adopcion_Agent_Browser_Web_Monitoring.md`
* `docs/decisiones/ADR-1117-Metodologia_Transcripcion_Fuentes_PDF_Escaneadas.md` (metodología de transcripción de `fuentes/260910_abc_reverte/`)
* `docs/decisiones/ADR-1118-No_Solapamiento_entre_Indices_de_Corpus_del_Mismo_Tipo_de_Entidad.md`
* `docs/Project_Definition/Chino_Definicion_Proyecto_2609_cronologia_west.md` (documento de trabajo que originó esta decisión)
