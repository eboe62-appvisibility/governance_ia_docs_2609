# ADR-1119: Verificación Obligatoria del Contenido Real de la Fuente (WebFetch) antes de Citar un Resumen de WebSearch como Hecho

Status: APPROVED

Date: 2026-08-26

Scope:
Knowledge

Category:
METHODOLOGY

Domain: (Optional)
Research, Documentation

Tags:
websearch, webfetch, verificacion-fuentes, alucinacion, evidencia

Related ADRs:
ADR-1116 (metodología de evaluación comparativa de fuentes — mismo principio de rigor de verificación, aplicado allí a la valoración de una fuente ya confirmada accesible, aquí a la confirmación previa de que una afirmación puntual existe realmente en ella), ADR-1115 (agent-browser — herramienta hermana de acceso a fuentes web, sujeta al mismo principio de esta decisión), ADR-1108 (patrón de índice derivado de corpus — punto 6 de esta Decision, añadido 2026-08-27, exige evaluar el `Scope` de estos índices tras cada verificación), ADR-1113 (criterio "sobre estudios chinos" del índice de fuentes curadas — ejemplo real de `Scope` aplicado en el punto 6), ADR-1118 (no solapamiento entre índices — comprobación previa obligatoria antes de proponer cualquier alta derivada del punto 6)

Supersedes:
NONE

Superseded By:
NONE

---

## Nota de Ampliación (2026-08-27) — Enriquecimiento de índices tras un contraste

Al contrastar externamente `conocimiento/260823-technology-economics-cxmt-debuto-bolsa-shanghai.md` (2026-08-26/27), el contraste confirmó — con matices y correcciones, pero confirmó — varios datos que el vídeo fuente daba sin verificar: la empresa CXMT y su comparación con ICBC, su fundador Zhu Yiming (no mencionado en absoluto por el vídeo), y dos medios de prensa (Xinhua, SCMP) usados como fuente del contraste. El usuario señaló que faltaba una segunda mitad de la disciplina que este ADR ya exige: no basta con verificar antes de citar (puntos 1-5) — cuando la verificación confirma una entidad o fuente que ya encaja en el `Scope` de un índice de corpus (`ADR-1108`) todavía no la tiene, el asistente debe **proponer activamente** esa incorporación, no esperar a que se le pregunte.

Se añade como punto 6 de la Decision:

6. **Tras completar una verificación de las exigidas por este ADR, evaluar activamente si alguna entidad o fuente confirmada encaja ya en el `Scope` de un índice de corpus existente (`conocimiento_index/*.md`) que todavía no la registre, y proponerlo como parte del mismo turno de trabajo — no solo cuando el usuario lo pida explícitamente.** La propuesta se hace, no se ejecuta sin más: sigue rigiendo `ADR-1108` (verificación de duplicados, no fabricar datos) y `ADR-1118` (una entidad, un único índice del mismo tipo — comprobar solapamiento antes de proponer). Simétricamente, **no toda fuente citada en una verificación entra en un índice**: solo las que además encajan en el `Scope` propio de ese índice (p. ej. "fuentes sobre estudios chinos" de `ADR-1113`) — una fuente usada puntualmente para contrastar un hecho, sin encajar en ningún `Scope` existente, se queda documentada en `Evidence and Sources` de la ficha, sin entrada de índice forzada. Precedente que motivó esta ampliación: en el mismo contraste se indexaron CXMT/ICBC/Zhu Yiming/Xinhua/SCMP por encajar en `Scope`, y se descartó explícitamente indexar CNBC/Bloomberg/TechTimes/MacRumors por no encajar — la misma verificación produjo altas y exclusiones, no todo lo verificado se indexa automáticamente en ninguna dirección.

---

## Nota de Ampliación (2026-08-27, segunda) — La convergencia entre resultados de `WebSearch` no sustituye a `WebFetch`

Al contrastar `conocimiento/260827-economics-xu-jiayin-evergrande-cadena-perpetua.md`, el asistente citó Global Times en `Evidence and Sources` basándose solo en que el resumen de `WebSearch` coincidía con otras 4 fuentes (France24, CNN, Fox Business, NPR) en los hechos centrales — sin abrir ninguna URL con `WebFetch`. Al responder después a una pregunta del usuario sobre esa misma fuente, se abrió por fin la URL real de Global Times y apareció un dato secundario incorrecto: la ficha decía "5 ejecutivos condenados, penas de 6-18 años"; la fuente real dice **56 personas relacionadas, penas de 1 año 10 meses a 18 años**. Ninguna de las otras 4 fuentes había sido abierta tampoco, así que su "coincidencia" con `WebSearch` no había servido de contraste real sobre ese dato secundario — probablemente todas heredan la misma síntesis de agencia, no una verificación independiente cada una.

Esto revela que el punto 2 de la Decision original, aunque no ambiguo en su letra ("abrir la URL de origen con WebFetch... antes de citar"), se estaba incumpliendo de facto mediante un atajo no contemplado explícitamente: tratar la convergencia de varios resultados de `WebSearch` sobre el mismo hecho como equivalente a haberlo verificado — un tipo de riesgo distinto del incidente original (una única fuente fabricada), pero un incumplimiento real de la misma regla.

Se añade como punto 7 de la Decision:

7. **La convergencia de varios resultados de `WebSearch` en el mismo hecho no exime de `WebFetch`.** Antes de citar un grupo de hechos relacionados (p. ej. "la condena, sus cargos y sus cifras"), abrir con `WebFetch` **al menos una** de las fuentes — preferiblemente la más autorizada o primaria disponible (agencia oficial, medio con acceso directo al hecho) — no solo confiar en que el resumen de `WebSearch` mencione varias fuentes coincidentes. No hace falta abrir todas las fuentes citadas, pero sí al menos una por cada bloque de hechos relacionados que se vaya a citar — un dato secundario dentro de ese bloque (como el número exacto de condenados en este caso) puede ser incorrecto en el resumen de `WebSearch` incluso cuando el hecho principal (la condena en sí) es correcto y está ampliamente corroborado.

---

## Context

El 2026-08-26, durante una sesión de investigación biográfica sobre Huang Xiaojie (fundadora de la Escuela de Chino Aihua de Madrid, ficha `conocimiento/260804-education-perfil-huang-xiaojie-escuela-aihua-madrid.md`), el asistente de IA usó `WebSearch` para localizar fuentes secundarias adicionales. El texto de respuesta generado automáticamente por `WebSearch` afirmó, citando un artículo de Magisnet (2018), que Huang Xiaojie era "licenciada en gestión de administración pública por la Universidad de Hangzhou, con más de 20 años de experiencia profesional en educación en China". El asistente reportó este dato al usuario como "hallazgo secundario", sin abrir la página real primero.

El usuario pidió explícitamente verificar el dato. Al hacerlo con `WebFetch` directamente sobre la URL del artículo, se descubrió que:

* El artículo **no menciona a Huang Xiaojie** — menciona a una "Ana Huang", identidad no confirmada como la misma persona.
* El artículo **no contiene ninguna afirmación** sobre universidad, titulación ni años de experiencia — ese contenido no existe en la página.

El resumen generado por `WebSearch` había fabricado o mezclado contenido de otra fuente y lo había presentado con la misma confianza que el resto de resultados de búsqueda genuinos. El asistente lo trasladó al usuario sin la comprobación adicional que sí había aplicado correctamente minutos antes, en la misma sesión, para otras dos fuentes (verificación del canal/programa CCTV-4 《华人世界》 y de la fuente Tencent News añadida a la misma ficha) — ambas confirmadas mediante `WebFetch` directo antes de darlas por buenas. La inconsistencia no estaba en el criterio general del proyecto (`22_USER_PREFERENCES.md` § Hallucination Handling ya exige no presentar información no verificada como hecho confirmado), sino en la ausencia de una regla operativa específica que convierta ese principio general en un paso obligatorio y mecánico para esta herramienta en concreto — sin ella, aplicarlo dependía de la disciplina puntual del asistente en cada caso, y falló una vez de dos.

---

## Decision

1. **El texto de respuesta autogenerado por `WebSearch` nunca es, por sí mismo, evidencia suficiente para una afirmación factual concreta** (dato biográfico, cifra, cita textual, fecha). Es un índice de resultados con un resumen orientativo, no una fuente primaria — puede fabricar, mezclar contenido entre resultados o atribuir incorrectamente una afirmación a una URL que no la contiene, como ocurrió en el caso que motiva este ADR.

2. **Antes de citar cualquier afirmación factual concreta obtenida vía `WebSearch`** — al incorporarla a una Knowledge Asset, o al reportarla al usuario como hallazgo, hecho o dato verificado — es obligatorio abrir la URL de origen con `WebFetch` (o lectura directa equivalente) y confirmar que la afirmación aparece efectivamente en el contenido real de esa página, con una cita textual o paráfrasis fiel de lo que dice el fichero/página, no del resumen de `WebSearch`.

3. **Excepción explícita**: no requiere esta verificación adicional el uso de `WebSearch` para tareas que no dependen de una afirmación factual puntual — p. ej., listar qué fuentes existen sobre un tema, orientar una búsqueda posterior, o identificar candidatas a investigar. La obligación se activa en el momento de **citar un hecho concreto**, no en el de buscar.

4. **Cuando `WebFetch` contradice o no confirma** lo que el resumen de `WebSearch` sugería (como en este caso): se reporta la discrepancia explícitamente al usuario — nunca se descarta en silencio ni se sustituye por una versión corregida sin explicar qué pasó —, y el dato no verificado no se incorpora a ninguna Knowledge Asset. Si el dato ya se había reportado como hallazgo antes de completar esta verificación, se retracta explícitamente en el siguiente turno, como ocurrió en la sesión que motiva este ADR.

5. **Aplica también a `agent-browser`** (`ADR-1115`) cuando esa herramienta produzca un resumen o extracción automática de una página, por el mismo principio: el resumen de la herramienta no sustituye la lectura del contenido real antes de citar un hecho concreto.

---

## Consequences

### Positive

* Cierra una vía de alucinación real y ya observada en la práctica, no hipotética — distinta de la alucinación "de conocimiento general" que el proyecto ya cubre (`22_USER_PREFERENCES.md`), porque aquí el asistente cita una fuente real y verificable que, al comprobarla, no dice lo que se afirmó.
* Da una regla mecánica y activable ("¿esto es un hecho concreto citado de `WebSearch`? → abrir la URL con `WebFetch` antes de reportarlo") en vez de depender de la disciplina puntual del asistente, mismo patrón de mitigación ya usado en `AI_ENTRYPOINT.md` para las verificaciones de `14_ADR_INDEX.md`.
* Refuerza, en vez de duplicar, el principio general ya existente en `22_USER_PREFERENCES.md` § Hallucination Handling — lo convierte en un paso operativo concreto para una herramienta específica.

### Negative

* Añade un paso adicional (una llamada a `WebFetch` por cada afirmación factual concreta) al flujo de investigación web, con el coste de tiempo/tokens correspondiente.
* No elimina el riesgo de que la propia página fuente contenga información falsa o desactualizada — solo garantiza que lo que se cita coincide con lo que dice la fuente, no que la fuente en sí sea correcta (ese es un riesgo distinto, ya cubierto por la evaluación de fiabilidad de `ADR-1116`).

### Risks Mitigated

* Incorporación a una Knowledge Asset de un dato biográfico o factual fabricado por la capa de resumen de `WebSearch`, atribuido a una fuente que en realidad no lo contiene.
* Reporte al usuario de un "hallazgo" que no resiste una verificación directa, socavando la confianza en hallazgos futuros genuinos.
* Aplicación inconsistente del principio general anti-alucinación ya existente, por falta de un punto de activación mecánico y específico por herramienta.

---

## References

* ai/governance/22_USER_PREFERENCES.md § Hallucination Handling, § Evidence Requirements (principio general que este ADR operacionaliza para `WebSearch`/`WebFetch`)
* docs/decisiones/ADR-1115-Adopcion_Agent_Browser_Web_Monitoring.md (herramienta hermana, punto 5 de esta Decision)
* docs/decisiones/ADR-1116-Metodologia_Evaluacion_Comparativa_Fuentes.md (precedente de formalizar en ADR una lección de rigor metodológico surgida de un incidente real)
* conocimiento/260804-education-perfil-huang-xiaojie-escuela-aihua-madrid.md (ficha que motivó el incidente — el dato erróneo no llegó a incorporarse a ella)
* docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md, docs/decisiones/ADR-1113-Generalizacion_Indices_Fuentes_Periodicas_a_Documentales.md, docs/decisiones/ADR-1118-No_Solapamiento_entre_Indices_de_Corpus_del_Mismo_Tipo_de_Entidad.md (punto 6 de esta Decision, añadido 2026-08-27)
* conocimiento/260823-technology-economics-cxmt-debuto-bolsa-shanghai.md, conocimiento_index/260806-index-society-organizaciones-instituciones.md, conocimiento_index/260825-index-society-personas-contemporaneas.md, conocimiento_index/260806-index-documentation-fuentes-curadas.md (caso real que motivó el punto 6: contraste que produjo tanto altas de índice — CXMT, ICBC, Zhu Yiming, Xinhua, SCMP — como exclusiones deliberadas — CNBC, Bloomberg, TechTimes, MacRumors)
