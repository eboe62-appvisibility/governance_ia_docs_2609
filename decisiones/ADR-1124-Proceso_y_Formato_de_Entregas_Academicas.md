# ADR-1124: Proceso y Formato de las Entregas Académicas

Status: APPROVED

Date: 2026-10-25

Scope:
Process

Category:
PROCESS

Domain: (Optional)
Education, Documentation

Tags:
governance, standards, curriculum, learning, validation

Related ADRs:
ADR-1100 (deslinde: una entrega no es un Knowledge Asset y no sigue su convención de nombres); ADR-1101 (las fuentes de apoyo de una entrega se guardan en `entregas/fuentes/`); ADR-1103 §8 (los PDF exportados quedan fuera del control de versiones); ADR-1115 (búsqueda de respaldo externo, opcional en una entrega); ADR-1120 (verificación de denominaciones y enlaces); ADR-1123 (formato de énfasis, aplicable por analogía a las respuestas)

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Enmienda Parcial (2026-10-27)

Punto 2 (Nombre y ubicación): el patrón de nombre cambia de `UAM_<Asignatura>_Entrega_<Tema>_<aammdd>.md` a `<aammdd>_<Asignatura>_Entrega_<Tema>.md` — se antepone la fecha y se retira el prefijo literal `UAM_`, para alinear la convención con la de los Knowledge Assets (`YYMMDD-domain_principal-slug.md`, ADR-1100 punto 3) y permitir ordenar cronológicamente la carpeta `entregas/` por nombre de fichero. Aplicado, por decisión del usuario, a `asignaturas/011_17615_Ingles_1/entregas/` (ADR-1124 §9 permite revisar entregas existentes cuando el usuario lo decide); el resto de carpetas `entregas/` del repositorio conserva el patrón anterior hasta que se revisen por el mismo motivo.

---

## Context

Las carpetas `asignaturas/*/entregas/` contienen los trabajos que el estudiante entrega para su evaluación: `UAM_Ingles01_Entrega_Tiempos_261005.md` (011-17615), `UAM_Micro-Presencia-Online_Entrega_261008_Referentes.md` y sus hermanas (000_261001) y `UAM_Historia_Intro_Entrega_Kapuscinski_261024.md` (011-17617). Ninguna ADR regulaba su proceso de generación ni su formato. Hay al menos dos patrones de nombre (fecha al final o antes del tema).

La elaboración de la entrega de *El Emperador* (texto de control 4 de Introducción a la Historia, 2026-10-24/25) fijó en la práctica una serie de reglas que conviene conservar:

* La separación entre análisis y ejecución.
* Un ámbito de conocimiento declarado por el usuario.
* Citas por página de la fuente.
* Referencias al manual sin dar por impartidas las sesiones.
* Respuestas personales nunca inventadas.
* Un anexo final que distingue las afirmaciones respaldadas de las que no lo están.

Una entrega es el punto en que el conocimiento curado del repositorio se convierte en un trabajo evaluable. Sin reglas propias, ese paso puede introducir afirmaciones sin fuente que pasan desapercibidas, o mezclar en el texto entregado material que el usuario no ha autorizado.

---

## Decision

### 1. Ámbito y naturaleza

Esta ADR rige toda carpeta `asignaturas/*/entregas/`. Una entrega es un **trabajo de salida**, no un Knowledge Asset:

* No usa la plantilla `ai/templates/30_knowledge_asset.md`.
* No sigue la convención de nombres de `ADR-1100`.
* Se nutre de los Knowledge Assets, pero no los sustituye ni los modifica.

### 2. Nombre y ubicación

* Nombre: `<aammdd>_<Asignatura>_Entrega_<Tema>.md`, donde `<aammdd>` es la fecha de creación de la entrega, antepuesta (enmienda 2026-10-27, por alineación con ADR-1100 punto 3).
* Sin tildes ni espacios. Los campos se separan con `_` y las palabras dentro de un campo con `-`.
* El material de apoyo (enunciados, documentos del profesor, recortes) se guarda en `entregas/fuentes/`.
* El PDF exportado lleva el mismo nombre base que el `.md` y queda fuera del control de versiones (`ADR-1103` §8).

### 3. Estructura del documento

1. **Título** en H3: «<Tipo de prueba>: <obra o tema>».
2. **Ficha descriptiva** en H4, en tabla de dos columnas con los campos fijos `Asignatura`, `Curso`, `Estudiante`, `Fecha de entrega`, `Tipo de prueba`, `Nº de ítems` y `Fuente`. En `Fuente` se cita la obra completa, sin desglosar capítulos.
3. **Respuestas**, numeradas y enunciadas como en la tarea, con el enunciado en negrita.
4. **Preguntas complementarias** (las del propio estudiante, por ejemplo las «Questions to tackle» de una actividad), bajo un encabezado propio y continuando la numeración de la tarea.
5. **Anexos** numerados en romanos y referenciados desde las respuestas («Anexo I»), para desarrollos largos que sostienen una respuesta, como listas, decálogos o tablas.
6. **Anexo final, «Afirmaciones que necesitan respaldo»** (punto 7).

### 4. Ámbito de conocimiento por entrega

* El usuario declara al inicio de cada entrega el ámbito de conocimiento autorizado. Si no lo declara, el ámbito es la carpeta `conocimiento/` de la asignatura más sus enlaces derivados (transcripciones y fuentes que esos Knowledge Assets enlazan).
* El material de otras carpetas `conocimiento*/` del repositorio no entra en el cuerpo de la entrega salvo autorización expresa del usuario. Mientras tanto, se ofrece como fuente para ampliar, con enlace al Knowledge Asset.
* El usuario puede ampliar el ámbito a lo largo de la elaboración. La ampliación vale para esa entrega, no para las siguientes.

### 5. Reglas de cita en el cuerpo

* **Fuente principal**: por página («(p. 54)», «(pp. 122-124)»). No se cita por capítulo cuando la página es localizable.
* **Manual o guía docente**: «el manual (Sesión N)». La redacción no debe dar por impartida ni comentada en clase una sesión: el estudiante puede estar leyendo por delante del calendario.
* **Literalidad**: si una paráfrasis del usuario o del asistente difiere de la cita literal de la fuente, prevalece la literal.
* **Otras lenguas**: las citas se traducen al español y se marcan «(trad. propia)».
* **Enlaces internos** a Knowledge Assets o a rutas del repositorio: nunca en el cuerpo, porque el profesor no puede abrirlos. Solo en anexos.

### 6. Respuestas personales

* El asistente no inventa experiencias, lecturas ni opiniones del estudiante. Hasta que el usuario aporta el contenido, la respuesta lleva el marcador `[PENDIENTE EOB: <qué falta>]`, y puede acompañarse de un andamiaje basado en las fuentes.
* Cuando el usuario aporta un borrador, se integra respetando su intención y su tono. Si contiene errores de exactitud (atribuciones, fechas, citas), el asistente los corrige o los señala explícitamente, nunca en silencio.
* Las reflexiones del estudiante que afirman hechos sin fuente se redactan como reflexión («salvando las distancias», «invita a preguntarse») y se registran en el anexo de respaldo.

### 7. Anexo «Afirmaciones que necesitan respaldo»

Anexo final con tres bloques en tabla:

* **A. Afirmaciones externas sin fuente en el repositorio** (prioridad alta), con qué buscar y, si existe, un punto de partida disponible.
* **B. Afirmaciones de la fuente principal sin página en el texto** (prioridad media), con la página ya localizada cuando sea posible.
* **C. Fuentes disponibles pendientes de cotejo** (prioridad baja): transcripciones automáticas, discrepancias entre fuentes, Knowledge Assets en DRAFT.

Este anexo pertenece al documento de trabajo (`.md`) y **se excluye del PDF que se entrega al profesor**. Los anexos de contenido (punto 3.5) sí se incluyen.

La búsqueda de respaldo para el bloque A (por ejemplo, con el procedimiento de `ADR-1115`) y la creación de los Knowledge Assets que falten son **opcionales**. El usuario puede omitirlas en una entrega concreta sin que sea necesario dejar constancia de la excepción.

### 8. Proceso

1. **Análisis**, sin escribir ficheros:
   * enunciado de la tarea y modelo de formato;
   * ámbito de conocimiento;
   * cobertura de cada pregunta con las fuentes del ámbito;
   * lagunas, con propuestas de fuentes para ampliar;
   * avisos de pensamiento crítico.
2. **Aprobación** del usuario.
3. **Borrador**: se escribe el fichero.
4. **Iteraciones**: cada ronda de mejoras se propone y se aplica sobre el fichero. El commit se hace solo cuando el usuario lo pide en ese turno, y el mensaje sigue el patrón `asignaturas/<carpeta>: <qué cambia>`.
5. **Cierre**:
   * enlaces del documento verificados;
   * ningún marcador `[PENDIENTE…]` en las respuestas;
   * anexo de respaldo revisado;
   * exportación a PDF sin ese anexo.

### 9. Retroactividad

Ninguna. Las entregas anteriores a esta ADR no se renombran ni se reestructuran. La norma se aplica a las entregas nuevas y a las existentes cuando se revisen por otro motivo, si el usuario lo decide.

---

## Consequences

### Positive

* Generación reproducible y formato homogéneo de las entregas entre asignaturas.
* Trazabilidad explícita de qué partes de una entrega carecen de respaldo, sin que esa trazabilidad llegue al profesor.
* Garantía de que el asistente no atribuye al estudiante lecturas, experiencias u opiniones que no ha aportado.
* Separación clara entre el conocimiento autorizado para la entrega y el material disponible en el resto del repositorio.

### Negative

* Conviven dos patrones de nombre (punto 9) hasta que se revisen las entregas de Micro-Presencia Online.
* Mantener el anexo de respaldo y excluirlo al exportar a PDF exige un paso manual.
* La norma nace de una sola entrega elaborada con este proceso. Se espera ajustarla con la experiencia de las siguientes.

### Risks Mitigated

* Afirmaciones sin fuente que pasan inadvertidas a un trabajo evaluado.
* Respuestas personales inventadas por el asistente.
* Enlaces internos inservibles para el profesor dentro del texto entregado.
* Redacciones que dan por impartidas sesiones que el estudiante aún no ha cursado.

---

## References

* `asignaturas/011_17617_Introduccion_Historia/entregas/261024_Historia_Intro_Entrega_Kapuscinski.md`: entrega que motiva y ejemplifica esta ADR (nombre actualizado el 2026-10-27 según la enmienda del punto 2).
* `asignaturas/011_17615_Ingles_1/entregas/261005_Ingles01_Entrega_Tiempos.md`: modelo de ficha descriptiva (nombre actualizado el 2026-10-27 según la enmienda del punto 2).
* `ai/governance/05_ANALYSIS_PROTOCOL.md` y `ai/governance/20_EXECUTION_PROTOCOL.md`: separación entre análisis y ejecución.
* `ai/governance/10_EVIDENCE_MODEL.md`: niveles de evidencia.
