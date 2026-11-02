# ADR-2101: Tratamiento de Datos Personales Sensibles (PII) en Knowledge Assets de Gestión

Status: APPROVED

Date: 2026-09-23

Scope:
Documentation

Category:
SECURITY

Domain: (Optional)
Education, Governance

Tags:
security, secrets, gestion-academica, knowledge-asset, adr

Related ADRs:
ADR-2100 (mismo dominio Gestión — establece la ubicación de estos Knowledge Assets); ADR-1100 (plantilla y estándar de evidencia que esta ADR no modifica)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

Varias fuentes administrativas oficiales de la UAM que el usuario aporta como fuente primaria para Knowledge Assets de Gestión (Impresos de Matrícula de Grado, resguardos de matrícula de microcredenciales) contienen datos personales sensibles: DNI/NIF completo, número de cuenta bancaria (IBAN) parcial, además de identificadores administrativos de menor sensibilidad (NIA, NIP, correo electrónico institucional del propio usuario).

El repositorio es un repositorio Git versionado sin garantía de que el remoto vaya a permanecer siempre privado.

Sin una política explícita, cada nueva fuente con PII exige decidir de nuevo, caso a caso, si redactar el dato y si excluir el fichero de git — riesgo real de inconsistencia o de commitear un DNI real por descuido. El criterio ya se ha aplicado dos veces en la práctica, siempre confirmado explícitamente con el usuario antes de proceder:

* `260718-education-impreso-matricula-2026-27.md` — DNI e IBAN parcial omitidos del texto; fichero excluido de git.
* `260916-education-impreso-matricula-micro-presencia-online.md` — DNI omitido del texto; fichero excluido de git.

Esta ADR formaliza ese criterio ya aplicado, en vez de dejarlo como una decisión implícita repetida ad hoc.

---

## Decision

1. Cuando la fuente primaria de un Knowledge Asset de Gestión contenga PII de categoría alta — DNI/NIF, número de cuenta bancario completo o parcial, o cualquier otro identificador que la normativa de protección de datos considere dato personal directo —, el Knowledge Asset resultante:
   a. **Omite literalmente esos valores** de su texto; referencia su existencia y ubicación en el PDF/documento de origen sin transcribirlos.
   b. **Se excluye de git** añadiendo su ruta exacta a `.gitignore`, bajo la sección "Datos personales propios sensibles (DNI, IBAN)".
   c. Incluye una nota explícita en su sección `Notes` señalando la exclusión y el motivo, y en `Prohibited Actions` la prohibición expresa de commitear el fichero o de transcribir el dato omitido en cualquier otro documento del repositorio.

2. Identificadores de menor sensibilidad y de uso administrativo interno (NIA, NIP, correo institucional del propio usuario) **no** activan por sí solos la regla 1 — solo el DNI/NIF y los datos bancarios lo hacen.

3. Antes de aplicar por primera vez la exclusión de git sobre un fichero nuevo, se **confirma explícitamente con el usuario** (nunca se asume en silencio) — mismo criterio ya seguido en los dos precedentes citados en el Context.

4. La regla de `.gitignore` se aplica **por ruta exacta de fichero**, nunca por un patrón glob amplio (p. ej. `*matricula*.md`), para no excluir por error futuras fichas de Gestión que no contengan PII.

---

## Consequences

### Positive

* Política explícita y repetible: no hace falta reconsiderar el criterio cada vez que aparece un nuevo Impreso de Matrícula o documento administrativo similar.
* Reduce el riesgo de exponer DNI/IBAN del usuario en el remoto del repositorio.

### Negative

* Introduce fichas de Gestión que existen en el árbol de trabajo pero nunca aparecen en `git status`/`git log` — quien audite el repositorio debe saber que `.gitignore` puede ocultar contenido legítimo, no solo ruido de sistema.
* Mantenimiento manual: cada nuevo fichero con PII exige una línea nueva en `.gitignore`, sin automatización.

### Risks Mitigated

* Exposición accidental de DNI/IBAN en un commit.
* Inconsistencia entre fichas de Gestión sobre qué se considera "dato sensible".

---

## References

* .gitignore (sección "Datos personales propios sensibles (DNI, IBAN)")
* 260718-education-impreso-matricula-2026-27.md
* 260916-education-impreso-matricula-micro-presencia-online.md
* ADR-2100-Ubicacion_Knowledge_Assets_Gestion_Academica.md
