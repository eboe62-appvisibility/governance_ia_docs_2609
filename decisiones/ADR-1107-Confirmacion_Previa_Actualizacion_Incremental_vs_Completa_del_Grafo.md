# ADR-1107: Confirmacion Previa entre Actualizacion Incremental y Completa del Grafo de Conocimiento

Status: APPROVED

Date: 2026-08-01

Scope:
Process

Category:
PROCESS

Domain: (Optional)
Documentation, Governance

Tags:
knowledge-repository, methodology

Related ADRs:
ADR-1103, ADR-1106

Supersedes:
NONE

Superseded By:
NONE

---

## Context

El skill `/understand` (Understand Anything) decide entre una actualizacion incremental y una reconstruccion completa (`--full`) del grafo de conocimiento mediante una regla binaria simple: si ya existe un grafo y un `meta.json` previos, ejecuta incremental; solo reconstruye completo si se pasa `--full` explicitamente o no existe grafo previo. Esta regla no contempla ningun umbral relacionado con la magnitud o naturaleza del cambio — un cambio de un archivo y un cambio que borra decenas de archivos o retira un mecanismo arquitectonico completo reciben el mismo tratamiento por defecto (incremental).

Durante la retirada del mecanismo Relationship Record (ADR-1106 — 58 archivos borrados, una capa arquitectonica completa retirada), se ejecuto una actualizacion incremental sin plantear la alternativa al usuario. Esa decision, tomada unilateralmente, expuso una limitacion real y no hipotetica de la actualizacion incremental: esta se basa en un inventario de archivos (`scan-result.json`) congelado en la fecha del ultimo escaneo completo. Cualquier archivo creado despues de esa fecha resulta invisible para el proceso incremental, aunque `git diff` lo detecte correctamente como nuevo. En la practica, dos archivos nuevos (`ADR-1106` y dos ficheros de configuracion de Obsidian) quedaron fuera del analisis y tuvieron que detectarse mediante comparacion manual (`git ls-files` contra el inventario) y corregirse con lotes de analisis suplementarios despues de la fusion inicial.

El usuario senalo explicitamente que esta decision — incremental frente a completo — debio plantearse como pregunta, acompanada de una valoracion de riesgo breve, en vez de resolverse en silencio.

---

## Decision

Antes de ejecutar `/understand` tras un cambio que borra o renombra un numero significativo de archivos, o que retira/introduce un componente arquitectonico completo (una capa, una carpeta raiz del Knowledge Repository, un mecanismo de gobernanza), el asistente de IA debe preguntar explicitamente al usuario si prefiere una actualizacion incremental o una reconstruccion completa (`--full`), acompanando la pregunta de una valoracion de riesgo breve de cada opcion:

* **Incremental**: mas rapida y barata; riesgo real de omitir archivos creados despues del ultimo escaneo completo, ya que el inventario base (`scan-result.json`) no se regenera.
* **Completa (`--full`)**: mas lenta y costosa; reescanea el proyecto desde cero, sin el riesgo de inventario desactualizado.

No se define un umbral numerico exacto de "cambio significativo" — queda a criterio del asistente identificar cuando el cambio es estructural (no solo numeroso), y en caso de duda, preguntar. Un cambio de pocos archivos aislados no requiere esta confirmacion; puede procesarse incrementalmente sin preguntar, conforme al Principio de Proporcionalidad.

Este principio se aplica, mas alla de `/understand`, a cualquier flujo de trabajo que ofrezca una bifurcacion entre una via barata/incremental y una via completa/autoritativa cuando el cambio en curso sea estructuralmente significativo.

---

## Consequences

### Positive

* Evita que se repita la perdida silenciosa de nodos por inventario de escaneo desactualizado.
* Da al usuario control explicito sobre el compromiso coste/exhaustividad en operaciones de regeneracion de grafo costosas.

### Negative

* Anade un paso de confirmacion adicional en tareas que, de otro modo, se ejecutarian de forma automatica.

### Risks Mitigated

* Nodos o aristas ausentes del grafo por archivos nuevos no presentes en el inventario de escaneo incremental.
* Decisiones de coste/exhaustividad tomadas unilateralmente por el asistente sin informar al usuario del riesgo real.

---

## References

* docs/decisiones/ADR-1103-Procedimiento_Validado_de_Integracion_de_Ramas.md
* docs/decisiones/ADR-1106-Retirada_Relationship_Record.md
* ai/governance/20_EXECUTION_PROTOCOL.md
