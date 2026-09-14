# Listado de Archivos Relacionados - Módulo Suppliers

## Documento: Arquitectura y Requisitos del Módulo Suppliers
**Base (patrón de pantalla):** GuardiasIneco.vue  
**Backend real:** entidad `Proveedor`, ya existente en el microservicio **gda-persona** (no gda-admingade/gda-finance — corregido 2026-07-16)  
**Fecha:** 2026-07-16 (revisado 2026-07-16)  
**Grafo de referencia:** [knowledge-graph-suppliers.json](knowledge-graph-suppliers.json)

> **Corrección importante (2026-07-16):** el análisis inicial asumía que Suppliers necesitaría un backend
> nuevo en `gda-admingade`, por analogía directa con Guardias. Verificado en código real: la entidad
> `Proveedor` YA EXISTE en `gda-persona` (`Proveedor.java`, `ProveedorRepository`, `ProveedorService`/`Impl`,
> `ProveedorController`) y su DTO (`ProveedorDto`) ya vive en `gda-persona-commons`. El Gateway también
> tiene ya su propia capa de composición para Proveedor (agrega datos de `gda-persona` y `gda-admingade`).
> Todo lo existente es **solo lectura**; hay que **ampliarlo** con create/update/delete, no crearlo desde cero.

---

## 1. ARCHIVOS A CREAR - NUEVOS

### Capa de Presentación (Views)

| Archivo | Ruta | Descripción | Prioridad |
|---------|------|-------------|-----------|
| **Suppliers.vue** | `GDA/frontend/src/modules/gestionAdmingade/views/Suppliers.vue` | Vista principal del módulo Suppliers | **CRÍTICA** |

### Capa de Presentación (Components)

| Archivo | Ruta | Descripción | Prioridad |
|---------|------|-------------|-----------|
| **suppliersList.vue** | `GDA/frontend/src/modules/gestionAdmingade/components/suppliers/suppliersList.vue` | Componente de tabla para listar suppliers | **CRÍTICA** |
| **supplierForm.vue** | `GDA/frontend/src/modules/gestionAdmingade/components/suppliers/supplierForm.vue` | Componente de formulario crear/editar supplier | **CRÍTICA** |

### Capa de Lógica de Negocio (Composables)

| Archivo | Ruta | Descripción | Prioridad |
|---------|------|-------------|-----------|
| **useFetchSuppliers.js** | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useFetchSuppliers.js` | Composable para obtener suppliers | **CRÍTICA** |
| **useSaveSupplier.js** | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useSaveSupplier.js` | Composable para crear supplier | **CRÍTICA** |
| **useUpdateSupplier.js** | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useUpdateSupplier.js` | Composable para actualizar supplier | **CRÍTICA** |
| **useDeleteSupplier.js** | `GDA/frontend/src/modules/gestionAdmingade/composables/service/suppliers/useDeleteSupplier.js` | Composable para eliminar supplier | **CRÍTICA** |

### Capa de Modelos (Models)

| Archivo | Ruta | Descripción | Prioridad |
|---------|------|-------------|-----------|
| **supplier.js** | `GDA/frontend/src/models/supplier.js` | Modelo de transformación frontend/backend | **CRÍTICA** |

---

## 2. ARCHIVOS A MODIFICAR - EXISTENTES

### Servicios

| Archivo | Ruta | Cambios Requeridos | Prioridad |
|---------|------|-------------------|-----------|
| **admingade.js** | `GDA/frontend/src/services/admingade.js` | Añadir métodos: `getSuppliers()`, `postSupplier()`, `putSupplier()`, `deleteSupplier()` | **ALTA** |

### Constantes

| Archivo | Ruta | Cambios Requeridos | Prioridad |
|---------|------|-------------------|-----------|
| **endpoints/* | `GDA/frontend/src/constants/endpoints/` | Añadir constante `SUPPLIERS` a `PATH_SERVICES_ADMIN_GADE` | **ALTA** |
| **common.js** | `GDA/frontend/src/constants/common.js` | Verificar/añadir `STATUS`, `TYPE_FORM`, constantes de supplier si aplica | **MEDIA** |

### Modelos

| Archivo | Ruta | Cambios Requeridos | Prioridad |
|---------|------|-------------------|-----------|
| **index.js** | `GDA/frontend/src/models/index.js` | Exportar `modelSupplier` y `modelBackSupplier` | **ALTA** |

### Traducciones (i18n)

| Archivo | Ruta | Cambios Requeridos | Prioridad |
|---------|------|-------------------|-----------|
| **es.json** | `GDA/frontend/src/i18n/locales/es.json` | Añadir claves de traducción para campos de supplier | **MEDIA** |
| **en.json** | `GDA/frontend/src/i18n/locales/en.json` | Añadir claves de traducción para campos de supplier | **MEDIA** |

### Router

| Archivo | Ruta | Cambios Requeridos | Prioridad |
|---------|------|-------------------|-----------|
| **index.js** | `GDA/frontend/src/router/index.js` | Registrar ruta `/suppliers` en módulo gestionAdmingade | **ALTA** |

---

## 3. ARCHIVOS DE REFERENCIA - NO REQUIEREN CAMBIOS

### Componentes Compartidos (Ya Existen)

| Componente | Ruta | Uso en Suppliers |
|-----------|------|----------------|
| **gda-dialog.vue** | `GDA/frontend/src/components/gda-dialog.vue` | Diálogos de formulario |
| **gda-dialog-ok-cancel.vue** | `GDA/frontend/src/components/gda-dialog-ok-cancel.vue` | Confirmación de eliminación |
| **gda-dialog-informative.vue** | `GDA/frontend/src/components/gda-dialog-informative.vue` | Mensajes de éxito/error |
| **gda-loading.vue** | `GDA/frontend/src/components/gda-loading.vue` | Indicador de carga |
| **gda-table.vue** | `GDA/frontend/src/components/gda-table.vue` | Tabla de listado |
| **gda-select.vue** | `GDA/frontend/src/components/gda-select.vue` | Selectores en formulario |
| **gda-form.vue** | `GDA/frontend/src/components/gda-form.vue` | Contenedor de formulario |
| **crudHeader.vue** | `GDA/frontend/src/modules/common/components/crudHeader.vue` | Encabezado CRUD |

### Composables Compartidos (Ya Existen)

| Composable | Ruta | Uso en Suppliers |
|-----------|------|----------------|
| **useCrud.js** | `GDA/frontend/src/composables/utils/useCrud.js` | Gestión de estado CRUD |
| **useForm.js** | `GDA/frontend/src/composables/utils/useForm.js` | Gestión de formularios |

### Utilidades (Ya Existen)

| Utilidad | Ruta | Uso en Suppliers |
|---------|------|----------------|
| **utils.js** | `GDA/frontend/src/utils/utils.js` | Funciones helper (parseValue, isFormTypeEdit, etc.) |
| **notifications.js** | `GDA/frontend/src/utils/notifications.js` | Servicio de notificaciones |
| **axiosCall.js** | `GDA/frontend/src/utils/axiosCall.js` | Cliente HTTP |

### Sistema de Internacionalización

| Módulo | Ruta | Uso en Suppliers |
|--------|------|----------------|
| **i18n** | `GDA/frontend/src/i18n/` | Sistema de traducción |

---

## 4. BACKEND - ARCHIVOS RELACIONADOS

**Backend real: `gda-persona` (dominio `Proveedor`), no `gda-admingade`.** Casi todo lo listado abajo
**ya existe** — el trabajo es ampliarlo con create/update/delete, no crearlo desde cero.

### API Gateway

| Archivo | Ruta | Descripción | Estado |
|---------|------|-------------|--------|
| **ProveedorController.java** | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/persona/ProveedorController.java` | Controller propio del Gateway. Expone hoy `GET {rsp_grp,rsp_eqp,admingade,super}/proveedores` y `GET .../proveedores/nombre/{nombre}` | **YA EXISTE (solo GET) — AMPLIAR con POST/PUT/DELETE** |
| **ProveedorService.java / ProveedorServiceImpl.java** | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/` | Compone datos de `gda-persona` (Proveedor) y `gda-admingade` (`encargoVigente`, vía `GdaAdmingadeApiClient`) — confirma en producción el patrón de ADR-0604 | **YA EXISTE (solo GET) — AMPLIAR con save/update/delete** |
| **GdaPersonaApiClient.java** | `GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/clients/GdaPersonaApiClient.java` | Cliente Feign; ya tiene `getProveedores()`, `getProveedoresByEncargoId()`, `getProveedorByNombre()`, `getProveedoresById()` | **AMPLIAR con addProveedor()/editProveedor()/deleteProveedor()** |
| **RouteLocatorConfig.java** | `GDA/api-gateway/.../config/RouteLocatorConfig.java` | Contiene una ruta de paso simple para `/admingade/proveedores/**` ya escrita pero **comentada** (superada por la capa de composición dedicada) | **NO REACTIVAR** — dejarla comentada |
| **ProveedorRequest.java / ProveedorResponse.java** (propuestos) | `GDA/api-gateway/.../dto/persona/request/` y `.../response/` | A diferencia de Guardias, hoy el Gateway reutiliza `ProveedorDto` directamente, sin DTOs propios | **DECISIÓN PENDIENTE** al añadir CRUD: ¿reutilizar `ProveedorDto` o introducir DTOs propios del Gateway? |

### Módulo Backend (gda-persona)

| Archivo | Ruta | Descripción | Estado |
|---------|------|-------------|--------|
| **ProveedorController.java** | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/ProveedorController.java` | Expone hoy `GET /proveedores`, `/proveedores?encargoId=`, `/proveedores/{id}`, `/proveedores/nombre/{nombre}`, `/proveedores/externo/encargo/{id}` | **YA EXISTE (solo GET) — AMPLIAR con POST/PUT/DELETE** |
| **ProveedorService.java / ProveedorServiceImpl.java** | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/service/` | Lógica de negocio | **YA EXISTE (solo GET) — AMPLIAR** |
| **ProveedorRepository.java** | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/repository/` | `JpaRepository<Proveedor, Long>` con `findAll`, `findByEncargoIdOrderByNombreAsc`, `findById`, `findByNombre`, `findByCodigo`, `findByCodigoNotAndEncargoId` | **YA EXISTE** — `save()`/`deleteById()` ya los hereda de `JpaRepository`, solo falta exponerlos en Service/Controller |
| **Proveedor.java** | `GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/data/model/` | Entidad JPA (`@Table(name = "proveedor")`): id, codigo, emailContacto, nombre, observaciones, personaContacto, telefonoContacto, encargoId | **YA EXISTE** — verificar si el formulario de alta/edición necesita campos adicionales |

### Módulo Backend Commons (gda-persona-commons) — ADR-0601

| Archivo | Ruta | Descripción | Estado |
|---------|------|-------------|--------|
| **ProveedorDto.java** | `GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java` | DTO de intercambio: id, codigo, emailContacto, nombre, observaciones, personaContacto, telefonoContacto, encargoId | **YA EXISTE** — ya cumple ADR-0601 de fábrica, no requiere cambio de ubicación |

> **Nota (ADR-0601):** `ProveedorDto` ya está correctamente en `-commons`, separado de la entidad/repositorio/service/controller del módulo principal `gda-persona` — el mismo patrón que `GuardiaDto.java` en `gda-admingade-commons`. No hay nada que mover; solo ampliar los campos si el formulario de Suppliers los necesita.

> **Nota (ADR-0604):** el Gateway ya agrega `Proveedor` (gda-persona) con `encargoVigente` (gda-admingade) — es la segunda confirmación real del patrón de composición del API Gateway, además de Guardias. Ver [ADR-0604](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md).

### Base de Datos

| Archivo | Ruta | Descripción | Estado |
|---------|------|-------------|--------|
| **tabla `proveedor`** | Schema `gda_persona` (instancia PostgreSQL compartida, ADR-0603) | Tabla de persistencia | **YA EXISTE** — no requiere migración de creación, solo verificar si faltan columnas para el formulario nuevo |

---

## 5. PATRONES Y CONVENCIONES A SEGUIR

### Estructura de Archivos

```
gestionAdmingade/
├── components/
│   └── suppliers/
│       ├── suppliersList.vue
│       └── supplierForm.vue
├── composables/
│   └── service/
│       └── suppliers/
│           ├── useFetchSuppliers.js
│           ├── useSaveSupplier.js
│           ├── useUpdateSupplier.js
│           └── useDeleteSupplier.js
└── views/
    └── Suppliers.vue
```

### Plantillas de Código - GuardiasIneco.vue

#### Vista: estructura base
```vue
<template>
  <crud-header v-model:keyword="keyword" :label-btn="titlePage" @add="onAdd" />
  <suppliers-list :suppliers="suppliersCustomized" @delete="showDialogOkCancelDelete" @edit="onEdit" />
  <gda-dialog v-model="isDialogForm" :title="dialogTitle" :no-backdrop-dismiss="true" @close="closeDialogForm">
    <supplier-form v-model="formModel" :type="formType" @save="saveFormSupplier" />
  </gda-dialog>
  <gda-loading :loading="isLoading" />
  <gda-dialog-ok-cancel v-model="isDialogOkCancelDelete" :title="`${$t('delete')} ${titlePage}`" />
  <gda-dialog-informative v-model="isDialogInformative" :message="messageInformative" />
</template>

<script>
import { ref, computed } from "vue";
import { useFetchSuppliers } from "@/modules/gestionAdmingade/composables/service/suppliers/useFetchSuppliers.js";
// ... más imports
export default {
  name: "suppliers-view",
  setup() {
    // Composables de servicio
    const { suppliers, fetchSuppliers, isLoading: isLoadingFetchSuppliers } = useFetchSuppliers();
    const { saveSupplier, status: statusSaveSupplier, isLoading: isLoadingSaveSupplier } = useSaveSupplier();
    // ... más composables
    
    // Gestión CRUD genérica
    const { titlePage, formType, formModel, ... } = useCrud({ list: suppliersAll });
    
    // Computed para filtrado y carga
    const suppliersCustomized = computed(() => (keyword.value ? filterList() : suppliersAll.value));
    const isLoading = computed(() => Boolean(isLoadingFetchSuppliers?.value || ...));
    
    // Métodos CRUD
    const saveFormSupplier = () => { isFormTypeEdit(formType) ? _editFormSupplier() : _addFormSupplier(); };
    
    return { suppliersCustomized, isLoading, ... };
  }
};

// Funciones helper externas (fuera del export default)
const generateSuppliersCustomized = (suppliers) => { /* transformación visual */ };
const generateModelSaveAdd = (row) => { /* mapeo para POST */ };
const generateModelSaveEdit = (row) => { /* mapeo para PUT */ };
const generateModelForm = ({ _model }) => { /* mapeo para form */ };
</script>
```

#### Composable Service: patrón
```javascript
import { ref, reactive, toRefs, readonly } from "vue";
import { notificationService } from "@/utils/notifications";
import { admingadeServices } from "@/services";
import { STATUS } from "@/constants/common.js";
import { cError } from "@/utils/utils.js";
import { modelSupplier } from "@/models";

export const useFetchSuppliers = () => {
  const suppliers = ref(null);
  const _state = reactive({ isLoading: false, status: "", error: "" });
  
  const fetchSuppliers = async () => {
    try {
      _state.isLoading = true;
      _state.status = STATUS.LOADING;
      const result = await admingadeServices.getSuppliers();
      suppliers.value = _transformParametersForFront(result);
      _state.status = STATUS.SUCCESS;
    } catch (error) {
      cError(error);
      _state.status = STATUS.ERROR;
      notificationService.error(error?.response?.data?.message || "errors.default");
    } finally {
      _state.isLoading = false;
    }
  };
  
  return {
    ...toRefs(readonly(_state)),
    suppliers: readonly(suppliers),
    fetchSuppliers,
  };
};

const _transformParametersForFront = (items) => items.map(modelSupplier);
```

#### Modelo: transformación
```javascript
// Mapeo 1:1 con los campos reales de ProveedorDto (gda-persona-commons)
import { parseValue } from "@/utils/utils.js";

export const modelSupplier = (item = {}) => ({
  id: parseValue(item.id),
  code: parseValue(item.codigo),
  name: parseValue(item.nombre),
  contactEmail: parseValue(item.emailContacto),
  contactPerson: parseValue(item.personaContacto),
  contactPhone: parseValue(item.telefonoContacto),
  observations: parseValue(item.observaciones),
  encargoId: parseValue(item.encargoId),
});

export const modelBackSupplier = (item = {}) => ({
  id: parseValue(item.id),
  codigo: parseValue(item.code),
  nombre: parseValue(item.name),
  emailContacto: parseValue(item.contactEmail),
  personaContacto: parseValue(item.contactPerson),
  telefonoContacto: parseValue(item.contactPhone),
  observaciones: parseValue(item.observations),
  encargoId: parseValue(item.encargoId),
});
```

#### Constantes: endpoints
```javascript
// En GDA/frontend/src/constants/endpoints/index.js o admingade.js
export const PATH_SERVICES_ADMIN_GADE = {
  // ... existentes
  SUPPLIERS: `${API_URL}/admingade/suppliers`,
  // ...
};
```

#### Servicio HTTP: métodos
```javascript
// En admingade.js
export const admingadeServices = {
  // ... existentes
  getSuppliers: () => apiClient.get({ url: PATH_SERVICES_ADMIN_GADE.SUPPLIERS }),
  postSupplier: (params) => apiClient.post({ url: PATH_SERVICES_ADMIN_GADE.SUPPLIERS, params }),
  putSupplier: (params) => apiClient.put({ url: PATH_SERVICES_ADMIN_GADE.SUPPLIERS, params }),
  deleteSupplier: (id) => apiClient.delete({ url: `${PATH_SERVICES_ADMIN_GADE.SUPPLIERS}/${id}` }),
  // ...
};
```

---

## 6. DEPENDENCIAS CRÍTICAS

### Frontend Dependencies (ya existentes)

```json
{
  "vue": "^3.x",
  "vue-router": "^4.x",
  "vue-i18n": "^9.x",
  "quasar": "^2.x",
  "axios": "^1.x",
  "@vueuse/core": "^latest"
}
```

### Backend Dependencies (verificar)

```xml
<!-- Spring Boot -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- JPA/Hibernate -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- PostgreSQL Driver -->
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
</dependency>
```

---

## 7. CHECKLIST DE IMPLEMENTACIÓN

### Fase 1: Backend Setup (gda-persona — ampliar, no crear)
- [x] ~~Crear entidad `Proveedor.java`~~ Ya existe en `gda-persona`
- [x] ~~Crear DTO `ProveedorDto.java` en `gda-persona-commons`~~ Ya existe, ya cumple ADR-0601
- [x] ~~Crear `ProveedorRepository`~~ Ya existe (`save()`/`deleteById()` ya heredados de `JpaRepository`)
- [ ] Añadir `addProveedor()`/`editProveedor()`/`deleteProveedor()` a `ProveedorService`/`ProveedorServiceImpl` (gda-persona)
- [ ] Añadir `POST`/`PUT`/`DELETE /proveedores` a `ProveedorController` (gda-persona)
- [ ] Verificar si faltan columnas en la tabla `proveedor` (schema `gda_persona`) para el formulario nuevo; si faltan, migración de columnas (no de tabla completa)
- [ ] Ampliar `ProveedorController`/`ProveedorService`/`ProveedorServiceImpl` del **API Gateway** con POST/PUT/DELETE
- [ ] Añadir `addProveedor()`/`editProveedor()`/`deleteProveedor()` al Feign client `GdaPersonaApiClient`
- [ ] Decidir si el Gateway sigue reutilizando `ProveedorDto` o introduce `ProveedorRequest`/`ProveedorResponse` propios
- [ ] Validar endpoints con Postman/Insomnia

### Fase 2: Frontend Models & Services
- [ ] Crear `GDA/frontend/src/models/supplier.js`
- [ ] Actualizar `GDA/frontend/src/models/index.js`
- [ ] Crear constante `SUPPLIERS` en endpoints
- [ ] Añadir métodos en `admingade.js`
- [ ] Añadir claves i18n

### Fase 3: Frontend Composables
- [ ] Crear `useFetchSuppliers.js`
- [ ] Crear `useSaveSupplier.js`
- [ ] Crear `useUpdateSupplier.js`
- [ ] Crear `useDeleteSupplier.js`
- [ ] Validar composables con mock data

### Fase 4: Frontend Components
- [ ] Crear `suppliersList.vue`
- [ ] Crear `supplierForm.vue`
- [ ] Validar componentes en aislamiento

### Fase 5: Frontend View
- [ ] Crear `Suppliers.vue`
- [ ] Integrar componentes y composables
- [ ] Validar flujos CRUD completos
- [ ] Añadir ruta en router

### Fase 6: Testing
- [ ] Tests unitarios backend
- [ ] Tests de integración backend
- [ ] Tests componentes frontend
- [ ] Tests E2E flujo completo

### Fase 7: Documentación
- [ ] Actualizar ADRs si aplica
- [ ] Documentar API
- [ ] Documentar decisiones de diseño

---

## 8. NOTAS IMPORTANTES

### Diferencias Potenciales vs GuardiasIneco

1. **Identificador de Supplier:** ~~GuardiasIneco filtra por `ID_SUPPLIER_INECO`. Suppliers podría: no
   tener este filtro / tener uno similar / ser global~~ **Resuelto (2026-07-16):** Proveedor ya filtra
   por `encargoId`/`encargoVigente` (resuelto por el Gateway vía `GdaAdmingadeApiClient.getEncargoVigente()`),
   no por un identificador de supplier fijo como `ID_SUPPLIER_INECO`.

2. **Relaciones de Entidad:** GuardiasIneco tiene relaciones complejas (person → group → initiative → package). Suppliers debe definir sus propias relaciones.

3. **Validaciones de Negocio:** Verificar requisitos específicos de validación para suppliers.

4. **Permisos y Autorización:** Definir qué roles pueden CRUD suppliers.

5. **Flujos Masivos:** GuardiasIneco tiene `GuardiasInecoMasivo.vue`. ¿Suppliers lo necesita?

---

## 9. REFERENCIAS

- **Ejemplo base:** [GuardiasIneco.vue](../../GDA/frontend/src/modules/gestionAdmingade/views/guardias/GuardiasIneco.vue)
- **Grafo de dependencias:** [knowledge-graph-suppliers.json](knowledge-graph-suppliers.json)
- **ADRs relevantes:** 
  - [ADR-0802-integracion-front-back.md](../../docs/decisiones/ADR-0802-integracion-front-back.md)
  - [ADR-0803-seguridad-secretos.md](../../docs/decisiones/ADR-0803-seguridad-secretos.md)
  - [ADR-0604-patron-composicion-api-gateway.md](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md)
