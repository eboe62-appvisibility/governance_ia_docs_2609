# ADR-1106: Retirada del Mecanismo Relationship Record y Migración de su Información a los Knowledge Assets

Status: APPROVED

Date: 2026-07-30

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Research, Documentation

Tags:
knowledge-asset, knowledge-repository, classification, relationship-record

Related ADRs:
ADR-1100, ADR-1101, ADR-1104, ADR-1105, ADR-0033

Supersedes:
NONE (este documento no reemplaza integramente ningun ADR anterior; enmienda parcialmente ADR-1100 y ADR-1105, ver seccion Decision)

Superseded By:
NONE

---

## Nota sobre el caracter de este documento

Este es un Architectural Decision Record provisional, de caracter operativo: su unico proposito es planificar y dejar trazabilidad de un trabajo de limpieza concreto (la retirada del mecanismo Relationship Record). No establece una regla de arquitectura de conocimiento permanente destinada a guiar decisiones futuras del mismo tipo — al contrario, documenta la eliminacion de una regla previa (Architectural Decision Record 1100, punto 4).

Se ha considerado usar en su lugar la plantilla `ai/templates/10_repository_event.md` (Repository Event), pero esa plantilla esta explicitamente reservada a documentar cambios ya completados ("Repository events should accurately describe implemented changes"; prohibe expresamente contenido de "intentions" y "future work"). Como este documento existe precisamente para planificar trabajo pendiente antes de ejecutarlo, el formato Architectural Decision Record es el que corresponde durante la fase de planificacion. Una vez completado el trabajo, el commit final de la rama sí debera redactarse siguiendo la plantilla Repository Event.

Sobre su destino final: el modelo de Architectural Decision Record de este proyecto (`docs/decisiones/04_adr_template.md`) solo contempla los estados PROPOSED, APPROVED, SUPERSEDED y DEPRECATED — no existe un estado de "eliminado", y `ai/governance/00_AI_ENTRYPOINT.md` establece que los registros historicos deben preservarse como prueba de trazabilidad, no sustituirse ni borrarse silenciosamente. Se recomienda que, al finalizar el trabajo aqui planificado, este documento se marque `Status: DEPRECATED` en lugar de eliminarse fisicamente del repositorio, precisamente para conservar el porque de la decision. La destruccion fisica del archivo queda como alternativa disponible si el usuario lo decide explicitamente, pero no es la recomendacion por defecto.

---

## Context

El componente "Knowledge Repository" (`docs/Project_Definition/02_MASTER_ARCHITECTURE.md`, componente 4) se materializo mediante Architectural Decision Record 1100 con dos piezas: la carpeta `conocimiento/` como ubicacion fisica con una convencion de nombres de archivo, y un artefacto adicional llamado Relationship Record (`ai/templates/31_relationship_record.md`), alojado en `conocimiento/relaciones/` con el mismo nombre base que su Knowledge Asset emparejado, pensado para registrar la relacion `derived_from` entre cada Knowledge Asset y su fuente.

Una revision del estado actual del repositorio (57 Knowledge Assets, 57 Relationship Records) muestra lo siguiente, verificado directamente sobre los archivos:

* El 100% de los Relationship Records existentes (57 de 57) usa el tipo de relacion `derived_from` apuntando exclusivamente a un documento fuente (`document:fuentes/...`). No existe en el repositorio ninguna relacion registrada entre dos Knowledge Assets.
* En todos los Knowledge Assets revisados, el campo `Related Knowledge Assets` permanece vacio ("ninguno registrado").
* La informacion de procedencia que cada Relationship Record documenta (`Evidence and Sources`: archivo fuente y lineas citadas) ya esta duplicada, casi literalmente, dentro de la seccion `Evidence and Sources` del propio Knowledge Asset.
* Excepcion detectada: en al menos un caso verificado (`conocimiento/relaciones/260726-travel-wang-jue-archaeological-adventures.md`), la seccion `Scope` del Relationship Record contiene una precision que no existe en el Knowledge Asset emparejado (la exclusion explicita de los fragmentos OCR individuales de `fuentes/260726_revista_CBI/`, ya cubiertos por otras aristas del grafo). Esto demuestra que, aunque la duplicacion sea la norma, no toda la informacion de los Relationship Records es redundante — algunos contienen matices de alcance (`Scope`) o notas (`Notes`) ausentes en su Knowledge Asset emparejado.

El proposito original que motivo la creacion del Relationship Record (Architectural Decision Record 1100, seccion Context) era evitar acoplar el nombre de archivo del Knowledge Asset a un identificador externo mutable (un codigo de escaneo OCR). Ese problema quedo resuelto por la convencion de nombres (`YYMMDD-domain_principal-slug.md`, Architectural Decision Record 1100 punto 3), que es independiente de si existe o no un archivo de Relationship Record separado. El requisito adicional de mantener un archivo de Relationship Record dedicado para la procedencia (Architectural Decision Record 1100 punto 4) no era estrictamente necesario para resolver ese problema, y en la practica ha derivado en duplicacion sistematica sin aportar, hasta la fecha, ninguna relacion entre Knowledge Assets.

Adicionalmente, se ha comprobado que Obsidian (la herramienta de visualizacion introducida durante esta misma rama) no tiene capacidad de descubrir relaciones semanticas entre documentos por si sola: solo visualiza enlaces explicitos ya escritos (`[[wikilink]]` o enlace markdown). El mecanismo Relationship Record, tal como se ha usado hasta ahora, no resuelve el problema de "que Knowledge Assets se relacionan entre si" — ese problema sigue sin resolver independientemente de si se conserva o se retira el Relationship Record.

---

## Decision

Se retira el mecanismo Relationship Record como artefacto de archivo separado. La informacion de procedencia (`derived_from` hacia un documento fuente) pasa a vivir exclusivamente dentro de la seccion `Evidence and Sources` de cada Knowledge Asset, que ya la documenta hoy. Las relaciones futuras entre Knowledge Assets (cuando se establezcan) se documentaran mediante wikilinks de Obsidian y, si se demuestra necesario tras probar herramientas de descubrimiento semantico (por ejemplo, el skill `understand-anything:understand-knowledge`), mediante un mecanismo a definir en un Architectural Decision Record posterior — no se reintroduce el Relationship Record para ese proposito sin una decision explicita nueva.

El trabajo se ejecuta en los siguientes pasos:

### Paso 1 — Revertir los wikilinks piloto no comiteados

Descartar, mediante `git restore` (o equivalente), los cambios no comiteados en los 3 pares de Knowledge Asset y Relationship Record usados como piloto de prueba de Obsidian (`260726-travel-wang-jue-archaeological-adventures`, `260726-politics-zheng-yongnian-dialogues`, `260726-economics-15th-five-year-plan`). Los wikilinks se volveran a crear, si procede, una vez completada la limpieza, apuntando unicamente a otros Knowledge Assets — nunca a Relationship Records, que dejaran de existir.

Nota operativa adicional (no parte del mecanismo Relationship Record, pero detectada durante la verificacion de este mismo paso): el archivo `.obsidian/graph.json`, generado automaticamente por Obsidian, aparece como no rastreado en el control de versiones. Se recomienda anadirlo a `.gitignore` junto con `.obsidian/app.json` y `.obsidian/workspace.json`, por ser tambien estado de sesion local sin valor compartible.

### Paso 2 — Documentar la retirada en este Architectural Decision Record

Este mismo documento.

### Paso 3 — Separacion de rama respetando Architectural Decision Record 1102

Decision final (revisa la version inicial de este paso, que habia aceptado mezclar gobernanza y conocimiento en la rama de trabajo): se mantiene el principio de Architectural Decision Record 1102 (Modelo de Ramas: Gobernanza vs. Contenido) sin excepciones.

* Este mismo Architectural Decision Record (documento de gobernanza, `docs/decisiones/`) se confirma en la rama `develop`, no en la rama de trabajo.
* Los cambios de conocimiento (migracion de informacion hacia los Knowledge Assets, eliminacion de los Relationship Records en `conocimiento/relaciones/`) se ejecutan integramente en la rama de trabajo, que permanece como rama de contenido hasta su integracion futura en `develop` siguiendo el procedimiento validado de Architectural Decision Record 1103.
* Los cambios de gobernanza que sí deben tocar archivos vivos citados en el Paso 6 (`ai/governance/00_AI_ENTRYPOINT.md`, `ai/skills/documentation/elaborar_knowledge_assets.md`, `ai/templates/31_relationship_record.md`, Architectural Decision Record 1100, Architectural Decision Record 1105) se ejecutan tambien en `develop`, no en la rama de trabajo, precisamente porque son documentos de gobernanza.

### Paso 4 — Migrar a los Knowledge Assets la informacion que solo existe en su Relationship Record

Antes de eliminar cada Relationship Record, revisar si contiene informacion no presente en su Knowledge Asset emparejado — no solo en la seccion `Evidence and Sources` (la mas frecuentemente duplicada), sino tambien en `Scope`, `Description` y `Notes`, donde se ha verificado que pueden existir precisiones ausentes en el Knowledge Asset (ver ejemplo de `wang-jue` en la seccion Context). La informacion identificada como no redundante se incorpora al Knowledge Asset correspondiente antes de borrar el Relationship Record.

### Paso 5 — Verificar y hacer commit en la rama de trabajo

Confirmar que ningun Knowledge Asset ha perdido informacion, que no quedan referencias rotas, y hacer commit siguiendo la plantilla `ai/templates/10_repository_event.md` para el mensaje de commit.

### Paso 6 — Eliminar las referencias al mecanismo Relationship Record en la documentacion viva; preservar el registro historico

No todas las menciones detectadas tienen el mismo tratamiento. Inventario verificado por busqueda directa sobre el repositorio:

**Documentos operativos que se actualizan de verdad** (describen el mecanismo como regla vigente o procedimiento reutilizable; dejarlos sin actualizar induciria a error en trabajo futuro):

* `ai/governance/00_AI_ENTRYPOINT.md` — eliminar la referencia a `ai/templates/31_relationship_record.md` en la lista de plantillas, y actualizar la regla de Architectural Decision Record 1100 citada (linea que dice "Knowledge Assets and Relationship Records follow the naming convention") para reflejar que la convencion aplica solo a Knowledge Assets.
* `ai/skills/documentation/elaborar_knowledge_assets.md` — este skill se usara en corpus futuros; debe dejar de instruir la creacion de Relationship Records emparejados.
* `ai/templates/31_relationship_record.md` — eliminar el archivo. Es una plantilla vacia sin contenido de decision ni valor historico propio.
* `docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md` — **enmienda parcial, no eliminacion**: los puntos 1 y 3 (ubicacion de `conocimiento/` y convencion de nombres) permanecen vigentes y en uso activo por los 57 Knowledge Assets existentes. Unicamente el punto 2 (subcarpeta `conocimiento/relaciones/`) y el punto 4 (obligacion de referenciar fuentes exclusivamente via Relationship Record) quedan revocados por este Architectural Decision Record. Se anadira una nota en Architectural Decision Record 1100 senalando la enmienda y remitiendo a este documento.
* `docs/decisiones/ADR-1105-Procedimiento_Elaboracion_Knowledge_Assets.md` — es un procedimiento operativo vivo, no solo narracion historica (se reutilizara en corpus futuros). Debe actualizarse el paso de "Emparejamiento" para eliminar la creacion de Relationship Records.

**Documentos historicos que no se modifican** (narran una decision o un evento ya ejecutado en su momento; reescribirlos falsificaria el registro historico del proyecto, en contra del principio de `ai/governance/00_AI_ENTRYPOINT.md` sobre preservacion de "ledgers" historicos):

* `docs/decisiones/ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md` — menciona que la migracion rompio rutas relativas en Relationship Records existentes en su momento. Es un hecho historico verificable, no una regla vigente.
* `docs/decisiones/ADR-1104-Ampliacion_Domain_Tags_Corpus_ABC.md` — documenta un cambio ya ejecutado sobre las plantillas cuando el Relationship Record todavia existia.

**Menciones cosmeticas, de baja prioridad, opcionales**:

* `docs/decisiones/ADR-0033-Taxonomia_ADRs.md` — mencion ilustrativa de "Relationship Record" como ejemplo de entidad clasificable por `Domain`. No condiciona ninguna regla del documento.
* `docs/Project_Definition/03_ONTOLOGY.md` — mencion ilustrativa equivalente; ademas, esta mencion es en si misma cuestionable independientemente de este Architectural Decision Record, ya que el documento declara explicitamente que "it does NOT describe... repository structure", y "Relationship Record" es un artefacto de estructura de repositorio, no un concepto ontologico.

**Hallazgo pendiente de decision aparte (2026-07-30), ya resuelto**: un ledger histórico de estado del proyecto, mantenido fuera de la serie numerada de `Project_Definition/`, fue eliminado en un commit de esta misma rama y posteriormente recreado, con contenido distinto y mas extenso, en un commit posterior (`ea6e748`) previo al inicio de esta conversacion sobre Obsidian. Su tratamiento (actualizar, marcar como historico, o archivar) quedo fuera del alcance de este Architectural Decision Record en su momento.

**Resolucion (2026-08-08)**: las decisiones de herramienta que ese ledger documentaba de forma informal (Understand-Anything, Obsidian, agent-browser) se extrajeron a `ADR-1109`, `ADR-1114` y `ADR-1115`, cada una con su propio anexo de fuentes. Con ese contenido ya migrado, el fichero se retira del repositorio — no se mantiene como historico ni se actualiza, dado que su proposito original (evitar reexplicar el proyecto en cada sesion) ya lo cubren `docs/arquitectura/.ua/meta.json` y `knowledge-graph.json`, que estan vivos.

### Paso 7 — No proceder con herramientas de descubrimiento de relaciones hasta completar los pasos anteriores

Unicamente cuando el repositorio este libre de referencias operativas al Relationship Record y validado, se evaluara el uso de `understand-anything:understand-knowledge` u otra herramienta para proponer relaciones entre Knowledge Assets. Cualquier relacion propuesta por una herramienta o por la propia IA tiene caracter de sugerencia (`docs/Project_Definition/03_ONTOLOGY.md`, seccion AI Responsibilities: "suggest missing relationships") y requiere validacion explicita del usuario antes de incorporarse como conocimiento canonico.

---

## Consequences

### Positive

* Elimina duplicacion sistematica de informacion de procedencia entre 57 pares de archivos.
* Simplifica el modelo mental del Knowledge Repository: un Knowledge Asset, un archivo.
* Libera la carpeta `conocimiento/relaciones/` y el termino "Relationship Record" para, si se decide en el futuro, redefinirlos exclusivamente como relaciones genuinas entre Knowledge Assets, sin la carga historica del uso puramente provenance actual.

### Negative

* Requiere revisar y potencialmente editar 57 pares de archivos (114 archivos) para migrar informacion no redundante antes de borrar cada Relationship Record.
* Introduce una excepcion de modelo de ramas (Architectural Decision Record 1102) aceptada explicitamente para esta limpieza.
* Deja pendiente, sin resolver, el problema de fondo que motivo esta conversacion: como se estableceran y mantendran relaciones reales entre Knowledge Assets.

### Risks Mitigated

* Riesgo de perdida de informacion: mitigado por el Paso 4 (migracion explicita antes de borrar).
* Riesgo de reescribir la historia del proyecto: mitigado distinguiendo explicitamente, en el Paso 6, entre documentos operativos (se actualizan) y documentos historicos (no se tocan).
* Riesgo de perder la trazabilidad de por que se elimino el Relationship Record: mitigado recomendando marcar este documento como DEPRECATED en vez de eliminarlo fisicamente al concluir el trabajo.

---

## References

* docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* docs/decisiones/ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md
* docs/decisiones/ADR-1104-Ampliacion_Domain_Tags_Corpus_ABC.md
* docs/decisiones/ADR-1105-Procedimiento_Elaboracion_Knowledge_Assets.md
* docs/decisiones/ADR-0033-Taxonomia_ADRs.md
* docs/Project_Definition/02_MASTER_ARCHITECTURE.md
* docs/Project_Definition/03_ONTOLOGY.md
* ai/templates/30_knowledge_asset.md
* ai/templates/31_relationship_record.md
* ai/templates/10_repository_event.md
* ai/skills/documentation/elaborar_knowledge_assets.md
