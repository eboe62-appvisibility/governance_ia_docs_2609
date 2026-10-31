# ADR-0035 – Consumo de la gobernanza y la documentación IA compartidas

Status: APPROVED
Date: 2026-10-31
Scope: System
Category: GOVERNANCE
Tags: governance, ai, cross-project, sparse-checkout, adr-index, reproducibility
Related ADRs: ADR-0033, ADR-0034
Supersedes: NONE
Superseded By: NONE
Validation Reference: herramienta de configuración de cada proyecto (§4); §Verification de governance/14_ADR_INDEX.md

---

## Context

Varios proyectos (server_monitoring_2509, GDA y, más adelante, Chino) comparten dos repositorios:

* `governance_ia_2607`: la gobernanza IA (protocolos, skills, plantillas, índice de ADR).
* `governance_ia_docs_2609`: la documentación IA.

Hasta el 31/10/2026 cada proyecto resolvía por su cuenta cómo consumirlos:

* Qué partes se traen a local solo quedaba en la configuración de git de cada copia (`.git/info/sparse-checkout`), que no se versiona.
* Los puntos de entrada (`CLAUDE.md`, `AI_ENTRYPOINT.md`) cambiaban entre fichero propio y enlace simbólico, y algunos proyectos los versionaban en su propio repositorio.
* Las ADR se copiaban entre repositorios sin una referencia única. ADR-0033 acabó con versiones distintas en server_monitoring_2509 y en governance_ia_docs_2609 sin que nada avisara.

Este ADR es transversal: se aplica a todo proyecto que consuma estos repositorios.

---

## Decision

### 1. Tres capas

| Capa | Repositorio | Contenido | En el proyecto |
|---|---|---|---|
| Proyecto | El de cada proyecto | Código, infraestructura y configuración. **No contiene gobernanza IA ni documentación IA** | Raíz |
| Gobernanza IA | `governance_ia_2607` | Protocolos, skills, plantillas, índice integral de ADR (`governance/14_ADR_INDEX.md`) y particularidades de cada proyecto (`governance/projects/<proyecto>/`: `CLAUDE.md`, instrucciones de Copilot, etc.) | `ai/` |
| Documentación IA | `governance_ia_docs_2609` | Toda la documentación IA de todos los proyectos: `decisiones/` (todas las ADR, propias y transversales) y, por tipo, `<Tipo>/<proyecto>/` (`Project_Definition`, `Project_Implementation`, `arquitectura`, …) | `docs/` |

* Cada documento existe una sola vez, en su repositorio.
* El repositorio del proyecto ignora `ai/`, `docs/` y los puntos de entrada (o el proyecto no es un repositorio git).
* Los documentos de trabajo locales que no deben publicarse (StatusQuo con datos del servidor, plan de acción, `Project_ADRs/`, `Project_Corrections/`, copias, `*_BORRAR*`) quedan ignorados por el `.gitignore` de `governance_ia_docs_2609`, con rutas relativas a su raíz.

### 2. ADR

1. **Conjunto completo.** Como `docs/` es un clon de `governance_ia_docs_2609`, cada entorno tiene todas las ADR de todos los proyectos en `docs/decisiones/`.
2. **Alcance.** Solo son vinculantes las ADR transversales y las del proyecto actual (sección Cross-project y sección del proyecto en `14_ADR_INDEX.md`). Las demás son de consulta y nunca prevalecen sobre las del proyecto.
3. **Propiedad.** Cada ADR la modifica solo el entorno al que pertenece, según su rango: 00xx server_monitoring_2509, 06xx y 08xx GDA, 11xx Knowledge/Chino. Una ADR transversal la comparten todos, pero cada entorno modifica solo su propia sección.
4. **Transversales.** Una ADR transversal es un único fichero. Las diferencias entre entornos van en secciones por entorno dentro de ella, nunca en variantes con el mismo número.
5. **Cambios.** Crear o modificar una ADR incluye, en el mismo bloque de trabajo:
   * Rama corta y PR en `governance_ia_docs_2609`.
   * Rama corta y PR en `governance_ia_2607` que actualice `governance/14_ADR_INDEX.md`.
   * Ejecutar §Verification desde la raíz del proyecto.

### 3. Reglas comunes de consumo

1. **Ubicación.** `ai/` y `docs/` son clones independientes en la raíz del proyecto.
2. **Rama.** Los dos repositorios se siguen en `develop` y se actualizan solo con `git pull --ff-only`.
3. **Checkout parcial.** Cada proyecto puede excluir partes que no use. Requisitos:
   * Nunca se excluyen `governance/`, `skills/` ni `templates/` de `governance_ia_2607`, ni `decisiones/` de `governance_ia_docs_2609`.
   * Los patrones y su motivo se documentan en el perfil del proyecto (§4).
4. **Puntos de entrada.** Su contenido vive en `governance/projects/<proyecto>/` (o, transitoriamente, donde indique el perfil). En la raíz del proyecto solo hay enlaces o punteros, no versionados, creados por la herramienta de configuración. El punto de entrada común es `governance/00_AI_ENTRYPOINT.md`.
5. **Finales de línea.** LF obligatorio en los dos repositorios, impuesto por `.gitattributes` (`* text=auto eol=lf`; binarios marcados), según `Configuracion_Equipo_2608`. Se recomienda `core.autocrlf=false`.
6. **Reproducibilidad.** Cada proyecto tiene una herramienta idempotente (target de make, script) que clona `ai/` y `docs/` si no existen, aplica su checkout parcial y crea los puntos de entrada. Volver a ejecutarla no cambia nada.
7. **Cambios en los repositorios compartidos.** Nunca se edita directamente en `develop`:
   * Antes de empezar, `git pull --ff-only`.
   * Rama corta `<tipo>/<aammdd>-<tema>` desde `develop` y PR a `develop`.
   * Promoción `develop → master` mediante PR ("Create a merge commit"). Después, fast-forward de `develop` a `master`.

### 4. Perfiles de proyecto

Añadir un proyecto consiste en añadir su columna; no requiere un ADR nuevo.

| | server_monitoring_2509 (`monitoring`) | GDA |
|---|---|---|
| `docs/` | Clon completo de `governance_ia_docs_2609` | Clon completo de `governance_ia_docs_2609` |
| Carpetas propias en `docs/` | `<Tipo>/monitoring/` | `Project_Definition/` ⚠️ pendiente de mover a `Project_Definition/GDA/` |
| Checkout parcial de `ai/` (modo) | no-cone | cone |
| Patrones de `ai/` | `/*`, `!/claude_code/`, `!/opencode/`, `!/governance/00_CLAUDE.md` | `/*`, `!/*/`, `/governance/`, `/skills/`, `/templates/` (los ficheros de la raíz sí entran) |
| Motivo de las exclusiones | `claude_code/` y `opencode/` se conservan solo en el remoto; `00_CLAUDE.md` es el de GDA | `claude_code/` (Docker) y `opencode/` no aplican: GDA se desarrolla en Windows sin Docker |
| `CLAUDE.md` | Symlink no versionado a `ai/governance/projects/monitoring/CLAUDE.md` | Fichero sin versionar en la raíz del workspace (no es repositorio git) con `@ai/governance/00_CLAUDE.md` ⚠️ pendiente de mover a `governance/projects/GDA/CLAUDE.md` |
| Punto de entrada | `AI_ENTRYPOINT.md` → symlink no versionado a `ai/governance/00_AI_ENTRYPOINT.md` | Sin fichero en la raíz: `00_CLAUDE.md` remite a `ai/governance/00_AI_ENTRYPOINT.md` |
| Copilot | `.github/copilot-instructions.md` → symlink no versionado a `ai/governance/projects/monitoring/copilot-instructions.md` | No existe `.github/` ⚠️ pendiente de decidir |
| Otros | `.claudecodeignore` y `.ua` (grafo de Understand-Anything, en `docs/arquitectura/monitoring/.ua`) → symlinks no versionados | — |
| Herramienta de configuración | `make ai-setup` | ⚠️ pendiente: script de Git Bash (`make` no está disponible) |
| `core.autocrlf` | `false` (WSL) | `true` solo en la configuración de sistema de Git for Windows; afecta a `ai/` y `docs/` ⚠️ pendiente de alinear |
| Necesita `ai/` y `docs/` en PRO | No: el servidor no los usa; `make audit` solo avisa si falta `docs/decisiones/` | No: el despliegue no los usa |

---

## Consequences

### Positive

* Cada documento existe una sola vez: no puede volver a divergir como ADR-0033.
* El repositorio de cada proyecto solo contiene el proyecto.
* Cada entorno ve todas las decisiones de los demás, aunque solo aplique las suyas.
* Reglas comunes para todos los proyectos. Lo que es propio de cada proyecto queda escrito en su perfil.
* Un clon nuevo reproduce el entorno con un solo comando.

### Negative

* Un cambio de ADR requiere dos PR (documentación e índice).
* Desde la web de GitHub, los enlaces del repositorio de un proyecto a `docs/…` no resuelven: la documentación está en otro repositorio.
* Las funciones de Copilot en github.com no ven las instrucciones del proyecto, que ya no están en su repositorio.
* Los repositorios compartidos no quedan fijados a un commit concreto (no son submódulos): cada proyecto avanza cuando decide.

### Risks Mitigated

* Divergencia de ADR y documentos copiados entre repositorios.
* Publicación accidental de documentos de trabajo locales con datos sensibles.
* Pérdida de la configuración de checkout parcial al reclonar.
* Sobrescritura del `CLAUDE.md` de un proyecto con el de otro.

---

## References

* governance/14_ADR_INDEX.md (§ADR Registry, §Verification)
* `.gitattributes` y `.gitignore` de `governance_ia_2607` y `governance_ia_docs_2609`
* server_monitoring_2509: Makefile, target `ai-setup`
* ADR-0033 (taxonomía de ADR), ADR-0034 (política de ramas de server_monitoring_2509; mismo modelo de ramas)
* Configuracion_Equipo_2608
