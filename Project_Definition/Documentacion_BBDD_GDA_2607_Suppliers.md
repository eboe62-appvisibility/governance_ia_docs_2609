# Documentación BBDD GDA — Módulo Suppliers (campos de auditoría)
Versión: 2607 (2026-07-22, revisada varias veces: renombrado `persona_id` → `usuario_modificacion_id`
con corrección final de la FK a `gda_persona.persona(id)`; y renombrado `fecha_efecto` →
`fecha_modificacion` con inversión de semántica, ver §8) · Basado en:
`Documentacion_BackEnd_GDA_2607_Suppliers.md` §14 y `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §15
(mismo módulo, capas backend/frontend) · Tabla: `gda_persona.proveedor` · Estado: **`ALTER TABLE`
originales ejecutados por el usuario; `ALTER TABLE` de renombrado `usuario_modificacion_id` + FK (a
`gda_persona.persona`) pendiente de ejecutar, sin riesgo de datos — código ya adaptado a ambos cambios**

==========================================
1. INTRODUCCIÓN
==========================================

Este documento recoge la ampliación de esquema de `gda_persona.proveedor` con 4 campos de auditoría
(`persona_id`, `fecha_alta`, `fecha_baja`, `fecha_efecto`), decidida y ejecutada como parte del mismo
trabajo de Suppliers documentado en `Documentacion_BackEnd_GDA_2607_Suppliers.md` (§14, Fase 4) y
`Documentacion_FrontEnd_GDA_2607_Suppliers.md` (§15, Fase 7). Es el tercer documento de la serie Suppliers,
centrado exclusivamente en la decisión de esquema de BD y su criterio, para no diluir esa decisión
dentro de los documentos de backend/frontend ya extensos.

A diferencia del resto del backend/frontend de Suppliers (donde el `ALTER TABLE` original de la tabla ya
existía, ver `Documentacion_BackEnd_GDA_2607_Suppliers.md` §5), esta ampliación de esquema forma parte
íntegra de este trabajo — no una tabla preexistente, sino un cambio de esquema decidido y ejecutado
durante esta conversación.

==========================================
2. MODELO DE REFERENCIA: `gda_estructura.iniciativa`
==========================================

La decisión de partida fue explícita: replicar el mismo criterio de campos de auditoría ya usado por
`gda_estructura.iniciativa`, cambiando únicamente `username` por `persona_id`:

```sql
-- gda_estructura.iniciativa (campos de auditoría, extracto)
fecha_alta date NULL,
fecha_baja date NULL,
fecha_efecto date NULL,
username varchar(128) NULL,
```

Verificado contra el código real de `Iniciativa.java`/`IniciativaServiceImpl.java` (backend y Gateway),
el criterio real de estos 4 campos en Iniciativa es más matizado de lo que sugiere la DDL por sí sola
(ver sección 3): no los 4 se comportan igual, y no todos son responsabilidad exclusiva de backend.

==========================================
3. DDL APLICADA
==========================================

```sql
-- gda_persona.proveedor: ampliación con campos de auditoría (2026-07-22)

ALTER TABLE gda_persona.proveedor
    ADD COLUMN fecha_alta date NULL,
    ADD COLUMN fecha_baja date NULL,
    ADD COLUMN fecha_efecto date NULL,
    ADD COLUMN persona_id int8 NULL;
```

Ejecutada por el usuario directamente contra la BD compartida de desarrollo (`gda_03_desa`). Sin
cambios de permisos: el `GRANT` ya existente sobre la tabla completa (`postgres`/`userdesarrollo`, ver
`Documentacion_BackEnd_GDA_2607_Suppliers.md` §1) cubre las columnas nuevas sin necesidad de un `GRANT`
adicional.

**Ampliación posterior (mismo día, ver sección 7 para el histórico completo de la decisión):** se
renombró `persona_id` a `usuario_modificacion_id` y se le añadió una FK real — **decisión final**:
apunta a `gda_persona.persona(id)` (no a `gda_autenticacion.usuario(id)`, que fue una decisión
intermedia revertida el mismo día, ver sección 7):

```sql
-- Renombrar columna persona_id a usuario_modificacion_id
ALTER TABLE gda_persona.proveedor
RENAME COLUMN persona_id TO usuario_modificacion_id;

-- Añadir clave foránea relacionando usuario_modificacion_id con gda_persona.persona(id)
ALTER TABLE gda_persona.proveedor
ADD CONSTRAINT fk_proveedor_usuario
FOREIGN KEY (usuario_modificacion_id)
REFERENCES gda_persona.persona(id);
```

Sin riesgo de violación de FK: la columna se rellena con `personaId` directamente (ver sección 4), que
es exactamente el tipo de valor que esta FK final espera.

==========================================
4. CRITERIO POR CAMPO (DECISIONES EXPLÍCITAS)
==========================================

| Campo | Tipo | FK | ¿Quién lo rellena? | Editable vía API |
|---|---|---|---|---|
| `usuario_modificacion_id` (antes `persona_id`) | `int8 NULL` | **Con FK** a `gda_persona.persona(id)` (`fk_proveedor_usuario`) — pese al nombre de la columna ("usuario"), la FK final referencia a la persona de negocio, no a `gda_autenticacion.usuario`; esa fue una decisión intermedia revertida el mismo día (ver sección 7) | Backend (Gateway): se obtiene `personaId` del `SecurityContext` y se asigna **directamente**, sin resolución adicional (a diferencia de la decisión intermedia, que sí requería una llamada extra a `gda-autenticacion`) | **Sí, en alta y en cada edición** — coherente con el nombre de la columna ("última modificación"), diverge deliberadamente de `username` en Iniciativa (que solo se fija en el alta) |
| `fecha_alta` | `date NULL` | — | Backend (`gda-persona`), automático: `LocalDate.now()` en el alta; preservado sin cambios en cada edición | No — nunca expuesto en `ProveedorRequest` (Gateway) ni en el frontend, igual que `username`/`fechaAlta` en Iniciativa no se envían desde el formulario |
| `fecha_baja` | `date NULL` | — | **Nadie, vía aplicación** — decisión explícita: "la baja de un Proveedor sólo podrá ser gestionada directamente en BBDD" | **No, en ningún modo.** El backend preserva siempre el valor original en `editProveedor`, ignorando cualquier valor entrante. El frontend lo muestra en Editar/Consultar como solo lectura, únicamente informativo. Diverge de `fechaBaja` en Iniciativa, que sí es editable vía el PUT normal cuando el `estado` pasa a Cerrada/Detenida — mecanismo que Proveedor no tiene |
| `fecha_efecto` | `date NULL` | — | **Frontend**, campo obligatorio del formulario Añadir/Editar | Sí, en alta y edición — mismo criterio que `fechaEfecto` en Iniciativa (campo de negocio obligatorio) |

**Nota sobre el borrado (`DELETE /proveedores/{id}`):** se valoró inicialmente sustituir el borrado
físico por uno lógico (marcar `fecha_baja` en vez de `DELETE`), pero se descartó: no existe en ningún
módulo del proyecto un precedente real de "botón Eliminar que hace UPDATE en vez de DELETE" — ni
siquiera Iniciativa, que mantiene un `DELETE` físico real (`deleteIniciativa`) totalmente separado del
mecanismo de `fecha_baja` (que en Iniciativa se gestiona por el PUT normal, no por el borrado). Decisión
final: Proveedor mantiene el mismo `DELETE` físico real que ya tenía, sin relación con `fecha_baja`.

==========================================
5. HALLAZGO: SERIALIZACIÓN DE `LocalDate` EN CONSULTAS FEIGN
==========================================

Durante la verificación end-to-end se detectó que los 3 campos `LocalDate` no se guardaban ni se leían
correctamente a través del API Gateway, pese a que la columna y el mapeo JPA eran correctos. Causa:
`ACSFeignConfiguration.java` (Gateway) usa, para **todas** sus llamadas Feign, un `ObjectMapper` sin
módulo de fechas Java 8 registrado — cualquier `LocalDate` sin anotación de serialización explícita se
pierde silenciosamente al pasar por Feign. `gda-persona-commons` ya tenía resuelto este problema en 9
DTOs existentes mediante `@JsonSerialize`/`@JsonDeserialize` con clases propias
(`com.ineco.gda.persona.util.LocalDateSerializer`/`LocalDateDeserializer`); se aplicó el mismo patrón a
`ProveedorDto`. Detalle completo, con rutas y líneas exactas, en
`Documentacion_BackEnd_GDA_2607_Suppliers.md` §14 (Hallazgo 1) y en el nuevo
[ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md).

Esto no es un problema de esquema de BD, pero se documenta aquí porque cualquier campo `date`/`LocalDate`
que se añada en el futuro a **cualquier** tabla de `gda_persona` (o de cualquier otro módulo consumido
vía Feign por el Gateway) reproducirá el mismo fallo si no se replica esta anotación.

==========================================
6. VERIFICACIÓN DE CUMPLIMIENTO CON LOS ADR
==========================================

| ADR | Requisito clave | ¿Cumple esta ampliación de esquema? |
|-----|------------------|----------------------------------|
| [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) | Sin migraciones versionadas (Flyway/Liquibase), gestión manual de esquema | ✅ Coherente con el resto del backend — `ALTER TABLE` manual ejecutado directamente por el usuario, mismo patrón ya documentado como deuda técnica en este ADR (evidencia añadida en esta fase) |
| [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) | Patrón de campos de auditoría (nuevo, creado en esta fase) | ✅ Es el segundo caso real de aplicación de este patrón (tras Iniciativa), con las divergencias documentadas en la sección 4 |

No se requieren cambios en ADR-0600/0601/0602/0604/0801/0802/0803 desde el punto de vista específico
de esquema de BD (sí hay hallazgos de otras capas que sí los afectan, ver
`Documentacion_BackEnd_GDA_2607_Suppliers.md` §14 y §6 de ese documento).

==========================================
7. AMPLIACIÓN: RENOMBRADO A `usuario_modificacion_id` + FK A `gda_autenticacion.usuario` (2026-07-22)
==========================================

**Resumen:** tras la implementación inicial de la sección 3/4 (columna `persona_id`, sin FK), se decidió
que el campo debía referenciar formalmente al **usuario de autenticación** (`gda_autenticacion.usuario`),
no a la persona de negocio, y que su nombre lo reflejara (`usuario_modificacion_id`). Esto tiene una
repercusión real en código que no es un simple renombrado de columna: **el valor que el Gateway extraía
del `SecurityContext` (`personaId`) NO es el mismo ID que ahora exige la FK.**

**Hallazgo — `persona_id` (JWT) ≠ `usuario.id` (FK nueva):** verificado en código real
(`SecurityFilter.java`, `JwtServiceImpl.java`, `Usuario.java` de `gda-autenticacion`): el JWT emitido
en el login lleva como `subject` el `persona_id` (`usuario.getPersonaId().toString()`), y el Gateway
reconstruye el `UserDetails` de Spring Security usando ese mismo valor como "username"
(`SecurityFilter.java`). Es decir, `personaId` es el ID de `gda_persona.persona`, mientras que la FK
nueva exige el ID **propio** de `gda_autenticacion.usuario` (su PK, `usuario.id`) — dos columnas
distintas de la entidad `Usuario` (`id` y `persona_id`), verificado en
`Usuario.java`/`UsuarioDto.java` (`gda-autenticacion`/`gda-autenticacion-commons`). Guardar `personaId`
directamente en una columna con FK a `usuario(id)` habría violado la constraint en cuanto
`persona_id != usuario.id` para ese usuario (caso general, no una coincidencia garantizada).

**Corrección aplicada:** se añadió una resolución adicional vía Feign, reutilizando un patrón ya
existente en el Gateway (`GrupoServiceImpl`, `EquipoServiceImpl`, `UsuarioServiceImpl` ya hacen la
misma llamada para otros propósitos):
```java
Long personaId = Long.parseLong(((UserDetails) securityContext.getAuthentication().getPrincipal()).getUsername());
Long usuarioId = gdaAutenticacionApiClient.getUsuarioByPersona(personaId).getBody().getId();
proveedorDto.setUsuarioUltModificacionId(usuarioId);
```

**Archivos modificados (además de los ya descritos en `Documentacion_BackEnd_GDA_2607_Suppliers.md` §14):**

| Archivo | Cambio |
|---|---|
| [Proveedor.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/Proveedor.java) | `personaId` → `usuarioModificacionId` (`@Column(name = "usuario_modificacion_id")`) |
| [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java) | Mismo renombrado |
| [ProveedorServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/impl/ProveedorServiceImpl.java) | `+` dependencia `GdaAutenticacionApiClient` (ya existía, reutilizado el método `getUsuarioByPersona`, sin cambios ahí); `addProveedor`/`editProveedor` resuelven `usuarioId` antes de asignar |

**No afectados:** frontend (el campo nunca se muestra ni se envía desde `supplierForm.vue`/`supplier.js`,
sigue siendo 100% backend); `ProveedorRequest.java` (Gateway, sin cambios, nunca tuvo este campo).

**Riesgo de violación de FK (evaluado, sin materializarse):** se planteó el riesgo de que filas de
prueba ya guardadas violaran la FK a `usuario(id)` si `persona_id != usuario.id`. Comprobado por el
usuario con la consulta de verificación: la columna estaba completamente a `NULL` en todas las filas
existentes, así que no hubo ningún dato que limpiar antes de continuar.

**⚠️ CORRECCIÓN FINAL (mismo día, antes de ejecutar nada de lo anterior contra la FK a `usuario`):**
se decidió que el valor de la columna sea en realidad **`gda_persona.persona.id`**, no
`gda_autenticacion.usuario.id` — la FK definitiva es la del inicio de la sección 3
(`REFERENCES gda_persona.persona(id)`), **no** `REFERENCES gda_autenticacion.usuario(id)`. Esto revierte
por completo la necesidad de resolución vía `gdaAutenticacionApiClient`: `personaId` (obtenido del
`SecurityContext`, ver Hallazgo de esta misma sección) **ya es** directamente el valor correcto a
persistir, sin ningún paso intermedio.

**Código revertido en consecuencia:**
```java
// ANTES (decisión intermedia, ya no vigente):
// Long usuarioId = gdaAutenticacionApiClient.getUsuarioByPersona(personaId).getBody().getId();
// proveedorDto.setUsuarioModificacionId(usuarioId);

// AHORA (decisión final):
Long personaId = Long.parseLong(((UserDetails) securityContext.getAuthentication().getPrincipal()).getUsername());
proveedorDto.setUsuarioModificacionId(personaId);
```
Se retiró de `ProveedorServiceImpl.java` (Gateway) la dependencia `GdaAutenticacionApiClient`, añadida
solo para la decisión intermedia y ya sin ningún uso. `Proveedor.java`/`ProveedorDto.java` (gda-persona)
no necesitaron ningún cambio adicional — el nombre de campo (`usuarioModificacionId`) ya era correcto,
solo cambia a qué tabla apunta conceptualmente el valor que contiene.

**Paso 6 — Revisión de ADR:** [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md)
actualizado para reflejar la decisión final (FK a `gda_persona.persona(id)`, sin resolución adicional) y
dejar constancia de la decisión intermedia descartada, por si vuelve a plantearse en otro módulo.

==========================================
8. RENOMBRADO `fecha_efecto` → `fecha_modificacion`: INVERSIÓN DE SEMÁNTICA (2026-07-22)
==========================================

**DDL:**
```sql
-- La columna no contenía datos en ninguna fila (verificado antes de ejecutar)
ALTER TABLE gda_persona.proveedor
RENAME COLUMN fecha_efecto TO fecha_modificacion;
```

**Resumen:** no es un simple renombrado — invierte por completo la naturaleza del campo. `fecha_efecto`
era un campo de **negocio**, obligatorio y editable por el usuario en el formulario Añadir/Editar
(Fase 7, `Documentacion_FrontEnd_GDA_2607_Suppliers.md` §15). `fecha_modificacion` es un campo de
**auditoría puro**: se fija automáticamente a `LocalDate.now()` en **cada** alta y edición, nunca
editable desde el frontend ni desde el `Request` del Gateway. Al mismo tiempo, se introdujo un campo
nuevo en el formulario — **Fecha Alta** — que ocupa el hueco que dejó "Fecha Efecto" en el formulario
(no en la BD: `fecha_alta` ya existía como columna desde la Fase 4, pero hasta ahora era 100%
invisible/backend, igual que en Iniciativa).

**Cambio de criterio de `fecha_alta` (columna ya existente, comportamiento nuevo):** deja de ser
puramente backend — ahora es un campo del formulario de Añadir, con valor por defecto la fecha del día
(prellenado explícitamente por el frontend, no solo confiando en el comportamiento del selector de
fecha) pero **modificable por el usuario** antes de guardar. En Editar/Consultar se muestra siempre
deshabilitado, sin marca de obligatorio, y el backend sigue preservando su valor original en cada
edición (esto no cambia).

**Resumen de los 4 campos de auditoría finales de `proveedor` (y patrón obligatorio para toda tabla
nueva, ver ADR-0606 actualizado):**

| Campo | ¿Quién lo rellena? | ¿Cuándo cambia? |
|---|---|---|
| `fecha_alta` | Frontend (Añadir), con valor por defecto = hoy, editable por el usuario | Solo una vez, en el alta — inmutable después (preservado por el backend en cada edición) |
| `fecha_modificacion` | Backend, automático | En el alta **y** en cada edición posterior |
| `fecha_baja` | Nadie vía API — solo BBDD directa | Nunca vía aplicación |
| `usuario_modificacion_id` | Backend (Gateway), desde el `SecurityContext` | En el alta **y** en cada edición posterior |

**Archivos modificados (backend, Gateway y frontend):**

| Archivo | Cambio |
|---|---|
| [Proveedor.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/Proveedor.java) | `fechaEfecto` → `fechaModificacion` (`@Column(name = "fecha_modificacion")`) |
| [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java) | Mismo renombrado |
| [ProveedorServiceImpl.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/impl/ProveedorServiceImpl.java) | `addProveedor`: ya **no** fuerza `fechaAlta = LocalDate.now()` (ahora viene del DTO/frontend); `+ fechaModificacion = LocalDate.now()`. `editProveedor`: sigue preservando `fechaAlta`/`fechaBaja` originales; `+ fechaModificacion = LocalDate.now()` en cada edición |
| [ProveedorRequest.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/dto/persona/request/ProveedorRequest.java) | `fechaEfecto` → `fechaAlta` (`@JsonFormat(pattern="dd/MM/yyyy")`) — ahora sí se acepta desde el frontend |
| [supplier.js](../../GDA/frontend/src/models/supplier.js) | `modelSupplier`: `+ dateHigh` (de `fechaAlta`), `dateEffect` → `dateModification` (de `fechaModificacion`). `modelBackSupplier`: `fechaEfecto` → `fechaAlta: item.dateHigh` |
| [supplierForm.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/suppliers/supplierForm.vue) | Campo "Fecha Efecto" → "Fecha Alta" (`dateHigh`): editable y prellenado con hoy solo en Añadir (`moment().format(FORMAT_MASK_DATE)`), deshabilitado y sin obligatoriedad en Editar/Consultar |
| [suppliersList.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/suppliers/suppliersList.vue) | Columna "FECHA EFECTO" → "FECHA MODIFICACION" (`dateModification`) |
| [i18n/es-ES/common.js](../../GDA/frontend/src/i18n/es-ES/common.js) | `+ dateModification: "Fecha modificación"` (clave nueva; `dateHigh`/`dateLow` ya existían, reutilizadas de Iniciativa) |

**Verificación:** `mvn compile` en verde (`gda-persona`, `gda-persona-commons`, `api-gateway`), 15/15
tests de `ProveedorServiceImplTest`, `npx eslint` sin errores en los 5 archivos frontend modificados.

**Paso 6 — Revisión de ADR:** [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md)
reescrito sustancialmente: eleva el patrón de 4 columnas de "recomendado" a **obligatorio para toda
tabla nueva** del backend GDA, con el criterio final (no el inicial) de `fecha_alta`/`fecha_modificacion`.

==========================================
9. REFERENCIAS
==========================================

- [Documentacion_BackEnd_GDA_2607_Suppliers.md](Documentacion_BackEnd_GDA_2607_Suppliers.md) §14 (Fase 4 — implementación backend/Gateway completa, con los 2 hallazgos de esta fase)
- [Documentacion_FrontEnd_GDA_2607_Suppliers.md](Documentacion_FrontEnd_GDA_2607_Suppliers.md) §15 (Fase 7 — implementación frontend)
- [ADR-0606-patron-campos-auditoria.md](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) (nuevo)
- [ADR-0603-gestion-bbdd.md](../../docs/decisiones/ADR-0603-gestion-bbdd.md)
- Patrón de referencia real: [Iniciativa.java](../../GDA/backend/gda-estructura/gda-estructura/src/main/java/com/ineco/gda/estructura/data/model/Iniciativa.java), [IniciativaServiceImpl.java](../../GDA/backend/gda-estructura/gda-estructura/src/main/java/com/ineco/gda/estructura/service/impl/IniciativaServiceImpl.java)
- Entidad/DTO ampliados: [Proveedor.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/Proveedor.java), [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java)
- Resolución `personaId` → `usuario.id` (sección 7): [SecurityFilter.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/security/SecurityFilter.java), [Usuario.java](../../GDA/backend/gda-autenticacion/gda-autenticacion/src/main/java/com/ineco/gda/autenticacion/data/model/Usuario.java), [UsuarioDto.java](../../GDA/backend/gda-autenticacion/gda-autenticacion-commons/src/main/java/com/ineco/gda/autenticacion/dto/UsuarioDto.java), [GdaAutenticacionApiClient.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/clients/GdaAutenticacionApiClient.java)
