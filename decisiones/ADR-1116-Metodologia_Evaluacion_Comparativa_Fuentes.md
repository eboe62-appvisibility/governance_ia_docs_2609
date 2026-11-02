# ADR-1116: Metodología de Evaluación Comparativa de Fuentes Documentales (rúbrica de 6 criterios)

Status: APPROVED

Date: 2026-08-09

Scope:
Knowledge

Category:
METHODOLOGY

Domain: (Optional)
Documentation, Research

Tags:
comparativa-fuentes, evaluacion-calidad, metodologia, agent-browser

Related ADRs:
ADR-1115 (agent-browser — motor de acceso que ha alimentado los 4 experimentos hasta ahora, aunque el método es agnóstico de herramienta), ADR-1113 (índice de fuentes documentales curadas que esta metodología mantiene actualizado, columna `Valoración`), ADR-1111 (criterios de interés — contraste de fuentes contradictorias), ADR-1108 (patrón de tabla/índice y mecanismo de "columnas adicionales según lo requiera el caso concreto"), ADR-1119 (exige confirmar por lectura directa el contenido real de una fuente antes de puntuarla o citar un hecho suyo — precondición previa a aplicar esta rúbrica)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

El 2026-08-09 el proyecto ejecutó su primer experimento comparativo formal — valorar varias fuentes documentales sobre un mismo tema (腊八节) con una rúbrica de 6 criterios — y documentó esa rúbrica dentro de la propia Knowledge Asset de resultados, `conocimiento_valoracion_fuentes/260809-culture-labajie-00-comparativa-fuentes.md`. Ese mismo día se repitió el proceso tres veces más, sobre temas distintos (北盘江第一桥, 高德地图, 百度地图), reutilizando siempre la misma rúbrica por referencia a esa primera ficha ("mismos 6 criterios que el experimento anterior").

El usuario señaló que esta ubicación es incorrecta: una Knowledge Asset de tipo `Document` cataloga el conocimiento sobre un tema concreto (腊八节), no una decisión metodológica de alcance transversal. La rúbrica ya gobierna 4 experimentos y previsiblemente gobernará los siguientes — es exactamente el tipo de decisión revisable, con criterios explícitos y consecuencias sobre cómo se prioriza el conocimiento del proyecto, para la que existen los ADR (mismo criterio ya aplicado en `ADR-1109` para Obsidian y en `ADR-1115` para agent-browser).

Un incidente concreto reforzó la necesidad de una definición canónica y no ambigua: en el cuarto experimento (百度地图), NCPSSD y Baidu Baike empataron a 50/60 puntos. Se resolvió el empate con un criterio editorial ad hoc ("preferir fuente independiente sobre autopublicación corporativa") que promocionó indebidamente a NCPSSD. El usuario detectó el error: NCPSSD aportaba 62 resultados académicos, pero todos sobre *usos particulares* del producto (estudios de accesibilidad urbana, pedagogía), ninguno explicaba qué es el producto en sí — mientras que Baidu Baike sí lo hacía. La distorsión no estaba en el desempate, sino en que el criterio **Profundidad** no distinguía "hay mucho contenido" de "ese contenido explica el tema base" — permitiendo que Verificabilidad/Diferenciación/Neutralidad altas (favorecidas estructuralmente por cualquier fuente académica, casi por definición) compensaran una Profundidad que en realidad debía haber sido mucho más baja. Corregir el criterio en vez de forzar el resultado deshizo el empate sin necesidad de ningún juicio editorial adicional.

---

## Decision

1. **Ubicación canónica**: esta rúbrica vive en este ADR, no en ninguna Knowledge Asset de resultados. Las fichas `*-00-comparativa-fuentes.md` y las fichas individuales por fuente referencian este ADR (`Related ADRs`) en vez de redefinir los criterios; `conocimiento_valoracion_fuentes/260809-culture-labajie-00-comparativa-fuentes.md` (origen histórico de la rúbrica) queda como precedente documentado, con una nota que remite aquí para la definición vigente.

2. **6 criterios, 0 a 10 en pasos de 2** (0, 2, 4, 6, 8, 10):
   1. **Profundidad** — cobertura del **tema o producto en sí** (ficha técnica, historia, contexto, significado). **Definición reforzada tras el incidente del 2026-08-09**: contenido sobre *usos particulares, aplicaciones o menciones tangenciales* del tema no cuenta como Profundidad alta, por extenso o rigurosamente citado que esté — Profundidad mide si un lector que no sabe nada del tema puede aprender qué es leyendo esta fuente. Ese contenido de aplicación/uso particular, cuando aporte una perspectiva genuinamente distinta (crítica, académica, cultural), se valora en el criterio de Diferenciación, no en este.
   2. **Verificabilidad** — presencia y calidad de citas a fuentes primarias o secundarias identificables (que el contenido de la cita sea consultable, no solo que exista un marcador de nota).
   3. **Limpieza** — fidelidad y facilidad de uso de la extracción tal como se obtuvo (ruido de navegación, texto cifrado, contaminación de contenido ajeno al tema — p. ej. plantillas de "entidades relacionadas" de otro producto de la misma empresa matriz —, necesidad de depuración manual).
   4. **Diferenciación** — qué aporta esta fuente que ninguna otra del mismo experimento ofrece, evaluada contra el conjunto completo, no solo contra la fuente más parecida.
   5. **Mantenimiento** — viabilidad de volver a consultar esta fuente de forma automatizada en el futuro (¿requiere login, CAPTCHA, reintentos por timing de carga?).
   6. **Neutralidad** — ausencia de sesgo oficialista, comercial o editorial detectable en el contenido concreto evaluado, no en la fuente en abstracto.

3. **Columna Total = suma directa, no promedio**: rango 0-60. Se revisará el paso a promedio cuando el número de experimentos por fuente lo justifique estadísticamente — decisión pendiente, no automática.

4. **Excepción de escala para fuentes accesibles sin contenido dedicado**: cuando una fuente es accesible (portada y, si aplica, buscador funcionan) pero no devuelve contenido dedicado al tema del experimento, se puntúa `Profundidad = 1` (deliberadamente fuera de la escala de pasos de 2) y el resto de criterios a `0` — nunca `0` en todos, para que el total nunca sea exactamente cero y no distorsione cálculos acumulados futuros que sumen estas tablas.

5. **Empates**: un empate en puntuación total **no se resuelve con un criterio de desempate ad hoc**. Antes de aplicar cualquier juicio editorial de desempate, se revisan los criterios individuales en busca de una distorsión de fondo (como el caso de Profundidad en el punto 2) — si existe, se corrige el criterio, no el resultado. Si tras esa revisión el empate persiste genuinamente entre fuentes con contenido sustancial, se documenta explícitamente como empate sin promoción forzada, y se solicita confirmación del usuario antes de promocionar cualquiera de las dos a `conocimiento/`.

6. **Organización de ficheros**: cada experimento genera una ficha `NNNNNN-domain-tema-00-comparativa-fuentes.md` (tabla + lectura de resultados) y, para cada fuente **con contenido**, una ficha individual — ambas en `conocimiento_valoracion_fuentes/`. La fuente con mayor puntuación total se traslada a `conocimiento/`, sin prefijo numérico en el nombre de archivo. Las fuentes sin contenido dedicado (Profundidad = 1) no generan ficha individual — su resultado queda documentado únicamente en la tabla comparativa y en la fila correspondiente de `conocimiento_index/260806-index-documentation-fuentes-curadas.md`.

7. **Excepción de nomenclatura — puntuación en el nombre de archivo**: mientras una fuente permanece en `conocimiento_valoracion_fuentes/`, su fichero incluye la puntuación total como prefijo numérico (p. ej. `260809-technology-baidumaps-48-ncpssd.md`) para facilitar la comparación visual — excepción deliberada a la convención general de nombres de `ADR-1100`/`ADR-1101`, limitada a esta carpeta. Al promocionarse a `conocimiento/`, el prefijo se retira.

8. **Alcance temático abierto**: la rúbrica no se limita a temas de curación cultural del corpus — los experimentos 3 y 4 (高德地图, 百度地图) la aplicaron a una investigación operativa (elegir herramienta de mapas para la rama `maps`), confirmando que el método es reutilizable para cualquier "¿qué fuente de las ya accesibles vía agent-browser es mejor para X?", no solo para curación de contenido del corpus.

---

## Consequences

### Positive

* Una única definición canónica, consultable y citable por ADR, en vez de una definición de facto dispersa entre 4 fichas de contenido que la referencian por convención tácita.
* La corrección de Profundidad queda documentada como refinamiento metodológico permanente, no como una nota aislada en una sola ficha — futuros experimentos heredan la corrección automáticamente en vez de arriesgarse a repetir la misma distorsión.
* La regla de empates (punto 5) evita que un futuro empate se resuelva otra vez con un criterio de desempate improvisado antes de descartar que el problema esté en la puntuación de base.

### Negative

* ~~Las 4 fichas comparativas ya existentes citan la rúbrica por referencia a la primera...~~ — corregido en el mismo cambio que creó este ADR: las 4 fichas y el índice principal ya declaran `ADR-1116` en su campo `Related ADRs`.
* La revisión "detectar distorsión antes de desempatar" (punto 5) añade un paso de análisis manual que no existía explícitamente antes — más lento que aplicar un desempate directo, pero evita el tipo de error ya ocurrido.

### Risks Mitigated

* Que la rúbrica siga percibiéndose como parte del contenido de una ficha sobre 腊八节 en vez de como una decisión de gobernanza del proyecto — mitigado por su reubicación aquí.
* Que la misma distorsión de Profundidad (premiar cantidad/rigor académico sobre explicación real del tema) se repita sin diagnosticarse en futuros experimentos — mitigado por la definición reforzada del punto 2.

---

## References

* `conocimiento_valoracion_fuentes/260809-culture-labajie-00-comparativa-fuentes.md` (origen histórico de la rúbrica, primer experimento — precedente, no ubicación canónica desde este ADR)
* `conocimiento_valoracion_fuentes/260809-engineering-beipanjiang-00-comparativa-fuentes.md` (segundo experimento)
* `conocimiento_valoracion_fuentes/260809-technology-gaode-00-comparativa-fuentes.md` (tercer experimento)
* `conocimiento_valoracion_fuentes/260809-technology-baidumaps-00-comparativa-fuentes.md` (cuarto experimento — incidente de Profundidad/empate que motiva este ADR)
* `conocimiento_index/260806-index-documentation-fuentes-curadas.md` (índice que la columna `Valoración` de esta metodología mantiene actualizado)
* `docs/decisiones/ADR-1115-Adopcion_Agent_Browser_Web_Monitoring.md`
