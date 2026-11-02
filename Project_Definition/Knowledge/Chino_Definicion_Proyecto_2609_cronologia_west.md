//Chino_Definicion_Proyecto_2609_cronologia_west.md
Versión 260904
Estado: Corrección aprobada (`docs/decisiones/ADR-1122-Particion_East_West_Indice_Cronologico_Historia.md`, APPROVED) — pendiente de ejecución (rename de East, creación de West, actualización de las 20 referencias)
Ámbito: Arquitectura de Conocimiento — `conocimiento_index/`, `conocimiento_west/`, dominio History (índices cronológicos derivados de corpus y ubicación de las Knowledge Assets de origen occidental)
Rama de trabajo: `cronologia_west`
Documento de referencia que corrige/amplía: `docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md`, `docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md`, `docs/decisiones/ADR-1118-No_Solapamiento_entre_Indices_de_Corpus_del_Mismo_Tipo_de_Entidad.md`, `docs/decisiones/ADR-1115-Adopcion_Agent_Browser_Web_Monitoring.md`, `docs/Project_Definition/03_ONTOLOGY.md`

==========================================
OBJETIVO
==========================================
Dividir el índice cronológico único del dominio History en dos índices paralelos, East y West, para poder analizar de forma comparada las líneas temporales de la historia de China/Asia Oriental y de la historia de Occidente tal y como está curada en este proyecto: paralelismos, convergencias, divergencias y solapamientos entre ambas.

Hoy el índice existente (`260806-index-history-cronologia.md`) es, de hecho, íntegramente oriental (China, Mongolia, Asia Central). Pasa a llamarse East y se crea un West nuevo, alimentado por fuentes que hasta ahora no tenían destino de indexación cronológica propio.

==========================================
DECISIÓN
==========================================
1. Renombrar `conocimiento_index/260806-index-history-cronologia.md` a `conocimiento_index/260806-index-history-east-cronologia.md`. Contenido y block-ids (`^260806-CRN-NNNNN`) no cambian.
2. Crear `conocimiento_index/YYMMDD-index-history-west-cronologia.md` (`YYMMDD` = fecha real de creación del archivo), mismo esquema de columnas y mismo orden cronológico descendente que East. Block-ids: `^YYMMDD-CRW-NNNNN` — acrónimo propio `CRW`, distinto del `CRN` de East, para diferenciar ambos índices por acrónimo además de por fecha de archivo.
3. Las entradas que documentan contacto directo entre ambas esferas (viaje de Marco Polo, invasión mongola de Rusia/Europa, Tamerlán, futuros casos de la Ruta de la Seda) llevan fila completa en ambos índices, marcadas como "convergencia East/West", apuntando a la(s) misma(s) Knowledge Asset(s).
4. El índice East se sigue nutriendo exactamente como hasta ahora: cada Knowledge Asset nueva de `Domain: History` sobre China/Asia Oriental que se cree añade su fila, sin distinción de proceso respecto al criterio ya vigente desde su creación el 2026-08-06.
5. Las Knowledge Assets elaboradas a partir de `fuentes/260828_abc_reverte/` y `fuentes/260824_mundo_west/` se alojan en `conocimiento_west/`, carpeta de primer nivel hermana de `conocimiento/` y `conocimiento_index/` — mismo criterio estructural (rol/corpus de origen, no dominio) que ya motivó la creación de `conocimiento_index/`. Cada una incorpora, en su cabecera, una franja `tiempo_inicio`/`tiempo_final` estimada por aproximación a partir del contenido del artículo, apoyándose cuando haga falta en fuentes externas (agent-browser, ADR-1115) sin necesidad de documentar ese apoyo en `Evidence and Sources` — es una aproximación declarada como tal, no un dato que exija el aparato de evidencia habitual del resto de la ficha.

==========================================
EXCEPCIONES DOCUMENTADAS A LA GOBERNANZA VIGENTE
==========================================
Esta decisión introduce dos desviaciones deliberadas de reglas ya aprobadas, ambas acotadas exclusivamente al par de índices East/West de cronología histórica — no se extienden a ningún otro índice del corpus:

**Respecto a ADR-1108 (punto 5, derivación mecánica de acrónimos)**: el acrónimo de bloque se deriva mecánicamente de las 3 primeras consonantes del dominio del índice; aplicado dos veces a "cronología" daría `CRN` en ambos índices. Se opta por un acrónimo propio para West (`CRW`) para que East y West queden identificables por su acrónimo de bloque, no solo por la fecha del archivo.

**Respecto a ADR-1118 (no solapamiento entre índices de corpus del mismo tipo de entidad)**: ADR-1118 exige que una entidad figure en exactamente un índice cuando dos índices cubren el mismo tipo de entidad. La razón de esa regla, según su propio texto, es evitar mantener sincronizado a mano un dato que cambia con el tiempo (el caso que la motivó fueron biografías y roles de personas vivas). Una entrada de cronología histórica no comparte ese riesgo: la fecha de un evento pasado no cambia, salvo corrección puntual de transcripción, ya cubierta por la disciplina de verificación habitual del proyecto. Por eso las entradas de convergencia llevan fila completa en ambos índices en vez de vivir en uno solo con referencia cruzada desde el otro.

==========================================
FUENTES Y ELABORACIÓN DEL WEST
==========================================
La construcción de West arranca por `fuentes/260828_abc_reverte/` (Arturo Pérez-Reverte, "Una historia de Europa", 126 piezas transcritas, sin Knowledge Assets elaboradas todavía). Cada Knowledge Asset que se elabore a partir de esta fuente se aloja en `conocimiento_west/` e incorpora, en el momento de crearse, su franja `tiempo_inicio`/`tiempo_final` estimada por aproximación —no todas las piezas de Reverte cubren un único periodo cerrado ni lo declaran con precisión de fecha— y da de alta su fila correspondiente en West. Por el volumen (126 piezas), la elaboración se planifica por lotes, con paralelización vía subagentes cuando proceda, no pieza a pieza en el hilo principal.

En paralelo, `fuentes/260824_mundo_west/` (100 piezas ya transcritas: fascículos de Alejandro Magno, Marx, y piezas sueltas de otros fascículos) pasa por una ronda de valoración adaptada del método ya usado con `mundo_east` — el criterio ya no es "¿es relevante para China?" sino "¿aporta una fecha datable comparable con la línea East?" — antes de decidir qué piezas se convierten en Knowledge Assets. Las que se elaboren siguen el mismo criterio de ubicación y franja temporal estimada que las de Reverte.

`fuentes/260824_mundo_west/` (61 PDF sin transcribir, colección `Reportaje_Historia_NN_Mundo_0010`, más un archivo suelto `Pais.pdf`) queda fuera del alcance de West mientras no esté completamente transcrita. No bloquea el arranque de West ni el resto de esta corrección; se retoma como trabajo posterior, momento en el que se verificará también su rango exacto de fascículos.

Esta estimación de franja temporal no hace falta para las Knowledge Assets que alimentan East: cada pieza transcrita de `fuentes/260824_mundo_east/` ya declara el periodo histórico exacto en su propio apartado "Fuente" (p. ej. "El Mundo... nº 24 — La India (3000 a.C. - 200 a.C.)"), heredado directamente del fascículo de origen, así que la fecha pasa al índice sin investigación adicional. Es la asimetría real entre ambos lados: East hereda la datación de la fuente ya transcrita; West la reconstruye por aproximación porque ni Reverte ni los fascículos de `mundo_west` la declaran siempre con esa misma precisión.

==========================================
IMPACTO DEL RENOMBRADO DE EAST
==========================================
El rename de `260806-index-history-cronologia.md` a `260806-index-history-east-cronologia.md` requiere actualizar:

- 20 referencias en 19 Knowledge Assets de `conocimiento/`, todas con el patrón `[Índice Cronológico del Corpus](../conocimiento_index/260806-index-history-cronologia.md#^260806-CRN-NNNNN)`.
- La lista de índices con enlace recíproco activo en `ai/skills/documentation/verificar_indices_derivados.md` (línea 64).
- El `Knowledge Asset ID` de cabecera del propio archivo renombrado (línea 8), que declara su propia ruta.

==========================================
ALCANCE Y EXCLUSIONES
==========================================
Esta corrección se limita al índice cronológico de History. No establece un precedente automático de partición East/West para el resto de `conocimiento_index/` (personas, destinos, organizaciones, etc.); cualquier partición similar en otro índice requeriría su propia decisión.

==========================================
GOBERNANZA: NUEVO ADR
==========================================
Se propone un ADR nuevo (`ADR-1122`, borrador adjunto) en vez de modificar ADR-1108 o ADR-1118 in situ. Motivo: ambas reglas (derivación mecánica de acrónimo, no solapamiento entre índices) son buenos valores por defecto que deben seguir aplicando a cualquier otro índice del corpus; esta decisión introduce dos excepciones puntuales y justificadas, no un cambio de regla general. Es el mismo patrón que el propio ADR-1118 ya siguió respecto a ADR-1108, y ADR-1119 respecto a ADR-1108 punto 6: un ADR nuevo, focalizado, que referencia y acota al anterior, sin reescribirlo. Por el mismo criterio, la creación de `conocimiento_west/` amplía ADR-1100 punto 2 siguiendo el precedente que la propia `conocimiento_index/` ya sentó (carpeta de primer nivel por rol/corpus de origen, no por dominio), sin reabrir ese ADR.
