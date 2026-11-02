# ADR-1111: Criterios de Interés para la Curación de Conocimiento en el Proyecto

Status: APPROVED

Date: 2026-08-05

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Research, Documentation

Tags:
interest-criteria, curation, living-document, methodology, corpus-curation

Related ADRs:
ADR-1100 (ubicación y convención de nombres de Knowledge Assets — regula el "dónde" y el "cómo se llama", no el "por qué se incorpora"), ADR-1101 (ubicación física de fuentes en `fuentes/`), ADR-0033 (vocabulario Domain/Tags — clasifica lo ya incorporado, no decide si se incorpora)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

Este ADR nació el 2026-08-05, durante la curación de la edición de enero de 2026 de 歐洲時報 (`fuentes/260101_oushinet/`), con un alcance inicial estrecho: un criterio de relevancia para decidir qué artículos de **prensa periódica china** justifican Knowledge Asset propio. Al redactarlo, se hizo evidente que ese alcance era accidental, no deliberado — la pregunta real que motiva este ADR ("¿este contenido merece incorporarse al proyecto, dado lo que le interesa al usuario ahora mismo?") no depende de que la fuente sea prensa, ni de que sea periódica, ni de que ya exista físicamente en el repositorio.

El usuario señaló explícitamente que:

- El proyecto no se limita a prensa periódica como fuente — a futuro puede incorporar búsqueda proactiva de información (periódica o no) en internet u otras fuentes.
- El ámbito de interés del usuario **evolucionará**: se ampliará a nuevos temas, se restringirá en otros, según avance el proyecto.
- No quiere verse obligado a crear un ADR nuevo cada vez que el ámbito de interés cambie o se aplique a un tipo de fuente distinto, solo por no haber adoptado desde el principio una perspectiva suficientemente amplia.

Este ADR se reescribe, por tanto, como documento **vivo**: un único lugar donde se acumulan y actualizan los criterios de interés del usuario a lo largo del tiempo, mediante entradas fechadas en la sección `Historial de Revisiones`, en vez de mediante ADRs nuevos que amplíen o parcheen uno anterior.

---

## Decision

1. **Objeto de este ADR**: responde a la pregunta "¿este contenido merece incorporarse al proyecto (como Knowledge Asset, o como fuente citada) dado el interés actual del usuario?" — con independencia de:
   - el tipo de fuente (prensa periódica, página web, red social, libro, vídeo, entrevista, observación de campo...);
   - si la fuente ya existe en el repositorio o se busca de forma proactiva;
   - el canal de descubrimiento (aportada por el usuario, encontrada por el asistente de IA, resultado de una búsqueda dirigida).

   No regula **dónde** se guarda el conocimiento incorporado (`ADR-1100`, `ADR-1101`) ni **cómo se clasifica** (`ADR-0033` § Domain/Tags) — solo si se incorpora.

2. **Mecanismo de documento vivo**: cualquier cambio en el ámbito de interés — ampliación, restricción, aplicación a un nuevo tipo de fuente, refinamiento de un criterio existente — se documenta como una nueva entrada fechada en `Historial de Revisiones`, editando este mismo ADR. **No se crea un ADR nuevo** para este propósito, salvo que el cambio propuesto sea en sí mismo una decisión arquitectónica ajena al criterio de interés (p. ej. cambiar la ubicación física del conocimiento, que seguiría perteneciendo a `ADR-1100`/`ADR-1101`).

3. **Criterios de interés vigentes** (ver también el historial para su origen y evolución):

   - **Criterio A**: contenido que sea opinión o perspectiva directa de la comunidad china sobre España y/o Occidente.
   - **Criterio B**: contenido que trate directamente sobre China o los chinos, dentro o fuera de China.

   Basta con cumplir uno de los dos para justificar la incorporación.

4. **Excepciones** — el contenido puede incorporarse aunque no cumpla A ni B en sentido estricto, cuando concurre alguna de estas razones, documentada explícitamente (en `Notes` del Knowledge Asset resultante, no asumida de forma implícita):

   - **(i) Referencia necesaria**: sirve de cita o índice para otro contenido que sí cumple el criterio.
   - **(ii) Valor comparativo futuro**: permite comparar a China con otra región en un dominio concreto (infraestructura, economía, políticas públicas...), aunque el contenido en sí no trate sobre China de forma central.
   - **(iii) Reacción de la comunidad china ante Occidente**: documenta una opinión o postura del medio/comunidad china ante un hecho o política occidental, aunque el hecho detonante en sí no sea chino.

5. **Revisión activa, no aplicación mecánica**: estos criterios son el punto de partida vigente, no una regla cerrada. Un caso límite que no encaje con naturalidad se documenta como precedente en el historial, no se fuerza dentro de una categoría existente.

---

## Historial de Revisiones

### 2026-08-05 — Origen y generalización

Redactado inicialmente como "Criterio de Relevancia para Curación de Knowledge Assets desde Medios de Comunicación Periódicos", con los criterios A/B y las excepciones (i)-(iii) ya listados arriba. Motivado por la curación de la edición de enero de 2026 de 歐洲時報: de 19 artículos candidatos, 2 se retiraron del corpus (`260804-infrastructure-accidente-ferroviario-adamuz-cronica.md`, `260804-infrastructure-investigacion-causas-red-ferroviaria-espannola.md` — noticia factual española sin ángulo chino) y 17 se conservaron, 3 de ellos por excepción en vez de por A/B directo:

- `260804-documentation-oushinet-260101-portada-ficha-tecnica.md` — excepción (i).
- `260804-infrastructure-envejecimiento-ferroviario-ue-plan-alta-velocidad.md` — excepción (ii).
- `260804-politics-editorial-eeuu-venezuela-dilema-europeo.md`, `260804-politics-editorial-groenlandia-relacion-ue-eeuu.md` — excepción (iii).

El mismo día, antes de cerrar la sesión, se generalizó el ADR de "medios periódicos" a "cualquier fuente" (ver Context) — el criterio A/B y las excepciones no cambian de contenido, pero dejan de estar acoplados a prensa periódica como tipo de fuente.

---

## Consequences

### Positive

* Un único lugar donde consultar y actualizar el criterio de interés vigente, independientemente de qué tipo de fuente lo origine.
* Evita la proliferación de ADRs estrechos (uno por tipo de fuente, uno por ampliación de ámbito) que habría sido difícil de mantener coherente.
* El historial fechado preserva la trazabilidad de cómo y por qué cambió el criterio, sin perder el precedente concreto que lo originó.

### Negative

* Es un criterio de juicio editorial, no mecánico — sigue dependiendo de que quien cure el contenido (usuario o asistente de IA) revise este ADR antes de decidir, en vez de aplicar una regla automática.
* Al ser un documento vivo que crece por entradas, requiere disciplina para no dejar que el historial se vuelva difícil de leer — si en el futuro crece mucho, valorar resumir entradas antiguas sin perder trazabilidad (no resuelto por adelantado, se abordará si llega a ser un problema real).

### Risks Mitigated

* Curación inconsistente entre sesiones o entre distintos tipos de fuente, sin criterio explícito al que volver.
* Fragmentación de la decisión de interés en varios ADRs estrechos y potencialmente contradictorios entre sí.
* Acumulación de Knowledge Assets sin relación real con el objeto del proyecto, solo por proceder de una fuente en principio relevante.

---

## References

* fuentes/260101_oushinet/transcripcion/260101_oushinet.md (fuente que motivó el origen de este ADR)
* conocimiento/260804-documentation-oushinet-260101-portada-ficha-tecnica.md (caso de excepción (i))
* conocimiento/260804-infrastructure-envejecimiento-ferroviario-ue-plan-alta-velocidad.md (caso de excepción (ii))
* conocimiento/260804-politics-editorial-eeuu-venezuela-dilema-europeo.md, conocimiento/260804-politics-editorial-groenlandia-relacion-ue-eeuu.md (casos de excepción (iii))
* docs/decisiones/ADR-1100-Knowledge_Repository_Ubicacion_y_Naming.md
* docs/decisiones/ADR-1101-Migracion_Fuentes_Miscelanea_y_Naming.md
* docs/decisiones/ADR-0033-Taxonomia_ADRs.md
