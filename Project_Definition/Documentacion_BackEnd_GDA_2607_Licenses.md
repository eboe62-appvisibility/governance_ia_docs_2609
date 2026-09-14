# Documentación BackEnd GDA — Módulo Licencias
Versión: 2607 (2026-07-24, revisada 2026-07-25, revisada 2026-07-26) · Basado en:
`Documentacion_BackEnd_GDA_2607_Suppliers.md`
(mismo modelo de desarrollo) y `Documentacion_FrontEnd_GDA_2607_Licenses.md` (completo, ya construido)
· Backend real: entidad `Licencia`, **nueva de principio a fin** en `gda-admingade` (a diferencia de
Suppliers/`Proveedor`, que ya existía en `gda-persona`) · Estado: **Backend CRUD completo** (GET/POST/PUT/
DELETE) en las 3 capas (`gda-persona`, `gda-admingade`, API Gateway), compilación en verde en los 3
módulos, **verificado de extremo a extremo en el navegador y confirmado por el usuario en su totalidad**
(2026-07-25 el CRUD base; 2026-07-26 el ciclo de "Baja Definitiva" y el filtro de Estados). **Subida a
GitLab de los 4 repositorios en curso, gestionada manualmente por el usuario.**

==========================================
1. INTRODUCCIÓN GENERAL
==========================================

Este documento recoge la construcción del backend del módulo **Licencias**: a diferencia de Suppliers
(ampliar 5 archivos ya existentes de `Proveedor` con create/update/delete), aquí no existía absolutamente
nada — ni entidad, ni repositorio, ni servicio, ni controller, en ningún microservicio. Es el documento
complementario de `Documentacion_FrontEnd_GDA_2607_Licenses.md` (que cubre el listado y los formularios
Añadir/Editar/Consultar/Eliminar construidos en `GDA/frontend/src/modules/gestionAdmingade/`).

**Fase 1 (sección 6): GET+POST**, replicando solo lo que el frontend ejercitaba en ese momento (listado
y Añadir). **Fase 2 (sección 7): PUT+DELETE**, añadidos cuando el frontend incorporó Editar/Consultar
(que necesitaban `PUT` para guardar) y Eliminar (que necesitaba `DELETE`) — con esto el backend queda con
las cuatro operaciones completas, simétrico con el frontend.

==========================================
2. INVESTIGACIÓN PREVIA: PATRÓN DE REFERENCIA
==========================================

A diferencia de Suppliers (cuyo patrón de referencia para el backend de escritura fue `Guardia` solo por
analogía), aquí `Guardia` es el patrón de referencia **directo y más cercano posible**: es la entidad
nativa de `gda-admingade` con CUD completo, con FKs cruzadas hacia `gda_persona` (`personaId`,
`proveedorId`). Verificado en código real antes de escribir nada:

- **Nunca hay `@ManyToOne` ni `@FeignClient` dentro de `gda-admingade` hacia otro esquema.** Los FK
  cruzados de `Guardia` (`personaId`, `proveedorId`) son campos `Long` planos, con un comentario explícito
  en el código ("Proveedor y Recurso no tienen relación de entidad"). Cero `@FeignClient` en todo
  `gda-admingade` — la resolución a objetos completos ocurre **solo** en el API Gateway.
- **Confirmado también a nivel de datasource:** `gda-admingade` y `gda-persona` comparten la misma
  instancia física de PostgreSQL (`172.16.22.114:5432/gda_03_desa`), solo cambia `currentSchema`. Un JOIN
  nativo cross-schema sería técnicamente posible, pero **no existe ni un solo precedente** de ello en el
  código — la convención exclusiva es composición vía Feign en el Gateway. Se sigue esa convención para
  Licencia, no el atajo técnico.
- `GuardiaServiceImpl` (Gateway) resuelve `personaId`/`proveedorId` con un fan-out de
  `CompletableFuture.supplyAsync(...)` (uno por Feign client) + `HashMap<Long, Dto>` por id, `.join()`eados
  antes de construir la lista de respuesta. Mismo patrón aplicado a `Licencia` (sección 5).

**Hallazgo crítico, con impacto directo en el frontend ya construido (Fases 1-2):** `licencia.estado_id`/
`localizacion_id` referencian `gda_persona.maestro_estado`/`maestro_localizacion` — entidades
**distintas** de las que ya reutiliza el resto de la app para conceptos con nombre parecido:
`Localizacion`/`LocalizacionDto` (campo `descripcion`, tabla distinta, ya expuesta como
`GET /localizaciones`) y el "Estado" genérico de `modelState.js` (también `descripcion`).
`MaestroEstadoDto`/`MaestroLocalizacionDto` (verificado leyendo el código real de `gda-persona`) usan
**`nombre`**, no `descripcion`. El frontend de la Fase 1/2 (que reutilizaba `modelLocation`/`modelState`/
`useFetchLocations`, pensados para las entidades genéricas) apuntaba a la entidad equivocada — con este
backend real habría mostrado Localización/Estado en blanco. Corregido en el mismo turno que este backend
(ver `Documentacion_FrontEnd_GDA_2607_Licenses.md`, sección de correcciones).

**Estado de partida de los maestros, verificado antes de tocar nada:**
- `MaestroLocalizacion`: servicio (`MaestroLocalizacionServiceImpl.findAll()`) y repositorio ya
  funcionando, pero su único controller (`POST /getMaestroLocalizacion`) **no está enrutado por el
  Gateway** — inalcanzable desde fuera de `gda-persona` hasta ahora.
- `MaestroEstado`: entidad y repositorio ya existían, pero `MaestroEstadoService`/`ServiceImpl`/
  `Controller` estaban **completamente vacíos** (stubs sin ningún método). Además,
  `MaestroEstadoServiceImpl.maestroEstadoRepository` no era `final`, así que `@RequiredArgsConstructor`
  no lo habría inyectado — bug latente en código que nunca se había llegado a ejecutar.

==========================================
3. ARQUITECTURA (RESUMEN)
==========================================

```
Frontend            Licenses.vue → useFetchLicenses/useSaveLicense → admingadeServices.{get,post}License
                     licenseForm.vue → useFetchStates/useFetchMaestroLocalizacion (dominio gestionAdmingade)
                          │
API Gateway (composición, ADR-0604)
    LicenciaController (Gateway) — admingade/licencias, admingade/maestrosEstado, admingade/maestrosLocalizacion
      → LicenciaServiceImpl: getLicencias() = fan-out CompletableFuture (gdaAdmingadeApiClient.getLicencias()
        + gdaPersonaApiClient.getMaestrosEstado() + getMaestrosLocalizacion()) + HashMap por id →
        LicenciaResponse (estado/localizacion anidados, patrón GuardiaServiceImpl.getGuardias()).
        addLicencia() = ReactiveSecurityContextHolder → usuarioModificacionId, Mono<LicenciaDto>,
        patrón GuardiaServiceImpl.addGuardia()
                          │
gda-admingade        LicenciaController (GET/POST /licencias) → LicenciaService/Impl
                      (ObjectAlreadyExistException si id ya existe; fechaAlta/fechaModificacion
                      automáticas) → LicenciaRepository → Licencia (entity, FKs Long planos, sin @ManyToOne)
                          │
gda-persona          MaestroEstadoController (+ GET /maestrosEstado, nuevo) → MaestroEstadoService/Impl
                      (findAll() implementado, bug de inyección corregido) → MaestroEstadoRepository (ya existía)
                      MaestroLocalizacionController (+ GET /maestrosLocalizacion, nuevo; el POST
                      /getMaestroLocalizacion existente se deja intacto, sin consumidores) → Service ya existente
                          │
Base de datos        gda_admingade.licencia (ya existía) · gda_persona.maestro_estado/maestro_localizacion
                      (ya existían, sin cambios de esquema)
```

**Decisión — sin ruta nueva en `RouteLocatorConfig.java` para `admingade/licencias`:** `GuardiaController`
del Gateway convive con una ruta activa `admingade/guardias/**` en ese archivo, pero Suppliers documentó
explícitamente que activar una ruta de paso a la vez que un controller de composición en la misma ruta
genera conflicto de enrutamiento (por eso la ruta `proveedores` sigue comentada). Se sigue el criterio más
cauto y más reciente de Suppliers: **ningún** route nuevo en `RouteLocatorConfig.java` para Licencia; todo
pasa por el `@RestController` de composición del Gateway.

**Decisión — endpoints de maestros nuevos con estilo REST limpio, sin imitar el `POST /getMaestroLocalizacion`
existente:** ese endpoint es un outlier de estilo (verbo en la ruta, método POST para una lectura) sin
consumidores reales. Los nuevos `GET /maestrosEstado`/`GET /maestrosLocalizacion` usan el mismo estilo
plano que el resto del backend (`/guardias`, `/localizaciones`, `/proveedores`). El endpoint legado no se
toca ni se elimina (fuera de alcance, cero riesgo por no tener consumidores).

**Decisión — maestros expuestos al frontend vía el mismo `LicenciaController` del Gateway:** en vez de un
módulo de maestros aparte, `admingade/maestrosEstado`/`admingade/maestrosLocalizacion` son dos endpoints
adicionales, pequeños, en el mismo controller de Licencias (passthrough directo a `gdaPersonaApiClient`,
sin composición) — sirven al desplegable del formulario. La resolución interna para el LISTADO (dentro de
`LicenciaServiceImpl.getLicencias()`) es independiente y no depende de estas rutas — llama a los Feign
directamente.

==========================================
4. ESTRUCTURA DE ARCHIVOS
==========================================

### `gda-persona` (rama nueva `GDAV3-446`, creada desde `develop` en esta fase)

| Archivo | Ruta | Cambio |
|---|---|---|
| `MaestroEstadoService.java` | `.../service/` | `+ List<MaestroEstadoDto> findAll();` |
| `MaestroEstadoServiceImpl.java` | `.../service/impl/` | `maestroEstadoRepository` → `final` (bug de inyección corregido); `+ findAll()` (idéntico a `MaestroLocalizacionServiceImpl.findAll()`) |
| `MaestroEstadoController.java` | `.../web/controller/` | `+ @GetMapping("/maestrosEstado")` |
| `MaestroLocalizacionController.java` | `.../web/controller/` | `+ @GetMapping("/maestrosLocalizacion")` (nuevo, junto al `POST /getMaestroLocalizacion` ya existente, sin tocarlo) |

No se modifica `MaestroLocalizacionService`/`ServiceImpl` (ya tenían `findAll()`), ni
`MaestroEstadoRepository` (ya extendía `JpaRepository`), ni ningún `Dto`/entidad (ya tenían los campos
necesarios: `id`/`nombre`/`observaciones`, y `ambito` en el caso de `MaestroEstado`).

### `gda-admingade` + `gda-admingade-commons` (mismo repo git, ya en `GDAV3-446`)

| Archivo | Ruta | Cambio |
|---|---|---|
| `Licencia.java` (entity) | `gda-admingade/.../data/model/` | Nuevo — `id`, `estadoId`/`localizacionId`/`usuarioModificacionId` (`Long` planos, sin `@ManyToOne` — viven en `gda_persona`), `licencia` (`String`), `expteCompra` (`Boolean`), `observaciones`, `fechaAlta`/`fechaModificacion`/`fechaBaja` (`LocalDate`) |
| `LicenciaRepository.java` | `.../data/repository/` | Nuevo — `extends JpaRepository<Licencia, Long>`, sin queries custom |
| `LicenciaService.java` / `LicenciaServiceImpl.java` | `.../service/`, `.../service/impl/` | Nuevo — `findAll()`; `addLicencia()` patrón `addGuardia`: `ObjectAlreadyExistException` si `id` ya existe, `fechaModificacion = LocalDate.now()` (`fechaAlta` ya no se fuerza, viene del DTO desde la Fase 2, sección 7); `editLicencia()` patrón `editProveedor`: `ObjectNotFoundException` si `id` no existe, preserva `fechaAlta`/`fechaBaja` originales leídos de BD, `fechaModificacion = LocalDate.now()`; `deleteLicencia()` patrón `deleteGuardia` (DELETE físico) |
| `LicenciaController.java` | `.../web/controller/` | Nuevo — `GET`/`POST`/`PUT`/`DELETE /licencias` (patrón `GuardiaController`: catch + rethrow `ObjectAlreadyExistException`/`ObjectNotFoundException`, `DataIntegrityViolationException` → 400 en `PUT`, catch genérico → 500) |
| `LicenciaDto.java` | `gda-admingade-commons/.../dto/` | Nuevo — mismos campos que la entidad; las 3 fechas con `@JsonSerialize`/`@JsonDeserialize` (`LocalDateSerializer`/`LocalDateDeserializer`) **desde el principio** — lección directa del Hallazgo 1 de Suppliers (`Documentacion_BackEnd_GDA_2607_Suppliers.md` §14), donde faltar esta anotación hizo que las fechas se perdieran silenciosamente al pasar por Feign |

`ObjectAlreadyExistException`/`ObjectNotFoundException` (ya en `gda-admingade-commons`) se reutilizan sin
cambios. `GlobalExceptionHandler.java` (ya genérico por tipo de excepción) no requiere cambios.

### `api-gateway` (ya en `GDAV3-446`)

| Archivo | Cambio |
|---|---|
| `web/clients/GdaPersonaApiClient.java` | `+ getMaestrosEstado()` → `GET /maestrosEstado`; `+ getMaestrosLocalizacion()` → `GET /maestrosLocalizacion` |
| `web/clients/GdaAdmingadeApiClient.java` | `+ getLicencias()`/`addLicencia()`/`editLicencia()`/`deleteLicencia()` → `GET`/`POST`/`PUT`/`DELETE /licencias` (bajo comentario `//Licencia`) |
| `dto/admingade/request/LicenciaRequest.java` | Nuevo — `id`, `estadoId`, `localizacionId`, `licencia`, `expteCompra`, `observaciones`, `fechaAlta` (`@JsonFormat(pattern="dd/MM/yyyy")` — añadido en la Fase 2, sección 7, cuando `fechaAlta` pasó a ser un campo del formulario) — coincide exactamente con el payload que envía `modelBackLicense()` del frontend |
| `dto/admingade/response/LicenciaResponse.java` | Nuevo — `id`, `licencia`, `localizacion` (`MaestroLocalizacionDto` anidado), `estado` (`MaestroEstadoDto` anidado), `expteCompra`, `observaciones`, `fechaAlta`, `fechaModificacion`, `fechaBaja` (fechas con `@JsonFormat(pattern="dd/MM/yyyy")`, patrón `GuardiaResponse`) |
| `service/admingade/LicenciaService.java` / `service/admingade/impl/LicenciaServiceImpl.java` | Nuevo — `getLicencias()` (fan-out `CompletableFuture` + `HashMap`, patrón `GuardiaServiceImpl.getGuardias()`); `addLicencia()`/`editLicencia(LicenciaRequest)` (`ReactiveSecurityContextHolder` → `usuarioModificacionId`, `Mono<LicenciaDto>`, patrón `GuardiaServiceImpl.addGuardia()`); `deleteLicencia(Long)` (`Mono<Void>`, patrón `ProveedorServiceImpl.deleteProveedor()` — `.doOnNext(...).then()`, ver sección 7); `getMaestrosEstado()`/`getMaestrosLocalizacion()` (passthrough) |
| `web/controller/admingade/LicenciaController.java` | Nuevo — `GET`/`POST`/`PUT`/`DELETE admingade/licencias`, `GET admingade/maestrosEstado`, `GET admingade/maestrosLocalizacion` |

No se toca `RouteLocatorConfig.java` (decisión de la sección 3) ni `SecurityConfiguration.java`
(`/admingade/**` ya cubre las rutas nuevas con los mismos roles que el resto de "Gestión Admingade":
`ADMINGADE`, `ADMINGADE_RSP_EQP`, `GDA_SUPERGESTOR`, `ADMINGADE_MANAGER`).

==========================================
5. VERIFICACIÓN DE CUMPLIMIENTO CON LOS ADR (docs/decisiones)
==========================================

| ADR | Requisito clave | ¿Cumple el backend de Licencias? |
|-----|------------------|----------------------------------|
| [ADR-0600](../../docs/decisiones/ADR-0600-estrategia-despliegue.md) | Cada módulo backend es un artefacto independiente | ✅ Se amplía `gda-admingade` (módulo ya existente); no se crea artefacto Maven nuevo |
| [ADR-0601](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md) | DTOs en `-commons`; entidades/repos/services/controllers en el módulo principal | ✅ `LicenciaDto` en `gda-admingade-commons`; `Licencia`/`LicenciaRepository`/`LicenciaService`/`LicenciaController` en `gda-admingade` — mismo patrón que `GuardiaDto`/`Guardia` |
| [ADR-0602](../../docs/decisiones/ADR-0602-comunicacion-interservicios.md) | REST/JSON, rutas planas sin versionar | ✅ `GET`/`POST /licencias`, `GET /maestrosEstado`, `GET /maestrosLocalizacion` — mismo estilo que `/guardias`/`/proveedores` |
| [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) | Schema propio por servicio, sin joins cruzados | ✅ Sin cambios de esquema; ningún JOIN cross-schema pese a ser técnicamente posible (mismo datasource) — resolución vía Feign en el Gateway, igual que `Guardia` |
| [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md) | Composición dedicada cuando el recurso necesita datos de otro microservicio | ✅ `Licencia` necesita `maestro_estado`/`maestro_localizacion` de `gda-persona`; el Gateway implementa la composición dedicada (`LicenciaServiceImpl`, fan-out + `HashMap`), mismo patrón que `GuardiaServiceImpl`. Se usa `LicenciaRequest` propio desde el principio (no reutiliza `LicenciaDto` como entrada), igual que la decisión final de Suppliers §4 |
| [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) | 4 columnas de auditoría obligatorias en toda tabla nueva | ✅ **Cumple de partida** — `licencia` ya nace con `fecha_alta`/`fecha_modificacion`/`fecha_baja`/`usuario_modificacion_id` con el criterio final (FK a `gda_persona.persona(id)`); sin necesidad de la migración correctiva que sí hizo falta para `proveedor` |
| [ADR-0801](../../docs/decisiones/ADR-0801-estrategia-empaquetado-despliegue.md) | JAR ejecutable vía CI Maven | ✅ Sin cambios de empaquetado |
| [ADR-0802](../../docs/decisiones/ADR-0802-integracion-front-back.md) | Todo vía API Gateway, manejo de errores centralizado | ✅ El frontend sigue llamando solo al Gateway; mismo `notifications.js`/`axiosCall.js` |
| [ADR-0803](../../docs/decisiones/ADR-0803-seguridad-secretos.md) | Auth JWT stateless, sin datos sensibles sin cifrar | ✅ Sin cambios; `Licencia` no almacena datos de persona física |

No se requiere ningún ADR nuevo — se reutilizan todos los patrones ya verificados con Suppliers, aplicados
esta vez a una entidad nativa de `gda-admingade` en vez de una ampliación de `gda-persona`.

==========================================
6. ESTADO REAL DE IMPLEMENTACIÓN — BACKEND GET+POST (2026-07-24)
==========================================

**Resumen:** construido de cero el backend de Licencia en las 3 capas (`gda-persona` para los maestros,
`gda-admingade` para la entidad propia, API Gateway para la composición), cerrando el hueco que dejaba
pendiente `Documentacion_FrontEnd_GDA_2607_Licenses.md` (Fases 1-2, frontend sin backend real detrás).
Alcance explícitamente limitado a GET+POST (sección 1).

**Verificación de compilación (todas en verde antes de dar la fase por completada):**
1. `mvn -pl gda-persona-commons,gda-persona -am compile` (repo `gda-persona`) — sin errores.
2. `mvn -pl gda-admingade,gda-admingade-commons -am compile` (repo `gda-finance`) — sin errores. Se
   necesitó además `mvn -pl gda-admingade-commons -am install` para que el API Gateway pudiera resolver
   `LicenciaDto` desde el `.m2` local (la versión `4.7.0-SNAPSHOT` ya estaba pineada localmente en el
   `pom.xml` del Gateway desde antes de esta fase, sin relación con Licencias).
3. `mvn clean compile` (repo `api-gateway`) — sin errores, tras el `install` anterior.

**Corrección de un hallazgo real, no introducido por Licencias:** `MaestroEstadoServiceImpl.
maestroEstadoRepository` no era `final`, así que `@RequiredArgsConstructor` habría generado un
constructor sin argumentos y Spring nunca lo habría inyectado — el bug nunca se manifestó porque el
servicio no tenía ningún método implementado hasta ahora. Corregido al implementar `findAll()`.

**Paso 4 (subida a GitLab):** pendiente en los 3 repos — la ejecuta el usuario manualmente
(`git add`/`commit`/`push`), igual que en todo el trabajo de Suppliers. No se ha ejecutado ningún comando
git de escritura desde esta conversación, salvo la creación local de la rama `GDAV3-446` en `gda-persona`
(`git checkout -b`, sin push).

**Paso 6 (revisión de ADR):** ver sección 5. No se crea ningún ADR nuevo — todos los patrones aplicados
ya estaban verificados y documentados para Suppliers; el único matiz nuevo (FK cruzada resuelta sin ningún
intento de acceso directo a otro schema, pese a compartir datasource físico) es evidencia adicional para
ADR-0603, no un patrón que requiera ADR propio.

**Pendiente (al cerrar esta fase):** verificación end-to-end real en local; `PUT`/`DELETE` de Licencia
(resuelto en la Fase 2, sección 7, más abajo); validación de campos obligatorios en servidor (sigue
pendiente, ver sección 7); subida a GitLab de los 3 repos, por el usuario.

==========================================
7. ESTADO REAL DE IMPLEMENTACIÓN — FASE 2 (PUT+DELETE) (2026-07-25)
==========================================

**Resumen:** el frontend incorporó Editar/Consultar (`Documentacion_FrontEnd_GDA_2607_Licenses.md` §10)
y Eliminar, que necesitaban `PUT`/`DELETE` reales para funcionar — hasta entonces solo existían GET/POST
(sección 6). Se amplían las mismas 3 capas, sin tocar nada de lo ya construido en la Fase 1.

**Archivos modificados (ningún archivo nuevo en esta fase):**

| Repo | Archivo | Cambio |
|---|---|---|
| `gda-admingade` | `LicenciaService.java`/`LicenciaServiceImpl.java` | `+ editLicencia()` (patrón `editProveedor`: `ObjectNotFoundException` si `id` no existe o es `null`; preserva `fechaAlta`/`fechaBaja` del registro original leído de BD antes de guardar, ignorando lo que llegue en el DTO); `+ deleteLicencia()` (patrón `deleteGuardia`, DELETE físico vía `deleteById`) |
| `gda-admingade` | `LicenciaController.java` | `+ @PutMapping("/licencias")` (catch + rethrow `ObjectNotFoundException`, `DataIntegrityViolationException` → 400, genérico → 500 — mismo bloque que `GuardiaController.editGuardia`); `+ @DeleteMapping("/licencias/{licenciaId}")` (patrón `GuardiaController.deleteGuardia`) |
| `gda-admingade` | `LicenciaServiceImpl.addLicencia()` | Ya **no** fuerza `fechaAlta = LocalDate.now()` — ahora viene del `LicenciaDto` entrante (el frontend la manda siempre, ver frontend §10); sigue fijando `fechaModificacion = LocalDate.now()` |
| `api-gateway` | `GdaAdmingadeApiClient.java` | `+ editLicencia(LicenciaDto)` → `PUT /licencias`; `+ deleteLicencia(Long)` → `DELETE /licencias/{id}` |
| `api-gateway` | `LicenciaRequest.java` | `+ fechaAlta` (`@JsonFormat(pattern="dd/MM/yyyy")`) — el `Request` ya podía llevarla desde el diseño inicial (sección 4), pero hasta ahora el frontend nunca la enviaba |
| `api-gateway` | `LicenciaService.java`/`LicenciaServiceImpl.java` | `+ editLicencia(LicenciaRequest)` (mismo patrón que `addLicencia`: resuelve `usuarioModificacionId` desde `ReactiveSecurityContextHolder`, `Mono<LicenciaDto>`); `+ deleteLicencia(Long)` (`Mono<Void>`, patrón `ProveedorServiceImpl.deleteProveedor()`: `ReactiveSecurityContextHolder.getContext().doOnNext(sc -> gdaAdmingadeApiClient.deleteLicencia(id)).then()`) |
| `api-gateway` | `LicenciaController.java` | `+ @PutMapping("admingade/licencias")`; `+ @DeleteMapping("admingade/licencias/{id}")` (patrón `ProveedorController` — rama `GDAV3-443`, consultada vía `git show` porque el `GDAV3-446` de `api-gateway` no incluye el CUD de Suppliers, solo `develop`) |

**Decisión — patrón de referencia para `DELETE` en el Gateway, no `Guardia`:** `GuardiaController`
(Gateway) no expone ningún `DELETE` (aunque `gda-admingade` sí tiene `deleteGuardia`) — el Gateway de
Guardias nunca compuso esa operación porque el frontend de Guardias no la necesitaba. Para Licencia sí
hace falta (el frontend ya tiene un botón Eliminar funcional), así que el patrón de referencia real es
`ProveedorServiceImpl.deleteProveedor()`/`ProveedorController.deleteProveedor()` (Suppliers, rama
`GDAV3-443` de `api-gateway`), el único `DELETE` de composición ya existente en el proyecto.

**Decisión inicial (2026-07-25) — corregida al día siguiente, ver sección 8:** se asumió inicialmente
que `licencia.estado_id` (a diferencia de Proveedor, sin ningún concepto de estado) no tenía ningún
valor "terminal" en `maestro_estado`, y se adoptó el criterio de Proveedor (`fecha_baja` intocable vía
API). Esa asunción resultó incorrecta — el usuario confirmó que sí existe un valor terminal real
("Baja Definitiva") y que `fecha_baja` debe cargarse con la fecha del día al seleccionarlo en Editar.
El detalle completo de la corrección (código y ADR) está en la sección 8.

**Verificación de compilación:** `mvn -pl gda-admingade,gda-admingade-commons -am compile` (repo
`gda-finance`) y `mvn clean compile` (repo `api-gateway`) — ambos en verde, sin necesidad de un nuevo
`install` (no se tocó ningún DTO de `-commons` en esta fase).

**Paso 4 (GitLab):** pendiente en los 3 repos, gestionado manualmente por el usuario.

**Paso 6 (ADR):** ningún patrón arquitectónico *nuevo* — pero, a diferencia de lo anotado inicialmente en
esta sección, sí se actualizaron dos ADRs existentes con evidencia real de esta fase (no se crea ningún
ADR nuevo):
- [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md) — Licencia añadida como
  **tercera confirmación real** del patrón de composición del Gateway, con dos matices no cubiertos por
  Guardias/Proveedores: el `DELETE` reactivo replica `ProveedorServiceImpl` (no `Guardia`, que nunca lo
  implementó), y `LicenciaController` combina composición real (`admingade/licencias`) con dos endpoints
  de passthrough puro (`admingade/maestrosEstado`/`maestrosLocalizacion`) en el mismo controller.
- [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) §4 — Licencia añadida como
  tercer caso real del patrón de auditoría, y como primer caso documentado donde la decisión de
  `fecha_baja` (intocable vía API vs. ligada a `estado_id`) no era obvia por tener la entidad su propio
  campo `estado_id` — la decisión tomada en esta fase (intocable, estilo Proveedor) quedó registrada
  explícitamente en el propio ADR. **Nota (2026-07-26): esta decisión resultó incorrecta y fue
  revertida al día siguiente — ver sección 8, punto 1, y la revisión correspondiente de este mismo ADR.**
  Se deja el texto original de esta sección sin reescribir, como registro histórico de la fase; la
  sección 8 documenta la corrección real.

**Verificación end-to-end confirmada por el usuario (2026-07-25):** listado, Añadir, Editar, Consultar y
Eliminar probados en el navegador contra los 3 backends locales (con el redirect temporal de Feign a
`localhost` en `application-local.yml` del Gateway) — funcionando correctamente de extremo a extremo.

**Pendiente:**
- Validación de campos obligatorios en servidor (`estadoId`/`localizacionId`/`licencia`) — el frontend ya
  valida client-side; server-side sigue como mejora futura, igual que se documentó para Suppliers (sección
  5, Hallazgo de `encargoId`).
- Revertir `application-local.yml` (Gateway) a las URLs remotas antes de subir a GitLab — es config de
  entorno local, no forma parte del commit de esta fase, igual que hizo Suppliers.
- Subida a GitLab de los 3 repos, por el usuario.

**Con esto, el backend de Licencia queda con las cuatro operaciones (GET/POST/PUT/DELETE) construidas en
las 3 capas, simétrico con el frontend, y verificado de extremo a extremo en el navegador.**

==========================================
8. CORRECCIONES POST-VERIFICACIÓN (2026-07-26): FECHA BAJA Y ESTADOS REPETIDOS
==========================================

Tras confirmar el CRUD funcionando de extremo a extremo (sección 7), el usuario reportó probando el
formulario Editar dos cosas: (1) un requisito funcional no cubierto — `fecha_baja` debe cargarse con la
fecha del día al seleccionar el estado **"Baja Definitiva"**; (2) un bug — el desplegable Estado muestra
valores repetidos.

**1. Fecha Baja al seleccionar "Baja Definitiva" — corrige la Decisión de la sección 7:**

La Fase 2 (sección 7) asumió, sin confirmarlo explícitamente, que ningún valor de `maestro_estado` era
"terminal" para Licencia y adoptó el criterio de Proveedor (`fecha_baja` intocable). Era una asunción
incorrecta. Corrección aplicada, backend:

| Archivo | Cambio |
|---|---|
| `gda-admingade/LicenciaServiceImpl.editLicencia()` | Ya **no** preserva `fechaBaja` del registro original — se persiste tal cual llegue en el `LicenciaDto`. `fechaAlta` sigue preservándose sin cambios |
| `api-gateway/LicenciaRequest.java` | `+ fechaBaja` (`@JsonFormat(pattern="dd/MM/yyyy")`) — antes solo tenía `fechaAlta`; `LicenciaServiceImpl` (Gateway) no necesitó cambio, `ModelMapper` la mapea automáticamente al existir en ambos DTOs |

Decisión de diseño — **quién decide cuándo escribir `fecha_baja`:** a diferencia de Iniciativa
(`isStateClosedOrStopped` en `iniciativaForm.vue`, que solo habilita el campo para que el usuario lo
rellene manualmente, comparando por **ID** contra `ID_TYPES_STATES_CLOSED`/`ID_TYPES_STATES_STOPPED` de
`constants/common.js`), Licencia **autorrellena** `fecha_baja` con la fecha del día en el propio
frontend en el instante en que se selecciona el estado, comparando por **texto** (`description` del
estado contra el literal `"Baja Definitiva"`, no un ID). Motivo: no hay ningún ID de `maestro_estado`
documentado como estable/conocido de antemano (a diferencia del catálogo dedicado de Iniciativa), y
`maestro_estado` resultó ser una tabla compartida por `ambito` (ver punto 2) — anclarse a un ID concreto
sería frágil si el mismo `nombre` puede existir con distinto `id` según el entorno. El campo queda
también editable manualmente en ese momento (`:readonly="!isStatusBajaDefinitiva"` en
`licenseForm.vue`), por si el usuario necesita ajustarlo — mismo criterio de "editable, no solo
mostrado" que Iniciativa. Backend y Gateway no imponen ninguna regla sobre qué valor de `fecha_baja`
llega — confían en el frontend, igual que confían en él para `fecha_alta`.

**Verificación de compilación:** `mvn -pl gda-admingade,gda-admingade-commons -am compile` y
`mvn clean compile` (api-gateway) — ambos en verde.

**Paso 6 (ADR):** [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) §4 corregido —
la nota de Licencia ahora documenta el mecanismo real (estilo Iniciativa, decidido por el frontend por
texto) en vez de la asunción inicial (estilo Proveedor, intocable). Ningún ADR nuevo.

**2. Estados repetidos en el desplegable — resuelto (2026-07-26):**

Confirmada la hipótesis con el contenido real de `gda_persona.maestro_estado` (aportado por el usuario,
29 filas): la tabla es un catálogo **compartido por varios dominios** vía la columna `ambito`
(`Vacantes`, `Certificaciones`, `Candidatos`, `Vacante_Principal`, `Vacante_Situacion`, ...), con
restricción `UNIQUE (nombre, ambito)` — el mismo `nombre` puede repetirse legítimamente bajo distintos
`ambito` (p. ej. `Cancelada` existe con `ambito = 'Vacante_Principal'` **y** con `ambito` a `NULL`).
`MaestroEstadoServiceImpl.findAll()` (Fase 1, sección 6) no filtraba por `ambito`, así que el desplegable
de Licencia mostraba las 29 filas de todos los dominios mezcladas.

**Hallazgo clave:** no existe ningún valor de `ambito` literal para "Licencias" — las 4 filas sin
`ambito` asignado (`NULL`): `Pendiente de Incorporación`, `Activo`, `Baja Definitiva`, `Cancelada`, son
las que corresponden al dominio de Licencia (estados de negocio plausibles para el ciclo de vida de una
licencia, e incluyen literalmente el valor `Baja Definitiva` usado en el punto 1). Se filtra por
`ambito IS NULL`, no por un valor de texto.

**Archivos modificados (`gda-persona`):**

| Archivo | Cambio |
|---|---|
| `MaestroEstadoRepository.java` | `+ List<MaestroEstado> findByAmbitoIsNull();` (query derivada de Spring Data) |
| `MaestroEstadoService.java` / `MaestroEstadoServiceImpl.java` | `+ findByAmbitoIsNull()`, mismo mapeo que `findAll()` mediante `ModelMapper` pero sobre el resultado filtrado |
| `MaestroEstadoController.java` | `getMaestrosEstado()` (endpoint `GET /maestrosEstado`) pasa a llamar `findByAmbitoIsNull()` en vez de `findAll()` |

**Decisión — no se renombra el endpoint ni se añade un parámetro de filtro genérico:** `findAll()` se
deja intacto (sigue devolviendo las 29 filas sin filtrar) por si algún consumidor futuro necesita
literalmente todo el catálogo; se añade `findByAmbitoIsNull()` como método nuevo específico, en vez de
convertir `findAll()` en un filtro sorpresa. El endpoint `/maestrosEstado` sigue con el mismo nombre y
firma — es, hasta ahora, exclusivo del formulario de Licencias, así que no hace falta un parámetro de
`ambito` genérico todavía; si un futuro módulo necesita otro `ambito` (o el catálogo completo), se
decidirá entonces si conviene generalizar a `findByAmbito(String ambito)` o crear un endpoint propio.

**Verificación de compilación:** `mvn -pl gda-persona-commons,gda-persona -am compile` — en verde.

**Paso 6 (ADR) — punto 2:** [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) (gestión de
BBDD) ampliado con este hallazgo como nueva evidencia de deuda técnica: catálogos compartidos por varios
dominios (`ambito`) sin ningún filtro documentado, con recomendación explícita para módulos futuros que
consulten cualquier tabla `maestro_*`. No se crea ningún ADR nuevo — se trata como deuda técnica de
gestión de BBDD, mismo tratamiento que la desincronización de secuencia y la ausencia de migraciones
versionadas ya documentadas en ese ADR para Suppliers.

**Verificación end-to-end confirmada por el usuario en el navegador (2026-07-26):** seleccionar "Baja
Definitiva" en Editar carga Fecha Baja con la fecha del día y permite ajustarla hasta guardar; el
desplegable Estado muestra únicamente las 4 opciones relevantes para Licencia, sin repetidos. Requirió
reiniciar `gda-persona` y `api-gateway` (mismo motivo que en la Fase 2: Spring Boot no recarga mappings
ni queries derivadas nuevas en caliente).

**Paso 4 (GitLab):** en curso — el usuario está subiendo los 4 repositorios
(`gda-persona`, `gda-finance`, `api-gateway`, `GDA/frontend`) tras confirmar todo el módulo funcionando.

**Con esto, el módulo Licencias (backend + frontend) queda completo, verificado de extremo a extremo, y
sin ningún ADR pendiente de actualizar.**

==========================================
9. REFERENCIAS
==========================================

- Documento hermano (frontend): [Documentacion_FrontEnd_GDA_2607_Licenses.md](Documentacion_FrontEnd_GDA_2607_Licenses.md)
- Patrón de referencia (Guardia, entidad nativa de gda-admingade con CUD completo): [Guardia.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Guardia.java), [GuardiaController.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/web/controller/GuardiaController.java), [GuardiaServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/GuardiaServiceImpl.java), [GuardiaController.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/admingade/GuardiaController.java)
- Patrón de referencia para `DELETE` reactivo en el Gateway (Suppliers, único precedente real, rama `GDAV3-443` de `api-gateway`): `ProveedorServiceImpl.java`/`ProveedorController.java` (`.../service/persona/impl/`, `.../web/controller/persona/`), consultados vía `git show GDAV3-443:...`
- Entidad/DTO nuevos: [Licencia.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Licencia.java), [LicenciaDto.java](../../GDA/backend/gda-finance/gda-admingade-commons/src/main/java/com/ineco/gda/admingade/dto/LicenciaDto.java)
- Composición en el Gateway: [LicenciaController.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/admingade/LicenciaController.java), [LicenciaServiceImpl.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/LicenciaServiceImpl.java), [LicenciaRequest.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/admingade/request/LicenciaRequest.java), [LicenciaResponse.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/admingade/response/LicenciaResponse.java)
- Maestros (gda-persona): [MaestroEstadoController.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/MaestroEstadoController.java), [MaestroEstadoServiceImpl.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/impl/MaestroEstadoServiceImpl.java), [MaestroLocalizacionController.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/MaestroLocalizacionController.java)
- Excepciones reutilizables: [ObjectAlreadyExistException.java](../../GDA/backend/gda-finance/gda-admingade-commons/src/main/java/com/ineco/gda/admingade/exception/ObjectAlreadyExistException.java)
- Referencia del mismo modelo de desarrollo (Suppliers): [Documentacion_BackEnd_GDA_2607_Suppliers.md](Documentacion_BackEnd_GDA_2607_Suppliers.md), [Documentacion_BBDD_GDA_2607_Suppliers.md](Documentacion_BBDD_GDA_2607_Suppliers.md)
- ADRs verificados: [ADR-0600](../../docs/decisiones/ADR-0600-estrategia-despliegue.md),
  [ADR-0601](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md),
  [ADR-0602](../../docs/decisiones/ADR-0602-comunicacion-interservicios.md),
  [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md),
  [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md),
  [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md),
  [ADR-0801](../../docs/decisiones/ADR-0801-estrategia-empaquetado-despliegue.md),
  [ADR-0802](../../docs/decisiones/ADR-0802-integracion-front-back.md),
  [ADR-0803](../../docs/decisiones/ADR-0803-seguridad-secretos.md)
