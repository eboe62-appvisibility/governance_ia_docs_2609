# ADR-1112: Prohibición de Siglas de Proyecto en el Contenido de Documentos del Repositorio

Status: APPROVED

Date: 2026-08-05

Scope:
Documentation

Category:
DOCUMENTATION

Domain: (Optional)
Documentation, Governance

Tags:
terminology, acronyms, style, documentation, knowledge-asset

Related ADRs:
NONE

Supersedes:
NONE

Superseded By:
NONE

---

## Context

El 2026-08-03, dos commits (`cbdb0d9`, `dee18c4`) sustituyeron las siglas "KA"/"KAs" y "RR" por "Knowledge Asset"/"Knowledge Assets" y "Relationship Record" en varios ADRs, con el mensaje "conforme a la convención establecida de no usar siglas en documentación de gobernanza". Esa convención, sin embargo, nunca se escribió en ningún documento del repositorio — se verificó explícitamente antes de este ADR (`grep` de "sigla"/"acrónimo" sin resultados en `ai/governance/`, `docs/decisiones/` y `docs/Project_Definition/`).

El 2026-08-05, durante la sesión de curación de la rama `oushinet`, el asistente de IA volvió a introducir la sigla "KA" en `Notes` de tres Knowledge Assets, al documentar la retirada de otras dos fichas del corpus. El usuario lo señaló como incumplimiento involuntario de una norma que, en ausencia de documentación explícita, dependía por completo de que el asistente la recordara sin ningún punto de referencia — el mismo patrón de fallo ya identificado en `AI_ENTRYPOINT.md` para el chequeo de verificación de ADRs (nota del 2026-08-02 sobre ADR-1106/1107/1108).

Se consideró documentar la regla en `AI_ENTRYPOINT.md` § "Language & Documentation", pero el usuario señaló que ese fichero (y, por el mismo motivo, todo `ai/governance/`, cuyos documentos se declaran explícitamente "General Purpose Use" en su primera línea) está pensado como marco reutilizable entre proyectos distintos del propio usuario — en otro proyecto podría preferirse el uso de siglas de gobernanza. La preferencia de no usar siglas es, por tanto, una decisión de **este** proyecto, no un principio general de interacción con IA, y corresponde documentarla en `docs/decisiones/` como ADR.

---

## Decision

1. **Regla**: las siglas que abrevian un término propio del proyecto (p. ej. "KA" por "Knowledge Asset", "RR" por "Relationship Record") no se usan en el contenido de ningún documento del repositorio destinado a persistir como conocimiento o gobernanza del proyecto — Knowledge Assets (`conocimiento/`), ADRs (`docs/decisiones/`), plantillas (`ai/templates/`), fuentes curadas (`fuentes/`), y cualquier otro documento versionado. Se escribe siempre el término completo.

2. **Excepción explícita — identificadores formales de documento**: esta regla no aplica a un identificador que referencia un documento concreto por su nombre propio, aunque ese nombre sea en sí mismo una sigla — p. ej. `ADR-1100`, `ADR-1109`. La diferencia con el caso prohibido: `ADR-1100` no sustituye a la locución "Architectural Decision Record número 1100" dentro de una frase (nadie escribe esa expansión en prosa), funciona como un identificador único, igual que un nombre de fichero. `KA` sí sustituía a un sustantivo común del proyecto ("Knowledge Asset") de forma intercambiable dentro de la prosa — ese es el uso prohibido.

3. **Ámbito**: aplica únicamente al contenido de documentos versionados. **No aplica** a la conversación entre el usuario y el asistente de IA (chat) — ahí las siglas técnicas de gobernanza (KA, ADR, etc.) son válidas y esperadas por practicidad, conforme a la petición explícita del usuario que motiva este ADR.

4. **Ubicación de esta regla**: `docs/decisiones/` (este ADR), no `ai/governance/` ni `AI_ENTRYPOINT.md` — ambos declarados explícitamente reutilizables entre proyectos del usuario (ver Context), mientras que esta preferencia es específica de este proyecto.

5. **Alcance retroactivo — sin barrido automático obligatorio**: este ADR no exige una corrección inmediata de todas las instancias preexistentes del corpus. Las instancias detectadas se corrigen de forma incremental, al tocar el fichero por otro motivo, o mediante un barrido explícito bajo petición del usuario. Instancia preexistente conocida y pendiente en el momento de este ADR: `conocimiento/260804-documentation-oushinet-260101-portada-ficha-tecnica.md`, línea 77 ("excluida de la fragmentación en KA").

---

## Consequences

### Positive

* Cierra un vacío de documentación que ya había causado dos incumplimientos detectados (2026-08-03, 2026-08-05) de una regla que solo existía como convención tácita.
* Distingue con precisión el caso prohibido (sigla sustituyendo a un sustantivo común en prosa) del caso permitido (identificador formal de documento tipo `ADR-XXXX`), evitando una interpretación excesivamente amplia que obligara a parafrasear cada cita de ADR.
* Queda en `docs/decisiones/`, coherente con que es una preferencia de este proyecto y no un principio a exportar a otros proyectos del usuario.

### Negative

* Sigue sin existir verificación automática (no hay CI en este repositorio) — el cumplimiento depende de la vigilancia del asistente de IA o del usuario en cada sesión, igual que otros chequeos ya documentados en `AI_ENTRYPOINT.md` § Self-Review Gate.
* No corrige de forma inmediata las instancias preexistentes del corpus (punto 5) — deuda documentada, no resuelta por este ADR.

### Risks Mitigated

* Reaparición futura de siglas de proyecto en el contenido de Knowledge Assets o ADRs, por falta de un lugar único donde comprobar la regla.
* Confusión entre "sigla prohibida en prosa" e "identificador de documento", que podría llevar a parafrasear innecesariamente referencias como `ADR-1100`.

---

## References

* AI_ENTRYPOINT.md § Self-Review Gate (precedente del mismo patrón de fallo: regla documentada que depende de recordarla, ya incumplida antes)
* Commits `cbdb0d9`, `dee18c4` (2026-08-03) — primera corrección reactiva, sin regla escrita
* conocimiento/260804-documentation-oushinet-260101-portada-ficha-tecnica.md (instancia preexistente pendiente, línea 77)
