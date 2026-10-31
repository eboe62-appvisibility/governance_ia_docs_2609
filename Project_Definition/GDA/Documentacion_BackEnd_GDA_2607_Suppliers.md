# Documentación BackEnd GDA — Módulo Suppliers
Versión: 2607 (2026-07-20, revisada 2026-07-20, revisada 2026-07-21, revisada 2026-07-22) · Basado en:
`Documentacion_FrontEnd_GDA_2607_Suppliers.md` (mismo módulo, lado frontend) y
`Documentacion_BBDD_GDA_2607_Suppliers.md` (esquema de BD y criterio de campos de auditoría) · Backend
real: entidad `Proveedor` en **gda-persona** (no `gda-admingade`/`gda-finance`) · Estado: **backend CUD
de Suppliers completo, con campos de auditoría** — Fase 0 (planificación), Fase 1 (`gda-persona`), Fase 2
(API Gateway), Fase 3 (verificación end-to-end, incluida corrección de mensaje de error 2026-07-21) y
Fase 4 (campos de auditoría `persona_id`/`fecha_alta`/`fecha_baja`/`fecha_efecto`, 2026-07-22) y Fase 5
(renombrado `fecha_efecto` → `fecha_modificacion`, con inversión de semántica, 2026-07-22)
completadas; pendiente subida a GitLab (gestionada manualmente por el usuario)

==========================================
1. INTRODUCCIÓN GENERAL
==========================================

Este documento recoge la evolución de la ampliación **backend** del módulo **Suppliers**: añadir
create/update/delete a la entidad `Proveedor`, ya existente y funcional en modo solo lectura en
`gda-persona` y en su capa de composición del API Gateway. Es el documento complementario de
`Documentacion_FrontEnd_GDA_2607_Suppliers.md` (que cubre la parte ya construida en
`GDA/frontend/src/modules/gestionAdmingade/`) y sigue su mismo formato y convención de versionado
para mantener coherencia entre ambos lados del módulo.

El frontend está terminado desde el 2026-07-17 (Fases 1 a 5: listado, añadir, consultar, editar,
eliminar) pero **no es funcional de extremo a extremo**: los botones Añadir/Editar/Eliminar llaman a
endpoints reales (`POST`/`PUT`/`DELETE` sobre `admingade/proveedores`) que hoy solo responden `GET`
en las cuatro capas backend involucradas. Este documento traza cómo cerrar ese hueco.

Documentos relacionados (mismo directorio `.vscode/docs/`):

- `Documentacion_FrontEnd_GDA_2607_Suppliers.md` — puerta de entrada del lado frontend; su sección 5
  ("Validaciones de formulario según el esquema real de BD") y su sección 7 ("Acciones pendientes
  antes de implementar") son la base directa de este documento.
- `knowledge-graph-suppliers.json`, `REQUISITOS_MODULO_SUPPLIERS.md`, `MAPA_VISUAL_SUPPLIERS.md`,
  `RESUMEN_EJECUTIVO_SUPPLIERS.md` — análisis técnico compartido por ambos lados del módulo.
- ADRs vigentes en `docs/decisiones/` (0600 a 0803) — ver sección 6.

==========================================
2. ARQUITECTURA PROPUESTA (RESUMEN)
==========================================

```
Backend gda-persona     ProveedorController → ProveedorService → ProveedorRepository →
                        Proveedor (entity)  — GET ya existe; AMPLIAR con addProveedor/
                        editProveedor/deleteProveedor (Repository y Entity NO requieren cambio,
                        save()/deleteById() ya los da JpaRepository)
                          │
Backend gda-persona-commons   ProveedorDto (DTO de intercambio interno, ADR-0601) · excepciones ya
                        existentes reutilizables: ObjectAlreadyExistException, ObjectNotFoundException
                          │
API Gateway (composición, ADR-0604)
                        ProveedorController → ProveedorServiceImpl (paquete .persona) — GET ya
                        existe; AMPLIAR con POST/PUT/DELETE + GdaPersonaApiClient (métodos Feign de
                        escritura, hoy inexistentes)
                          │
Contrato Gateway        ProveedorDto (reutilizado hoy solo para lectura) → decisión tomada: introducir
                        ProveedorRequest propio del Gateway para create/update (ver sección 4)
                          │
Base de datos           tabla `proveedor`, schema `gda_persona` — YA EXISTE, sin cambios de esquema
```

(YA EXISTE) = backend real, solo lectura, verificado en código. (AMPLIAR) = añadir create/update/delete
a código ya existente. No se crea ningún microservicio, módulo Maven ni tabla nueva.

**Patrón de referencia verificado en código real:** `Guardia` (`gda-admingade` + su capa de
composición en el Gateway) ya implementa el mismo flujo completo de escritura que necesita
`Proveedor`, incluyendo manejo de `ObjectAlreadyExistException`/`ObjectNotFoundException` y de
`DataIntegrityViolationException` → 400 en el controller
([GuardiaController.java (backend)](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/web/controller/GuardiaController.java)),
y un `GuardiaRequest` propio en el Gateway para desacoplar el contrato de entrada del DTO interno
([GuardiaServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/GuardiaServiceImpl.java)).

==========================================
3. ESTRUCTURA DE ARCHIVOS (RESUMEN)
==========================================

**A ampliar (ya existen, verificado en código — añadir create/update/delete):**

| Archivo | Ruta | Cambio |
|---|---|---|
| `ProveedorService.java` / `ProveedorServiceImpl.java` (gda-persona) | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/` | `+ addProveedor()`, `+ editProveedor()`, `+ deleteProveedor()` |
| `ProveedorController.java` (gda-persona) | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/` | `+ @PostMapping`, `+ @PutMapping`, `+ @DeleteMapping` sobre `/proveedores` |
| `ProveedorService.java` / `ProveedorServiceImpl.java` (Gateway) | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/` | `+` métodos reactivos equivalentes (`Mono<...>`) |
| `ProveedorController.java` (Gateway) | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/persona/` | `+ @PostMapping`, `+ @PutMapping`, `+ @DeleteMapping` sobre las mismas rutas prefijadas del `GET` |
| `GdaPersonaApiClient.java` | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/clients/` | `+` métodos Feign `addProveedor`/`editProveedor`/`deleteProveedor` |

**A crear (nuevo, backend, decisión de diseño ya tomada — ver sección 4):**

| Archivo | Ruta prevista | Motivo |
|---|---|---|
| `ProveedorRequest.java` | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/persona/request/` | Contrato de entrada propio del Gateway para create/update, con validación Bean Validation (`@NotNull` en `encargoId`, `@Size(max = 200)` en `observaciones`) sin tocar el `ProveedorDto` compartido — ver sección 5 |

**No requieren cambio (verificado en código):**
`ProveedorRepository` (gda-persona) — `save()`/`deleteById()` ya los provee `JpaRepository`.
`Proveedor.java` (entidad JPA) y `ProveedorDto.java` (`gda-persona-commons`) — fuera de alcance, ver
sección 5. `ProveedorDto` sigue siendo el contrato interno gateway↔gda-persona (vía Feign); solo se
añade un `ProveedorRequest` como capa de entrada adicional en el Gateway, no un reemplazo.

**Frontend:** ya completo, sin cambios previstos (ver `Documentacion_FrontEnd_GDA_2607_Suppliers.md`,
secciones 9 a 13).

==========================================
4. DECISIÓN DE DISEÑO: CONTRATO DE ENTRADA EN EL GATEWAY
==========================================

**Pregunta abierta heredada** de `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §4: a diferencia de
Guardias (`GuardiaRequest`/`GuardiaResponse` propios del Gateway), `ProveedorController` del Gateway
reutiliza hoy `ProveedorDto` directamente como entrada/salida. Al añadir create/update/delete había
que decidir si se mantiene esa reutilización o se introduce un contrato propio.

**Decisión (2026-07-20):** introducir `ProveedorRequest` en el Gateway para las operaciones de
escritura (`POST`/`PUT`), manteniendo `ProveedorDto` como contrato de lectura (`GET`) y como DTO
interno de comunicación con `gda-persona` vía Feign.

**Motivo, verificado en código real:** `ProveedorDto` no es exclusivo de Suppliers — lo usan también
`GuardiaServiceImpl`, `PersonaServiceImpl` y otras clases de lectura en el Gateway y en
`gda-admingade`. Es además un DTO plano, sin ninguna anotación de validación. Añadirle `@NotNull`/
`@Size` para las reglas de escritura de Suppliers (sección 5) arrastraría esas reglas a todos sus
consumidores de solo lectura. Un `ProveedorRequest` propio, validado con `@Valid` en el controller,
aísla esas reglas al único punto de entrada que las necesita — mismo patrón que `GuardiaRequest`.

Queda pendiente decidir, al implementar, si conviene también un `ProveedorResponse` por simetría con
Guardias; no es necesario para el `GET`, que ya funciona con `ProveedorDto` sin problemas.

==========================================
5. ANOMALÍAS DEL ESQUEMA REAL DE BD A CUBRIR EN EL BACKEND
==========================================

Heredadas de `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §5, donde se documentaron como
responsabilidad de validación del **frontend** (`supplierForm.vue`) frente al esquema real de
`gda_persona.proveedor`. Con la ampliación CUD del backend, estas mismas discrepancias deben
cubrirse también en el lado servidor, en defensa de datos que no lleguen desde `supplierForm.vue` (o
lleguen manipulados). Verificado contra
[Proveedor.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/Proveedor.java)
y [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java):

| Campo | Entidad JPA (`Proveedor.java`) | DDL real (`gda_persona.proveedor`) | Tratamiento en el backend |
|---|---|---|---|
| `observaciones` | `length = 250` | `varchar(200)` | Sin validación explícita (ver nota de la Fase 2 más abajo: el Gateway no usa Bean Validation en ningún `Request` existente); un valor de 201-250 caracteres fallará en la BD como truncamiento, capturado como `DataIntegrityViolationException` → 400 en `gda-persona` (mismo mecanismo que `codigo`, fila siguiente) |
| `encargoId` | nullable (sin `nullable = false`) | `NOT NULL DEFAULT 0` | **Actualizado en la Fase 2 (2026-07-20):** no se usa `@NotNull` (sin precedente de Bean Validation en el Gateway). Comprobación manual en `ProveedorServiceImpl` del Gateway (`if (proveedorRequest.getEncargoId() == null) throw new IllegalArgumentException(...)`), capturada por un `@ExceptionHandler(IllegalArgumentException.class)` nuevo en `GlobalExceptionHandler` del Gateway → 400. Verificado que este tratamiento **no tiene precedente real**: `RecursoServiceImpl`/`PedidoServiceImpl` (`gda-admingade`) ya usan `IllegalArgumentException` para el mismo tipo de caso ("campo obligatorio ausente"), pero sus controllers no la capturan — cae al `catch (Exception e)` genérico → 500. Decisión explícita: mantener el nuevo `@ExceptionHandler` como **mejora deliberada** sobre ese patrón existente (que hoy da 500 donde debería dar 400), no como reproducción de un precedente |
| `codigo` | sin `unique` en la entidad | `UNIQUE (codigo)` + `UNIQUE (codigo, nombre)` | no es validable con Bean Validation (requiere consulta a BD); `ProveedorController`/`ProveedorServiceImpl` de `gda-persona` capturan `DataIntegrityViolationException` en `addProveedor`/`editProveedor` y devuelven 400 (implementado en la Fase 1), replicando `GuardiaController.editGuardia`; el Gateway ya traduce el `FeignException.BadRequest` resultante a 400 vía su propio `GlobalExceptionHandler` (verificado en la Fase 2, sin cambios necesarios ahí) y los composables del frontend (`useSaveSupplier.js`/`useUpdateSupplier.js`, ya implementados) se limitan a propagar y mostrar ese error |

No se corrige la entidad `Proveedor.java` ni el esquema de BD (fuera de alcance, igual que en el
frontend); estas notas existen para que el backend nuevo (`ProveedorRequest`, manejo de excepciones)
no reproduzca del lado servidor el mismo desajuste ya detectado y resuelto del lado cliente.

==========================================
6. VERIFICACIÓN DE CUMPLIMIENTO CON LOS ADR (docs/decisiones)
==========================================

Mismo resultado que en `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §6 — la ampliación CUD no cambia
ninguna conclusión, solo añade evidencia de las partes que faltaban por construir:

| ADR | Requisito clave | ¿Cumple la ampliación backend de Suppliers? |
|-----|------------------|----------------------------------|
| [ADR-0600](../../docs/decisiones/ADR-0600-estrategia-despliegue.md) | Cada módulo backend es un artefacto independiente | ✅ Se amplía `gda-persona`, módulo ya existente; no se crea artefacto Maven nuevo |
| [ADR-0601](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md) | DTOs/excepciones en `-commons`; entidades/repos/services/controllers en el módulo principal | ✅ `ProveedorDto` sigue en `gda-persona-commons`; `ObjectAlreadyExistException`/`ObjectNotFoundException` ya están ahí y se reutilizan sin cambio; el `ProveedorRequest` nuevo va en el Gateway (no en `-commons`), igual que `GuardiaRequest` |
| [ADR-0602](../../docs/decisiones/ADR-0602-comunicacion-interservicios.md) | REST/JSON, rutas planas sin versionar | ✅ Los nuevos `POST`/`PUT`/`DELETE` se añaden sobre las mismas rutas planas ya usadas por el `GET` (`/proveedores`, `admingade/proveedores`, etc.) |
| [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) | Schema propio por servicio, sin joins cruzados | ✅ Sin cambios de esquema; `save()`/`deleteById()` de `JpaRepository` operan sobre la misma tabla `proveedor` en `gda_persona` |
| [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md) | Composición dedicada en el Gateway cuando el recurso necesita datos de otro microservicio | ✅ La capa de composición ya existente (`ProveedorServiceImpl` agrega `encargoVigente` de `gda-admingade`) se amplía sin cambiar su patrón. **Revisado el 2026-07-20** al cerrar la Fase 2: se actualizó su ejemplo de Proveedores, que ya no es la "variante sin `Request`/`Response` propios" (ahora usa `ProveedorRequest`, igual que Guardias) |
| [ADR-0605](../../docs/decisiones/ADR-0605-modelo-autorizacion-rbac-api-gateway.md) | Autorización por prefijo de ruta en el Gateway, sin distinción lectura/escritura; microservicios de dominio sin autorización propia | ✅ Creado el 2026-07-20 a raíz de la decisión de permisos de Suppliers (sección 7.5/8); Suppliers reutiliza el prefijo/roles de "Gestión Admingade" sin necesidad de tocar `SecurityConfiguration.java`, exactamente el criterio que formaliza este ADR |
| [ADR-0801](../../docs/decisiones/ADR-0801-estrategia-empaquetado-despliegue.md) | JAR ejecutable vía CI Maven | ✅ Sin cambios de empaquetado, mismo pipeline de `gda-persona` y del Gateway |
| [ADR-0802](../../docs/decisiones/ADR-0802-integracion-front-back.md) | Todo vía API Gateway, manejo de errores centralizado | ✅ El frontend (ya construido) sigue llamando solo al Gateway; los nuevos errores 400/409 de unicidad se propagan por el mismo canal que ya usa `notifications.js` |
| [ADR-0803](../../docs/decisiones/ADR-0803-seguridad-secretos.md) | Auth JWT stateless, sin datos sensibles sin cifrar | ✅ Sin cambios; `Proveedor` sigue sin datos sensibles (confirmado 2026-07-16 en el análisis frontend) |

==========================================
7. IMPACTO EN OTRAS FUNCIONALIDADES (VERIFICADO EN CÓDIGO, 2026-07-20)
==========================================

Antes de tocar código se ha comprobado explícitamente si la ampliación CUD de `Proveedor` puede
afectar negativamente a alguna otra funcionalidad ya construida. Resultado, por archivo/área
verificada:

1. **`ProveedorDto` compartido** — sin impacto: no se modifica (decisión de la sección 4: el
   contrato de escritura nuevo es `ProveedorRequest`, aparte). `ProveedorDto` sigue idéntico para
   sus otros consumidores de solo lectura ya existentes (`GuardiaServiceImpl`, `PersonaServiceImpl`,
   entre otros).
2. **`GlobalExceptionHandler.java`** (`gda-persona`) — sin impacto: ya es genérico por tipo de
   excepción (`ObjectNotFoundException`, `ObjectAlreadyExistException`), compartido hoy por `Persona`
   y `PersonaHistorico`; reutilizarlo para `Proveedor` no requiere modificarlo.
3. **`GdaPersonaApiClient.java`** — sin impacto: la ampliación solo añade métodos Feign nuevos
   (`addProveedor`/`editProveedor`/`deleteProveedor`); ninguno de los métodos existentes se toca.
4. **Ruta de paso comentada en `RouteLocatorConfig.java`** (líneas 236-240) — riesgo real, ya
   señalado en `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §4: existe una ruta simple de Spring
   Cloud Gateway para `/admingade/proveedores/**` (+ `rsp_grp`/`rsp_eqp`/`super`), comentada y
   superada por la capa de composición (`ProveedorController`). Verificado que sigue comentada. Si se
   reactivara mientras el `ProveedorController` de composición ya expone `POST`/`PUT`/`DELETE` sobre
   esas mismas rutas, se produciría un conflicto de enrutamiento entre la ruta de paso y el
   controller propio de la app Gateway. **Debe permanecer comentada** durante y después de esta
   ampliación.
5. **Seguridad/permisos del Gateway** (`SecurityConfiguration.java`, Gateway) — verificado: la
   autorización se aplica por **prefijo de ruta**, sin distinguir verbo HTTP
   (`.pathMatchers("/admingade/**").hasAnyAuthority("ADMINGADE","ADMINGADE_RSP_EQP","GDA_SUPERGESTOR","ADMINGADE_MANAGER")`,
   y equivalentes para `/rsp_grp/**`, `/rsp_eqp/**`, `/super/**`). Consecuencia doble: (a) no hace
   falta configuración de seguridad nueva para que `POST`/`PUT`/`DELETE` funcionen, se heredan los
   mismos roles que ya protegen el `GET`; y (b) cualquier usuario que hoy puede leer proveedores por
   esa ruta podrá, en cuanto se implemente, crear/editar/eliminar también — no existe un rol de
   "escritura" separado de "lectura" en el modelo actual.
   **Decisión de producto (2026-07-20):** esto es exactamente lo que se quiere — solo deben acceder a
   la gestión de Proveedores las mismas personas que ya acceden al resto de pantallas del menú
   "Gestión Admingade", y esas personas (pocas) deben tener CRUD completo, sin rol de escritura
   separado. Verificado en [menu.js](../../GDA/frontend/src/app/menu.js) (bloque
   `MENU_MANAGEMENT_ADMINGADE`, líneas 121-179): `MANAGEMENT_ADMINGADE_SUPPLIERS` ya está agrupado junto
   al resto de pantallas de ese menú (Guardias, Servicios Suministros, Personas ETT, Incurridos,
   Consultas...), todas ellas servidas por rutas bajo `/admingade/**`. El modelo de permisos actual del
   Gateway ya cumple el requisito tal cual está — **no se necesita ningún cambio en
   `SecurityConfiguration.java` ni un rol nuevo.**
6. **`gda-persona` (microservicio interno)** — verificado en su propio `SecurityConfiguration.java`:
   no aplica autorización por ruta/rol a nivel de microservicio, solo CORS; confía en que el Gateway
   ya filtró la petición antes de reenviarla. Mismo modelo de confianza que ya usa `Guardia` en
   `gda-admingade` hoy — no es un riesgo nuevo.
7. **Tests existentes** (`ProveedorServiceImplTest.java`) — al ser cambios aditivos (métodos nuevos,
   ninguno existente se modifica ni se elimina), no se espera que rompan; se evaluará al implementar
   si conviene añadir tests para los métodos nuevos.

**Conclusión:** no se detecta impacto negativo en funcionalidades ya construidas. Única condición que
sigue en pie: la ruta comentada de `RouteLocatorConfig.java` (punto 4) debe permanecer comentada. El
punto de permisos (punto 5) queda resuelto: el modelo actual ya es el deseado, sin cambios de
seguridad pendientes.

==========================================
8. ACCIONES PENDIENTES ANTES DE IMPLEMENTAR
==========================================

1. ~~Decidir si el Gateway reutiliza `ProveedorDto` o introduce un contrato propio para create/update.~~
   **Resuelto (sección 4):** se introduce `ProveedorRequest`.
2. ~~Confirmar el nombre y ubicación exactos de las excepciones a lanzar en `gda-persona`.~~
   **Resuelto (Fase 1):** `ObjectAlreadyExistException` en `addProveedor` si el `id` ya existe,
   `ObjectNotFoundException` en `editProveedor`/`deleteProveedor` si el `id` no existe — reutilizadas
   de `gda-persona-commons` sin crear excepciones nuevas.
3. ~~Confirmar si el Gateway necesita también un `ProveedorResponse` propio.~~ **Resuelto (Fase 2):**
   no hace falta — el `GET` sigue devolviendo `ProveedorDto` sin cambios; solo se introdujo
   `ProveedorRequest` para la entrada de `POST`/`PUT` (sección 12).
4. ~~Verificar permisos/roles para las operaciones create/update/delete de Suppliers.~~ **Resuelto
   (2026-07-20, sección 7.5):** solo acceden las mismas personas que ya acceden al resto de pantallas
   de "Gestión Admingade" (roles `ADMINGADE`, `ADMINGADE_RSP_EQP`, `GDA_SUPERGESTOR`,
   `ADMINGADE_MANAGER` sobre `/admingade/**`), con CRUD completo, sin rol de escritura separado. El
   modelo actual del Gateway ya lo cumple tal cual — no se toca `SecurityConfiguration.java`.
5. Mantener comentada la ruta `proveedores` de `RouteLocatorConfig.java` (sección 7.4) durante y
   después de esta ampliación.
6. Una vez implementado: probar de extremo a extremo con el frontend ya construido (Añadir → Editar
   → Eliminar desde `Suppliers.vue`), confirmando en particular el caso de `codigo` duplicado (sección 5).

==========================================
9. PROCEDIMIENTO DE IMPLEMENTACIÓN POR FASE
==========================================

Acordado el 2026-07-20 (paso 6 añadido el mismo día, tras cerrar la Fase 1). Cada fase de este
documento (gda-persona, API Gateway, y las que surjan) sigue el mismo procedimiento de seis pasos,
en este orden:

1. **Corregir/ampliar el código** de la fase correspondiente.
2. **Detalle de archivos afectados**, indicando el microservicio/módulo de cada uno (`gda-persona`,
   `gda-persona-commons`, API Gateway, frontend...), entregado como resumen antes de continuar.
3. **Validación** por parte del usuario. Si la fase toca más de un microservicio (p. ej. `gda-persona`
   + API Gateway, como la Fase 2), probar en local requiere primero **redirigir temporalmente los
   clientes Feign del Gateway a `localhost`**: en `GDA/api-gateway/src/main/resources/config/
   application-local.yml`, cada dependencia (`gda-persona`, `gda-admingade`, etc.) tiene su URL local
   ya escrita pero comentada, junto a la URL remota compartida (`172.16.22.114`) activa por defecto:
   ```yaml
   gda-persona:
     feign:
       # url: http://localhost:9108
       url: http://172.16.22.114:9108
   ```
   Solo hace falta descomentar/comentar la URL del/los microservicios que la fase haya modificado
   (los no tocados pueden seguir apuntando al remoto compartido, sin necesidad de levantarlos en
   local — así se hizo con `gda-admingade` en la Fase 2/3, sin cambios). Tras editar, **reiniciar el
   proceso del Gateway** (Spring Boot no recarga YAML en caliente) y, antes de dar la fase por
   validada, dejar el archivo como estaba (URL remota activa) para no dejar ese cambio local mezclado
   en el commit de la fase — es un ajuste de entorno de pruebas, no un cambio de código de la fase.
4. **Subida al repositorio GitLab**, en la rama correspondiente, solo si procede tras la validación.
   **La ejecuta el usuario manualmente** (`git add`/`commit`/`push`) — no se ejecuta ningún comando
   git desde esta conversación, ni siquiera con validación previa. Cada microservicio bajo
   `GDA/backend/` (y `GDA/frontend`) tiene su propio repositorio git independiente con remoto propio
   (ver sección 11, Fase 1); no hay un único repo de workspace que los agrupe.
5. **Documentación de la fase completada**, añadiendo su sección correspondiente en este documento
   (siguiendo el formato de `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §§9-13).
6. **Revisión y/o ampliación de ADRs** (`docs/decisiones/`, plantilla `adr_template.md`, índice
   obligatorio `ai/governance/14_ADR_INDEX.md`) en base a la experiencia adquirida durante la
   implementación de la fase que se cierra: ¿algún ADR existente queda desactualizado por lo
   implementado? ¿se ha detectado algún patrón real, ya en producción, que no estuviera documentado
   en ningún ADR? Primer caso real: al cerrar la Fase 1 se detectó que ADR-0604 quedará desactualizado
   cuando la Fase 2 implemente `ProveedorRequest` (revisión diferida a esa fase, no ahora) y que el
   modelo de autorización RBAC del Gateway no estaba documentado en ningún ADR — se creó ADR-0605
   para cerrar esa laguna (ver sección 6).

No se avanza al paso siguiente sin completar el anterior; en particular, no se documenta una fase
como completada (paso 5) sin que el usuario confirme que el paso 4 ya se ha realizado, y la fase no
se da por cerrada del todo hasta completar también el paso 6.

==========================================
10. ESTADO REAL DE IMPLEMENTACIÓN — FASE 0 (PLANIFICACIÓN) (2026-07-20)
==========================================

**Resumen:** completada. Se verificó en código real el estado actual (solo lectura en las 4 capas
backend), se tomó la decisión de diseño de la sección 4, se trasladaron las anomalías de esquema de
la sección 5 del documento frontend al lado backend, se comprobó el impacto en otras funcionalidades
(sección 7) y se cerró la decisión de permisos (sección 7.5/8). No se modificó ningún archivo de
código en esta fase.

==========================================
11. ESTADO REAL DE IMPLEMENTACIÓN — FASE 1 (GDA-PERSONA) (2026-07-20)
==========================================

**Resumen:** se amplió `gda-persona` con create/update/delete para `Proveedor`, siguiendo el patrón
de `GuardiaServiceImpl`/`GuardiaController` (sección 2) y el manejo de excepciones acordado en las
secciones 5 y 8. Compilación (`mvn -pl gda-persona -am compile`) y tests (`mvn -pl gda-persona -am
test -Dtest=ProveedorServiceImplTest`) verificados en verde (15/15) antes de dar la fase por
completada. Esta fase **no es funcional de extremo a extremo todavía**: el API Gateway (Fase 2) sigue
sin exponer `POST`/`PUT`/`DELETE`, así que el frontend ya construido sigue sin poder completar
Añadir/Editar/Eliminar hasta que esa fase se implemente.

**Archivos modificados (microservicio `gda-persona`):**

| Archivo | Ruta | Cambio |
|---|---|---|
| `ProveedorService.java` | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/` | `+ addProveedor()`, `+ editProveedor()`, `+ deleteProveedor()` en la interfaz |
| `ProveedorServiceImpl.java` | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/impl/` | Implementación de los 3 métodos: `addProveedor` comprueba `id` ya existente (`ObjectAlreadyExistException`); `editProveedor`/`deleteProveedor` comprueban `id` inexistente (`ObjectNotFoundException`) |
| `ProveedorController.java` | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/` | `+ @PostMapping`, `+ @PutMapping`, `+ @DeleteMapping` sobre `/proveedores`; captura `DataIntegrityViolationException` → 400 (duplicado de `codigo`, sección 5) |
| `ProveedorServiceImplTest.java` | `GDA/backend/gda-persona/gda-persona/src/test/java/com/ineco/gda/persona/service/impl/` | `+ 6` tests: `addProveedor_ok/_alreadyExists`, `editProveedor_ok/_nullId/_notFound`, `deleteProveedor_ok/_notFound` |

**No tocados (confirmado):** `ProveedorRepository`, `Proveedor.java` (entidad), `ProveedorDto.java`,
`GlobalExceptionHandler.java` — tal como se preveía en las secciones 3 y 7.

**Decisión de procedimiento (2026-07-20):** la subida a GitLab (paso 4 del procedimiento de la
sección 9) la gestiona el usuario manualmente, fuera de esta conversación — no se ejecuta ningún
`git add`/`commit`/`push` desde aquí. Corrección de contexto (2026-07-20): el repo raíz del workspace
(`C:\WorkSpace\2408_GDA-REIR`) no tiene commits ni remoto y su `.gitignore` excluye `GDA/` por
completo, pero **cada microservicio dentro de `GDA/backend/` tiene su propio repositorio git
independiente** — `GDA/backend/gda-persona` está conectado a
`http://172.16.22.114/gdact-reir/gdact-3.0/backend/gda-persona.git` (rama de trabajo `GDAV3-443` en
el momento de esta fase). El usuario mantiene ahí una convención propia de archivos "copy" (revisión
local de cambios, excluidos vía `.gitignore` del propio repo); ya retirados antes del commit
definitivo.

**Verificación final antes de la subida (2026-07-20):** con los archivos "copy" ya eliminados,
`mvn -pl gda-persona -am clean test` completo en verde (15/15 tests, 0 errores). Commit y push a
`GDAV3-443` realizados por el usuario.

**Paso 6 — Revisión de ADRs (2026-07-20):** se revisaron los 8 ADRs vigentes en `docs/decisiones/`
contra lo implementado en esta fase (sección 6). Dos hallazgos: (1) ADR-0604 quedará parcialmente
desactualizado cuando la Fase 2 implemente `ProveedorRequest` — revisión **diferida a esa fase**, no
corregido ahora; (2) el modelo de autorización RBAC del Gateway (base de la decisión de permisos de
la sección 7.5/8) no estaba documentado en ningún ADR — creado
[ADR-0605](../../docs/decisiones/ADR-0605-modelo-autorizacion-rbac-api-gateway.md) para cerrar esa
laguna, y actualizado `ai/governance/14_ADR_INDEX.md` en consecuencia.

**Fase 1 subida a GitLab y documentada — procedimiento de la sección 9 completado de principio a fin
(pasos 1 a 6).**

**Pendiente:** Fase 2 (API Gateway: `ProveedorRequest`, `GdaPersonaApiClient`,
`ProveedorService`/`ProveedorServiceImpl`/`ProveedorController` del Gateway — incluida la revisión de
ADR-0604 señalada arriba) y Fase 3 (verificación end-to-end desde `Suppliers.vue`).

Este documento se irá actualizando con una sección por fase completada, siguiendo el mismo formato de
`Documentacion_FrontEnd_GDA_2607_Suppliers.md` §§9-13, y el procedimiento de la sección 9.

==========================================
12. ESTADO REAL DE IMPLEMENTACIÓN — FASE 2 (API GATEWAY) (2026-07-20)
==========================================

**Resumen:** se amplió el API Gateway con `POST`/`PUT`/`DELETE` para `Proveedor`, cerrando el hueco
que dejaba pendiente la Fase 1 — el frontend ya construido puede ahora completar Añadir/Editar/
Eliminar de extremo a extremo. Se siguió el patrón reactivo ya existente en el propio
`ProveedorServiceImpl` del Gateway (`ReactiveSecurityContextHolder` + `Mono`, no el estilo síncrono de
`CandidatoServiceImpl`), por ser el precedente del mismo archivo. Compilación verificada en verde
(`mvn clean compile`) antes de dar la fase por completada; no se añadieron tests porque el módulo
`api-gateway` no tiene ningún test existente (verificado: `src/test/` vacío en todo el módulo, ni
Guardias ni Candidato lo tienen tampoco).

**Archivos modificados/nuevos (API Gateway):**

| Archivo | Ruta | Cambio |
|---|---|---|
| `ProveedorRequest.java` | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/persona/request/` | **Nuevo** — DTO plano sin Bean Validation (mismo patrón que `GuardiaRequest`/`CandidatoRequest`/`PersonaRequest`, ninguno de los cuales usa `@NotNull`/`@Size`) |
| `GdaPersonaApiClient.java` | `.../apigateway/web/clients/` | `+ addProveedor()`, `+ editProveedor()`, `+ deleteProveedor()` (Feign), agrupados bajo el comentario `// Proveedor` ya existente |
| `ProveedorService.java` (Gateway) | `.../apigateway/service/persona/` | `+ addProveedor()`, `+ editProveedor()`, `+ deleteProveedor()` (`Mono<...>`) |
| `ProveedorServiceImpl.java` (Gateway) | `.../apigateway/service/persona/impl/` | Implementación reactiva; `addProveedor`/`editProveedor` comprueban `encargoId != null` (`IllegalArgumentException` si falta, ver hallazgo más abajo); `deleteProveedor` vía `.doOnNext().then()` |
| `ProveedorController.java` (Gateway) | `.../apigateway/web/controller/persona/` | `+ @PostMapping`, `+ @PutMapping`, `+ @DeleteMapping` sobre las mismas 4 rutas prefijadas del `GET` |
| `GlobalExceptionHandler.java` (Gateway) | `.../apigateway/exception/` | `+ @ExceptionHandler(IllegalArgumentException.class)` → 400 (nuevo, ver hallazgo más abajo) |

**No tocados:** `gda-persona` (Fase 1, verificado con `git status` sin cambios), `ACSFeignConfiguration.java`, `RouteLocatorConfig.java` (ruta comentada, sigue así, sección 7.4).

**Hallazgo 1 — sin Bean Validation en el Gateway:** ningún `Request` existente (`GuardiaRequest`,
`CandidatoRequest`, `PersonaRequest`) usa `@NotNull`/`@Size`, y el `pom.xml` del Gateway no tiene
`spring-boot-starter-validation`. Se descartó el plan original de la sección 5 (Bean Validation) a
favor de una comprobación manual, sin nueva dependencia — decisión tomada explícitamente con el
usuario antes de escribir código.

**Hallazgo 2 — propagación de errores 400 ya funciona sin cambios adicionales:** el Gateway ya tiene
su propio `GlobalExceptionHandler` (`@ControllerAdvice`), que traduce `FeignException.BadRequest`
(lanzada por Feign vía su `ErrorDecoder.Default` cuando `gda-persona` responde 400, p. ej. por
`codigo` duplicado) a una respuesta 400 con cuerpo `ExceptionResponse`. Es el mismo mecanismo que ya
usa Guardias en producción; no hizo falta construir nada nuevo para que el caso de `codigo` duplicado
(sección 5) llegue como 400 al frontend.

**Hallazgo 3 — `IllegalArgumentException` para `encargoId` obligatorio, verificado en detalle:**
`RecursoServiceImpl`/`PedidoServiceImpl` (`gda-admingade`) ya usan `IllegalArgumentException` para el
mismo tipo de caso ("campo obligatorio ausente"), pero sus controllers **no la capturan** — cae al
`catch (Exception e)` genérico → 500, no 400. El `@ExceptionHandler(IllegalArgumentException.class)`
añadido al `GlobalExceptionHandler` del Gateway (que sí traduce esto a 400) **no reproduce un
precedente real**; es una mejora deliberada sobre ese patrón existente, decidida explícitamente con
el usuario. Ver sección 5 (fila `encargoId`) para el detalle completo.

**Paso 6 — Revisión de ADRs (2026-07-20):** actualizado
[ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md) (diferido desde el
cierre de la Fase 1): su ejemplo de Proveedores como "variante sin `Request`/`Response` propios" ya
no es exacto — se documentó como historia (estado en 2026-07-16) y se añadió una actualización
(2026-07-20) señalando que Proveedores ahora usa `ProveedorRequest`, igual que Guardias. No se creó
ningún ADR nuevo en esta fase — los Hallazgos 1 y 3 son decisiones de implementación puntuales, no
patrones de arquitectura nuevos que requieran gobernanza propia.

**Fase 2 subida a GitLab y documentada — procedimiento de la sección 9 completado de principio a fin
(pasos 1 a 6).**

==========================================
13. ESTADO REAL DE IMPLEMENTACIÓN — FASE 3 (VERIFICACIÓN END-TO-END) (COMPLETADA, 2026-07-21)
==========================================

**Resumen:** en curso. Verificado: **Añadir, Editar y Eliminar Proveedor funcionan de extremo a
extremo** (frontend → Gateway → `gda-persona` → BD compartida de desarrollo), incluido el caso de
`código` duplicado mostrando ya un mensaje de error real en el frontend. Se detectaron y resolvieron
tres incidencias durante la propia verificación — dos de datos/producto (puntos 1 y 2, sin cambio de
código) y una sí de código, ya corregida y subida a GitLab (punto 3):

1. **Secuencia de `id` desincronizada en la BD compartida** (`gda_03_desa`): el primer intento de
   Añadir devolvió 500 por `duplicate key value violates unique constraint "pk_proveedor"` — la
   secuencia IDENTITY de `proveedor.id` estaba por detrás del máximo `id` real de la tabla (datos
   insertados en algún momento sin pasar por la secuencia). Resuelto por el usuario contra la BD
   compartida con `SELECT setval(pg_get_serial_sequence('gda_persona.proveedor', 'id'), (SELECT
   MAX(id) FROM gda_persona.proveedor));`. Incidencia de datos del entorno compartido, no de la Fase 1.

2. **Encargo seleccionado no coincidía con el "encargo vigente"**: tras el primer Añadir exitoso, el
   proveedor no aparecía en el listado, ni siquiera tras recargar la página o volver a entrar en la
   app. Causa: `ProveedorServiceImpl.findAll()` del Gateway filtra el listado al `encargoVigente`
   (`GdaAdmingadeApiClient.getEncargoVigente()`, valor `3` en el entorno de pruebas en el momento de
   esta verificación), pero el desplegable "Encargo" del formulario (`useFetchJobs()`) muestra
   **todos** los encargos, no solo el vigente — se guardó por error con `encargoId = 2`. No es un bug
   — es el comportamiento ya previsto (documentado también como mejora de UX pendiente en
   `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §7) — pero es una nota operativa importante para
   seguir probando Editar/Eliminar: **verificar siempre que el `encargoId` usado coincide con el
   valor real de `GET <gda-admingade>/encargos/encargoVigente`**, contrastando con
   `GET <gda-persona>/proveedores/nombre/{nombre}` (sin pasar por el Gateway) si algo no aparece
   donde se espera.

3. **`código` duplicado devolvía 400 sin mensaje visible en el frontend** — probado en Añadir y en
   Editar. Causa: `ProveedorController.java` (`gda-persona`) capturaba `DataIntegrityViolationException`
   con `ResponseEntity.status(HttpStatus.BAD_REQUEST).build()`, **sin cuerpo** — mismo patrón que ya
   usa `GuardiaController.editGuardia` (no es un bug introducido por la Fase 1, es un hueco
   preexistente replicado siguiendo el patrón de Guardias). Sin cuerpo real, el
   `@ExceptionHandler(FeignException.BadRequest.class)` del Gateway generaba un mensaje reducido a un
   único espacio en blanco (`" "`), que `vue-i18n` no podía traducir (`[intlify] Not found ' ' key`) —
   y como `" "` es un string no vacío (truthy en JS), el *fallback* del frontend a `"errors.default"`
   nunca se activaba (`error?.response?.data?.message || "errors.default"`). **Corregido (2026-07-21,
   commit `3fa57a2`, subido a GitLab):** `addProveedor`/`editProveedor` en `ProveedorController.java`
   pasan de `ResponseEntity<ProveedorDto>` a `ResponseEntity<Object>` (necesario para poder devolver
   un `String` de error sin romper el tipo genérico del método) y el `catch` de
   `DataIntegrityViolationException` ahora responde `.body("El código de proveedor ya existe")` en
   vez de `.build()` — mismo precedente que ya usa el `GlobalExceptionHandler` de `gda-persona` para
   `ObjectAlreadyExistException`/`ObjectNotFoundException` (devolver `ex.getMessage()` en el cuerpo).
   Deliberadamente **no se corrige el mismo hueco en `GuardiaController`** (fuera de alcance de
   Suppliers). Verificado por el usuario: el mensaje ya aparece correctamente en el frontend, en Añadir
   y en Editar.

**Revisión de ADR (paso 6, 2026-07-21):** el incidente de la secuencia desincronizada (punto 1) es
evidencia real de una deuda técnica que
[ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) ya documentaba desde el 2026-06-11 (sin
migraciones versionadas); se añadió como evidencia concreta en su sección de deuda técnica, sin
cambiar ninguna decisión. El resto de hallazgos de esta fase (desajuste `encargoId`/`encargoVigente`,
necesidad de tocar `application-local.yml` para pruebas locales) son de producto/UX y de
procedimiento respectivamente, no arquitectónicos — no requieren ADR nuevo ni corrección de los
existentes (evaluado explícitamente, ver conversación de esta fase).

**Fase 3 (backend) verificada y cerrada — procedimiento de la sección 9 completado de principio a fin
(pasos 1 a 6) para el punto 3 anterior.** No queda ningún pendiente de backend para Suppliers CUD.

**Pendiente (frontend, fuera de este documento):** retirar los tooltips provisionales de "backend aún
no disponible" en Añadir/Editar/Eliminar (`supplierSaveNotAvailable`, `supplierDeleteNotAvailable`,
`Suppliers.vue`/`supplierForm.vue`), y la mejora de UX ya anotada en
`Documentacion_FrontEnd_GDA_2607_Suppliers.md` §7 (desplegable de Encargo no restringido al vigente).

==========================================
14. ESTADO REAL DE IMPLEMENTACIÓN — FASE 4 (CAMPOS DE AUDITORÍA) (2026-07-22)
==========================================

**Resumen:** con Suppliers CUD ya funcional de extremo a extremo (Fase 3), se añadieron a `Proveedor`
los campos de auditoría `persona_id`, `fecha_alta`, `fecha_baja` y `fecha_efecto`, replicando
deliberadamente el criterio ya usado por `gda_estructura.iniciativa` (que usa `username` en vez de
`persona_id`), con dos decisiones explícitas que **divergen** de ese modelo de referencia (ver
`Documentacion_BBDD_GDA_2607_Suppliers.md` para el detalle completo de la decisión y su motivo):
`persona_id` se reasigna en cada edición (no solo en el alta, a diferencia de `username`), y `fecha_baja`
es intocable vía API en ambas entidades — en Proveedor por decisión explícita ("solo gestionable
directamente en BBDD"); en Iniciativa porque su alta baja pasa por el PUT normal con el propio
`estado`, mecanismo que Proveedor no tiene.

**Archivos modificados:**

| Archivo | Módulo | Cambio |
|---|---|---|
| [Proveedor.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/Proveedor.java) | gda-persona | `+ usuarioModificacionId` (`Long`, `@Column(name = "usuario_modificacion_id")` — renombrado desde `personaId`, ver `Documentacion_BBDD_GDA_2607_Suppliers.md` §7), `+ fechaAlta/fechaBaja/fechaEfecto` (`LocalDate`) |
| [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java) | gda-persona-commons | Mismos 4 campos; las 3 fechas anotadas con `@JsonSerialize(using = LocalDateSerializer.class)` / `@JsonDeserialize(using = LocalDateDeserializer.class)` (ver Hallazgo 1 más abajo) |
| [ProveedorServiceImpl.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/impl/ProveedorServiceImpl.java) | gda-persona | `addProveedor`: `fechaAlta = LocalDate.now()`. `editProveedor`: preserva `fechaAlta` **y** `fechaBaja` del registro original leído de BD, ignorando lo que llegue en el DTO — `fechaBaja` queda así intocable vía API. `deleteProveedor`: **sin cambios**, sigue siendo `deleteById` físico (ver Hallazgo 2) |
| [ProveedorRequest.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/persona/request/ProveedorRequest.java) | API Gateway | `+ fechaEfecto` (`@JsonFormat(pattern="dd/MM/yyyy")`, único campo de fecha que acepta el Request; ni `fechaAlta` ni `fechaBaja` son de entrada, igual que `IniciativaRequest` no admite `username`) |
| [ProveedorServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/impl/ProveedorServiceImpl.java) | API Gateway | `addProveedor`/`editProveedor` obtienen `personaId` de `ReactiveSecurityContextHolder` (mismo patrón que `IniciativaServiceImpl.addIniciativa`) y lo **resuelven a `usuario.id`** vía `gdaAutenticacionApiClient.getUsuarioByPersona(personaId).getBody().getId()` (nueva dependencia `GdaAutenticacionApiClient`, reutilizando un método ya existente) antes de asignarlo a `usuarioModificacionId` — detalle completo del porqué en `Documentacion_BBDD_GDA_2607_Suppliers.md` §7 |

**No tocados (deliberado):** `ProveedorController` y `ProveedorRepository` de `gda-persona` (el mapeo
genérico vía `ModelMapper` ya persiste los campos nuevos sin cambio de código); `GdaPersonaApiClient`
(sigue enviando el `ProveedorDto` completo, sin métodos nuevos).

**Hallazgo 1 — serialización de `LocalDate` en Feign, bug real detectado y corregido (2026-07-22):**
[`ACSFeignConfiguration.java`](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/config/ACSFeignConfiguration.java#L29)
define, para **todas** las llamadas Feign del Gateway (no solo `gda-persona`), un `ObjectMapper`
desnudo (`new ObjectMapper()`) sin módulo de fechas Java 8 registrado. Los campos `LocalDate` sin
anotación de serialización específica se pierden silenciosamente al pasar por Feign (guardado y
lectura). `gda-persona-commons` ya tenía resuelto esto en 9 DTOs existentes (`CandidatoDto`,
`PersonaDto`, etc.) mediante `@JsonSerialize`/`@JsonDeserialize` con clases propias
(`com.ineco.gda.persona.util.LocalDateSerializer`/`LocalDateDeserializer`, formato `dd/MM/yyyy`);
`ProveedorDto` fue el único DTO nuevo sin esas anotaciones, causando que `fechaEfecto`/`fechaBaja` no
se guardaran ni se mostraran en los formularios de Editar/Consultar. Corregido replicando el patrón ya
establecido. **Es un hallazgo con impacto potencial en cualquier módulo futuro** que añada un campo
`LocalDate` a un DTO consumido vía Feign — ver ADR-0602 (deuda técnica actualizada).

**Hallazgo 2 — desajuste de versión `gda-persona-commons` en `api-gateway/pom.xml`, real, no
introducido por esta fase:** `gda-persona-parent` está en `4.4.0-SNAPSHOT`, pero
[`api-gateway/pom.xml`](../../GDA/api-gateway/pom.xml#L70-L74) fijaba la dependencia a `4.3.0` (release,
no SNAPSHOT) — mismo criterio que el resto de dependencias `-commons` del Gateway (`gda-estructura-commons:4.4.1`,
`gda-admingade-commons:4.6.1`, `gda-agrupacion-commons:4.2.0`, todas fijadas a release, ninguna a
SNAPSHOT). Verificado con `git log` que esta línea no ha cambiado en los últimos commits de
`api-gateway`: es un desajuste preexistente, no introducido por Suppliers ni por esta fase. Para poder
compilar y probar en local, se actualizó temporalmente a `4.4.0-SNAPSHOT` (decisión explícita del
usuario) — **este cambio de `pom.xml` se excluye deliberadamente del commit de esta fase**; antes de
publicar de verdad haría falta o bien revertir el pin a una versión de release real de
`gda-persona-commons` publicada en Nexus, o publicar `4.4.0` como release. Ver ADR-0600 (deuda técnica
actualizada) para el detalle.

**Verificación:** `mvn compile` en verde en `gda-persona`/`gda-persona-commons` y `api-gateway`; 15/15
tests de `ProveedorServiceImplTest` sin regresión; probado de extremo a extremo por el usuario en local
(Añadir con Fecha Efecto, Editar/Consultar mostrando las fechas correctamente).

**Paso 6 — Revisión de ADRs (2026-07-22):** ver sección 6 actualizada y los ADRs referenciados en los
Hallazgos 1 y 2. Se creó además [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md)
para codificar como convención del proyecto el patrón de campos de auditoría (`persona_id`/`fecha_alta`
autogestionados en backend, `fecha_efecto` como campo de negocio obligatorio, `fecha_baja` opcionalmente
inmutable vía API), ya aplicado ahora a dos entidades (`Iniciativa`, `Proveedor`).

**Pendiente:** subida a GitLab por el usuario (excluyendo `pom.xml`, ver Hallazgo 2, y
`application-local.yml`, config de entorno local).

==========================================
15. ESTADO REAL DE IMPLEMENTACIÓN — FASE 5 (RENOMBRADO `fecha_efecto` → `fecha_modificacion`) (2026-07-22)
==========================================

**Resumen:** corrección del criterio de la Fase 4 — `fecha_efecto` (campo de negocio editable por el
frontend) se renombra a `fecha_modificacion` y se invierte su naturaleza a **100% auditoría/backend**,
actualizada en cada alta y edición. Al mismo tiempo, `fecha_alta` (ya existente desde la Fase 4, hasta
ahora 100% backend) pasa a ser editable por el usuario en el formulario de Añadir, con valor por
defecto la fecha del día. Detalle completo, campo a campo, en
`Documentacion_BBDD_GDA_2607_Suppliers.md` §8.

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| [Proveedor.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/Proveedor.java) | `fechaEfecto` → `fechaModificacion` |
| [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java) | Mismo renombrado |
| [ProveedorServiceImpl.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/impl/ProveedorServiceImpl.java) | `addProveedor`: ya no fuerza `fechaAlta` (ahora la decide el DTO entrante); `+ fechaModificacion = LocalDate.now()`. `editProveedor`: sigue preservando `fechaAlta`/`fechaBaja` originales; `+ fechaModificacion = LocalDate.now()` |
| [ProveedorRequest.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/persona/request/ProveedorRequest.java) | `fechaEfecto` → `fechaAlta` — ahora sí aceptado desde el frontend |

**No tocados:** `ProveedorServiceImpl.java` (Gateway) — el mapeo de `fechaAlta` de `Request` a `Dto` es
automático vía `ModelMapper`, sin código adicional necesario.

**Verificación:** `mvn compile` en verde (`gda-persona`, `gda-persona-commons`, `api-gateway`), 15/15
tests de `ProveedorServiceImplTest` sin regresión.

**Paso 6 — Revisión de ADR:** [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md)
reescrito: eleva el patrón de 4 columnas de auditoría de "recomendado" a **obligatorio para toda tabla
nueva** del backend GDA (petición explícita del usuario, ver `Documentacion_BBDD_GDA_2607_Suppliers.md` §8).

==========================================
16. REFERENCIAS
==========================================

- Documento hermano (frontend): [Documentacion_FrontEnd_GDA_2607_Suppliers.md](Documentacion_FrontEnd_GDA_2607_Suppliers.md)
- Esquema de BD y criterio de campos de auditoría: [Documentacion_BBDD_GDA_2607_Suppliers.md](Documentacion_BBDD_GDA_2607_Suppliers.md)
- Patrón de referencia de campos de auditoría (Iniciativa): [Iniciativa.java](../../GDA/backend/gda-estructura/gda-estructura/src/main/java/com/ineco/gda/estructura/data/model/Iniciativa.java), [IniciativaServiceImpl.java](../../GDA/backend/gda-estructura/gda-estructura/src/main/java/com/ineco/gda/estructura/service/impl/IniciativaServiceImpl.java), [IniciativaServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/estructura/impl/IniciativaServiceImpl.java)
- Backend real de Suppliers (gda-persona): [ProveedorController.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/ProveedorController.java), [ProveedorService.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/ProveedorService.java), [ProveedorServiceImpl.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/impl/ProveedorServiceImpl.java), [Proveedor.java (entity)](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/Proveedor.java), [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java)
- Composición en el Gateway: [ProveedorController.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/persona/ProveedorController.java), [ProveedorService.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/ProveedorService.java), [ProveedorServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/impl/ProveedorServiceImpl.java), [ProveedorRequest.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/persona/request/ProveedorRequest.java), [GdaPersonaApiClient.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/clients/GdaPersonaApiClient.java), [GlobalExceptionHandler.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/exception/GlobalExceptionHandler.java)
- Patrón de referencia (Guardias, escritura ya funcional): [GuardiaController.java (gda-admingade)](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/web/controller/GuardiaController.java), [GuardiaController.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/admingade/GuardiaController.java), [GuardiaServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/GuardiaServiceImpl.java)
- Precedente de `IllegalArgumentException` sin traducción a 400 (Hallazgo 3, Fase 2): [RecursoServiceImpl.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/service/impl/RecursoServiceImpl.java), [PedidoServiceImpl.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/service/impl/PedidoServiceImpl.java)
- Excepciones reutilizables: [ObjectAlreadyExistException.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/exception/ObjectAlreadyExistException.java), [ObjectNotFoundException.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/exception/ObjectNotFoundException.java)
- [knowledge-graph-suppliers.json](knowledge-graph-suppliers.json)
- [REQUISITOS_MODULO_SUPPLIERS.md](REQUISITOS_MODULO_SUPPLIERS.md)
- [MAPA_VISUAL_SUPPLIERS.md](MAPA_VISUAL_SUPPLIERS.md)
- [RESUMEN_EJECUTIVO_SUPPLIERS.md](RESUMEN_EJECUTIVO_SUPPLIERS.md)
- ADRs verificados: [ADR-0600](../../docs/decisiones/ADR-0600-estrategia-despliegue.md),
  [ADR-0601](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md),
  [ADR-0602](../../docs/decisiones/ADR-0602-comunicacion-interservicios.md),
  [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md),
  [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md),
  [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) (nuevo, Fase 4),
  [ADR-0801](../../docs/decisiones/ADR-0801-estrategia-empaquetado-despliegue.md),
  [ADR-0802](../../docs/decisiones/ADR-0802-integracion-front-back.md),
  [ADR-0803](../../docs/decisiones/ADR-0803-seguridad-secretos.md)
