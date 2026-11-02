# ADR-1101: Migración de Fuentes Primarias Ubicadas en asignaturas/ y Convención de Nombres en fuentes/

Status: APPROVED

Date: 2026-07-26

Scope:
System

Category:
ARCHITECTURE

Domain: (Optional)
Documentation, Research

Tags:
architecture, repository-structure, migration, sources

Related ADRs:
ADR-0033, ADR-1100

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Ampliación (2026-08-20) — Nombrado de archivos individuales en colecciones de fuentes/

El punto 3 cubre el caso de una colección de fragmentos de escaneo OCR (identificador de escáner inmutable) más un documento consolidado. No cubre el caso, distinto, de una colección multi-archivo cuyos elementos son documentos ya legibles y diferenciables entre sí (p. ej. varios reportajes de prensa transcritos), sin documento consolidado único. Este vacío se hizo evidente al racionalizar `fuentes/260818_viajero/` (9 reportajes de *El Viajero*, suplemento de *El País*, cada uno con nombre libre e inconsistente).

Se añade un cuarto patrón, para archivos individuales dentro de una colección de este tipo:

```
AAMMDD_FUENTE_region[-ciudad].{md,pdf}
```

* `AAMMDD`: fecha de **emisión/publicación original** del documento (a diferencia del `YYMMDD` del punto 2, que es la fecha de migración/curación de la carpeta-colección).
* `FUENTE`: código breve en mayúsculas de la cabecera o publicación de origen (p. ej. `VJR` = *El Viajero*).
* `region`: la unidad geográfica más específica que declara el propio documento. Si el documento no especifica nada más fino que el país, el país hace de región (p. ej. un artículo de alcance nacional sin ciudad concreta).
* `ciudad` (opcional): localidad concreta, añadida tras `region` con guion, cuando el documento la identifica.
* Un dato ausente en la fuente simplemente se omite del nombre — no se rellena por inferencia externa (principio de veracidad, `00_PROJECT_VISION_Chino_Definicion_Proyecto.md`).
* **Sin transliteración de `ñ` ni tildes**: a diferencia del slug de Knowledge Asset (ADR-1100 punto 3, acotado a `conocimiento/` por integrarse en la bóveda Obsidian), los archivos de `fuentes/` no pasan por ningún proceso de vínculo que exija portabilidad estricta de nombre de fichero — se conserva la ortografía española original (`españa`, no `espanna` ni `espana`).

Primera aplicación — `fuentes/260818_viajero/`:

| Antes | Después |
|---|---|
| `China Macao Viajero 0806.md` | `080607_VJR_china-macao.md` |
| `China Pekin Viajero 0911.md` | `091107_VJR_china-pekin.md` |
| `China Rio Li Viajero 1002.md` | `100227_VJR_china-rio-li.md` |
| `China Shanghai Viajero 0706.md` | `070609_VJR_china-shanghai.md` |
| `China Shanghai Viajero 0806.md` | `080628_VJR_china-shanghai.md` |
| `China Shanghai Viajero 1008.md` | `100828_VJR_china-shanghai.md` |
| `China Viajero 0602.md` | `060204_VJR_china.md` |
| `China Xian Viajero 0208.md` | `020810_VJR_china-xian.md` |
| `Chinos Madrid Viajero 0901.md` | `090110_VJR_españa-madrid.md` |

Cada `.pdf` gemelo se renombra igual que su `.md` (sin seguimiento en Git, conforme a ADR-1103).

Se detecta, sin retro-aplicar el cambio, que `fuentes/260726_ABC/` ya usaba de facto un patrón afín no documentado (`AAMMDD_Fuente_Tema.md`, p. ej. `131001_Viajero_Pekin.md`) — se reconoce como precedente que informó esta regla, no como pendiente de renombrado.

---

## Nota de Ampliación (2026-08-22) — Artículos gramaticales y comunidad autónoma en `region`/`ciudad`

Al racionalizar `fuentes/260819_viajero/` (lote de España, más de 90 reportajes) surgieron dos afinamientos del patrón del punto anterior, ninguno de los cuales requiere alterar su estructura `AAMMDD_FUENTE_region[-ciudad]`:

1. **Artículos y preposiciones puramente gramaticales se omiten.** `el`, `la`, `los`, `las`, `de`, `del` se eliminan del slug cuando son mero enlace gramatical sin valor distintivo — se conserva siempre la palabra con contenido semántico que los acompaña. Ejemplos: "Jerez de los Caballeros" → `jerez-caballeros`; "Jerez de la Frontera" → `jerez-frontera`; "La Rochelle" → `rochelle`. La prueba es que dos topónimos que comparten la primera palabra (los dos "Jerez" del ejemplo) siguen siendo perfectamente distinguibles tras la omisión — por la palabra final y por `region` — sin que se pierda información real.

2. **Una comarca o accidente geográfico sub-provincial poco reconocible por sí solo lleva su comunidad autónoma como envoltorio en `region`.** Provincias y comunidades autónomas ya son reconocibles sin más contexto (`burgos`, `cadiz`, `asturias`, `navarra`...) y no necesitan envoltorio. Pero una comarca (`Alt Berguedà`) o un accidente geográfico (`Guadalquivir`) que por sí solo no sitúa al lector se antepone con su comunidad autónoma: `cataluña-alt-bergueda`, `andalucia-guadalquivir`. Frontera de juicio, no criterio mecánico — aplicar con el mismo criterio editorial que ya rige el resto de `region`.

---

## Nota de Ampliación (2026-08-26) — Transcripciones de vídeo (`*_youtube/`)

Las tres colecciones `fuentes/260801_youtube/`, `fuentes/260808_youtube/` y `fuentes/260809_youtube/` usan hasta ahora el título literal del vídeo como nombre de fichero — sin fecha, sin identificar el canal, con las mayúsculas, tildes y erratas del propio título de YouTube ("Adrían Díaz" en vez de "Adrián"). Es el mismo problema, sin resolver todavía, que motivó las dos notas anteriores para colecciones de prensa. Se detecta al auditar qué Knowledge Assets derivan de `fuentes/260809_youtube/` (solo una, `conocimiento/260823-technology-economics-cxmt-debuto-bolsa-shanghai.md`, que ya documenta canal y URL del vídeo en su propia sección `Evidence and Sources` aunque el fichero fuente no lo refleje en el nombre).

Se añade un quinto patrón, para archivos individuales de transcripción de vídeo dentro de una colección `*_youtube/`, análogo al del punto anterior pero sustituyendo `region` (sin equivalente natural en la mayoría de estos vídeos) por el canal de origen:

```
AAMMDD_CANAL_tema-slug.md
```

* `AAMMDD`: fecha de **publicación real del vídeo en YouTube** — a diferencia de buena parte de la prensa histórica del punto anterior, esta fecha es siempre un metadato público de la propia plataforma; nunca se omite por desconocimiento salvo que el vídeo mismo la oculte.
* `CANAL`: código breve y reconocible del canal o autor de YouTube (mismo criterio que `VJR` para *El Viajero* — abreviatura legible elegida editorialmente, no derivación mecánica de 3 consonantes como los identificadores de bloque de `ADR-1108`, que resuelven un problema distinto).
* `tema-slug`: kebab-case con el tema real del vídeo — nunca el titular clickbait literal —, mismo criterio de la Nota de Ampliación anterior (fuera artículos/preposiciones sin valor distintivo, dentro las palabras con contenido semántico).
* Un dato no verificable (fecha o canal ambiguos) se omite del nombre, nunca se infiere — mismo principio de veracidad de las notas anteriores.

**Criterio de elección frente al patrón `YYMMDD_fuenteid_tema.md`** (usado de facto, sin ADR propio hasta ahora, por las extracciones `agent-browser` de `ADR-1115`/`ADR-1116` — p. ej. los 4 ficheros `260809_wikipedia_zh_beipanjiang.md` y análogos en `260808_youtube/`, que no son vídeos pese a la carpeta): los dos patrones no son intercambiables, aplican a dos tipos de fuente distintos.

* Usar **`AAMMDD_CANAL_tema-slug`** (este patrón) cuando la fuente es un canal, creador o programa con nombre propio de longitud variable (a menudo 2-4 palabras) **y** el documento tiene una fecha de publicación única y estable — un vídeo se publica una vez.
* Usar **`YYMMDD_fuenteid_tema.md`** cuando la fuente es una plataforma o institución con identificador ya corto y canónico por sí mismo (`wikipedia`, `douban`, `ncpssd`...) **y** no existe una fecha de publicación única real — son páginas vivas, editadas continuamente; el `YYMMDD` de ese patrón representa la fecha de extracción/curación, no de publicación, precisión que ese patrón nunca ha declarado explícitamente hasta esta nota.
* Comprimir un nombre de canal ya corto (`douban` → `DBN`) no aporta valor real; expandir un identificador institucional a un código de canal (`wikipedia_zh` → `WKZ`) tampoco. Cada patrón rinde mejor en el tipo de fuente para el que se pensó — no hay un criterio único superior entre ambos.

**Primera aplicación — `fuentes/260809_youtube/`** (2026-08-26): los 5 ficheros ya traían canal (`author`) y fecha de publicación (`published`) en su propio YAML de cabecera — capturados por la herramienta de recorte al crear el fichero, no hubo que buscarlos externamente. Renombrados con `git mv`:

| Antes | Después |
|---|---|
| `China no pide permiso y Occidente se queda atrás.md` | `260618_INA_adrian-diaz-ia-soberania-tecnologica.md` |
| `Claves para entender China. Tradiciones, cultura, economía. Adrián Díaz Marro.md` | `250822_CMR_adrian-diaz-claves-cultura-economia.md` |
| `EEUU LES BLOQUEÓ LOS CHIPS Y CHINA HA HECHO ESTO.md` | `NTV_cxmt-salida-bolsa-chips.md` |
| `TODO lo que NO te CUENTAN sobre CHINA  Adrián Díaz.md` | `250723_CPP_adrian-diaz-economia-sociedad-china.md` |
| `Viví 20 años en CHINA y esto es lo que los medios NUNCA te van a contar  Adrían Díaz.md` | `260526_UCYM_adrian-diaz-veinte-años-china.md` |

Códigos de canal asignados editorialmente a partir del campo `author` de cada fichero: `INA` (Inteligencia Artificial), `CMR` (Caminante Rojo), `NTV` (Negocios TV), `CPP` (ConPdePodcast), `UCYM` (Un Chino y Medio — no `UCM`, deliberadamente, para no colisionar con el código ya en uso en el proyecto para la Universidad Complutense de Madrid/GISEC, ver `ADR-1115`).

Un caso (`EEUU LES BLOQUEÓ...`) tenía el campo `published` vacío en su YAML; comprobado que tampoco es recuperable vía `WebFetch` (YouTube redirige a un muro de consentimiento de cookies antes de servir la página) ni vía búsqueda externa — conforme al principio de veracidad, se omite `AAMMDD` del nombre en vez de sustituirlo por la fecha de creación del fichero (`created`, que es la fecha de recorte/curación, no de publicación del vídeo — dato de naturaleza distinta, `ADR-1119`) o por cualquier otra fecha inferida.

**Segunda aplicación — `fuentes/260801_youtube/`** (2026-08-26): los 3 ficheros, todos con `published` completo en su YAML. Renombrados con `git mv`:

| Antes | Después |
|---|---|
| `HSK 1 Curso COMPLETO Con 150 Oraciones de Ejemplo - 150 Palabras BÁSICAS - aprender Chino Mandarín.md` | `220201_SMC_hsk1-150-oraciones.md` |
| `HSK 2 Curso COMPLETO Con 150 Oraciones de Ejemplo - Vocabulario Chino para Principiantes.md` | `220630_SMC_hsk2-150-oraciones.md` |
| `¿Qué cambió en el nuevo HSK 3.0 (Nivel 1 explicado en 25 min).md` | `260424_CHD_hsk30-nivel1-cambios.md` |

Códigos: `SMC` (Spanish - Mandarin Corner: iniciales de las tres palabras con contenido, el guion separador se descarta) y `CHD` (Chino con Din: se descarta "con" por gramatical, `H` de "C**h**ino" añadida para desambiguar frente al demasiado corto y genérico `CD`). En el slug del tercer fichero, "3.0" se simplificó a `30` (sin punto, poco habitual en kebab-case) — pendiente de fijar como regla general si aparecen más casos de versión con decimales.

Dos Knowledge Assets citaban estos ficheros por su nombre antiguo — actualizadas en el mismo cambio: `conocimiento/260801-language-hsk-1-cambios-v2-vs-v3.md` y `conocimiento_index/260801-index-language-hsk1-mandarin-corner-150-oraciones.md`.

**Evaluación de otras fuentes afectadas, pendiente de aplicar** (2026-08-26): 2 de los 6 ficheros de `fuentes/260808_youtube/` (`5 skills gratis de Claude Code que nadie conoce.md` y `China's Shuangbao Bridge A Record-Breaking Marvel.md`) tienen el mismo problema, con `published` igualmente vacío en su YAML — mismo caso límite que `EEUU LES BLOQUEÓ...` (`AAMMDD` se omitiría). Los otros 4 ficheros de esa carpeta no son transcripciones de vídeo — ver el criterio de elección de patrón más arriba. No renombrados todavía.

**Tercera aplicación — `fuentes/260809_youtube/`, fichero nuevo** (2026-08-27): `🇨🇳 CADENA PERPETUA AL HOMBRE MÁS RICO DE CHINA.md` (canal El Confidencial, `published` vacío en su YAML — mismo caso límite, `AAMMDD` omitido, confirmado también no recuperable vía `WebFetch` por el muro de consentimiento de YouTube) → `ELC_xu-jiayin-evergrande-cadena-perpetua.md`. Código de canal `ELC` (El Confidencial) — excepción a la regla de descartar artículos gramaticales (2026-08-22): aquí "El" es parte del nombre propio de la cabecera, no un artículo genérico modificando un sustantivo, mismo tratamiento que recibiría "El País" o "El Mundo".

---

## Nota de Ampliación (2026-08-29) — Transcripciones de libros de texto escaneados (`*_libros_texto/`)

`ADR-1117` (punto 3) exige renderizar y transcribir cualquier PDF escaneado sin capa de texto desde imagen corregida, y su punto 7 fija cómo nombrar el `.md` resultante en función del nombre del PDF de origen: si el nombre del PDF ya identifica el artículo, el `.md` lo conserva tal cual (7a); si el PDF es genérico y contiene varios artículos, cada `.md` usa `nombrebase__articulo-slug.md` (7b). Ninguno de los dos cubre el caso real que motiva esta nota: un PDF de nombre puramente generado por el escáner (`Escaneado_20260828-1104.pdf`, sin ningún dato editorial) que además es una obra unitaria (una sola pieza, no varios artículos) — `fuentes/260828_libros_texto/transcripcion/`, primera colección de extractos de libros de texto escaneados del repositorio.

Aplicar 7b tal cual (`Escaneado_20260828-1104__ciencias-sociales-...md`) ata el nombre del `.md` a un identificador de escáner sin ningún valor mnemotécnico, distinto en cada PDF aunque dos extractos vengan del mismo libro. Se opta en su lugar por extender aquí el mismo criterio de código-de-fuente ya usado en los patrones `AAMMDD_FUENTE_region` (prensa) y `AAMMDD_CANAL_tema-slug` (vídeo):

```
FUENTE_slug.md
```

* `FUENTE`: código breve y reconocible del libro/serie de origen, elegido editorialmente (mismo criterio que `VJR`, `SMC`, `ELC`...) — no una derivación mecánica. Debe ser lo bastante específico para distinguir obras distintas si en el futuro conviven varios libros de la misma colección/curso en la misma carpeta (p. ej. dos libros distintos de 8º de EGB no deberían compartir código).
* `slug`: kebab-case con el tema real del extracto transcrito (mismo criterio de las notas anteriores: fuera artículos/preposiciones sin valor distintivo).
* **Sin `AAMMDD`**, a diferencia de los patrones de prensa y vídeo: un libro de texto no tiene, por extracto, una fecha de publicación propia y distinta — su fecha editorial (año de la obra) ya queda documentada en la sección "Datos editoriales y legales" del propio `.md` (colofón, depósito legal), y la fecha de migración/curación ya la lleva la carpeta contenedora (`YYMMDD_slug/`, punto 2). Anteponer una fecha aquí sería redundante, no informativo.
* El PDF de origen **no se renombra** (a diferencia del patrón de prensa, donde el `.pdf` gemelo sí se renombra igual que su `.md`): conforme al punto 3 de esta ADR, el identificador de escáner del PDF es evidencia inmutable del lote físico, y esta carpeta reúne extractos de libros potencialmente distintos, no artículos de una misma cabecera. La trazabilidad entre el `.md` y su PDF de origen queda documentada en la sección "Nota de transcripción" del propio `.md`, no en el nombre de fichero.

**Primera aplicación — `fuentes/260828_libros_texto/transcripcion/`** (2026-08-29): `Escaneado_20260828-1104.pdf` (PDF sin renombrar) → `EGB08_ciencias-sociales-imperialismos-extremo-oriente.md`. Código `EGB08`: siglas reales de la colección tal como aparecen impresas en portada y colofón del libro (*Educación General Básica*) + curso (8.º).

---

## Nota de Ampliación (2026-08-31) — Reseñas de lectura personal (`fuentes/260830_bibliografia_propia/{md,docx}/`)

`fuentes/260830_bibliografia_propia/` reúne la bibliografía personal leída por el usuario, con dos carpetas gemelas de reseñas (`md/` y `docx/`, mismo nombre base salvo extensión) y una tabla índice `LitMusVi.md` que, tras la sesión del 2026-08-30/31, se estableció como SSOT de la fecha de lectura de cada libro (renombrado de 36 pares de archivos para hacerla coincidir, commit `8fd1967`). Los nombres de archivo resultantes de ese renombrado seguían el patrón de facto ya usado en la carpeta (`AAMMDD_Autor_Titulo Libre.md`), heredado sin decisión explícita — mismo vacío de gobernanza que motivó las tres notas anteriores para prensa, vídeo y libros de texto.

Este caso no encaja con el patrón `FUENTE_slug` sin fecha de la nota anterior (libros de texto): allí se omite `AAMMDD` porque un extracto de libro de texto no tiene fecha de publicación propia y distinta. Aquí sí la hay — es la fecha de lectura personal, un dato real, verificable por archivo y ya fijado como SSOT en `LitMusVi.md` —, exactamente el mismo tipo de dato que motiva `AAMMDD` en los patrones de prensa y vídeo. Se añade un sexto patrón, construido sobre esos dos:

```
AAMMDD_AUTOR_titulo-slug.{md,docx}
```

* `AAMMDD`: fecha de lectura registrada en `LitMusVi.md` (SSOT) cuando la fila existe y la tiene. Si la fila existe pero no tiene fecha (caso único a esta fecha: Maquiavelo, *El Príncipe*), o si el archivo no corresponde a ninguna fila de `LitMusVi.md`, se conserva el `AAMMDD` ya presente en el nombre de archivo sin verificar — no se infiere ni se inventa una fecha nueva (principio de veracidad).
* `AUTOR`: apellido principal del autor, en mayúsculas. A diferencia de `FUENTE`/`CANAL` en las notas anteriores (siglas artificiales que comprimen un nombre institucional largo), un apellido ya es corto y reconocible por sí mismo — no se aplica compresión a siglas, solo mayusculización. Nombre compuesto o muy largo: se elige editorialmente el término más distintivo (mismo criterio que el resto de esta ADR).
* `titulo-slug`: kebab-case, minúsculas, sin artículos/preposiciones puramente gramaticales sin valor distintivo (mismo criterio de la nota 2026-08-22). Un archivo que agrupa más de una fila de `LitMusVi.md` porque la obra reseñada las cubre a todas (p. ej. *Tragedias completas* de Esquilo = "Prometeo encadenado" + "Los persas"; *Martín Fierro* de Hernández = ambas partes) usa el título de la obra agrupadora, no el de una fila individual.
* **Excepción explícita al criterio general de `fuentes/` sobre tildes**: a diferencia de las cinco notas anteriores de esta ADR (que preservan la ortografía española sin transliterar), aquí **se eliminan las tildes** (`á/é/í/ó/ú` → `a/e/i/o/u`) en los tres segmentos, por decisión expresa del usuario al aplicar este patrón (2026-08-31): los nombres de archivo con tilde generan problemas reales de búsqueda/indexado en las herramientas que maneja. **La `ñ` no se transcribe** — no es una tilde sino una letra distinta del alfabeto español, y el usuario confirmó explícitamente mantenerla tal cual (p. ej. `BLASCO-IBAÑEZ_cañas-barro`, `BERCEO_..._señora`, `HOVASSE_..._niños`) — mismo criterio de no transliterar `ñ` que el resto de `fuentes/`, la excepción se acota solo a la tilde.
* Ambos gemelos (`.md` y `.docx`) se renombran igual.
* **Los archivos renombrados se trasladan a `md_new/`/`docx_new/`** (subcarpetas nuevas, hermanas de `md/`/`docx/`), no se sustituyen in situ: permite distinguir de un vistazo qué reseñas ya siguen el patrón nuevo y cuáles quedan pendientes o fuera de alcance, sin necesidad de auditar 72 nombres de archivo cada vez. Consolidar `md_new/`→`md/` (retirando la carpeta antigua) queda como decisión futura, una vez agotado el contenido pendiente de `md/`/`docx/`.

**Riesgo de colisión identificado, no aplicado todavía**: `LitMusVi.md` registra tanto a César Vidal como a Gore Vidal; hoy solo el segundo tiene reseña (`VIDAL_...`), así que no hay colisión real, pero la habría si se añadiera una reseña del primero. Se resuelve entonces, no antes, añadiendo inicial o nombre de pila (`VIDAL-G`/`VIDAL-C` o equivalente) — frontera de juicio, no criterio mecánico, mismo tratamiento que el resto de esta ADR da a la desambiguación de `region`.

**Alcance real de la primera aplicación (2026-08-31)**: de los 72 archivos en `md/`+`docx/`, 8 quedan **fuera de este patrón y sin mover**, no por falta de dato sino por no ser reseñas de lectura:

* 5 son reseñas de otro medio, con su propia cabecera de campos (`Director`/`Película`, `Reportaje`, conferencia con `Título`, exposición sin autor personal): `010715_Mandoki_nacida_inocente` (película), `030728_Dale Caminando entre cavernicolas` (documental TV), `031224_Budi como vender mas` (conferencia), `060610_CaixaForum...` (exposición, sin autor personal), `160715_Marteles reforma fiscal en españa` (documental de radio).
* 3 usan la misma plantilla `CITAS LITERARIAS` pero sin editorial ni obra real detrás — notas personales o conversaciones transcritas, no reseñas de un libro: `060729_Choni` (Autor: Desconocido, Libro vacío — una cita oral), `060822_Connie Ries vacas` y `090906_Martin_feria artesanía Gargantilla` (ambos titulados "Comentarios durante..." — apuntes de una visita/feria, no una obra publicada).

Los 64 restantes sí son reseñas de libro genuinas (`Autor`+`Libro` reales) y se renombran: 49 corresponden a una fila de `LitMusVi.md` con fecha SSOT; 1 (Maquiavelo, *El Príncipe*) corresponde a una fila de `LitMusVi.md` sin fecha registrada, así que conserva su `AAMMDD` de archivo existente en vez de quedar excluida (tiene datos de autor/obra reales, el único dato ausente es la fecha); 14 no están en `LitMusVi.md` en absoluto (reseñas de libro fuera de esa tabla índice) y también conservan su `AAMMDD` de archivo existente sin verificar contra ninguna SSOT.

---

## Nota de Ampliación (2026-10-17) — Enlace dual PDF+MD en Knowledge Assets, con independencia del layout de carpetas

Dos casos ya existentes resolvían el mismo problema de fondo —un Knowledge Asset cita una transcripción `.md` cuyo PDF de origen también existe y es localizable— con tratamiento distinto. `asignaturas/011_17617_Introduccion_Historia/conocimiento/261006-*.md` ya enlazaba ambos ("Fuente principal: `[X.pdf](...)` (transcripción en `[X.md](...)`)."), con el PDF y el `.md` como gemelos en la misma carpeta. Varias colecciones de `fuentes/` solo enlazaban la transcripción.

**Decisión**: toda referencia en `Evidence and Sources` a una transcripción `.md` enlaza también su PDF de origen, siempre que exista y pueda identificarse cuál es. La vía habitual de identificación es que el nombre del PDF sea derivable de forma estable a partir del nombre del `.md` (la derivación la rige `ADR-1117` punto 7), pero **no es la única**: basta cualquier identificación fiable ya disponible, incluida la que el propio `.md` declara en su "Nota de transcripción" o en su cita de fuente (caso típico de un PDF con nombre generado por el escáner, p. ej. `S45C-....pdf`, que no es derivable del nombre del `.md` pero sí está documentado dentro de él). Un nombre de PDF no mnemónico no es, por sí mismo, motivo para omitir el enlace — se enlaza con el nombre que el PDF realmente tiene, derivable o no. La regla es independiente de dónde viva cada fichero: aplica igual si PDF y `.md` son gemelos en la misma carpeta (caso Introducción a la Historia) que si el PDF vive en la raíz de la colección y el `.md` en una subcarpeta separada (caso frecuente en varias colecciones de `fuentes/`, con la subcarpeta de transcripciones nombrada `transcripcion/`). Esta ADR no fija un único layout de carpeta como obligatorio — solo la obligación de enlazar ambos ficheros cuando los dos existen y el PDF es identificable.

El único caso que queda genuinamente fuera es aquel en que ni el nombre ni ninguna documentación existente permiten saber **cuál**, de varios PDF de una misma colección, es el que corresponde a una cita concreta — no porque el nombre no sea mnemónico, sino porque la correspondencia en sí no está establecida (p. ej. una colección de varias páginas escaneadas sueltas volcadas en un único `.md` consolidado, sin mapeo página-a-cita documentado todavía). Ese vacío de mapeo se resuelve estableciendo la correspondencia cuando el coste de hacerlo es razonable; cuando no lo es (p. ej. exigiría renderizar y leer visualmente, página a página, un PDF escaneado sin capa de texto solo para enlazar una cita puntual — decisión expresa del usuario, 2026-10-24), se enlaza en su lugar la carpeta completa que contiene el conjunto de PDF candidatos, dejando constancia explícita en la propia cita de que la correspondencia página-a-cita no está establecida. No es una exención del enlace — sigue habiendo un enlace real y localizable —, es un nivel de precisión distinto al de un PDF identificado de forma unívoca.

Aplicado el 2026-10-17 a 738 referencias (701 en `conocimiento_west_mundo/`, 37 en `conocimiento/`), insertando `(PDF: [nombre.pdf](ruta))` junto al enlace de transcripción ya existente, sin alterar el resto de la cita; detalle de las colecciones de `fuentes/` afectadas y de qué se movió en cada una, en el historial de commits de esa fecha.

**Corrección**: el punto 4 de la `Decision` original de esta ADR quedó alterado de forma no intencionada durante el barrido automático de enlaces de esta misma reorganización (la ruta textual que describía coincidía con el patrón de sustitución). Se ha restituido su redacción original — la `Decision` de un ADR documenta lo decidido en su fecha, no se reescribe para seguir la ubicación física actual; ese seguimiento corresponde a notas de ampliación como ésta.

**Corrección 2026-10-24**: un barrido posterior (a petición del usuario, tras cuantificar que 5 colecciones de `fuentes/` —`260101_oushinet/`, `260718_UAM/`, `260726_revista_CBI/`, `260101_calendario/`, `260828_libros_texto/`— nunca habían recibido este enlace) interpretó la frase "nombre derivable de forma estable" como una exención de las dos colecciones con PDF de nombre generado por escáner (`260726_revista_CBI/`, `260101_calendario/`), dejándolas sin corregir. El usuario corrigió ese criterio: no es la exención que esta ADR pretendía fijar. Se reescribe el párrafo de la `Decisión` (más arriba) para dejarlo explícito, y se aplica en consecuencia: `260101_calendario/` ya tiene su único PDF enlazado (su nombre de escáner ya estaba documentado en la propia "Nota de transcripción" del `.md`, solo faltaba convertirlo en enlace); `260726_revista_CBI/` (3 citas, un único `.md` consolidado de ~20 páginas escaneadas sueltas sin mapeo página-a-cita todavía establecido) queda pendiente de que se determine esa correspondencia, no exenta de la norma.

---

## Context

`AI_ENTRYPOINT.md` documenta el contrato de `asignaturas/` explícitamente: *"asignaturas/ (folders 011 to 042, one per UAM subject)"*. La carpeta `asignaturas/000-00000_miscelanea/` queda fuera de ese rango numérico — nunca fue una asignatura real, sino un cajón misceláneo que aloja material de origen externo (la revista China Book International, en `Revista_CBI_2606/`: 19 fragmentos OCR con nombre de escaneo más una versión consolidada), no estructura curricular.

`fuentes/` ya cumple, de facto, el rol de repositorio de material de referencia externo/primario (`04_SOURCE_MODEL.md`), y contiene tres patrones de nombrado no documentados pero consistentes:

* `NN_slug-kebab.{md,pdf}` — referencias institucionales oficiales numeradas.
* `YYMMDD_Tema.txt` — notas/compilaciones personales fechadas.
* `autor_tema_YYMMDD.md` — obras externas de autoría identificada.

Ninguno de los tres cubre el caso de una colección multi-archivo (una revista completa, con decenas de fragmentos escaneados). Este vacío se hizo evidente al crear Knowledge Assets (ADR-1100) que citan como fuente contenido ubicado en `asignaturas/000-00000_miscelanea/`, fuera del contrato documentado de esa carpeta.

---

## Decision

1. El contenido cuya naturaleza es material de fuente externa/primaria (según las categorías de `04_SOURCE_MODEL.md`) reside en `fuentes/`, con independencia de a qué asignatura o bloque temático sirva pedagógicamente. `asignaturas/` queda reservada exclusivamente para el árbol curricular oficial (carpetas 011-042, una por asignatura UAM), conforme a `AI_ENTRYPOINT.md`.

2. Las colecciones de fuente multi-archivo (p. ej. un número completo de revista escaneada) se organizan como subcarpeta de `fuentes/`, nombrada:

   ```
   YYMMDD_slug
   ```

   donde `YYMMDD` es la fecha de migración/curación de la colección al repositorio (no necesariamente la fecha de publicación original, que puede no conocerse con precisión de día) y `slug` es una etiqueta descriptiva breve.

3. Dentro de una colección así, el documento consolidado/curado (cuando existe) se renombra para coincidir con el nombre de su carpeta contenedora (`YYMMDD_slug.md`), de forma que sea el punto de entrada evidente. Los artefactos de escaneo individuales (fragmentos OCR con identificador generado por el escáner) **no se renombran**: su identificador original se preserva como evidencia inmutable del lote de escaneo físico, conforme al principio de trazabilidad de `04_SOURCE_MODEL.md`.

4. Como primera aplicación de esta regla: `asignaturas/000-00000_miscelanea/Revista_CBI_2606/` se migra a `fuentes/260726_revista_CBI/`, y su documento consolidado se renombra a `fuentes/260726_revista_CBI/260726_revista_CBI.md`. La carpeta `asignaturas/000-00000_miscelanea/`, ya vacía, se elimina. (Nota: esta ruta describe la ubicación decidida en esa fecha; su ubicación actual, tras la reorganización de 2026-10-17, se documenta en la Nota de Ampliación correspondiente más arriba — el texto de esta Decision no se reescribe retroactivamente.)

5. **Separación por rama**: esta ADR es un artefacto de gobernanza y reside en la rama `develop`. Su ejecución material (el `git mv` descrito en el punto 4, y la actualización consecuente de los Knowledge Assets de ADR-1100) se realiza en la rama que contiene el conocimiento adquirido correspondiente (`cbi_260726` a fecha de este ADR), conforme al criterio explícito del usuario: *gobernanza en `develop`, conocimiento adquirido en su rama propia*. `develop` conserva la regla; la rama de contenido aplica el cambio físico.

---

## Consequences

### Positive

* `asignaturas/` vuelve a cumplir exactamente su contrato documentado (011-042, una por asignatura).
* `fuentes/` gana un cuarto patrón de nombrado explícito y documentado, coherente con los tres ya existentes de facto.
* Corrige la inconsistencia en su origen en lugar de documentarla como excepción permanente.

### Negative

* Rompe las rutas relativas ya citadas en los 3 Knowledge Assets y 3 Relationship Records de ADR-1100 — requiere actualización coordinada en la rama de contenido.
* Invalida el `knowledge-graph.json` actual (`layer:asignaturas`, paso 9 del tour, y el `filePath` del nodo del documento afectado) hasta una nueva ejecución de `/understand`.
* Los 19 fragmentos OCR individuales permanecen con nombre de escaneo no legible — aceptado como deuda temporal, ya que según el usuario probablemente se eliminarán.

### Risks Mitigated

* Perpetuar un contrato de repositorio documentado pero incumplido.
* Ambigüedad sobre dónde ubicar futuro material de fuente no curricular (p. ej. próximos números de revista).

---

## References

* ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* docs/Project_Definition/04_SOURCE_MODEL.md
* AI_ENTRYPOINT.md
* docs/Project_Definition/02_MASTER_ARCHITECTURE.md
