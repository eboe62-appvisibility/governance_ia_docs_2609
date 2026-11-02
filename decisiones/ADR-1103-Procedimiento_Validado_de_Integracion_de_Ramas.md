# ADR-1103: Procedimiento Validado de Integración de Ramas (develop → master)

Status: APPROVED

Date: 2026-07-26

Scope:
System

Category:
PROCESS

Domain: (Optional)
Documentation, Research

Tags:
branch-policy, repository-structure, governance

Related ADRs:
ADR-1100, ADR-1101, ADR-1102

Supersedes:
NONE

Superseded By:
NONE

---

## Context

`ADR-1102` estableció el modelo de tres niveles (`master ← develop ← rama(s) de contenido`) y la separación transitoria entre gobernanza y conocimiento adquirido, pero dejó explícitamente sin definir dos cosas: el criterio de madurez que dispara la integración ascendente de una rama de contenido, y si la integración de `develop` en `master` es automática o deliberada.

Desde la aprobación de `ADR-1102` se ha completado, por primera vez, un ciclo completo: trabajo de contenido en `cbi_260726` → decisión de cerrar y borrar esa rama → fusión en `develop` → regeneración completa del knowledge graph sobre `develop` → decisión de proceder a integrar en `master`. Esta ADR documenta ese ciclo ya validado en la práctica, respondiendo a las preguntas que `ADR-1102` dejó abiertas.

Adicionalmente, se ha confirmado durante esta sesión que los archivos `.pdf` que acompañan a un `.md` transcrito (p. ej. `260718-UAM-01_grado-en-estudios-de-asia-y-africa-arabe.pdf`, `260726_revista_CBI/*.pdf`) ya están excluidos de git mediante la regla genérica `*.pdf` de `.gitignore` — nunca se han rastreado, diffeado ni fusionado. Esto nunca se declaró como decisión intencional sobre el modelo de fuentes, solo existía como efecto colateral de una línea de `.gitignore` sin explicación.

---

## Decision

### 1. Confirmación del modelo de ADR-1102

`develop` aloja exclusivamente gobernanza; la(s) rama(s) de contenido alojan exclusivamente conocimiento adquirido (`fuentes/`, `conocimiento/`, `vocabulary/`). Sin cambios respecto a `ADR-1102`.

### 2. Criterio de madurez para la integración rama-de-contenido → develop

El cierre de una rama de contenido (para archivarla o borrarla, típicamente al abrir una rama nueva para una nueva adquisición de conocimiento) es una señal de madurez que dispara su fusión en `develop`, pero no es un requisito (corregido 2026-08-06): el usuario puede pedir la fusión de una rama de contenido activa en `develop` en cualquier momento, sin cerrarla.

### 3. Puerta de regeneración del grafo

Tras fusionar una rama de contenido en `develop`, y **antes** de considerar la integración en `master`, se regenera el knowledge graph (`/understand`) sobre `develop`. Esto asegura que la representación estructurada canónica del conocimiento (`02_MASTER_ARCHITECTURE.md`) refleje el estado real antes de promoverlo.

Si el último escaneo completo no cubre archivos nuevos o renombrados desde entonces (como ocurrió en esta sesión), se ejecuta `/understand --full`, no incremental.

### 4. Puerta de revisión de Knowledge Assets tipo Índice (añadido 2026-08-06)

Además de la puerta de regeneración del grafo (punto 3), tras fusionar una rama de contenido en `develop` y antes de integrar en `master`, se revisa si algún Knowledge Asset marcado `Knowledge Type: Index` (p. ej. `260802-index-geography-ubicaciones-conocimiento.md`) necesita actualizarse con el contenido recién madurado. A diferencia del grafo, esta actualización no es mecánica — exige juicio editorial (deduplicar, clasificar, decidir qué incluir), como ya hizo `ADR-1108` con ese índice.

"Revisar" puede concluir en "sin cambios" (se documenta igual) o en una actualización editorial del índice afectado. Un índice puede ir por detrás de una rama de contenido todavía activa sin que sea una desviación, siempre que se revise en el momento correcto (tras fusionar, antes de subir a `master`) — mismo criterio ya aceptado para el grafo en el punto 3.

### 5. Integración develop → master: deliberada, no automática — con recordatorio activo

El merge de `develop` en `master` **nunca es automático**. Es un punto de control que el usuario decide explícitamente, en el momento que considere oportuno.

No obstante, el asistente de IA debe **recordarlo proactivamente** cuando detecte que se cumplen ambas condiciones, verificables al inicio de cualquier tarea sobre este repositorio:

1. `develop` está por delante de `master`:
   ```bash
   git rev-list master..develop --count
   ```
   (resultado > 0)

2. El knowledge graph no está obsoleto respecto a `develop` — usando el mismo procedimiento de verificación de frescura ya empleado por la skill `/understand-explain` (comparar `gitCommitHash` de `docs/arquitectura/.ua/meta.json` contra `git rev-parse develop`, con diff de rutas del proyecto vacío o commit coincidente).

Si ambas condiciones se cumplen, el asistente debe informar al usuario de que existen cambios en `develop` no integrados en `master` y el grafo ya está actualizado — preguntando si desea proceder con la integración — **sin ejecutar el merge sin confirmación explícita en cada ocasión**.

**Precisión explícita (incidente del 2026-08-02)**: "en cada ocasión" significa literalmente eso — una confirmación dada para un ciclo de integración concreto **no autoriza ciclos posteriores**, ni siquiera dentro de la misma sesión de conversación. Durante la sesión del 2026-08-01/02, el usuario confirmó explícitamente un merge `develop → master` en un momento dado; en un ciclo posterior de la misma sesión (nueva actualización del knowledge graph tras cambios adicionales), el asistente de IA ejecutó el mismo merge sin volver a pedir confirmación, asumiendo por el patrón repetido de la sesión que la autorización seguía vigente. El usuario señaló que, en adelante, es él quien decide y ordena explícitamente cada integración a `master` — el asistente de IA no debe iniciarla por inercia de sesión, por muchas veces que se haya confirmado antes el mismo patrón. El recordatorio proactivo del punto 5 sigue vigente (avisar cuando se cumplan las condiciones), pero avisar y preguntar no es lo mismo que ejecutar: la ejecución exige respuesta afirmativa expresa a esa pregunta concreta, no una extrapolación de turnos anteriores.

### 6. Recordatorio proactivo generalizado (no solo en el gate develop → master)

El punto 5 ya exige que el asistente de IA recuerde proactivamente el gate `develop → master` cuando se cumplan sus condiciones, sin ejecutar el merge sin confirmación. Esta misma exigencia se generaliza a **toda** la secuencia de esta ADR, no solo a su último paso:

* Antes de ejecutar una acción que dependa del estado de madurez de una rama de contenido (§2) o del momento de regeneración del knowledge graph (§3), el asistente de IA debe verificar si esa acción es coherente con la secuencia definida aquí.
* Si detecta una desviación — p. ej., regenerar el knowledge graph sobre una rama de contenido todavía activa, en lugar de recomendar primero su cierre y fusión en `develop` — debe **recomendar la secuencia correcta antes de ejecutar**, no limitarse a señalar la desviación después de haberla ejecutado ya.
* Señalar el problema a posteriori, aunque sea honesto, no sustituye a la recomendación previa: el coste de rehacer trabajo (o de decidir asumirlo) recae sobre el usuario si el aviso llega tarde.

**Contexto de este punto**: durante la sesión que incorporó el corpus `260726_ABC`, el asistente de IA ejecutó `/understand --full` estando en la rama de contenido `abc_260726` — todavía activa, sin haberse decidido su cierre — en lugar de recomendar primero fusionarla en `develop` conforme a §2/§3. El usuario detectó la desviación y decidió no descartar el trabajo ya realizado (asumir el coste), pero dejó constancia expresa de que el aviso debía haber llegado *antes* de ejecutar, no después.

### 7. Incidente del 2026-10-23: sincronización directa de todas las ramas, sin pasar por los puntos 2-5

Una instrucción puntual ("actualizamos todas las ramas para ponerlas en el mismo commit") llevó al asistente de IA a ejecutar `git branch -f` sobre `develop`, `gestion`, `guias`, `shi_zhilong` y `master`, apuntándolas directamente al commit vigente de la rama de contenido `mundo`. Esto fue fast-forward seguro (las cinco ramas eran ancestros estrictos de `mundo`, sin commits propios que perder), pero **se saltó por completo la secuencia de esta ADR**: no hubo merge de contenido en `develop`, no se regeneró el knowledge graph (punto 3), no se revisaron los Knowledge Assets tipo Índice (punto 4), y la integración `develop → master` no recibió la confirmación deliberada y específica que exige el punto 5 — se ejecutó como efecto colateral de sincronizar "todas las ramas", no como una decisión separada sobre `master` en concreto.

El asistente de IA señaló la desviación únicamente después de que el usuario preguntara explícitamente si se había saltado alguna norma, no antes de ejecutar el `branch -f` (incumpliendo también el punto 6 de esta misma ADR). Preguntado por las opciones, el usuario decidió explícitamente **no revertir y asumir el estado resultante** (las seis ramas en el mismo commit) como excepción puntual a esta ADR para este caso concreto, en lugar de rehacer la secuencia formal. Esta ADR no cambia su procedimiento por este incidente — se documenta la excepción, no se reescribe la regla ni el historial, mismo criterio ya aplicado en el incidente que documenta `ADR-1102 §Context`.

---

### 8. Exclusión intencional de PDFs del control de versiones

Cuando una fuente tiene par PDF+MD, el PDF es soporte visual/gráfico del `.md` transcrito — nunca la fuente autoritativa para el conocimiento del proyecto. Se excluye intencionalmente del control de versiones (regla `*.pdf` en `.gitignore`, ya existente); el `.md` es el artefacto rastreado, diffeado, fusionado y citable. Esta regla se documenta también en `04_SOURCE_MODEL.md`.

---

## Consequences

### Positive

* Cierra las dos preguntas que `ADR-1102` dejó explícitamente abiertas, con criterios extraídos de la práctica real en lugar de teorizados de antemano.
* El recordatorio activo del punto 5 evita que la integración a `master` dependa únicamente de que el usuario se acuerde de pedirlo.
* Formaliza como decisión intencional algo que ya funcionaba correctamente por accidente (exclusión de PDFs).

### Negative

* El criterio de madurez ("se cierra la rama") es retrospectivo — solo se confirma en el momento de cerrar, no permite anticipar cuánto durará una rama de contenido activa.
* El recordatorio de IA depende de que una sesión futura efectivamente ejecute la verificación del punto 5 — requiere que quede registrado en `AI_ENTRYPOINT.md` para no perderse (ver Related Artifacts).

### Risks Mitigated

* Que la integración a `master` se olvide indefinidamente por falta de recordatorio.
* Que un futuro asistente de IA reintroduzca sin querer el seguimiento de `.pdf` en git al no saber que su exclusión es intencional y no un descuido.
* Que el asistente de IA ejecute pasos de esta secuencia fuera de orden (p. ej. regenerar el grafo antes de fusionar/cerrar la rama de contenido) y solo lo señale después de haberlo hecho, trasladando al usuario un coste de "asumir o rehacer" que una recomendación previa habría evitado.
* Que el asistente de IA ejecute un merge `develop → master` asumiendo que una confirmación de un ciclo anterior de la misma sesión sigue vigente para un ciclo posterior (incidente del 2026-08-02, ver punto 5).

---

## References

* ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md
* ADR-1102-Modelo_de_Ramas_Gobernanza_vs_Contenido.md
* docs/Project_Definition/04_SOURCE_MODEL.md
* docs/Project_Definition/02_MASTER_ARCHITECTURE.md
* ai/governance/00_AI_ENTRYPOINT.md (registro de ADRs aprobados, para que el recordatorio del punto 5 se ejecute en sesiones futuras)
