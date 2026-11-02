# ADR-1118: No Solapamiento entre Índices de Corpus del Mismo Tipo de Entidad

Status: APPROVED

Date: 2026-08-25

Scope:
Knowledge

Category:
KNOWLEDGE

Domain: (Optional)
Documentation

Tags:
knowledge-asset, indice-corpus, personas, deduplicacion, adr-1108, bibliografia, excepcion-documentada

Related ADRs:
ADR-1100 (ubicación `conocimiento_index/` y convención de nombres), ADR-1108 (patrón general de índice derivado de corpus — este ADR lo refina cuando coexisten varios índices sobre el mismo tipo de entidad), ADR-1109 (criterio de relación verificada y exclusión de recopilatorios de la bóveda — mismo espíritu de mantener los índices de corpus limpios y no redundantes), ADR-1122 (primera excepción a este ADR, puntos 4/14/17 — convergencia East/West entre los índices cronológicos; precedente directo del punto 6 de este ADR, aplicado aquí a los índices de bibliografía con una mecánica distinta, ver punto 6)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

El 2026-08-25 se creó `conocimiento_index/260825-index-society-personas-contemporaneas.md`, un índice de corpus (patrón `ADR-1108` punto 3) acotado a personas contemporáneas, chinas o de origen chino, no políticas — a petición del usuario, con el objetivo explícito de mantenerlo separado de `260806-index-history-personas.md` (creado 2026-08-06), que mezcla figuras históricas y contemporáneas sin distinguir nacionalidad ni excluir a políticos.

Durante la curación, 4 personas (Huang Xiaojie, Zheng Yongnian, Wang Jianlin, Wang Jue) quedaron registradas simultáneamente en ambos índices: ya existían como fila en el índice histórico desde su creación, y se añadieron también al índice nuevo por cumplir su criterio. El usuario señaló explícitamente que una persona no debería figurar en dos índices de corpus a la vez, y se corrigió retirando las 4 filas duplicadas del índice histórico, dejándolas solo en el especializado.

El usuario indicó además que el criterio de exclusión de "políticos" quedará pendiente de refinar más adelante (p. ej. cargos políticos secundarios, dentro o fuera de España) — lo que apunta a que en el futuro podría crearse un tercer índice (p. ej. de figuras políticas contemporáneas), reabriendo la misma pregunta de solapamiento que motivó este ADR. No existía, hasta ahora, ninguna regla explícita sobre qué hacer cuando dos índices de corpus cubren el mismo tipo de entidad (`ADR-1108` define la mecánica de un índice individual, pero no la relación entre varios índices del mismo tipo).

### Contexto de la ampliación (2026-10-18) — excepción para bibliografía de interés cruzado

El 2026-10-17/18, al corregir el fascículo nº 38 de "El Reportaje de la Historia" (*El Mundo*, Los Mongoles), se añadieron deliberadamente las mismas 6 filas de libros tanto a `conocimiento_index/260822-index-documentation-bibliografia-recomendada.md` (BBL, dominio chino/East) como a `conocimiento_index/260822-index-documentation-west-mundo-bibliografia-recomendada.md` (RMW, colección West-Mundo), con enlace cruzado explícito en ambos sentidos. El usuario señaló que esto contradice el punto 1 y, en particular, el punto 5 de este ADR ("nunca se deja una entidad en dos índices 'porque ambos aplican un poco'"), y pidió evaluar si la excepción estaba justificada o si procedía revertirla.

La diferencia con el caso que motivó este ADR (personas: cada individuo pertenece a un único cajón taxonómico, sin ambigüedad real) es que un **libro reseñado en una página de prensa** puede ser, de forma verificable y no solo editorial, objeto de interés propio y documentado en dos dominios distintos del corpus a la vez — no por "aplicar un poco" a ambos, sino por tener Knowledge Assets dedicadas en ambos lados. Se verificó un segundo caso (fascículo nº 50, Marco Polo / *Il Milione*) con el mismo patrón: tres Knowledge Assets East (`conocimiento/260901-history-marco-polo-viajero-global.md`, `conocimiento/260904-history-marco-polo-debate-historicidad-china.md`, `conocimiento/260904-history-marco-polo-embustes-libro-maravillas.md`) debaten su historicidad frente a fuentes chinas, pese a que el libro solo estaba catalogado en RMW. Se revisaron los 70 fascículos de la colección y no se encontraron más casos con esta tensión temática genuina (el más cercano, fascículo 27/Atila, se descartó por conexión temática débil — tratado de pasada en un libro ya duplicado, sin Knowledge Assets East propias sobre Atila).

**No es la primera excepción a este ADR**: `ADR-1122` (puntos 4, 14 y 17) ya había introducido, desde 2026-08-25/09-04, una excepción equivalente para los índices cronológicos (East/West/West-Mundo/West-España) — las "entradas de convergencia East/West", que llevan fila completa en ambos índices cuando documentan contacto directo entre esferas, marcadas explícitamente como tales. Se revisaron los tres índices cronológicos en busca de casos semejantes sin convergencia (2026-10-18): el único caso ya identificado, el viaje de Marco Polo a la corte del Gran Kan (1271-1295), está correctamente triple-enlazado desde 2026-09-04/06 (`^260806-CRN-00011`, `^260904-CRW-00003`, `^260904-CRM-00014`). La fundación del Imperio mongol (`^260806-CRN-00004`) y la invasión mongola de Rusia/Europa y batalla de Kulikovo (`^260806-CRN-00025`) no tienen fila espejo en ningún índice West pese a cumplir el criterio de "contacto directo documentado" mejor incluso que el propio caso de Marco Polo — pero no por un fallo de indexación: simplemente no existe todavía ninguna Knowledge Asset en `conocimiento_west/`, `conocimiento_west_mundo/` ni `conocimiento_west_espana/` que documente esos eventos desde el lado West. El mecanismo de convergencia enlaza Knowledge Assets ya existentes a ambos lados; no genera contenido nuevo, por lo que no hay nada que corregir aquí — a diferencia del caso de bibliografía, donde la fuente (la página de reseñas) ya existía físicamente en `fuentes/260824_mundo_west/` en ambos casos corregidos.

**Diferencia de mecánica, documentada aquí para evitar confusión futura**: la convergencia de `ADR-1122` enlaza Knowledge Assets **independientes**, cada una con su propio ángulo y fuente (biográfico en East, crítica literaria en West-Mundo, marco narrativo de Reverte en West) — no duplica texto, documenta el mismo evento histórico desde fuentes distintas. La excepción del punto 6 de este ADR, en cambio, sí duplica el contenido literal de la fila (título, autor, año, editorial) entre BBL y RMW, porque una cita bibliográfica no tiene "ángulos" distintos que documentar por separado — es el mismo dato editorial citado desde la misma página de reseña en ambos índices. Ambas mecánicas comparten el mismo espíritu (visibilidad verificada en ambos dominios, nunca por conveniencia editorial) pero no son intercambiables.

Se decide formalizar una excepción estrecha y auditable (punto 6), en vez de ampliar la regla general o tratar cada caso de forma ad hoc.

---

## Decision

1. **Regla general**: cuando dos o más Knowledge Assets de `conocimiento_index/` son índices de corpus (`ADR-1108` punto 3) sobre el **mismo tipo de entidad** (p. ej. "personas"), una entidad concreta debe figurar como fila en **exactamente uno** de ellos, nunca en más de uno simultáneamente.

2. **Criterio de asignación**: la entidad se indexa en el índice **más específico** cuyo `Scope` la cubra. El índice general (`260806-index-history-personas.md`) actúa como cajón de cierre: retiene lo que ningún índice especializado reclama todavía (figuras históricas, y personas contemporáneas que no encajan en ningún índice especializado existente — p. ej. no chinas, o políticas). Un índice especializado nuevo (como `260825-index-society-personas-contemporaneas.md`) nunca amplía sus propias filas copiando del general — mueve la fila, no la duplica.

3. **Mecánica de la migración** (cuando una entidad pasa de un índice al otro):
   - Se retira la fila completa del índice de origen. El identificador de bloque liberado (`^YYMMDD-xxx-NNNNN`) **no se reasigna** a otra entrada — permanece huérfano y documentado en `Notes`, mismo principio de estabilidad de identificadores que rige el resto de índices del proyecto.
   - Se añade la fila en el índice de destino, con su propio identificador de bloque nuevo.
   - Las Knowledge Assets de origen que enlazaban al índice antiguo actualizan su `Related Assets` para apuntar solo al índice nuevo — nunca a ambos.
   - Ambos índices documentan la migración en su sección `Notes`, con fecha y referencia cruzada al otro fichero, para que quede trazable por qué una fila desapareció o apareció.

4. **No aplica** a índices de corpus sobre tipos de entidad *distintos* (p. ej. `260806-index-society-organizaciones-instituciones.md` vs. este índice de personas): cada tipo de entidad tiene su propio espacio de nombres de bloque (`RGN` vs. `PRS`/`CNT`) y no compite por las mismas filas.

5. **Precedente para índices especializados futuros**: cuando se cree un nuevo índice especializado que reclame entidades ya presentes en `260806-index-history-personas.md` o en cualquier índice especializado existente (p. ej. un futuro índice de figuras políticas contemporáneas), se aplica el mismo procedimiento del punto 3 — nunca se deja una entidad en dos índices "porque ambos aplican un poco".

6. **Excepción para bibliografía de interés cruzado documentado (añadido 2026-10-18)**: cuando una página de reseñas de prensa trata una figura o tema con relevancia propia y verificable tanto para el dominio chino/East (`conocimiento/`, `260822-index-documentation-bibliografia-recomendada.md`) como para una colección West (`260822-index-documentation-west-*-bibliografia-recomendada.md`), sus entradas pueden figurar como fila en ambos índices, como excepción explícita a la regla general del punto 1. La relevancia cruzada debe ser **verificable, no editorial**: exige la existencia de Knowledge Assets dedicadas al tema en el lado donde se duplica, no solo un juicio de que "el tema podría interesar también allí". Condiciones:
   - (a) cada fila duplicada enlaza explícitamente a su par en el índice hermano (bloque exacto, no solo el fichero), documentando la duplicación como deliberada en el propio texto de la fila;
   - (b) la excepción se documenta en la sección `Notes`/changelog de ambos índices, con fecha, justificación y lista de las Knowledge Assets que motivan la relevancia cruzada;
   - (c) no se automatiza ni se aplica por defecto a fascículos "exóticos" o temáticamente lejanos — cada caso requiere revisión y decisión explícitas, incluyendo una comprobación activa de que no existen más casos semejantes sin tratar en el resto del corpus;
   - (d) a diferencia del punto 3 (migración), aquí **no hay identificador liberado**: ambas filas son permanentes y ninguna se retira al crear la otra.
   
   Precedentes: fascículo 38 (Gengis Kan, `^260822-BBL-00030` a `00035` / `^260822-RMW-01340` a `01345`), fascículo 50 (Marco Polo, `^260822-BBL-00036` / `^260822-RMW-00951`).

---

## Consequences

### Positive

* Evita que el mismo dato (nombre, rol, hanzi/pinyin) tenga que mantenerse sincronizado manualmente en dos sitios, con el riesgo de que diverjan en ediciones futuras.
* Mantiene cada índice fiel a su propio `Scope` declarado, en vez de degradarse en un cajón desastre que acumula copias de otros índices.
* Dado que el corpus ya muestra intención de crecer con más índices especializados de personas (políticos, mencionado por el usuario como trabajo futuro), fija la regla antes de que el problema se repita, en vez de resolverlo de forma ad hoc cada vez.

### Negative

* Introduce un paso de mantenimiento adicional (migrar filas, no solo añadirlas) cada vez que se crea un índice especializado nuevo sobre un tipo de entidad ya cubierto por un índice general.
* Dispersa la información sobre una misma familia de entidades ("personas") en varios ficheros — mitigado porque cada índice referencia a los demás en su propio `Purpose`/`Key Characteristics` (ver `260806-index-history-personas.md` y `260825-index-society-personas-contemporaneas.md`).

### Risks Mitigated

* Duplicación silenciosa de una misma entidad en dos índices de corpus, ya observada en la práctica (Huang Xiaojie, Zheng Yongnian, Wang Jianlin, Wang Jue) antes de que este ADR existiera.
* Identificadores de bloque reasignados a una entrada distinta tras retirar una fila, lo que rompería cualquier enlace externo ya escrito hacia el bloque antiguo.
* Repetición futura del mismo problema al crear el índice de figuras políticas u otros índices especializados de personas, sin una regla escrita a la que remitirse.

### Positive (ampliación 2026-10-18, punto 6)

* Reconoce, con una excepción estrecha y auditable, que un libro —a diferencia de una persona— puede tener relevancia propia y verificable en dos dominios del corpus a la vez, sin forzar una elección artificial entre ambos índices.
* El requisito de verificación activa (Knowledge Assets dedicadas, no solo juicio editorial) y de revisión completa del corpus al aplicar la excepción evita que se convierta en una puerta trasera para duplicar libros por conveniencia.

### Negative (ampliación 2026-10-18, punto 6)

* Introduce, por primera vez en este ADR, filas verdaderamente duplicadas y permanentes (sin identificador liberado), lo que exige mantener sincronizados dos enlaces cruzados si alguna de las dos filas se edita en el futuro.

---

## References

* docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md (mecánica de un índice de corpus individual)
* docs/decisiones/ADR-1109-Distincion_Grafo_Obsidian_vs_Grafo_Understand_Anything.md (precedente de refinar reglas de índice tras un caso real, mismo estilo de revisión)
* conocimiento_index/260806-index-history-personas.md (índice general, 4 filas migradas el 2026-08-25 — ver su Notes)
* conocimiento_index/260825-index-society-personas-contemporaneas.md (índice especializado que motivó este ADR — ver su Notes)
* conocimiento_index/260822-index-documentation-bibliografia-recomendada.md (índice East/BBL — excepción del punto 6 aplicada a los fascículos 38 y 50, ver su Notes)
* conocimiento_index/260822-index-documentation-west-mundo-bibliografia-recomendada.md (índice West-Mundo/RMW — par cruzado de la excepción del punto 6, ver su Notes)
