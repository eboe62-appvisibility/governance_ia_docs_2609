# ADR-1114: Adopción de Understand-Anything como Knowledge Provider del proyecto

Status: APPROVED

Date: 2026-08-08

Scope:
Knowledge

Category:
ARCHITECTURE

Domain: (Optional)
Documentation, Research

Tags:
knowledge-provider, knowledge-graph, understand-anything, tooling

Related ADRs:
ADR-1103 (puerta de regeneración del grafo antes de fusionar a `master`), ADR-1107 (confirmación previa entre actualización incremental y completa del grafo), ADR-1109 (distinción entre el grafo de Obsidian y el grafo de Understand-Anything)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

`docs/Project_Definition/02_MASTER_ARCHITECTURE.md` § COMPONENTS define el "Knowledge Provider" como una capa de abstracción entre los asistentes de IA y la implementación subyacente del Knowledge Graph, dejando explícitamente abierta la elección de tecnología concreta: *"Examples include: Understand Anything; Neo4j; RDF/OWL; future technologies."*

Esa elección de tecnología concreta nunca quedó registrada formalmente en ningún ADR. Se adoptó y se ha venido usando desde antes del 2026-07-29 (primera referencia registrada en un ledger histórico de estado del proyecto, mantenido fuera de la serie numerada de `Project_Definition/` y retirado del repositorio el 2026-08-08 una vez incorporado aquí su contenido relevante) sin más respaldo formal que esa nota informal.

Desde entonces, `understand-anything` se ha convertido en dependencia operativa de facto de varios ADRs ya aprobados — `ADR-1103` exige regenerar su grafo antes de fusionar a `master`; `ADR-1107` regula cuándo usar actualización incremental frente a completa; `ADR-1109` distingue explícitamente su grafo del de Obsidian — sin que exista un ADR que documente la decisión original de adoptar esta herramienta en concreto.

---

## Decision

1. Se adopta formalmente `understand-anything` (https://github.com/Egonex-AI/Understand-Anything.git) como la implementación concreta del Knowledge Provider definido en `02_MASTER_ARCHITECTURE.md`, generando y manteniendo `docs/arquitectura/.ua/knowledge-graph.json` como grafo semántico autoritativo del repositorio.

2. Esta elección no cierra la puerta a otras implementaciones (Neo4j, RDF/OWL, tecnologías futuras) que `02_MASTER_ARCHITECTURE.md` ya deja abiertas — es la implementación vigente, no la única contemplada por el diseño arquitectónico. Un cambio de implementación futuro requeriría un nuevo ADR que documente el motivo (p. ej. limitaciones de escala, discontinuación de la herramienta) y declare `Supersedes: ADR-1114`.

3. El grafo generado por esta herramienta se rige por las reglas ya fijadas en `ADR-1103` (regeneración antes de merge a `master`), `ADR-1107` (elección incremental vs. completa) y `ADR-1109` (independencia respecto al grafo de Obsidian). Este ADR no redefine esas reglas — formaliza retroactivamente la elección de herramienta que ya asumían.

---

## Consequences

### Positive

* Cierra un vacío de trazabilidad: una dependencia operativa citada por otros tres ADRs (1103, 1107, 1109) queda por fin respaldada por su propia decisión formal.
* Deja constancia explícita de que la elección es una implementación entre varias posibles según `02_MASTER_ARCHITECTURE.md`, no un compromiso arquitectónico permanente — facilita revisar la decisión sin tocar la arquitectura conceptual del proyecto.

### Negative

* No se realizó en su momento una comparativa formal con alternativas (Neo4j, RDF/OWL) más allá de la mención genérica ya presente en `02_MASTER_ARCHITECTURE.md`. Este ADR formaliza una elección ya hecha en la práctica, no el resultado de una evaluación estructurada entre opciones — ver Anexo.

### Risks Mitigated

* Que una futura sesión reconsidere o sustituya `understand-anything` sin darse cuenta de que otros tres ADRs dependen operativamente de su presencia.

---

## References

* docs/Project_Definition/02_MASTER_ARCHITECTURE.md § COMPONENTS (Knowledge Provider)
* docs/decisiones/ADR-1103-Procedimiento_Validado_de_Integracion_de_Ramas.md
* docs/decisiones/ADR-1107-Confirmacion_Previa_Actualizacion_Incremental_vs_Completa_del_Grafo.md
* docs/decisiones/ADR-1109-Distincion_Grafo_Obsidian_vs_Grafo_Understand_Anything.md
* Anexo de este mismo documento (registro histórico íntegro de la adopción — el ledger original que lo contenía se retiró del repositorio el 2026-08-08)

---

## Anexo: Comparativa y fuentes de inspiración

No se realizó una comparativa formal con tabla de pros/contras frente a Neo4j o RDF/OWL — la elección se apoyó en una demostración concreta de la herramienta, validada después en la práctica durante meses de uso real (grafo regenerado varias veces, con capas arquitectónicas y tour guiado, sobre un corpus creciente).

Se conserva aquí, íntegra y con formato adaptado a Markdown (el original era texto plano), la antigua sección "Herramientas" de un ledger histórico de estado del proyecto — el registro original de esta adopción y de otras decisiones de herramienta, retirado del repositorio el 2026-08-08 una vez incorporado aquí su contenido íntegro. No es parte de la Decision en sí, sino el camino que condujo a ella; cubre más herramientas que solo Understand-Anything (Obsidian, sub-agentes, RAG/MCP, otras fuentes evaluadas) porque así estaba registrado originalmente en un único documento y se prefiere preservarlo completo antes que fragmentado.

### Antigua sección "Herramientas" (registro histórico del proyecto, versión 260729)

| Herramienta | Estado | ROI | Recomendación |
|---|---|---|---|
| Understand Anything | Implementado | Alto | Ya en uso — grafo regenerado y verificado |
| Obsidian | Rama creada | Alto | Sí — siguiente paso, por etapas reversibles |
| Sub-agentes (Task/Agent) | En uso puntual | Alto | Sí — ya validado para elaboración de fichas por lotes |
| RAG / MCP local | No evaluado | Por evaluar | Futuro, condicionado a crecimiento de volumen |

**Understand Anything**

Grafo semántico ya generado (222 nodos, 218 aristas, 10 capas, tour de 10 pasos — cifras de la fecha original del registro, no del estado actual). Fuente de indexación técnica autoritativa del repositorio.

Fuentes:
* https://github.com/Egonex-AI/Understand-Anything.git
* https://www.youtube.com/shorts/GnFTbPlJBN4

**Obsidian**

Capa de navegación humana sobre el grafo ya indexado — no debe redefinir la organización documental existente ni generar una taxonomía paralela. Rama `obsidian` ya creada y sincronizada; plan de implementación por etapas reversibles ya acordado (adopción formalizada después en `ADR-1109`).

Fuentes:
* https://obsidian.md/
* https://www.youtube.com/shorts/Avvn90oqAcY
* https://www.youtube.com/shorts/8XRJfTk7Jgc
* https://www.youtube.com/shorts/73yz8gTrj1A
* https://graphify.net/

**Sub-agentes**

Usados para paralelizar la elaboración de Knowledge Assets por lotes (~9 documentos por lote), con checklist de verificación posterior. Procedimiento formalizado en `ADR-1105` y en el skill `elaborar_knowledge_assets`.

Fuentes:
* https://www.youtube.com/shorts/NOecCy8aSWc

**Otras fuentes evaluadas (sin decisión de adopción asociada)**

*Claude Skills Marketplace* — tomar como referencia la organización por agentes especializados, evitando incorporar habilidades innecesarias.
* https://github.com/mhattingpete/claude-skills-marketplace.git
* https://www.youtube.com/shorts/DwkTuMo5qLA

*LightRAG* — implementar recuperación semántica para minimizar el contexto enviado a los modelos de IA.
* https://www.youtube.com/shorts/ODJN0LvQgws
