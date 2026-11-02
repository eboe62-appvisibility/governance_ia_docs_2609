# ADR-1117: Metodología Obligatoria de Transcripción de Fuentes PDF Escaneadas sin Capa de Texto

Status: APPROVED

Date: 2026-08-18

Scope:
System

Category:
PROCESS

Domain: (Optional)
Documentation, Research

Tags:
sources, validation, transcription, audit, ocr

Related ADRs:
ADR-1101, ADR-1103

Supersedes:
NONE

Superseded By:
NONE

---

## Context

Al transcribir a Markdown los una serie de PDF de `fuentes/260818_viajero/` (recortes escaneados del suplemento *Viajero* de El País), una primera tanda de 8 archivos se generó leyendo el PDF directamente con la herramienta de lectura multimodal. El usuario reportó que el resultado no estaba simplemente desordenado, sino que **faltaba texto real del artículo** — es decir, la herramienta produjo una reconstrucción plausible pero incompleta, presentada como transcripción fiel.

Verificación de causa raíz:

* `pdfinfo` mostró que estos PDF son salida directa de un escáner/multifunción (`Producer: Develop ineo+ 450i`), sin metadatos de autor ni texto seleccionable.
* `pdftotext` sobre el PDF devolvió una salida vacía (0 caracteres de texto extraíble): son imágenes puras, no PDF con capa OCR.
* Los PDF declaran `Page rot: 270`, y ese valor no basta por sí solo para obtener una imagen en orientación de lectura correcta al renderizar: distintas páginas de un mismo PDF multipágina (mismo lote físico de escaneo) requirieron correcciones de rotación distintas entre sí (algunas necesitaron −90° adicionales, otras +90°) para quedar legibles.

Leer directamente un PDF así de rotado y sin capa de texto llevó a un resultado que viola dos principios ya establecidos en `AI_ENTRYPOINT.md` (Truthfulness — *never invent data*; Hallucination Handling — *never present unverified information as a confirmed fact*), con el agravante de que el fallo no era evidente a primera vista: el texto resultante leía como prosa coherente, no como un error obvio.

La corrección aplicada — renderizar cada página a imagen de alta resolución con `pdftoppm`, corregir la rotación verificando visualmente cada página, y transcribir solo desde la imagen ya legible — produjo transcripciones verificablemente completas (recuadros, ladillos y secciones enteras que la primera pasada había omitido aparecieron íntegros). Esta corrección debe fijarse como procedimiento obligatorio, no quedar como conocimiento tácito de esta sesión, porque `fuentes/` ya contiene al menos una colección más de PDF escaneados (`260726_revista_CBI/`) y previsiblemente contendrá más en el futuro.

Además del riesgo técnico de omitir texto por una orientación incorrecta, una fuente escaneada puede contener varios artículos o secciones editoriales. La ausencia de una regla explícita sobre la unidad de extracción permite entregar contenido ajeno al artículo identificado por el nombre del archivo o mezclar artículos independientes en un único Markdown.

### Contexto de la ampliación (2026-08-24) — coste por transcripción y apoyo de OCR local

El usuario planteó reducir el coste (en tokens y en tiempo) de aplicar esta metodología, tanto para este asistente como para cualquier otro asistente de IA con menos recursos disponibles, mediante la introducción de Tesseract OCR (5.3.4, ya instalado en el entorno de ejecución) como apoyo del paso de transcripción.

Antes de decidir el diseño se hicieron dos pruebas reales sobre fuentes ya presentes en el proyecto, en vez de asumir que Tesseract funciona igual de bien para cualquier caso:

* **`fuentes/260101_oushinet/` (prensa china, maquetación densa a varias columnas)**: con los modelos de mayor precisión disponibles (`chi_sim`+`chi_tra`, `tessdata_best`, instalados sin privilegios de administrador — ver punto 11), la salida de Tesseract sobre una página real fue una secuencia de caracteres sin estructura de frase reconocible — inservible incluso como borrador.
* **`fuentes/260726_revista_CBI/` (revista en español/inglés, maquetación más simple)**: con los paquetes `spa`+`eng` ya instalados de fábrica, la salida fue texto corrido mayormente correcto y legible, con solo errores puntuales alrededor de elementos gráficos.

Esta diferencia empírica — no una suposición — es la que fundamenta que el apoyo de Tesseract se adopte de forma diferenciada por tipo de fuente (puntos 8-11 de Decision) en vez de como regla única. El caso concreto que motivó la comprobación, `fuentes/260824_mundo/Reportaje_Historia_11_Mundo_0011.pdf` (mismo escáner `Develop ineo+ 450i` y misma rotación `270°` que el caso original de este Architectural Decision Record), se analiza aparte como evaluación de calidad de una entrega puntual, sin que ese análisis en sí mismo forme parte de esta ampliación.

### Contexto de la ampliación (2026-08-31) — omisión por reconstrucción de memoria al regenerar un archivo ya transcrito

El usuario borró intencionadamente tres transcripciones ya existentes (`fuentes/260828_abc_reverte/250309_HRR_historia_europa_101.md`, `102.md`, `103.md`) y pidió transcribir de nuevo el PDF de origen (`Historia_Europa_Reverte_2503_22.pdf`) "como si las versiones eliminadas no hubieran existido nunca". El asistente había renderizado las páginas y generado un borrador `tesseract` correcto y completo en la misma tanda de trabajo, pero al redactar el Markdown final reconstruyó el cuerpo de memoria (a partir de una lectura de esas mismas páginas hecha minutos antes en la conversación) en lugar de transcribir directamente desde el borrador `tesseract` recién generado o la imagen corregida.

El resultado en `101.md` fue un párrafo gramaticalmente coherente al que le faltaba un fragmento real de aproximadamente 50 palabras (la turbulencia del reinado de Carlos I de Portugal), cosido de forma imperceptible entre dos frases que sí eran correctas. El usuario detectó el hueco por conocimiento del contenido original, no porque el texto resultante pareciera incompleto — mismo patrón de riesgo que el incidente original de este ADR (ver Context): una reconstrucción plausible pero incompleta, presentada como transcripción fiel.

La causa no fue la regla de orden de columnas ni el tratamiento de la cartela central (destacado), que se aplicaron correctamente; fue confiar en la memoria conversacional de una lectura previa en lugar de reverificar contra la fuente ya disponible en el momento de escribir. El punto 2 de este ADR prohíbe la lectura directa del PDF rotado como base de la transcripción, pero no cubría explícitamente el caso de regenerar un Markdown a partir del recuerdo de una transcripción anterior dentro de la misma sesión.

### Contexto de la ampliación (2026-09-03) — incompletitud declarada por inferencia estilística, sin evidencia textual

Al transcribir `fuentes/260828_libros_texto/transcripcion/BUP01_ciencias-sociales_23-imperialismo-colonialismo-europeo.md` (Tema 23 de un libro de B.U.P.), el asistente de IA marcó la transcripción como incompleta en su "Nota de transcripción" al llegar a la última página del PDF: el tema no cerraba con cuestionario ni resumen (elementos presentes en el Tema 22, completo, del mismo libro) y el número de páginas resultaba notablemente inferior pese a tener más apartados numerados. Ninguna de las páginas transcritas contenía una marca de continuidad explícita del tipo "Pasa a la página X" hacia una página ausente del PDF — el criterio del punto 5 no se cumplía. El usuario confirmó posteriormente, con conocimiento directo de la fuente, que el Tema 23 estaba completo: simplemente era más corto que el Tema 22, sin que eso implique nada anómalo.

Es el mismo riesgo que motiva este ADR (presentar como hecho verificado algo que no lo es) pero en dirección inversa al incidente original: en vez de completar u omitir contenido de forma imperceptible, el asistente añadió un juicio de incompletitud no respaldado por evidencia directa de la propia fuente, basado en una expectativa estilística (comparación con el tema anterior) que resultó errónea.

---

## Decision

1. **Antes de transcribir** cualquier PDF de `fuentes/` como fuente escaneada, se debe verificar si tiene capa de texto real: `pdftotext archivo.pdf -` (o equivalente). Si la salida es vacía o solo ruido, el PDF se trata como imagen pura a efectos de transcripción.

2. **Está prohibido** usar una lectura directa del PDF rotado como base de una transcripción entregada. La lectura directa del PDF solo puede usarse para triaje rápido (p. ej., decidir si el artículo de una página coincide con el nombre de archivo), nunca como fuente del texto final.

3. **Procedimiento obligatorio** para PDF sin capa de texto:
   a. Renderizar cada página a imagen de alta resolución: `pdftoppm -r 300 -png archivo.pdf salida`.
   b. Verificar visualmente la orientación de **cada página**, sin excepción y sin reutilizar por supuesto la rotación de otra página del mismo PDF, aunque se crea conocida de antemano (leyendo la imagen, no asumiendo el metadato `/Rotate`), y corregirla si es necesario con una herramienta de imagen (p. ej. Pillow `Image.rotate(±90/180, expand=True)`).
   c. Transcribir directamente desde la imagen ya corregida y legible.

4. **Toda transcripción** resultante debe incluir una sección "Nota de transcripción" que documente: formato de origen (escaneado, maquetación a N columnas, rotación aplicada), y el nivel de fidelidad alcanzado.

5. **Si el artículo continúa en una página física no incluida en el PDF** (reclamo de continuidad tipo "Pasa a la página X" hacia una página ausente del archivo), la transcripción se marca explícitamente como incompleta en la propia "Nota de transcripción" y no se completa por inferencia. No se reintenta completarla hasta disponer de una fuente PDF más completa — criterio ya aplicado en `fuentes/260818_viajero/China Viajero 0602.md`, dejado incompleto a petición explícita del usuario en espera de un nuevo escaneo del original.

6. Esta metodología aplica a toda colección presente o futura de PDF escaneados bajo `fuentes/` (incluye como mínimo `260726_revista_CBI/` y `260818_viajero/`), sustituyendo la práctica ad hoc anterior.

7. **Unidad de extracción y granularidad de entrega:**
   a. Si el nombre del PDF identifica un artículo, solo se transcribe ese artículo y sus elementos editoriales asociados, aunque el PDF contenga material adicional. El Markdown conserva el mismo nombre base del PDF.
   b. Si un PDF genérico contiene varios artículos completos, se crea un Markdown independiente por cada artículo. Los archivos usan el nombre base del PDF seguido de `__` y un *slug* descriptivo del artículo: `revista__articulo-slug.md`.
   c. No se crea un Markdown consolidado para un PDF genérico salvo petición expresa del usuario.
   d. Pies de foto, recuadros, listas y tablas se incluyen cuando pertenecen al artículo seleccionado. La publicidad, los índices y los artículos independientes quedan fuera, salvo que el usuario solicite su extracción.
   e. La identificación del artículo debe basarse en el nombre del archivo y en la evidencia editorial visible del PDF; no se debe seleccionar contenido por mera proximidad física dentro del escaneo.
   f. Antes de declarar un artículo incompleto, se debe comprobar la continuidad real entre todas las páginas incluidas en el PDF: una lectura inicial truncada, un cambio de columna o un salto de página no son evidencia suficiente de ausencia de contenido.
   g. La estructura editorial debe reflejarse explícitamente cuando exista: datos de publicación, subtítulo, cuerpo del artículo, subtítulos internos, recuadros o guías prácticas y pies de foto. Los elementos auxiliares se separan del cuerpo sin perder su pertenencia al artículo.
   h. La corrección de errores del OCR debe hacerse mediante contraste con la imagen legible. Se deben revisar especialmente nombres propios, topónimos, fechas, cifras, teléfonos, direcciones web, signos de puntuación y créditos de fotografía; no se deben corregir por intuición ni sustituir por datos externos.
   i. La salida debe reconstruir el orden de lectura editorial y no convertir cada página, columna o salto de página en una sección independiente. Los artículos contiguos del mismo escaneo se excluyen aunque compartan espacio físico con el artículo seleccionado.
   j. Las guías prácticas, fichas de servicio y pies de foto se consideran parte del artículo cuando están vinculados editorialmente a él; una pieza editorial adyacente no se incorpora por el mero hecho de aparecer en la misma página.

8. **Tesseract como generador de borrador, no como fuente de verdad (añadido 2026-08-24)** — limitado a fuentes en script latino (español/inglés) con maquetación regular (columnas simples, sin mezcla intensiva de imagen y texto): tras renderizar y corregir la rotación de cada página (punto 3), se ejecuta `tesseract pagina.png - -l spa+eng --psm 3` (u otro `--psm` apropiado a la maquetación) para obtener un borrador de texto en bruto por página. El asistente de IA parte de ese borrador para verificar y corregir contra la imagen ya corregida — en vez de transcribir cada palabra desde cero — sin relajar el nivel de fidelidad ya exigido por el punto 3.c ni las reglas de granularidad y estructura del punto 7. El borrador de Tesseract nunca se entrega tal cual: sigue prohibido, por el mismo principio del punto 2, presentar una salida automática sin verificar como transcripción final.

9. **Fuentes en script chino u otra maquetación de alta densidad multicolumna comparable quedan fuera de este apoyo, por ahora (añadido 2026-08-24)** — evidencia empírica (ver Context) mostró que Tesseract, incluso con los modelos de mayor precisión disponibles, produce sobre este tipo de fuente una salida sin estructura de frase reconocible, no utilizable ni como borrador. Para estas fuentes sigue vigente, sin cambios, la metodología de lectura multimodal directa desde la imagen corregida (puntos 1-7), sin apoyo de Tesseract. Esta exclusión se revisa solo si se valida con evidencia real un pipeline de segmentación de columnas + reconocimiento específico para maquetación densa — no se asume resuelto por adelantado.

10. **Automatización de la detección de rotación, aplica a cualquier script (añadido 2026-08-24)** — el paso 3.b puede apoyarse en `tesseract pagina.png - --psm 0` (modo OSD, cubierto por el paquete `tesseract-ocr-osd` ya instalado de fábrica) para obtener una sugerencia mecánica de rotación. No sustituye la verificación visual del punto 3.b, la acelera: la corrección final sigue confirmándose visualmente antes de transcribir.

11. **Instalación de los modelos de idioma para chino, sin privilegios de administrador (añadido 2026-08-24)** — `chi_sim` y `chi_tra` no vienen instalados de fábrica en este entorno (solo `eng`, `spa`, `osd`), y el directorio de idiomas del sistema pertenece a `root`. Se instalan en una ruta de usuario, sin `sudo`, referenciada mediante la variable de entorno `TESSDATA_PREFIX` — reproducible por cualquier sesión o asistente sin depender de privilegios de administrador:

    ```bash
    mkdir -p ~/.local/share/tessdata
    for f in chi_sim chi_tra chi_sim_vert chi_tra_vert; do
      curl -sL -o ~/.local/share/tessdata/$f.traineddata \
        "https://github.com/tesseract-ocr/tessdata_best/raw/main/$f.traineddata"
    done
    TESSDATA_PREFIX=~/.local/share/tessdata tesseract --list-langs
    ```

    Las variantes `_vert` (texto vertical) son necesarias porque Tesseract las exige al cargar `chi_sim`+`chi_tra` juntos, aunque el documento de origen use maquetación horizontal. Instalados y verificados en esta sesión (2026-08-24); **no promovidos a uso en producción** por la exclusión del punto 9 — quedan disponibles para cuando se valide un pipeline apto para maquetación densa en chino.

12. **Regenerar un Markdown ya transcrito no exime de reverificar contra la fuente (añadido 2026-08-31)** — al recrear un archivo previamente eliminado o superado dentro de la misma sesión (p. ej. a petición explícita del usuario de "transcribir como si la versión anterior no hubiera existido"), el cuerpo debe redactarse transcribiendo directamente desde el borrador `tesseract` ya generado para esa página o desde la imagen corregida, nunca de memoria de una lectura previa de esas mismas páginas hecha en la conversación, aunque esa lectura previa sea reciente y parezca fiable. La memoria conversacional está sujeta al mismo riesgo de omisión silenciosa e imperceptible que la prohibición del punto 2 sobre lectura directa del PDF rotado, y no la sustituye como fuente válida del paso de transcripción.

13. **No se infiere incompletitud por ausencia estilística (añadido 2026-09-03)** — el punto 5 exige evidencia textual explícita para marcar una transcripción como incompleta (una llamada de continuidad real hacia una página ausente del PDF, del tipo "Pasa a la página X"), o confirmación directa del usuario/de la fuente física. No es evidencia suficiente, y no debe usarse para declarar una transcripción incompleta: la ausencia de elementos de cierre esperables (cuestionario, resumen, vocabulario), un número de páginas inferior al de una pieza hermana, o cualquier otra expectativa estilística del asistente de IA sobre cómo "debería" cerrar la pieza. Ante duda genuina sin evidencia textual en ningún sentido, se pregunta al usuario en la propia respuesta en vez de afirmar unilateralmente que la pieza está completa o incompleta.

---

## Consequences

### Positive

* Elimina la clase de fallo detectada: transcripciones incompletas que se leen como completas y fidedignas.
* Evita mezclar artículos distintos o entregar contenido no solicitado cuando un PDF contiene varias piezas editoriales.
* Reduce falsos avisos de incompletitud causados por cortes de columna, saltos de página o OCR preliminar defectuoso.
* Mejora la trazabilidad de la maquetación original al distinguir el cuerpo del artículo de sus subtítulos, recuadros, guías y créditos gráficos.
* Mantiene el alcance temático del archivo cuando conviven varios artículos en una misma página escaneada.
* Procedimiento reproducible y verificable por cualquier sesión futura, sin depender de que se recuerde tácitamente.
* La "Nota de transcripción" deja trazabilidad explícita del nivel de fidelidad, alineado con el Modelo de Confianza/Evidencia del proyecto.
* (2026-08-24) Reduce el coste por artículo en fuentes de script latino y maquetación simple, al convertir el trabajo del asistente de IA de "generar todo el texto desde la imagen" a "verificar y corregir un borrador mecánico" — accesible también a asistentes de IA con menos recursos.

### Negative

* Coste por archivo significativamente mayor (render + verificación visual + rotación por página) frente a una lectura directa del PDF. Para colecciones grandes (p. ej. los 204 PDF restantes de `260818_viajero/`) esto exige planificación por lotes (paralelización vía subagentes), no ejecución archivo a archivo en el hilo principal.
* Requiere herramientas externas al propio asistente (`poppler-utils`, un entorno Python con Pillow) disponibles en el entorno de ejecución.
* (2026-08-24) La política diferenciada (punto 8 vs. punto 9) añade una decisión previa a cada transcripción (¿aplica el apoyo de Tesseract a esta fuente o no?), en vez de una regla única y mecánica. Los modelos `chi_sim`/`chi_tra` quedan instalados pero deliberadamente inactivos (punto 11) — deuda de instalación sin uso hasta que se valide un pipeline apto.

### Risks Mitigated

* Presentar como fuente fidedigna un texto parcialmente inventado/reconstruido a partir de una lectura de imagen mal orientada.
* Repetición del mismo fallo en sesiones futuras sin memoria de este incidente (mismo patrón de riesgo ya documentado para el check de índice de ADRs en `14_ADR_INDEX.md`: *"a documented check that depends entirely on being remembered has already failed silently once in this project's history"*).
* (2026-08-24) Adoptar Tesseract como apoyo general sin comprobación previa habría repetido, con una herramienta distinta, el mismo incidente que motivó este Architectural Decision Record — presentar como fiable una extracción automática no verificada para el caso (chino, maquetación densa) en el que la evidencia real muestra que no lo es.
* (2026-08-31) Repetición del incidente original con una causa distinta: confiar en la memoria conversacional de una transcripción previa, en vez de en el borrador `tesseract` o la imagen ya disponibles, al regenerar un archivo dentro de la misma sesión.
* (2026-09-03) Declarar una transcripción incompleta sin evidencia textual que lo respalde, por una expectativa estilística del asistente de IA (comparación con una pieza hermana) que resulta errónea — mismo riesgo de afirmar como hecho verificado algo que no lo es, en dirección inversa al incidente original de este ADR.

---

## References

* `fuentes/260818_viajero/` — 8 archivos regenerados bajo esta metodología (`China Macao Viajero 0806.md`, `China Pekin Viajero 0911.md`, `China Rio Li Viajero 1002.md`, `China Shanghai Viajero 0706.md`, `China Shanghai Viajero 0806.md`, `China Shanghai Viajero 1008.md`, `China Xian Viajero 0208.md`); sus versiones previas (incompletas) se conservan como `*copy.md` a modo de evidencia comparativa.
* `fuentes/260818_viajero/China Viajero 0602.md` — caso de transcripción deliberadamente incompleta por artículo truncado en el PDF de origen.
* ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md
* ADR-1103-Procedimiento_Validado_de_Integracion_de_Ramas.md
* docs/Project_Definition/04_SOURCE_MODEL.md
* docs/Project_Definition/05_EVIDENCE_CONFIDENCE.md
* (2026-08-24) `fuentes/260101_oushinet/260101_oushinet_contenido.pdf` — prueba real que descartó Tesseract como apoyo para chino/maquetación densa (punto 9).
* (2026-08-24) `fuentes/260726_revista_CBI/S45C-0i26072306440.pdf` — prueba real que validó Tesseract como borrador para español/inglés (punto 8).
* (2026-08-24) `fuentes/260824_mundo/Reportaje_Historia_11_Mundo_0011.pdf` — caso que motivó la comprobación; evaluado aparte como valoración de calidad de una entrega puntual, no como evidencia de esta ampliación.
* (2026-08-31) `fuentes/260828_abc_reverte/250309_HRR_historia_europa_101.md` — caso que motivó el punto 12: omisión de ~50 palabras al regenerar el archivo de memoria en lugar de desde el borrador `tesseract`/imagen corregida; corregido por el usuario tras detectar el hueco.
* (2026-09-03) `fuentes/260828_libros_texto/transcripcion/BUP01_ciencias-sociales_23-imperialismo-colonialismo-europeo.md` — caso que motivó el punto 13: marcado como incompleto por inferencia estilística sin evidencia textual; el usuario confirmó que la transcripción estaba completa y se corrigió la nota en el mismo cambio.
