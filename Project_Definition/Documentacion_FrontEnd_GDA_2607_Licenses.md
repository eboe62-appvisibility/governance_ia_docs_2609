# Documentación FrontEnd GDA — Módulo Licencias
Versión: 2607 (2026-07-24, revisada 2026-07-25, revisada 2026-07-26) · Basado en:
`Documentacion_{FrontEnd,BackEnd,BBDD}_GDA_2607_Suppliers.md` (mismo modelo de desarrollo, aplicado al
módulo Licencias) · Backend real: entidad `Licencia` en **gda-admingade** (a diferencia de
Suppliers/`Proveedor`, que ya existía en `gda-persona`, `Licencia` era una entidad **completamente nueva**
— ya construida, ver `Documentacion_BackEnd_GDA_2607_Licenses.md`) · Ticket: `GDAV3-446` · Estado:
**módulo Licencias completo y verificado de extremo a extremo** — listado, Añadir, Editar, Consultar,
Eliminar, Fecha Alta/Fecha Baja con el ciclo "Baja Definitiva", desplegable de Estados filtrado, y
bloqueo del formulario Editar tras la baja — todo confirmado funcionando por el usuario (2026-07-26).
Backend GET/POST/PUT/DELETE completo en las 3 capas (`gda-persona`, `gda-admingade`, API Gateway).
**Subida a GitLab de los 4 repositorios (gda-persona, gda-finance, api-gateway, frontend) en curso,
gestionada manualmente por el usuario.**

==========================================
1. INTRODUCCIÓN GENERAL
==========================================

Este documento recoge la construcción del nuevo módulo **Licencias** dentro de
`GDA/frontend/src/modules/gestionAdmingade/`, siguiendo el mismo modelo de desarrollo ya documentado
para **Suppliers** (Proveedores): arquitectura vista orquestadora + composables de servicio + modelos de
transformación + componentes UI compartidos, con documentación y validación del usuario por fases.

Diferencia de partida respecto a Suppliers, relevante para todo lo que sigue: `Proveedor` ya existía
(solo lectura) en `gda-persona` antes de ese trabajo — el esfuerzo fue ampliar 5 archivos backend ya
existentes con create/update/delete. `Licencia` era una entidad **nueva de principio a fin**: la tabla
`gda_admingade.licencia` ya existía en BD (aportada por el usuario, ver sección 3), y no existía ninguna
capa de backend — construida en una fase posterior a esta, documentada en
`Documentacion_BackEnd_GDA_2607_Licenses.md`.

==========================================
2. ARQUITECTURA (RESUMEN)
==========================================

```
Presentación        Licenses.vue → licensesList.vue / licenseForm.vue → crudHeader.vue (SHR)
                          │
UI Kit global (SHR)  gda-dialog / gda-dialog-informative / gda-loading / gda-table / gda-select /
                     gda-input / gda-form
                          │
Composables          useCrud.js / useForm.js (SHR) · useFetchLicenses / useSaveLicense (NUEVOS) ·
                     useFetchStates / useFetchMaestroLocalizacion (NUEVOS, dominio gestionAdmingade —
                     ver sección 6 sobre por qué no se reutilizan los genéricos existentes)
                          │
Modelos              license.js → modelLicense() / modelBackLicense() / modelMaestroEstado() /
                     modelMaestroLocalizacion() (NUEVO)
                          │
Servicios            admingadeServices.{get,post}License, getMaestrosEstado, getMaestrosLocalizacion
                     (MODIFICAR admingade.js)
                          │
API Gateway (composición, ADR-0604)   LicenciaController → LicenciaServiceImpl — YA EXISTE
                                       (ver Documentacion_BackEnd_GDA_2607_Licenses.md)
Backend gda-admingade                 LicenciaController → LicenciaService → LicenciaRepository →
                                       Licencia (entity) — YA EXISTE
                          │
Base de datos        tabla `licencia`, schema `gda_admingade` — YA EXISTÍA (ver sección 3)
```

(SHR) = compartido/reutilizado sin cambios. (NUEVO) = creado para Licencias.

==========================================
3. ESQUEMA DE BASE DE DATOS (DE PARTIDA)
==========================================

Tabla ya existente, aportada por el usuario, sin cambios:

```sql
CREATE TABLE gda_admingade.licencia (
	id bigserial NOT NULL,
	estado_id int8 NULL,
	localizacion_id int8 NULL,
	licencia varchar(255) NULL,
	expte_compra bool NOT NULL,
	observaciones varchar(200) NULL,
	fecha_alta date NULL,
	fecha_modificacion date NULL,
	fecha_baja date NULL,
	usuario_modificacion_id int8 NULL,
	CONSTRAINT fk_licencia_localizacion FOREIGN KEY (localizacion_id) REFERENCES gda_persona.maestro_localizacion(id),
	CONSTRAINT fk_licencia_maestro_estado FOREIGN KEY (estado_id) REFERENCES gda_persona.maestro_estado(id),
	CONSTRAINT fk_licencia_usuario FOREIGN KEY (usuario_modificacion_id) REFERENCES gda_persona.persona(id)
);
```

Nótese que `licencia` **ya nace** con los 4 campos de auditoría que
[ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) exige como obligatorios para toda
tabla nueva. `estado_id`/`localizacion_id` referencian `maestro_estado`/`maestro_localizacion` — ver
sección 6 para una precisión importante sobre estas dos entidades.

==========================================
4. ESTRUCTURA DE ARCHIVOS
==========================================

**Archivos nuevos:**

| Archivo | Ruta | Estado |
|---|---|---|
| `license.js` (modelo) | `GDA/frontend/src/models/license.js` | ✅ `modelLicense()`, `modelBackLicense()`, `modelMaestroEstado()`, `modelMaestroLocalizacion()` |
| `useFetchLicenses.js` | `.../composables/service/licenses/useFetchLicenses.js` | ✅ (patrón `useFetchSuppliers.js`) |
| `useSaveLicense.js` | `.../composables/service/licenses/useSaveLicense.js` | ✅ (patrón `useSaveSupplier.js`) |
| `useFetchStates.js` | `.../composables/service/states/useFetchStates.js` | ✅ Fetcher propio del dominio `gestionAdmingade`, apunta a `admingade/maestrosEstado` |
| `useFetchMaestroLocalizacion.js` | `.../composables/service/maestroLocalizacion/useFetchMaestroLocalizacion.js` | ✅ Fetcher propio, apunta a `admingade/maestrosLocalizacion` (ver sección 6, no reutiliza `useFetchLocations`) |
| `licensesList.vue` | `.../components/licenses/licensesList.vue` | ✅ 8 columnas (ver sección 5) |
| `licenseForm.vue` | `.../components/licenses/licenseForm.vue` | ✅ Modos `ADD`/`EDIT`/`DETAIL` (patrón `supplierForm.vue` completo) |
| `Licenses.vue` | `.../views/Licenses.vue` | ✅ Buscador + botón Añadir + tabla + diálogo de formulario (Añadir/Editar/Consultar) + diálogo de confirmación de borrado + diálogo informativo + loading |
| `useUpdateLicense.js` | `.../composables/service/licenses/useUpdateLicense.js` | ✅ (patrón `useUpdateSupplier.js`) |
| `useDeleteLicense.js` | `.../composables/service/licenses/useDeleteLicense.js` | ✅ (patrón `useDeleteSupplier.js`) |

**Archivos modificados:**

| Archivo | Cambio |
|---|---|
| `constants/endpoints/admingade.js` | `+ LICENSES`, `+ MAESTROS_ESTADO`, `+ MAESTROS_LOCALIZACION` (rutas `admingade/licencias`, `admingade/maestrosEstado`, `admingade/maestrosLocalizacion`) |
| `services/admingade.js` | `+ getLicenses()`, `+ postLicense()`, `+ getMaestrosEstado()`, `+ getMaestrosLocalizacion()` |
| `models/index.js` | `+ export modelLicense`, `modelBackLicense`, `modelMaestroEstado`, `modelMaestroLocalizacion` |
| `app/routes.js` | `+ MANAGEMENT_ADMINGADE_LICENSES` → `/administracionGade/licencias` → `Licenses.vue`, junto a `MANAGEMENT_ADMINGADE_PERSONS_ETT` |
| `app/configItemsMenuBreadcrumbs.js` | `+ CONFIG_VIEWS.MANAGEMENT_ADMINGADE_LICENSES` (título `i18n("licenses")`, mismos permisos que `PERSONS_ETT`/`SUPPLIERS`) |
| `app/menu.js` | `+` entrada de menú **nivel 1** dentro de "Gestión Admingade", entre `Personas ETT` y `Consultas` (posición pedida explícitamente por el usuario) |
| `i18n/es-ES/common.js` | `+ license`, `+ licenses`, `+ location`, `+ purchaseFile`, `+ dateModification`, `+ yes`, `+ no` (claves nuevas; `dateLow`/`state`/`observations`/`actions`/`seeDetail`/`deleteRecord` ya existían, reutilizadas sin cambio) |

==========================================
5. COLUMNAS DEL LISTADO IMPLEMENTADAS
==========================================

8 columnas, en este orden:

| Columna | Campo (frontend) | Origen | Notas |
|---|---|---|---|
| LICENCIA | `license` | `item.licencia` | texto plano |
| LOCALIZACIÓN | `location.description` | `item.localizacion` (objeto anidado, `MaestroLocalizacionDto`) | aplanado con `modelMaestroLocalizacion()` (`nombre` → `description`, **no** `modelLocation()` genérico — ver sección 6) |
| ESTADO | `status.description` | `item.estado` (objeto anidado, `MaestroEstadoDto`) | aplanado con `modelMaestroEstado()` (`nombre` → `description`, **no** `modelState()` genérico — ver sección 6) |
| EXPEDIENTE COMPRA | `purchaseFile` | `item.expteCompra` (boolean) | se muestra como `$t('yes')`/`$t('no')` |
| OBSERVACIONES | `notes` | `item.observaciones` | texto plano, `maxlength` real de BD 200 |
| FECHA MODIFICACIÓN | `dateModification` | `item.fechaModificacion` | `format: formatDate` |
| FECHA BAJA | `dateLow` | `item.fechaBaja` | `format: formatDate`. No forma parte del formulario Añadir; en Editar es de solo lectura salvo en el instante en que se selecciona el estado "Baja Definitiva" (se autorrellena y queda editable), y vuelve a quedar intocable una vez guardada — ver sección 11 (autorrelleno) y sección 12 (bloqueo tras guardar). No es "intocable vía API" como `proveedor.fecha_baja` — diverge deliberadamente de ese criterio |
| ACCIONES | — | — | shapeada igual que el estado final de `suppliersList.vue` tras su Fase 5: botón nativo Editar de `gda-table`, y Consultar/Eliminar añadidos a mano en el slot `buttons-append`. Completamente funcional desde la Fase 3 (sección 10): `Licenses.vue` escucha `edit`/`show-detail`/`delete` y abre los diálogos correspondientes |

`dateHigh` (`fecha_alta`) forma parte del modelo (`modelLicense`/`modelBackLicense`) y del formulario
Añadir/Editar desde la Fase 3 (sección 10), pero no es una columna del listado — decisión deliberada,
igual que `suppliersList.vue` tampoco muestra `dateHigh`. `usuarioModificacionId` sigue 100% fuera del
frontend (ni modelo, ni formulario, ni listado) — resuelto enteramente en el Gateway desde el
`SecurityContext`, igual que en Suppliers.

==========================================
6. HALLAZGOS REALES (CORRECCIONES APLICADAS)
==========================================

**Hallazgo 1 — `MaestroEstado`/`MaestroLocalizacion` no son las entidades genéricas que ya reutiliza el
resto de la app.** Detectado durante la investigación de la fase de backend
(`Documentacion_BackEnd_GDA_2607_Licenses.md` §2), antes de que hubiera ningún dato real para
comprobarlo visualmente: `licencia.estado_id`/`localizacion_id` apuntan a `gda_persona.maestro_estado`/
`maestro_localizacion`, cuyos DTOs (`MaestroEstadoDto`, `MaestroLocalizacionDto`, verificados en el
código real del backend) usan el campo **`nombre`**. La primera versión de `license.js` (Fase 1)
reutilizaba `modelLocation()`/`modelState()` (`GDA/frontend/src/models/location.js`/`state.js`), pensados
para otras entidades del backend (`Localizacion`, con campo `descripcion`) — con el backend real habrían
mostrado Localización/Estado en blanco. Corregido: `license.js` define ahora sus propios
`modelMaestroEstado()`/`modelMaestroLocalizacion()` locales (`nombre` → `description`), y
`licenseForm.vue` usa `useFetchMaestroLocalizacion` (nuevo) en vez de `useFetchLocations` (genérico,
entidad equivocada) para el desplegable de Localización. `useFetchStates.js` (ya existía desde antes,
apuntando a un endpoint adivinado `admingade/estados`) se corrigió para apuntar al endpoint real
`admingade/maestrosEstado`.

**Hallazgo 2 — interceptor global de 404 (`axiosCall.js`).** `axiosCall.js`
([líneas 94-96](../../GDA/frontend/src/utils/axiosCall.js#L94-L96)) redirige **toda la SPA** a
`#/notFound` ante cualquier respuesta 404 de cualquier llamada a la API, no solo la pantalla que la
originó. Mientras `admingade/licencias` no tuvo ningún backend detrás (antes de
`Documentacion_BackEnd_GDA_2607_Licenses.md`), esto impedía ver la pantalla de Licencias en absoluto —
no un listado vacío, sino una navegación forzada fuera de la pantalla. Confirmado interactivamente con
el usuario, verificado con un stub temporal de datos (revertido antes de continuar). Con el backend ya
construido y verificado end-to-end (`Documentacion_BackEnd_GDA_2607_Licenses.md` §6), este
comportamiento ya no aplica a Licencias. Relevante para cualquier módulo futuro que construya frontend
antes que backend: no hay forma de ver la pantalla hasta que el primer GET responda con éxito.

==========================================
7. VERIFICACIÓN DE CUMPLIMIENTO CON LOS ADR (docs/decisiones)
==========================================

Mismo resultado que `Documentacion_BackEnd_GDA_2607_Licenses.md` §5 desde el lado frontend — ver ese
documento para el detalle completo por ADR. Resumen: ADR-0600/0601/0602/0603/0604/0606/0801/0802/0803
cumplidos, sin necesidad de ningún ADR nuevo. La pregunta abierta que señalaba la versión anterior de
este documento (¿necesita Licencia una capa de composición propia en el Gateway?) quedó **resuelta con
evidencia real** al construir el backend: sí la necesita, y ya existe (`LicenciaServiceImpl`, patrón
`GuardiaServiceImpl`).

==========================================
8. ESTADO REAL DE IMPLEMENTACIÓN — FASE 1 (LISTADO) (2026-07-24)
==========================================

**Resumen:** implementada la pantalla de listado de Licencias, con las columnas pedidas por el usuario y
la entrada de menú entre "Personas ETT" y "Consultas". Detalle de archivos: sección 4.

**Paso 4 (GitLab):** pendiente, lo gestiona el usuario manualmente.
**Paso 6 (ADR):** ver sección 7 — sin ADR nuevo.

==========================================
9. ESTADO REAL DE IMPLEMENTACIÓN — FASE 2 (BOTÓN AÑADIR + FORMULARIO) (2026-07-24)
==========================================

**Resumen:** activado el botón "Añadir Licencia" y construido su formulario, siguiendo explícitamente el
mismo procedimiento que `supplierForm.vue`/`Suppliers.vue` en la Fase 2 de Suppliers (commit `7fdbf22`,
formulario solo en modo `ADD`).

**Campos del formulario** (`licenseForm.vue`):

| Campo | Componente | Obligatorio | Origen de las opciones |
|---|---|---|---|
| Licencia | `gda-input` (texto) | Sí | — |
| Localización | `gda-select` | Sí | `useFetchMaestroLocalizacion()` (nuevo, ver sección 6) |
| Estado | `gda-select` | Sí | `useFetchStates()` (dominio `gestionAdmingade`, ver sección 6) |
| Expediente compra | `q-checkbox` | — (booleano, siempre tiene valor) | — |
| Observaciones | `gda-input` (textarea) | No | `maxlength 200` |

**Decisiones de diseño:**
- Igual que `chargeId` en Suppliers (no igual que `person.js`): el formulario liga
  `modelForm.location`/`modelForm.status` al objeto completo del `gda-select`, y es la vista
  (`Licenses.vue`, `generateModelSaveAdd`) quien extrae `.id` antes de construir el payload
  (`locationId`/`statusId`).
- `fechaModificacion`/`usuarioModificacionId` fuera del formulario — 100% backend, en las tres
  operaciones. `fechaAlta` se incorporó como campo del formulario en la Fase 3 (sección 10).
- Sin tooltip de "guardado no disponible": a diferencia de la primera versión de esta fase, el backend
  ya existe (`Documentacion_BackEnd_GDA_2607_Licenses.md`), así que no aplica el aviso provisional que sí
  usó Suppliers en su propia Fase 2.

**Verificación:** `npx eslint` sin errores en los archivos nuevos/modificados de Licencias.

**Paso 4 (GitLab):** pendiente, lo gestiona el usuario manualmente.
**Paso 6 (ADR):** ver sección 7.

==========================================
10. ESTADO REAL DE IMPLEMENTACIÓN — FASE 3 (FECHA ALTA, EDITAR, CONSULTAR, ELIMINAR) (2026-07-25)
==========================================

**Resumen:** corrección solicitada por el usuario tras probar la Fase 2: (1) Fecha Alta como campo del
formulario Añadir, con la fecha del día por defecto; (2) los botones Editar y Consultar no abrían ningún
diálogo; (3) el botón Eliminar no tenía ninguna funcionalidad. Las tres correcciones, más el backend
`PUT`/`DELETE` que hizo falta construir para que Editar/Eliminar funcionen de verdad, siguen
explícitamente el patrón ya completo de Suppliers (`supplierForm.vue`/`Suppliers.vue`, versión final,
incluidas sus Fases 3, 4, 5 y 8) — ver `Documentacion_BackEnd_GDA_2607_Licenses.md` §4 para el detalle
del backend.

**1. Fecha Alta (`dateHigh`):**
- `license.js`: `modelLicense` `+ dateHigh: parseDate(item.fechaAlta)`; `modelBackLicense`
  `+ fechaAlta: item.dateHigh`.
- `licenseForm.vue`: `+ gda-input-date` para `dateHigh` (editable y obligatorio solo en Añadir,
  `:readonly="!isFormTypeAdd"`), prellenado con `moment().format(FORMAT_MASK_DATE)` cuando está vacío en
  modo Añadir — mismo patrón que `supplierForm.vue` (Fase 8 de Suppliers), no solo confiando en el
  comportamiento del selector de fecha. `+ gda-input-date` de solo lectura para `dateLow`, visible solo
  fuera de modo Añadir (`v-if="!isFormTypeAdd"`) — igual que Suppliers, informativo únicamente (la baja
  solo se gestiona directamente en BBDD, igual que `proveedor.fecha_baja`).
- Backend: `fechaAlta` ya no se fuerza en `addLicencia` (antes se forzaba a `LocalDate.now()`; ahora
  viene del `LicenciaRequest`, que ganó el campo `fechaAlta`); `editLicencia` preserva siempre
  `fechaAlta`/`fechaBaja` originales leídos de BD, ignorando lo que llegue en el DTO — igual que
  `editProveedor`.

**2. Editar y Consultar (botones ya existían en el listado, sin diálogo detrás):**
- `licenseForm.vue`: `type` pasa a admitir `TYPE_FORM.EDIT`/`TYPE_FORM.DETAIL` (antes solo `ADD`); todos
  los campos reciben `:readonly="isFormTypeDetail"` (`:disable` en el caso del `q-checkbox`); nuevo
  `onAccept()` para el botón "Aceptar" que `form-footer` muestra en modo detalle; LIMPIAR en modo edición
  restaura los valores originales campo a campo desde `_model` (no un `Object.assign` masivo), replicando
  `supplierForm.vue`.
- `Licenses.vue`: `+ onShowDetail()`, `+ onEdit()`, `+ generateModelForm()` (aplana la fila del listado al
  modelo del formulario); `saveFormLicense()` bifurca entre añadir/editar vía `isFormTypeEdit(formType)`;
  wiring `@show-detail`/`@edit` desde `licensesList.vue` y `@accept` desde el diálogo.
- **Diferencia real con Suppliers en la hidratación de Localización/Estado:** Suppliers necesitaba buscar el
  objeto `Encargo` completo en `optionsJobs` a partir de un `chargeId` plano, porque el listado de
  Proveedores solo devuelve el ID. El listado de Licencias, al pasar por la composición del Gateway
  (`Documentacion_BackEnd_GDA_2607_Licenses.md` §3), ya devuelve `location`/`status` como objetos
  completos (`{id, description}`) — así que `generateModelForm()` los pasa directamente, sin necesitar
  ninguna búsqueda. `licenseForm.vue` conserva de todas formas una búsqueda de reserva
  (`_findStateById`/`_findMaestroLocalizacionById`) por si el objeto llegara alguna vez sin `description`
  poblada, pero en el camino normal no se ejecuta.
- `useSaveLicense`/`useUpdateLicense` comparten `modelBackLicense()`; `Licenses.vue` añade
  `generateModelSaveEdit()` (= `generateModelSaveAdd()` + `id`), igual que Suppliers.
- Nuevo: `useUpdateLicense.js` (patrón `useUpdateSupplier.js`), `+ putLicense()` en `services/admingade.js`.

**3. Eliminar (botón ya existía, sin funcionalidad):**
- Nuevo: `useDeleteLicense.js` (patrón `useDeleteSupplier.js`), `+ deleteLicense(id)` en
  `services/admingade.js`.
- `Licenses.vue`: `+` diálogo `gda-dialog-ok-cancel` (título "Eliminar Licencia", mensaje con el nombre
  de la licencia vía `messageDeleteRecord`, clave i18n ya existente), `+ onDeleteLicense()`; `@delete` de
  la lista va directo a `showDialogOkCancelDelete` (de `useCrud`), sin pasar por `generateModelForm`,
  igual que Suppliers.
- **DELETE físico real** (`deleteById`), igual que `deleteGuardia`/`deleteProveedor` — sin borrado lógico
  vía `fecha_baja`, mismo criterio ya documentado como decisión de Suppliers (sin precedente de "Eliminar
  que hace UPDATE en vez de DELETE" en ningún módulo del proyecto).

**Verificación:** `npx eslint` sin errores en los 7 archivos frontend nuevos/modificados de esta fase.
Compilación en verde en `gda-admingade` y `api-gateway` (backend, ver documento hermano). **Verificación
end-to-end real en el navegador confirmada por el usuario (2026-07-25)**: listado, Añadir, Editar,
Consultar y Eliminar probados y funcionando correctamente contra los 3 backends locales. Dos incidencias
de proceso detectadas y resueltas durante la propia verificación (ninguna de código): `PUT`/`DELETE`
devolvían 405/404 la primera vez porque `gda-admingade`/`api-gateway` seguían corriendo con el build
anterior a esta fase — resuelto reiniciando ambos procesos (Spring Boot no recarga mappings nuevos en
caliente).

**Paso 4 (GitLab):** pendiente, lo gestiona el usuario manualmente en los 3 repos.
**Paso 6 (ADR):** ver sección 7 — se actualizaron [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md)
(Licencia como tercera confirmación real del patrón de composición) y
[ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) (Licencia como tercer caso real de
auditoría, y decisión explícita de `fecha_baja` documentada por primera vez para una entidad con
`estado_id` propio) — ver `Documentacion_BackEnd_GDA_2607_Licenses.md` §7 para el detalle. Ningún ADR
nuevo.

**Con esto, el módulo Licencias queda con las cinco operaciones (listado, Añadir, Editar, Consultar,
Eliminar) construidas en frontend, respaldadas por backend real en las tres capas, y verificadas de
extremo a extremo en el navegador.**

==========================================
11. CORRECCIONES POST-VERIFICACIÓN (2026-07-26): FECHA BAJA Y ESTADOS REPETIDOS
==========================================

Reportado por el usuario tras probar Editar de extremo a extremo (sección 10): (1) `fecha_baja` debía
cargarse con la fecha del día al seleccionar el estado **"Baja Definitiva"** — funcionalidad no
cubierta hasta ahora; (2) el desplegable Estado muestra valores repetidos — bug, resuelto en backend
(ver `Documentacion_BackEnd_GDA_2607_Licenses.md` §8).

**1. Fecha Baja al seleccionar "Baja Definitiva":**

| Archivo | Cambio |
|---|---|
| `licenseForm.vue` | `+ @update:model-value="onStatusChange"` en el `gda-select` de Estado; `onStatusChange(newStatus)` compara `newStatus.description` (sin distinguir mayúsculas/espacios) contra el literal `"Baja Definitiva"` y, si coincide, fija `modelForm.value.dateLow = moment().format(FORMAT_MASK_DATE)`. El campo `dateLow` pasa de `readonly` fijo a `:readonly="!isStatusBajaDefinitiva"` (computado sobre `modelForm.value.status`) — editable, no solo autorrellenado, mismo criterio que `dateLow` en `iniciativaForm.vue`. `+` restauración de `dateLow` en `onClear()` (modo Editar), que antes no se restauraba; `+ "dateLow"` en `FIELDS_THAT_CAN_CHANGE` |
| `license.js` | `modelBackLicense`: `+ fechaBaja: item.dateLow` |
| `Licenses.vue` | `generateModelSaveEdit`: `+ dateLow: row.dateLow` (nunca en `generateModelSaveAdd`, el campo no existe en modo Añadir) |

**Decisión — comparación por texto, no por ID:** a diferencia de `iniciativaForm.vue`
(`isStateClosedOrStopped`, compara `modelForm.value.state?.id` contra `ID_TYPES_STATES_CLOSED`/
`ID_TYPES_STATES_STOPPED`, IDs hardcodeados en `constants/common.js`), aquí se compara
`status.description` contra el literal `"Baja Definitiva"`. Motivo: no hay ningún ID de
`maestro_estado` conocido de antemano como estable entre entornos (Iniciativa tiene su propio catálogo
de estados dedicado, con IDs ya asumidos como constantes en el resto de la app; `maestro_estado`, según
la investigación en curso del punto 2, podría ser una tabla compartida por varios dominios vía `ambito`,
donde fiarse de un ID concreto sería frágil).

**Decisión — autorrelleno, no solo campo habilitado:** Iniciativa solo habilita `dateLow` cuando el
estado es terminal, sin rellenarlo automáticamente (el usuario lo introduce a mano). Aquí, siguiendo
literalmente lo pedido por el usuario, `dateLow` se autorrellena con la fecha del día en el mismo
instante en que se selecciona "Baja Definitiva", quedando además editable por si hace falta ajustarlo.

**Corrección de alcance en el backend:** `LicenciaServiceImpl.editLicencia()` (`gda-admingade`) ya no
preserva `fechaBaja` del registro original (sí sigue preservando `fechaAlta`); `LicenciaRequest.java`
(Gateway) ganó el campo `fechaBaja`. Ver `Documentacion_BackEnd_GDA_2607_Licenses.md` §8 para el detalle
completo, incluida la corrección de [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md)
§4 (la nota de Licencia pasó de "intocable, estilo Proveedor" a "mecanismo estilo Iniciativa").

**2. Estados repetidos — resuelto en backend, sin cambios en frontend:** confirmado con el contenido
real de la tabla (aportado por el usuario) que `gda_persona.maestro_estado` es un catálogo compartido
por varios dominios vía `ambito`, y que las 4 filas relevantes para Licencia (incluida `Baja Definitiva`)
son precisamente las que tienen `ambito` a `NULL`. `MaestroEstadoServiceImpl.findAll()` (`gda-persona`)
no filtraba por `ambito`; el endpoint `GET /maestrosEstado` pasa ahora a usar un método nuevo
`findByAmbitoIsNull()` — ver `Documentacion_BackEnd_GDA_2607_Licenses.md` §8. `useFetchStates.js`
(frontend) no tenía ningún bug propio — simplemente mostraba lo que el backend le devolvía; no requiere
ningún cambio.

**Verificación:** `npx eslint` sin errores en los 3 archivos frontend modificados de la Fecha Baja.
Compilación en verde en `gda-admingade`, `api-gateway` y, para el filtro de Estados, `gda-persona`.
**Confirmado por el usuario en el navegador (2026-07-26):** ambas correcciones funcionando.

**Paso 4 (GitLab):** en curso — el usuario está subiendo los 4 repos.
**Paso 6 (ADR):** [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md) §4 corregido, y
[ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md) ampliado con el hallazgo de catálogos
compartidos sin filtro documentado (`ambito`) — ver `Documentacion_BackEnd_GDA_2607_Licenses.md` §8-9.
Sin ADR nuevo.

==========================================
12. CORRECCIÓN (2026-07-26): FORMULARIO EDITAR BLOQUEADO CUANDO YA TIENE FECHA BAJA
==========================================

**Resumen:** una vez una Licencia tiene `fecha_baja` asignada (guardada previamente, típicamente al
llegar a "Baja Definitiva" — sección 11), el formulario Editar debe abrir con todos los campos
deshabilitados — la baja queda como un estado terminal real, no solo una fecha informativa.

**Archivo modificado:** `licenseForm.vue`.

| Cambio | Detalle |
|---|---|
| `+ isEditLocked` (computed) | `isFormTypeEdit && Boolean(_originalDateLow)` — cierto cuando se abre Editar sobre una fila que **ya traía** `dateLow` desde el listado (persistido en BD) |
| `+ isReadonlyField` (computed) | `isFormTypeDetail \|\| isEditLocked` — sustituye a `isFormTypeDetail` suelto en todos los `:readonly`/`:disable` de los campos (`license`, `location`, `status`, `purchaseFile`, `notes`) |
| `dateLow` | `:readonly="isReadonlyField || !isStatusBajaDefinitiva"` (antes `!isStatusBajaDefinitiva` a secas) — sigue editable únicamente en el instante de la transición a "Baja Definitiva" (sección 11), nunca ya una vez guardada |

**Bug real detectado y corregido el mismo día:** la primera versión de `isEditLocked` leía
`modelForm.value.dateLow` (el valor **en vivo** del formulario) en vez del valor originalmente cargado.
Como `onStatusChange` (sección 11) escribe `modelForm.value.dateLow` en el mismo instante en que se
selecciona "Baja Definitiva" — antes de guardar nada — `isEditLocked` se activaba inmediatamente y
bloqueaba el propio campo `dateLow` que se acababa de autorrellenar, impidiendo ajustarlo. Corregido
capturando el valor original una sola vez, antes de cualquier mutación: `const _originalDateLow =
props.modelValue.dateLow;` (fuera de cualquier `computed`, leído en el momento en que `licenseForm.vue`
se monta con los datos ya guardados de la fila), y `isEditLocked` compara contra esa constante, no
contra `modelForm.value.dateLow`. Con esto: mientras se elige el estado y se autorrellena la fecha,
`_originalDateLow` sigue vacío → formulario editable; solo tras Guardar y volver a abrir Editar sobre la
fila ya persistida, `_originalDateLow` trae el valor real de BD → formulario bloqueado.

**Decisión — no se toca `dateHigh` ni el botón Guardar:** `dateHigh` ya era `:readonly="!isFormTypeAdd"`,
siempre deshabilitado en Editar independientemente de esta corrección, sin cambio necesario. El botón
Guardar no necesitó ninguna lógica nueva: al quedar todos los campos `readonly`, `isFormChange` (que
compara `FIELDS_THAT_CAN_CHANGE` contra los valores originales) nunca puede volverse `true`, y
`form-footer` ya lo deshabilita automáticamente (`:disable="isFormTypeEdit && !isFormChange"`) — mismo
mecanismo que ya usa el resto de la app, sin caso especial para Licencias.

**No se oculta el botón Editar de la fila ni se cambia a modo Consultar:** el usuario puede seguir
abriendo el diálogo de Editar sobre una licencia ya dada de baja (para verla, igual que Consultar), solo
que no puede modificar nada — se decidió no forzar el tipo `DETAIL` en su lugar para no complicar
`Licenses.vue`/`useCrud` con una tercera variante de "Editar-pero-en-realidad-Detail", quedando resuelto
enteramente dentro de `licenseForm.vue`.

**Verificación:** `npx eslint` sin errores. **Confirmado por el usuario en el navegador (2026-07-26)**,
junto con las correcciones de la sección 11 — con esto el módulo Licencias queda verificado de extremo
a extremo en su totalidad (listado, CRUD completo, ciclo de "Baja Definitiva" y bloqueo posterior).

**Paso 4 (GitLab):** en curso — el usuario está subiendo los 4 repos.
**Paso 6 (ADR):** sin cambios — no es un patrón nuevo, es una regla de UI local a `licenseForm.vue`.

==========================================
13. REFERENCIAS
==========================================

- Documento de backend (mismo módulo): [Documentacion_BackEnd_GDA_2607_Licenses.md](Documentacion_BackEnd_GDA_2607_Licenses.md)
- Documento de referencia del mismo modelo de desarrollo (Suppliers): [Documentacion_FrontEnd_GDA_2607_Suppliers.md](Documentacion_FrontEnd_GDA_2607_Suppliers.md), [Documentacion_BackEnd_GDA_2607_Suppliers.md](Documentacion_BackEnd_GDA_2607_Suppliers.md), [Documentacion_BBDD_GDA_2607_Suppliers.md](Documentacion_BBDD_GDA_2607_Suppliers.md)
- Vista/formulario de referencia (Suppliers, Fase 2, commit `7fdbf22`): [supplierForm.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/suppliers/supplierForm.vue), [Suppliers.vue](../../GDA/frontend/src/modules/gestionAdmingade/views/Suppliers.vue) (rama `GDAV3-443`, sin mergear, consultado vía `git show`)
- Modelo de Licencias: [license.js](../../GDA/frontend/src/models/license.js)
- Hallazgo del interceptor global de 404: [axiosCall.js](../../GDA/frontend/src/utils/axiosCall.js)
- ADRs verificados: [ADR-0600](../../docs/decisiones/ADR-0600-estrategia-despliegue.md),
  [ADR-0601](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md),
  [ADR-0602](../../docs/decisiones/ADR-0602-comunicacion-interservicios.md),
  [ADR-0603](../../docs/decisiones/ADR-0603-gestion-bbdd.md),
  [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md),
  [ADR-0606](../../docs/decisiones/ADR-0606-patron-campos-auditoria.md),
  [ADR-0801](../../docs/decisiones/ADR-0801-estrategia-empaquetado-despliegue.md),
  [ADR-0802](../../docs/decisiones/ADR-0802-integracion-front-back.md),
  [ADR-0803](../../docs/decisiones/ADR-0803-seguridad-secretos.md)
