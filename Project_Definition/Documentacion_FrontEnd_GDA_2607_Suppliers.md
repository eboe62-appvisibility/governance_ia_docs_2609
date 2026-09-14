# Documentación FrontEnd GDA — Módulo Suppliers
Versión: 2607 (2026-07-16, revisada 2026-07-17, revisada 2026-07-20, revisada 2026-07-21, revisada
2026-07-22)
Basado en: `GuardiasIneco.vue` (patrón de pantalla) e `iniciativaForm.vue` (campos de fecha) · Backend
real: entidad `Proveedor` en **gda-persona** (no `gda-admingade`/`gda-finance` — corrección mayor
aplicada el 2026-07-16) · Estado: Fase 1 (listado), Fase 2 (Añadir), Fase 3 (Consultar), Fase 4
(Editar) y Fase 5 (Eliminar) completadas 2026-07-17; backend CUD completo desde 2026-07-21; Fase 6
(retirada de tooltips provisionales) completada 2026-07-21; Fase 7 (campos de auditoría Fecha Efecto/
Fecha Baja en formulario y listado) completada 2026-07-22; Fase 8 (Fecha Efecto → Fecha Alta editable
en Añadir, Fecha Modificación en el listado) completada 2026-07-22
(`Documentacion_BackEnd_GDA_2607_Suppliers.md` §15, `Documentacion_BBDD_GDA_2607_Suppliers.md` §8) —
**módulo Suppliers funcional de extremo a extremo**; pendiente solo la mejora de UX opcional del
desplegable de Encargo (sección 7)

==========================================
1. INTRODUCCIÓN GENERAL
==========================================

Este documento consolida el análisis realizado para la construcción del nuevo módulo **Suppliers**
dentro de `GDA/frontend/src/modules/gestionAdmingade/`, tomando como referencia directa la pantalla
ya existente `guardias/GuardiasIneco.vue`. El objetivo es reutilizar el mismo patrón arquitectónico
(vista orquestadora + composables de servicio + modelos de transformación + componentes UI
compartidos) para minimizar el riesgo técnico y mantener la coherencia con el resto del ecosistema
GDA descrito en `Documentacion_FrontEnd_GDA_2511.txt`.

Este documento no sustituye al análisis técnico detallado, que vive junto a este (movido desde
`docs/decisiones/` a `.vscode/docs/` el 2026-07-16 para mantener todo el análisis de Suppliers en un
único sitio):

- `knowledge-graph-suppliers.json` — grafo de nodos/edges/capas/flujos de datos.
- `REQUISITOS_MODULO_SUPPLIERS.md` — listado de archivos a crear/modificar/reutilizar.
- `MAPA_VISUAL_SUPPLIERS.md` — diagrama ASCII por capas.
- `RESUMEN_EJECUTIVO_SUPPLIERS.md` — estadísticas, checklist de fases y consideraciones críticas.

Su función es servir de **puerta de entrada** (entrypoint) legible junto al resto de
`Documentacion_*_GDA_*` de `.vscode/docs/`, y dejar constancia expresa de la verificación de
cumplimiento frente a los ADRs vigentes en `docs/decisiones/` (los ADRs permanecen en
`docs/decisiones/`; solo se movieron los documentos de análisis de Suppliers).

==========================================
2. ARQUITECTURA PROPUESTA (RESUMEN)
==========================================

```
Presentación        Suppliers.vue → suppliersList.vue / supplierForm.vue → crudHeader.vue (SHR)      [NUEVO]
                          │
UI Kit global (SHR)  gda-dialog / gda-dialog-ok-cancel / gda-dialog-informative /
                     gda-loading / gda-table / gda-select / gda-input / gda-form
                          │
Composables          useCrud.js (SHR) · useFetchSuppliers / useSaveSupplier /
                     useUpdateSupplier / useDeleteSupplier (NUEVOS)
                          │
Modelos              supplier.js → modelSupplier() / modelBackSupplier() (NUEVO)
                          │
Servicios            admingadeServices.{get,post,put,delete}Supplier (MODIFICAR admingade.js)
                          │
API Gateway          ProveedorController → ProveedorServiceImpl (YA EXISTE, solo GET —
(composición, ADR-0604)  agrega gda-persona + encargoVigente de gda-admingade — AMPLIAR con CUD)
                          │
Backend gda-persona     ProveedorController → ProveedorService → ProveedorRepository →
                        Proveedor (entity)  — TODO YA EXISTE, solo GET — AMPLIAR con CUD
Backend gda-persona-commons   ProveedorDto (DTO de intercambio, ADR-0601 — YA EXISTE)
                          │
Base de datos        tabla `proveedor`, schema `gda_persona` — YA EXISTE
```

(SHR) = compartido/reutilizado sin cambios. (NUEVO) = a crear. (YA EXISTE) = backend real, solo lectura.
(AMPLIAR) = añadir create/update/delete a código ya existente.

**Corrección mayor (2026-07-16):** el backend de Suppliers NO es un dominio nuevo en `gda-admingade`.
Es la entidad `Proveedor`, ya existente en `gda-persona`, con lectura completa (`Controller`,
`Service`, `Repository`, `Entity`, `Dto` en `-commons`) y con su propia capa de composición ya
funcionando en el API Gateway (agrega `gda-persona` con `encargoVigente` de `gda-admingade`, ADR-0604).
El trabajo de backend es ampliar 5 archivos existentes con create/update/delete, no crear un backend
desde cero.

**Flujos CRUD** (ver `knowledge-graph-suppliers.json` → `dataFlow` para el detalle capa por capa):
Read ya funciona end-to-end (backend); Create/Update/Delete no existen en ninguna capa backend y
hay que construirlos completos, siguiendo la secuencia Vista → Composable → Servicio Axios →
API Gateway (composición) → Backend gda-persona → Base de datos, con transformación de datos en
ambos sentidos vía `models/supplier.js`.

==========================================
3. ESTRUCTURA DE ARCHIVOS (RESUMEN)
==========================================

**A crear (nuevos, solo frontend):**
`Suppliers.vue`, `suppliersList.vue`, `supplierForm.vue`, `useFetchSuppliers.js`, `useSaveSupplier.js`,
`useUpdateSupplier.js`, `useDeleteSupplier.js`, `models/supplier.js`.

**A ampliar (backend, ya existen — añadir create/update/delete):**
- `gda-persona`: `ProveedorController.java`, `ProveedorService.java`/`ProveedorServiceImpl.java`
  (el `Repository` y la `Entity` no requieren cambio, `save()`/`deleteById()` ya los provee `JpaRepository`).
- API Gateway: `ProveedorController.java`, `ProveedorServiceImpl.java` (paquete `.persona`),
  `GdaPersonaApiClient.java` (añadir métodos Feign de escritura).
- `ProveedorDto.java` (`gda-persona-commons`) probablemente no requiere cambios, salvo que el
  formulario necesite campos nuevos.

**A modificar (frontend, existentes):**
`services/admingade.js`, `constants/endpoints/*`, `constants/common.js` (verificar),
`models/index.js`, `i18n` (es/en), `router/index.js`, `app/routes.js` /
`app/configItemsMenuBreadcrumbs.js` / `app/menu.js`.

**Reutilizables sin cambio:** `useCrud.js`, `useForm.js`, `crudHeader.vue`, `formFooter.vue`,
componentes `gda-*`, `utils/utils.js`, `utils/validations.js`, `utils/notifications.js`,
`utils/axiosCall.js`.

El detalle completo, con rutas exactas y prioridad de cada archivo, está en
`REQUISITOS_MODULO_SUPPLIERS.md` (secciones 1 a 4, mismo directorio).

==========================================
4. INTEGRACIÓN CON BACKEND / API GATEWAY
==========================================

Siguiendo el patrón ya usado por `guards/*`, todas las llamadas de Suppliers se canalizan a través
del API Gateway existente, sin acceso directo del frontend a ningún microservicio:

```
Suppliers.vue → useFetchSuppliers/useSaveSupplier/useUpdateSupplier/useDeleteSupplier
            → admingadeServices.{get,post,put,delete}Supplier()
            → axiosCall.js (Axios + interceptor JWT)
            → API Gateway: ProveedorController → ProveedorServiceImpl (YA EXISTE para GET)
            → gda-persona: ProveedorController → ProveedorService → ProveedorRepository
            → tabla `proveedor` (PostgreSQL, schema gda_persona) — YA EXISTE
```

El manejo de errores se centraliza en `utils/notifications.js` (`notificationService.error(...)`)
y `utils/utils.js` (`cError(...)`), igual que en `useFetchGuards.js`.

> **Hallazgo resuelto (auditoría 2026-07-16, verificado en `GDA/api-gateway` y `GDA/backend/gda-persona`):**
> la laguna de diseño señalada en la revisión anterior (¿necesita Suppliers una capa de composición
> propia en el Gateway, como Guardias?) queda **resuelta con evidencia real, no con una decisión de
> diseño**: el API Gateway **ya tiene** su propia capa de composición para `Proveedor`
> (`apigateway.web.controller.persona.ProveedorController` + `apigateway.service.persona.impl.
> ProveedorServiceImpl`), que agrega datos de `gda-persona` (Proveedor) con `gda-admingade`
> (`encargoVigente`, vía `GdaAdmingadeApiClient`) para devolver solo los proveedores del encargo
> vigente — exactamente el mismo patrón que `GuardiaServiceImpl` (ADR-0604), y una segunda
> confirmación real del mismo. Existe además una ruta de paso simple para `/admingade/proveedores/**`
> ya escrita pero **comentada** en `RouteLocatorConfig.java`, superada por esta capa de composición;
> no debe reactivarse. Lo único que falta añadir a esta capa (Gateway + gda-persona) es
> create/update/delete — hoy solo existe lectura en ambos niveles.
>
> **Decisión aún abierta:** a diferencia de Guardias (que tiene `GuardiaRequest`/`GuardiaResponse`
> propios del Gateway), el `ProveedorController` del Gateway hoy reutiliza `ProveedorDto`
> directamente como entrada/salida. Al añadir create/update/delete hay que decidir si se mantiene
> esa reutilización directa o se introducen `ProveedorRequest`/`ProveedorResponse` propios del
> Gateway, siguiendo el patrón más elaborado de Guardias.

==========================================
5. VALIDACIONES DE FORMULARIO SEGÚN EL ESQUEMA REAL DE BD (responsabilidad frontend)
==========================================

**Alcance:** el backend (`gda-persona`) ya está construido y no se toca — nuestra responsabilidad
es el frontend. Verificado el 2026-07-16 contra la DDL real de `gda_persona.proveedor` (aportada
por el usuario), hay 2 discrepancias entre la entidad JPA (`Proveedor.java`) y la tabla real que
`supplierForm.vue` debe respetar, en vez de confiar en los límites (potencialmente erróneos) de la
entidad Java:

| Campo | Entidad JPA (`Proveedor.java`) | DDL real (`gda_persona.proveedor`) | Validación que debe implementar `supplierForm.vue` |
|---|---|---|---|
| `observaciones` | `length = 250` | `varchar(200)` | **maxlength 200**, no 250 — la entidad tiene un límite incorrecto que no se va a corregir |
| `encargoId` | nullable (sin `nullable = false`) | `NOT NULL DEFAULT 0` | **campo obligatorio** en el formulario; el frontend debe enviar siempre un valor, no confiar en que la ausencia de valor caiga al `DEFAULT 0` de la BD |
| `codigo` | sin `unique` en la entidad | `UNIQUE (codigo)` + `UNIQUE (codigo, nombre)` | no existe endpoint de verificación de disponibilidad; `supplierForm.vue`/`useSaveSupplier.js`/`useUpdateSupplier.js` deben capturar el error 400/409 de la respuesta y mostrarlo con `notificationService.error(...)`, igual que el patrón de Guardias con `DataIntegrityViolationException` |

No se corrige la entidad ni la BD (fuera de nuestro alcance); estas notas existen para que el
formulario nuevo no reproduzca el mismo desajuste del lado del frontend, y para que el manejo de
errores de los composables esté preparado desde el primer día para un 400 por violación de
unicidad. Ver también [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) §Deuda técnica,
donde se documenta además una FK real cruzando `gda_persona` → `gda_admingade` (`encargo_id`),
sin relevancia directa para el frontend pero registrada por completitud.

==========================================
6. VERIFICACIÓN DE CUMPLIMIENTO CON LOS ADR (docs/decisiones)
==========================================

Se ha revisado el análisis de Suppliers (este documento + `knowledge-graph-suppliers.json` +
`REQUISITOS_MODULO_SUPPLIERS.md`) frente a los 8 ADRs vigentes en `docs/decisiones/` (se añade
ADR-0604 desde la revisión anterior). Resultado, **actualizado tras confirmar que el backend real
es `gda-persona`, no `gda-admingade`**:

| ADR | Estado ADR | Requisito clave | ¿Cumple el análisis de Suppliers? |
|-----|-----------|------------------|----------------------------------|
| [ADR-0600](../../docs/decisiones/ADR-0600-estrategia-despliegue.md) — Microservicios independientes (Maven multi-module) | Approved | Cada módulo backend es un artefacto independiente; prohibido un `pom.xml` padre que fuerce compilación en bloque | ✅ **Cumple** — Suppliers se añade dentro del módulo ya existente `gda-persona` (no `gda-admingade`; no se crea un módulo Maven nuevo ni un padre reactivo). |
| [ADR-0601](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md) — Patrón módulo-commons | Approved | DTOs, clientes de comunicación, excepciones y constantes van en `-commons`; entidades/repos/services/controllers van en el módulo principal | ✅ **Cumple, y ya existe:** `ProveedorDto.java` ya está en `gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/`, y `Proveedor.java`/`ProveedorRepository`/`ProveedorService`/`ProveedorController` ya están en el módulo principal `gda-persona` — el mismo patrón que `GuardiaDto.java` en `gda-admingade-commons`, verificado en código real, no solo planeado. |
| [ADR-0602](../../docs/decisiones/ADR-0602-comunicacion-interservicios.md) — Contratos REST/JSON + OpenAPI + rutas planas | Approved (revisado 2026-07-16) | Exponer API REST/JSON; generación base de OpenAPI vía `springdoc`; rutas planas sin versionar (práctica real, versionado solo si un cambio futuro lo exige) | ✅ **Cumple, y ya existe:** `GET admingade/proveedores` (entre otros prefijos) ya es REST/JSON y plano, igual que `GuardiaController` (`/guardias`). Falta añadir POST/PUT/DELETE con el mismo estilo. Documentación OpenAPI enriquecida y versionado quedan como mejora incremental opcional (ADR-0602 §Deuda técnica). |
| [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) — BD por microservicio (schema dedicado en instancia compartida) | Approved (revisado 2026-07-16) | Schema propio por servicio en la instancia PostgreSQL compartida; sin joins cruzados; `ddl-auto: none` + gestión de esquema fuera del ciclo de vida de la app (Flyway/Liquibase es mejora futura, no requisito actual) | ✅ **Cumple, y ya existe:** la tabla `proveedor` ya reside en el schema `gda_persona` (mismo patrón que `gda_admingade` para `Guardia`), sin joins cruzados. Formalizar con Flyway/Liquibase sigue como mejora incremental (ADR-0603 §Deuda técnica), igual que para el resto del backend. |
| [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md) — Composición del API Gateway (BFF) | Approved | Usar capa de composición dedicada cuando el recurso necesita datos de otro microservicio; ruta de paso simple si es autocontenido | ✅ **Cumple, y ya existe:** `Proveedor` **sí** necesita datos de otro microservicio (`encargoVigente` de `gda-admingade`), y el Gateway **ya** implementa la capa de composición dedicada (`ProveedorController`/`ProveedorServiceImpl` en `apigateway.*.persona`) — es la segunda confirmación real de este ADR, además de Guardias. |
| [ADR-0801](../../docs/decisiones/ADR-0801-estrategia-empaquetado-despliegue.md) — Empaquetado como JAR ejecutable vía CI Maven | Approved (revisado 2026-07-16) | JAR Spring Boot ejecutable, release/versionado por GitLab CI; sin Dockerfile por módulo (contenerización es mejora futura, no requisito actual) | ✅ **Cumple** — Al no crearse un módulo Maven nuevo (Suppliers vive dentro de `gda-persona`), hereda su mismo empaquetado JAR y su mismo pipeline de release; no aplica Dockerfile, igual que el resto del backend. |
| [ADR-0802](../../docs/decisiones/ADR-0802-integracion-front-back.md) — Integración frontend/backend | Proposed | Todo vía API Gateway, CORS en el gateway, manejo de errores centralizado, cliente HTTP estándar (Axios) | ✅ **Cumple** — Suppliers reutiliza `admingadeServices` + `axiosCall.js` (Axios) + `notifications.js`, exactamente el mismo patrón que `GuardiasIneco.vue`; no se plantea ninguna llamada directa a ningún microservicio. |
| [ADR-0803](../../docs/decisiones/ADR-0803-seguridad-secretos.md) — Seguridad y gestión de secretos | Proposed | Auth JWT stateless, secretos en `.env`/vault, cifrado de datos sensibles | ✅ **Cumple** — se reutiliza el interceptor JWT existente de `axiosCall.js`, sin introducir secretos nuevos. Confirmado (2026-07-16): `Proveedor` (Supplier) **no almacena datos sensibles** (son datos de contacto de empresa, no de persona física), por lo que no aplica cifrado en reposo. |

**Conclusión de la verificación:** el análisis de Suppliers es **compatible en su diseño** con los
8 ADRs vigentes y no contradice ninguno de ellos. Los ocho se cumplen sin reservas, y a diferencia
de la revisión anterior, ahora **la mayoría se cumplen con evidencia de código real** (ADR-0600,
ADR-0601, ADR-0602, ADR-0603, ADR-0604), no solo con un diseño planeado — porque el backend de
Suppliers ya existe parcialmente (solo lectura) en `gda-persona`.

> Nota: tras la revisión de 2026-07-16, ADR-0602, ADR-0603 y ADR-0801 ya no exigen versionado de
> API, migraciones Flyway/Liquibase ni Dockerfile como requisito — quedaron documentados como
> "Deuda técnica / mejoras pendientes" dentro de cada ADR, alineados con lo que realmente hace
> Guards y ahora también Proveedores hoy en el backend GDA. Por eso Suppliers los cumple igual que
> Guards, sin necesidad de implementar esas mejoras antes que el resto del backend.

==========================================
7. ACCIONES PENDIENTES ANTES DE IMPLEMENTAR
==========================================

> Nota (2026-07-17): los puntos 1 y 2 quedaron resueltos antes de empezar la implementación. El
> resto (3, 5 y 6) sigue vigente sin cambios — pertenecen al backend CUD y al formulario, que son
> objeto de la Fase 2 (ver sección 10). La Fase 1 (listado, solo lectura) no ha tocado el backend.

1. ~~Separar `SupplierDTO.java` hacia `gda-admingade-commons` (ADR-0601).~~ **Ya no aplica tal cual:**
   el backend real es `gda-persona`, y `ProveedorDto.java` ya está correctamente en
   `gda-persona-commons` desde antes de este análisis — no hay nada que mover.
2. ~~Confirmar con Producto/Seguridad si `Supplier` almacenará datos sensibles (ADR-0803).~~ Resuelto
   2026-07-16: `Proveedor` (Supplier) no almacena datos sensibles; no aplica cifrado en reposo.
3. Resolver las preguntas abiertas ya listadas en `RESUMEN_EJECUTIVO_SUPPLIERS.md` §"Consideraciones
   críticas" (relaciones de entidad, operaciones masivas, validaciones, permisos para create/update/delete).
4. ~~Evaluar si Suppliers necesita una capa de composición propia en el API Gateway...~~ **Resuelto
   con evidencia real (2026-07-16):** sí la necesita, y ya existe — `ProveedorController`/
   `ProveedorServiceImpl` en el Gateway ya agregan `gda-persona` con `encargoVigente` de
   `gda-admingade` (ADR-0604). Solo falta ampliarla con create/update/delete.
5. Añadir POST/PUT/DELETE en 4 puntos concretos (ver `REQUISITOS_MODULO_SUPPLIERS.md` §4):
   `ProveedorController`/`ProveedorService` en `gda-persona`, y `ProveedorController`/
   `ProveedorServiceImpl` en el API Gateway, más el método Feign correspondiente en
   `GdaPersonaApiClient`.
6. Decidir si el Gateway sigue reutilizando `ProveedorDto` directamente para create/update/delete,
   o introduce `ProveedorRequest`/`ProveedorResponse` propios (como Guardias).

**Mejoras opcionales (backlog, no bloquean la implementación — ver "Deuda técnica" en cada ADR):**
- Versionado de ruta y anotaciones OpenAPI/Swagger en el controller (ADR-0602) — igual de
  pendiente en Guards y en Proveedores.
- Migración formal con Flyway/Liquibase en vez de cambios manuales de esquema (ADR-0603) — igual
  de pendiente en todo el backend.
- **Detectado durante la verificación end-to-end de la Fase 3 del backend (2026-07-21):** el
  desplegable "Encargo" de `supplierForm.vue` (alimentado por `useFetchJobs()`) muestra **todos** los
  encargos existentes, no solo el `encargoVigente` — pero el listado de `suppliersList.vue` solo
  muestra proveedores cuyo `encargoId` coincide con el vigente (filtro aplicado en el Gateway, ver
  `Documentacion_BackEnd_GDA_2607_Suppliers.md` §2). Es fácil seleccionar un encargo no vigente sin
  darse cuenta: el alta funciona (backend) pero el proveedor no aparece en el listado, ni siquiera
  tras recargar. No es un bug bloqueante — es el comportamiento ya previsto (filtrado por
  `encargoVigente`) — pero valdría la pena, como mejora de UX futura, restringir el desplegable al
  encargo vigente por defecto, o avisar si se selecciona uno distinto.

==========================================
8. PROCEDIMIENTO DE IMPLEMENTACIÓN POR FASE
==========================================

Acordado el 2026-07-20 en el documento hermano (`Documentacion_BackEnd_GDA_2607_Suppliers.md` §9, paso
6 añadido el mismo día), trasladado aquí adaptado al contexto frontend. **No aplica
retroactivamente** a las Fases 1 a 5 (secciones 9 a 13): esas ya estaban completadas el 2026-07-17,
antes de formalizar este procedimiento. Rige para cualquier trabajo frontend nuevo sobre Suppliers a
partir de esa fecha (por ejemplo, retirar los avisos de "backend pendiente" de
`supplierForm.vue`/`Suppliers.vue` una vez la Fase 2 del backend esté operativa).

Cada fase sigue el mismo procedimiento de seis pasos, en este orden:

1. **Corregir/ampliar el código** de la fase correspondiente.
2. **Detalle de archivos afectados**, indicando la capa/carpeta de cada uno dentro de
   `GDA/frontend/src/` (`views`, `components`, `composables/service`, `models`, `services`, `i18n`,
   `router`/`app`...), entregado como resumen antes de continuar.
3. **Validación** por parte del usuario.
4. **Subida al repositorio GitLab**, en la rama correspondiente, solo si procede tras la validación.
   **La ejecuta el usuario manualmente** (`git add`/`commit`/`push`) — no se ejecuta ningún comando
   git desde esta conversación. `GDA/frontend` tiene su propio repositorio git independiente
   (`http://172.16.22.114/gdact-reir/gdact-3.0/frontend.git`), distinto del de cada microservicio
   backend, aunque comparta rama de trabajo con ellos para el mismo ticket (`GDAV3-443` en el momento
   de escribir esto).
5. **Documentación de la fase completada**, añadiendo su sección correspondiente en este documento,
   siguiendo el mismo formato que las Fases 1 a 5 (secciones 9 a 13).
6. **Revisión y/o ampliación de ADRs** (`docs/decisiones/`, plantilla `adr_template.md`, índice
   obligatorio `ai/governance/14_ADR_INDEX.md`) en base a la experiencia adquirida durante la
   implementación de la fase que se cierra — en el contexto frontend, principalmente los ADR-08xx
   (ADR-0801 a ADR-0803: empaquetado/despliegue, integración frontend-backend, seguridad), sin excluir
   los ADR-06xx de backend si el cambio frontend revela algo sobre el contrato con el Gateway. Primer
   caso real detectado desde el lado backend: la decisión de permisos de Suppliers dio lugar a
   ADR-0605 (modelo de autorización RBAC del Gateway) — ver
   `Documentacion_BackEnd_GDA_2607_Suppliers.md` §11.

No se avanza al paso siguiente sin completar el anterior; en particular, no se documenta una fase
como completada (paso 5) sin que el usuario confirme que el paso 4 ya se ha realizado, y la fase no
se da por cerrada del todo hasta completar también el paso 6.

==========================================
9. ESTADO REAL DE IMPLEMENTACIÓN — FASE 1 (LISTADO, SOLO LECTURA) (2026-07-17)
==========================================

**Resumen:** se ha implementado la pantalla de listado de Suppliers en modo solo lectura (búsqueda +
tabla), siguiendo el patrón arquitectónico previsto en las secciones 2 y 3. El formulario de
alta/edición, la columna ACCIONES y el backend CUD (sección 7) quedan para la Fase 2.

**Prerrequisito aplicado — liberar la etiqueta "Proveedores":** la pantalla ya existente
`Guardias / Intervenciones > Proveedores` se renombró a **"Proveedores-Guardias"** (menú,
breadcrumb y título de pantalla) para poder reutilizar la clave i18n `suppliers` ("Proveedores") en
este módulo nuevo sin colisión de nombres. Se añadió la clave `suppliersGuards` en
`i18n/es-ES/common.js` y se actualizó el título de `MANAGEMENT_ADMINGADE_GUARDS_SUPPLIERS` en
`configItemsMenuBreadcrumbs.js` para usarla.

**Archivos nuevos (frontend):**

| Archivo | Ruta | Estado |
|---|---|---|
| `supplier.js` (modelo) | `GDA/frontend/src/models/supplier.js` | ✅ Creado — solo `modelSupplier()` (transformación de lectura); `modelBackSupplier()` queda pendiente de Fase 2 |
| `useFetchSuppliers.js` | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useFetchSuppliers.js` | ✅ Creado |
| `suppliersList.vue` | `GDA/frontend/src/modules/gestionAdmingade/components/suppliers/suppliersList.vue` | ✅ Creado — 6 columnas, sin columna ACCIONES |
| `Suppliers.vue` | `GDA/frontend/src/modules/gestionAdmingade/views/Suppliers.vue` | ✅ Creado — buscador (`crud-header`) + tabla + loading, sin diálogos de alta/edición |
| `supplierForm.vue` | — | ⏳ Pendiente (Fase 2) |
| `useSaveSupplier.js` / `useUpdateSupplier.js` / `useDeleteSupplier.js` | — | ⏳ Pendientes (Fase 2) |

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| `services/admingade.js` | `+ getSuppliers()`, reutiliza `PATH_SERVICES_ADMIN_GADE.SUPPLIERS` (endpoint `admingade/proveedores`, ya existente y funcional) — no hizo falta crear una constante de endpoint nueva, confirmando lo ya apuntado en la sección 4 |
| `models/index.js` | `+ export modelSupplier` |
| `app/routes.js` | `+ MANAGEMENT_ADMINGADE_SUPPLIERS` → `/administracionGade/proveedores` → `Suppliers.vue` |
| `app/configItemsMenuBreadcrumbs.js` | `+ CONFIG_VIEWS.MANAGEMENT_ADMINGADE_SUPPLIERS` (título = `i18n("suppliers")`) |
| `app/menu.js` | `+` entrada de menú **nivel 1** dentro de "Gestión Admingade", ubicada entre `Personas ETT` y `Consultas` (decisión de ubicación tomada en esta fase, no recogida antes en este documento) |
| `i18n/es-ES/common.js` | corregido bug preexistente: la clave `phone` estaba duplicada (`"Telefono"` sin tilde pisaba a `"Teléfono"`) — corrección global de la app, no solo de Suppliers |

**Columnas del listado implementadas** (decisión tomada durante esta fase, no estaba en el
documento original): CÓDIGO (`codigo`), EMPRESA (`nombre`), RESPONSABLE (`personaContacto`), EMAIL
(`emailContacto`), TELÉFONO (`telefonoContacto`), OBSERVACIONES (`observaciones`). El campo
`encargoId` queda **fuera del listado** (decisión explícita: el listado ya viene filtrado por
`encargoVigente` desde el Gateway, así que mostrar el ID no aporta valor; se usará únicamente en el
formulario de alta/edición, donde es obligatorio).

**Pendiente para Fase 2:** columna ACCIONES (Editar / Consultar / Eliminar, replicando el patrón de
`iniciativasList.vue`), `supplierForm.vue`, composables de escritura, `modelBackSupplier()`, y la
ampliación backend CUD ya descrita en la sección 7 (sin cambios ahí, sigue vigente).

==========================================
10. ESTADO REAL DE IMPLEMENTACIÓN — FASE 2 (BOTÓN AÑADIR + FORMULARIO) (2026-07-17)
==========================================

**Resumen:** se implementó el botón "Añadir Proveedor" y su formulario de alta, en modo **solo
frontend** — el `POST` apunta al endpoint real (`admingade/proveedores`) pero el backend aún no lo
soporta (confirmado antes de construirlo), por lo que el guardado fallará hasta que se implemente
la sección 7.

**Archivos nuevos:**

| Archivo | Ruta | Estado |
|---|---|---|
| `useSaveSupplier.js` | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useSaveSupplier.js` | ✅ Creado (patrón `useSaveInitiative.js`) |
| `supplierForm.vue` | `GDA/frontend/src/modules/gestionAdmingade/components/suppliers/supplierForm.vue` | ✅ Creado — solo modo `ADD` (el validador del prop `type` aún no admite `EDIT`/`DETAIL`) |

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| `models/supplier.js` | `+ modelBackSupplier()` |
| `models/index.js` | `+ export modelBackSupplier` |
| `services/admingade.js` | `+ postSupplier()` (reutiliza el endpoint `SUPPLIERS`, igual que `getSuppliers`) |
| `views/Suppliers.vue` | `+` botón "Añadir Proveedor" (`crud-header` con botón visible), diálogo con `supplierForm`, diálogo informativo de éxito, `saveFormSupplier()` |
| `modules/common/components/formFooter.vue` | `+` prop opcional `tooltipSave` (compartido por todos los formularios de la app, sin efecto salvo que se informe explícitamente) |
| `i18n/es-ES/common.js` | `+ supplierSaveNotAvailable` (texto del tooltip de aviso) |

**Decisiones de diseño tomadas en esta fase:**
- Título en singular ("Añadir Proveedor", no "Añadir Proveedores"): `useCrud({ title: "supplier", ... })`, mismo patrón que `PerfilLicitacionPedido.vue`.
- Campo **Encargo** (`encargoId`): `gda-select` obligatorio, alimentado por `useFetchJobs()` (composable ya existente, reutilizado sin cambios) — se guarda el objeto completo y se extrae `.id` al construir el payload.
- Campos obligatorios: Código, Empresa, Encargo. Responsable/Email/Teléfono/Observaciones quedan opcionales (datos de contacto).
- `Observaciones`: `maxlength 200` (límite real de BD, distinto del límite de la entidad JPA — ver sección 5).
- Botón GUARDAR **no deshabilitado**, pero con tooltip explicando que el backend aún no soporta la operación.

**Pendiente:** columna ACCIONES (Editar/Consultar/Eliminar), `useUpdateSupplier.js`/`useDeleteSupplier.js`, ampliar `supplierForm.vue` a `EDIT`/`DETAIL`, y el backend CUD (sección 7, sin cambios).

==========================================
11. ESTADO REAL DE IMPLEMENTACIÓN — FASE 3 (COLUMNA ACCIONES — BOTÓN CONSULTAR) (2026-07-17)
==========================================

**Resumen:** se añadió la columna ACCIONES al listado con el botón **Consultar** (ver detalle),
plenamente funcional — a diferencia de Añadir/Editar/Eliminar, no depende de ningún endpoint nuevo:
solo muestra en modo lectura los datos ya obtenidos por el `GET` existente. Editar y Eliminar quedan
fuera de esta fase (mismo bloqueo de backend que Añadir).

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| `suppliersList.vue` | `+` columna ACCIONES; se ocultan los 3 botones nativos de `gda-table` (`show-btn-edit/detail/delete="false"`) y se añade manualmente el botón Consultar (icono `visibility`) vía slot `#buttons-append`, emitiendo `show-detail` — mismo patrón que `iniciativasList.vue` |
| `supplierForm.vue` | El prop `type` pasa a admitir `TYPE_FORM.ADD` y `TYPE_FORM.DETAIL` (antes solo `ADD`); todos los campos reciben `:readonly="isFormTypeDetail"`; nuevo `onAccept()` para el botón "Aceptar" que `form-footer` muestra en modo detalle; el campo Encargo se hidrata buscando en `optionsJobs` el que coincide con el `chargeId` de la fila (la lista no trae el objeto Encargo completo, solo su ID) |
| `Suppliers.vue` | `+ onShowDetail()`, `+ generateModelForm()` (aplana la fila de la tabla al modelo del formulario), wiring `@show-detail` / `@accept` |

**Decisión de diseño:** de las tres acciones (Editar/Consultar/Eliminar), se priorizó Consultar por ser
la única que no requiere backend nuevo (Editar necesita `PUT`, Eliminar necesita `DELETE`, ninguno
existe todavía — sección 7).

**Pendiente:** botones Editar y Eliminar (con el mismo aviso de "backend pendiente" ya usado en
Añadir), `useUpdateSupplier.js`/`useDeleteSupplier.js`, ampliar `supplierForm.vue` a `EDIT`, y el backend
CUD (sección 7, sin cambios).

==========================================
12. ESTADO REAL DE IMPLEMENTACIÓN — FASE 4 (BOTÓN EDITAR) (2026-07-17)
==========================================

**Resumen:** se añadió el botón Editar (nativo de `gda-table`) al listado. El diálogo se abre
precargado con los datos de la fila (incluido el Encargo, resuelto por `chargeId` contra
`useFetchJobs`). El `PUT` apunta al endpoint real, pero el backend aún no lo implementa (mismo
aviso por tooltip que Añadir) — sigue sin ser funcional de extremo a extremo.

**Archivos nuevos:**

| Archivo | Ruta |
|---|---|
| `useUpdateSupplier.js` | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useUpdateSupplier.js` (patrón `useSaveSupplier.js`) |

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| `models/supplier.js` | `modelBackSupplier()` incluye ahora `id` cuando está presente (necesario para el `PUT`) |
| `services/admingade.js` | `+ putSupplier()` |
| `suppliersList.vue` | Se habilita el botón nativo Editar de `gda-table`, emite `edit` |
| `supplierForm.vue` | `type` admite `EDIT`; el Encargo se hidrata también en edición; LIMPIAR en modo edición restaura los valores originales campo a campo desde `_model`, replicando exactamente el patrón de `iniciativaForm.vue` (no un `Object.assign`/clonado masivo) |
| `Suppliers.vue` | `+ onEdit()`, `+ generateModelSaveEdit()`, `saveFormSupplier()` bifurca entre añadir/editar |

**Nota de depuración relevante:** el botón LIMPIAR tuvo dos correcciones antes de quedar bien:
1. Cerraba el diálogo (por un `@clear="closeDialogForm"` erróneo en `Suppliers.vue`, no presente en
   ninguna otra pantalla) — corregido eliminando ese listener.
2. El problema reportado después ("LIMPIAR vacía el formulario") resultó ser una recarga de
   navegador obsoleta (HMR), no un fallo de código — confirmado reproduciendo la lógica de forma
   aislada con el motor de reactividad real del proyecto.

**Pendiente:** botón Eliminar, y el backend CUD (sección 7, sin cambios).

==========================================
13. ESTADO REAL DE IMPLEMENTACIÓN — FASE 5 (BOTÓN ELIMINAR) (2026-07-17)
==========================================

**Resumen:** se añadió el botón Eliminar con diálogo de confirmación (`gda-dialog-ok-cancel`),
replicando exactamente el patrón de `Iniciativa.vue`. El `DELETE` apunta al endpoint real, pero el
backend aún no lo implementa — el botón "Aceptar" del diálogo de confirmación muestra el mismo tipo
de aviso por tooltip que Añadir/Editar.

**Archivos nuevos:**

| Archivo | Ruta |
|---|---|
| `useDeleteSupplier.js` | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useDeleteSupplier.js` (patrón `useDeleteGuard.js`) |

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| `dialogOkCancel.vue` | Nuevo prop opcional `tooltipAccept` (mismo patrón que `tooltipSave` de `formFooter.vue`), sin efecto en el resto de pantallas que usan `gda-dialog-ok-cancel` |
| `services/admingade.js` | `+ deleteSupplier(id)` |
| `i18n/es-ES/common.js` | `+ supplierDeleteNotAvailable` |
| `suppliersList.vue` | Columna ACCIONES completa: solo Editar queda como botón nativo de `gda-table`; Consultar y Eliminar se añaden manualmente en el slot `#buttons-append`, en ese orden — replicando `iniciativasList.vue` para conseguir el orden Editar → Consultar → Eliminar (habilitar el botón nativo de eliminar lo dejaba en tercera posición, detrás de Consultar; se corrigió) |
| `Suppliers.vue` | `+` diálogo `gda-dialog-ok-cancel` (título "Eliminar Proveedor", mensaje con el nombre de la empresa), `+ onDeleteSupplier()`; `@delete` de la lista va directo a `showDialogOkCancelDelete` (de `useCrud`), sin pasar por `generateModelForm` |

**Pendiente:** el backend CUD completo (sección 7, sin cambios) — con esto quedan implementados en
frontend los tres botones de ACCIONES (Editar/Consultar/Eliminar) y el botón Añadir.

==========================================
14. ESTADO REAL DE IMPLEMENTACIÓN — FASE 6 (RETIRADA DE TOOLTIPS PROVISIONALES) (2026-07-21)
==========================================

**Resumen:** con el backend CUD de Suppliers ya completo y verificado de extremo a extremo
(`Documentacion_BackEnd_GDA_2607_Suppliers.md` §13), se retiraron los avisos por tooltip de "backend
aún no disponible" en Guardar y en Eliminar, introducidos en las Fases 2 y 5 respectivamente. Sigue
pendiente, deliberadamente sin aplicar en esta fase, la mejora de UX ya anotada en la sección 7 sobre
el desplegable de Encargo (no restringido al vigente).

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| `supplierForm.vue` | Retirado `:tooltip-save="$t('supplierSaveNotAvailable')"` de `form-footer` |
| `Suppliers.vue` | Retirado `:tooltip-accept="$t('supplierDeleteNotAvailable')"` de `gda-dialog-ok-cancel` |
| `i18n/es-ES/common.js` | Eliminadas las claves `supplierSaveNotAvailable` y `supplierDeleteNotAvailable`, ya sin ningún uso (verificado con búsqueda global antes de borrarlas) |

**No tocados (deliberado):** `formFooter.vue` (prop `tooltipSave`) y `dialogOkCancel.vue` (prop
`tooltipAccept`) — se dejan tal cual, son infraestructura genérica y opcional compartida por toda la
app (sin efecto si no se informa), introducida en las Fases 2 y 5; no hay motivo para retirarla solo
porque Suppliers ya no la necesite, podría servir a otra pantalla en el futuro.

**Verificación:** `npx eslint` sobre los 3 archivos modificados, sin errores ni warnings. Confirmado
además por búsqueda global que **ninguna otra pantalla de la app** (incluida `GuardiasIneco.vue` y
las ~17 vistas que usan `gda-dialog-ok-cancel`) pasaba `tooltip-save`/`tooltip-accept` — tras esta
fase, Suppliers queda con exactamente el mismo comportamiento por defecto (sin tooltip) que Guardias y
el resto, no una ausencia anómala.

**Paso 4 — GitLab:** commit y push a `GDA/frontend` (rama `GDAV3-443`) realizados por el usuario con
éxito.

**Paso 6 — Revisión de ADRs:** evaluado explícitamente — es una retirada de UI provisional sin ningún
patrón arquitectónico nuevo ni decisión de sistema implicada; no requiere ADR nuevo ni corrección de
los existentes.

**Fase 6 completada — procedimiento de la sección 8 completado de principio a fin (pasos 1 a 6). Con
esto, el módulo Suppliers (frontend + backend) queda funcional de extremo a extremo, sin pendientes de
código.**

**Pendiente (backlog, no bloqueante):** mejora de UX del desplegable de Encargo (sección 7) —
pospuesta a petición explícita, no forma parte de esta fase.

==========================================
15. ESTADO REAL DE IMPLEMENTACIÓN — FASE 7 (CAMPOS DE AUDITORÍA: FECHA EFECTO / FECHA BAJA) (2026-07-22)
==========================================

**Resumen:** con el backend ampliado (`Documentacion_BackEnd_GDA_2607_Suppliers.md` §14) con los campos
`persona_id`/`fecha_alta`/`fecha_baja`/`fecha_efecto`, se incorporó al frontend el tratamiento de
`fecha_efecto`/`fecha_baja`, replicando explícitamente el criterio de `iniciativaForm.vue`
(`persona_id` y `fecha_alta` son puramente de backend, sin ningún campo ni columna en el frontend —
igual que `username` no aparece en el formulario de Iniciativa).

**Decisiones tomadas (confirmadas por el usuario):**
1. **Fecha Efecto** (`fechaEfecto`): campo obligatorio del formulario Añadir/Editar (deshabilitado solo
   en Consultar), vía `gda-input-date`. No se prellena por código con la fecha de hoy — el propio
   componente `gda-input-date` ya posiciona el calendario en el día actual al abrir un campo vacío
   ([inputDate.vue:178-182](../../GDA/frontend/src/components/inputDate.vue#L178-L182)), igual que en
   `iniciativaForm.vue`, que tampoco prellena `dateEffect` por código.
2. **Fecha Baja** (`fechaBaja`): visible solo en Editar/Consultar (oculto en Añadir), siempre
   deshabilitado (`readonly`). A diferencia de `dateLow` en `iniciativaForm.vue` (editable cuando el
   `estado` es Cerrada/Detenida — mecanismo que Proveedor no tiene), en Proveedor es **intocable desde
   la app en cualquier modo**: "la baja de un Proveedor solo podrá ser gestionada directamente en BBDD"
   (decisión explícita del usuario). El backend refuerza esto preservando siempre el valor original en
   `editProveedor` (ver `Documentacion_BackEnd_GDA_2607_Suppliers.md` §14).
3. Se mantiene el **DELETE físico real** del botón Eliminar (`deleteById`), igual que
   `IniciativaController.deleteIniciativa` — descartada la idea inicial de borrado lógico vía
   `fecha_baja`, al no existir precedente real de ese patrón en ningún módulo del proyecto.
4. Dos columnas nuevas en `suppliersList.vue`: **FECHA EFECTO** y **FECHA BAJA**, colocadas entre
   TELÉFONO y OBSERVACIONES.

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| [supplier.js](../../GDA/frontend/src/models/supplier.js) | `modelSupplier`: `+ dateEffect`/`dateLow` vía `parseDate(item.fechaEfecto)`/`parseDate(item.fechaBaja)`. `modelBackSupplier`: `+ fechaEfecto: item.dateEffect` — **nunca** `fechaBaja` (punto 2) |
| [supplierForm.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/suppliers/supplierForm.vue) | `+ gda-input-date` para `dateEffect` (obligatorio, editable salvo en Consultar); `+ gda-input-date` para `dateLow` (`v-if="!isFormTypeAdd"`, siempre `readonly`); `MODEL_FORM_EMPTY`/`FIELDS_THAT_CAN_CHANGE`/`onClear` actualizados para `dateEffect`; expuesto `isFormTypeAdd` del composable `useForm` |
| [suppliersList.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/suppliers/suppliersList.vue) | `+` columnas `dateEffect`/`dateLow` (`i18n("dateEffect")`/`i18n("dateLow")`, ya existentes — reutilizadas de Iniciativa, sin claves i18n nuevas), con `format: formatDate` — mismo patrón que `iniciativasList.vue` |
| [Suppliers.vue](../../GDA/frontend/src/modules/gestionAdmingade/views/Suppliers.vue) | `generateModelForm`: `+ dateEffect`/`dateLow` (para Editar/Consultar). `generateModelSaveAdd`: `+ dateEffect` (nunca `dateLow`, heredado también por `generateModelSaveEdit` vía spread) |

**No tocados:** `i18n/es-ES/common.js` (las claves `dateEffect`/`dateLow` ya existían, creadas para
Iniciativa — no hizo falta ninguna clave nueva); `useSaveSupplier.js`/`useUpdateSupplier.js`/
`useDeleteSupplier.js`/`useFetchSuppliers.js` (sin cambios, la transformación ya vive en `supplier.js`).

**Verificación:** `npx eslint` sobre los 4 archivos modificados, sin errores ni warnings. Probado de
extremo a extremo por el usuario tras corregir dos bugs reales de backend detectados durante esta
verificación (serialización `LocalDate` en Feign y desajuste de versión de `gda-persona-commons` —
ambos documentados en `Documentacion_BackEnd_GDA_2607_Suppliers.md` §14, Hallazgos 1 y 2): Añadir con
Fecha Efecto y Editar/Consultar mostrando las fechas correctamente.

**Paso 6 — Revisión de ADRs (2026-07-22):** ver `Documentacion_BackEnd_GDA_2607_Suppliers.md` §14 y el
nuevo [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md).

**Fase 7 completada.** Con esto, el módulo Suppliers queda con campos de auditoría completos en las tres
capas (BD, backend, frontend), sin pendientes de código salvo la mejora de UX ya anotada en la sección 7.

==========================================
16. ESTADO REAL DE IMPLEMENTACIÓN — FASE 8 (FECHA EFECTO → FECHA ALTA, FECHA MODIFICACIÓN) (2026-07-22)
==========================================

**Resumen:** corrección del criterio de la Fase 7, a raíz de un cambio de esquema en BD
(`Documentacion_BBDD_GDA_2607_Suppliers.md` §8): la columna `fecha_efecto` se renombra a
`fecha_modificacion` e invierte su naturaleza (de campo de negocio editable a campo 100% de auditoría,
backend). El campo "Fecha Efecto" del formulario se sustituye por **"Fecha Alta"**, que pasa a ocupar
su lugar visualmente pero con un comportamiento distinto por modo:

1. **Añadir:** "Fecha Alta" editable, prellenada con la fecha del día (`moment().format(FORMAT_MASK_DATE)`
   al abrir el formulario, no solo confiando en el comportamiento del selector de `gda-input-date` como
   en la Fase 7 — aquí sí hace falta el prellenado explícito porque el valor debe estar presente incluso
   si el usuario no interactúa con el campo).
2. **Editar:** "Fecha Alta" deshabilitada y sin marca de obligatorio (el backend ignora cualquier valor
   entrante y preserva el original, sin cambios respecto a la Fase 4).
3. **Consultar:** igual que Editar — deshabilitada, sin obligatoriedad.
4. Columna del listado "FECHA EFECTO" → **"FECHA MODIFICACIÓN"** (misma posición, entre TELÉFONO y
   OBSERVACIONES), ahora alimentada por `fecha_modificacion` (auto, backend) en vez de por el campo de
   negocio que antes rellenaba el usuario.

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| [supplier.js](../../GDA/frontend/src/models/supplier.js) | `modelSupplier`: `+ dateHigh` (de `fechaAlta`); `dateEffect` → `dateModification` (de `fechaModificacion`). `modelBackSupplier`: `fechaEfecto` → `fechaAlta: item.dateHigh` |
| [supplierForm.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/suppliers/supplierForm.vue) | Campo `dateEffect` → `dateHigh`, label `$t('dateHigh')` ("Fecha alta", clave i18n ya existente); `:readonly="!isFormTypeAdd"`, `:required="isFormTypeAdd"`; `+` prellenado en `_initialize()` cuando `isFormTypeAdd` y el campo está vacío; `MODEL_FORM_EMPTY`/`FIELDS_THAT_CAN_CHANGE`/`onClear` actualizados |
| [suppliersList.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/suppliers/suppliersList.vue) | Columna `dateEffect` → `dateModification`, label `$t('dateModification')` |
| [Suppliers.vue](../../GDA/frontend/src/modules/gestionAdmingade/views/Suppliers.vue) | `generateModelForm`/`generateModelSaveAdd`: `dateEffect` → `dateHigh` |
| [i18n/es-ES/common.js](../../GDA/frontend/src/i18n/es-ES/common.js) | `+ dateModification: "Fecha modificación"` (clave nueva; `dateHigh`/`dateLow` ya existían, reutilizadas de Iniciativa) |

**No tocados:** `useSaveSupplier.js`/`useUpdateSupplier.js`/`useDeleteSupplier.js`/`useFetchSuppliers.js` (sin
cambios, la transformación vive en `supplier.js`); la clave i18n `dateEffect` (sigue en uso por
`iniciativaForm.vue`, no se toca aunque Suppliers deje de usarla).

**Verificación:** `npx eslint` sobre los 5 archivos modificados, sin errores ni warnings.

**Paso 6 — Revisión de ADR:** [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md)
reescrito: eleva el patrón de 4 columnas de auditoría de "recomendado" a **obligatorio para toda tabla
nueva** del backend GDA.

**Fase 8 completada.**

==========================================
17. REFERENCIAS
==========================================

- Vista de referencia (patrón de pantalla): [GuardiasIneco.vue](../../GDA/frontend/src/modules/gestionAdmingade/views/guardias/GuardiasIneco.vue)
- Patrón de campos de fecha (Iniciativa): [iniciativaForm.vue](../../GDA/frontend/src/modules/common/components/iniciativas/iniciativaForm.vue), [initiative.js](../../GDA/frontend/src/models/initiative.js), [inputDate.vue](../../GDA/frontend/src/components/inputDate.vue)
- Esquema de BD y criterio de campos de auditoría: [Documentacion_BBDD_GDA_2607_Suppliers.md](Documentacion_BBDD_GDA_2607_Suppliers.md)
- Backend real de Suppliers: [ProveedorController.java (gda-persona)](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/ProveedorController.java), [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java)
- Composición en el Gateway: [ProveedorController.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/persona/ProveedorController.java), [ProveedorServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/impl/ProveedorServiceImpl.java)
- [knowledge-graph-suppliers.json](knowledge-graph-suppliers.json)
- [REQUISITOS_MODULO_SUPPLIERS.md](REQUISITOS_MODULO_SUPPLIERS.md)
- [MAPA_VISUAL_SUPPLIERS.md](MAPA_VISUAL_SUPPLIERS.md)
- [RESUMEN_EJECUTIVO_SUPPLIERS.md](RESUMEN_EJECUTIVO_SUPPLIERS.md)
- ADRs verificados: [ADR-0600](../../docs/decisiones/ADR-0600-estrategia-despliegue.md),
  [ADR-0601](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md),
  [ADR-0602](../../docs/decisiones/ADR-0602-comunicacion-interservicios.md),
  [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md),
  [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md),
  [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) (nuevo, Fase 7),
  [ADR-0801](../../docs/decisiones/ADR-0801-estrategia-empaquetado-despliegue.md),
  [ADR-0802](../../docs/decisiones/ADR-0802-integracion-front-back.md),
  [ADR-0803](../../docs/decisiones/ADR-0803-seguridad-secretos.md)
- Documentación general del frontend: [Documentacion_FrontEnd_GDA_2511.txt](Documentacion_FrontEnd_GDA_2511.txt)
