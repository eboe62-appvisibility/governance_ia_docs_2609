# Mapa Visual de Dependencias - Módulo Suppliers

## Vista Jerárquica por Capas

```
┌─────────────────────────────────────────────────────────────────┐
│                        CAPA PRESENTACIÓN                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     VISTA PRINCIPAL                      │   │
│  │                   Suppliers.vue (NEW)                      │   │
│  │  - Orquestación de componentes                           │   │
│  │  - Gestión de estado CRUD (useCrud)                      │   │
│  │  - Composables de servicio                              │   │
│  │  - Diálogos y loading                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                          │                                        │
│        ┌─────────────────┼─────────────────┐                    │
│        │                 │                 │                    │
│  ┌─────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐              │
│  │ crudHeader │  │  suppliersList │  │ supplierForm  │              │
│  │   (SHR)    │  │   (NEW)      │  │   (NEW)     │              │
│  └────────────┘  └──────────────┘  └─────────────┘              │
│        │                 │                 │                    │
│        └─────────────────┼─────────────────┘                    │
│                          │                                        │
│                    ┌─────▼─────┐                                │
│                    │  gda-table │                               │
│                    │   (SHR)    │                               │
│                    └────────────┘                               │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    COMPONENTES UI                        │   │
│  │  ├─ gda-dialog (SHR)                                    │   │
│  │  ├─ gda-dialog-ok-cancel (SHR)                         │   │
│  │  ├─ gda-dialog-informative (SHR)                       │   │
│  │  ├─ gda-loading (SHR)                                  │   │
│  │  ├─ gda-select (SHR)                                   │   │
│  │  ├─ gda-form (SHR)                                     │   │
│  │  └─ Quasar Components (QSelect, QInput, etc.)          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   CAPA LÓGICA DE NEGOCIO                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  COMPOSABLES DE SERVICIO (useCrud = Compartido)                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  useCrud.js (SHR)                                       │   │
│  │  - Gestión de diálogos                                  │   │
│  │  - Gestión de tipo de forma (ADD/EDIT)                  │   │
│  │  - Búsqueda y filtrado                                  │   │
│  │  - Modelo de formulario                                 │   │
│  │  - Título de página                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  COMPOSABLES DE DATOS (CRUD específico de Suppliers)              │
│  ┌───────────────────┬─────────────────┬─────────────────────┐  │
│  │ useFetchSuppliers   │ useSaveSupplier   │ useUpdateSupplier    │  │
│  │ (NEW)             │ (NEW)           │ (NEW)              │  │
│  │                   │                 │                    │  │
│  │ getSuppliers()      │ saveSupplier()    │ updateSupplier()     │  │
│  │ ├─ GET request    │ ├─ POST request │ ├─ PUT request     │  │
│  │ ├─ Transform in   │ ├─ Transform out│ ├─ Transform out   │  │
│  │ │  (modelSupplier)  │ │ (modelBack)   │ │ (modelBack+id)   │  │
│  │ ├─ Status mgmt    │ ├─ Status mgmt  │ ├─ Status mgmt     │  │
│  │ └─ Notifications  │ └─ Notifications│ └─ Notifications   │  │
│  │                   │                 │                    │  │
│  └───────────────────┴─────────────────┴─────────────────────┘  │
│              │                                 │                 │
│              │        ┌────────────────────────┤                 │
│              │        │                        │                 │
│              ▼        ▼                        ▼                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ MODELOS (supplier.js - NEW)                              │   │
│  │ ├─ modelSupplier()      : API → Frontend                 │   │
│  │ └─ modelBackSupplier()  : Frontend → API                 │   │
│  └──────────────────────────────────────────────────────────┘   │
│              │                        │                         │
│              └────────────┬───────────┘                         │
│                           ▼                                      │
│            ┌──────────────────────────┐                         │
│            │  useDeleteSupplier (NEW)   │                         │
│            │                          │                         │
│            │  deleteSupplier(id)        │                         │
│            │  ├─ DELETE request       │                         │
│            │  ├─ Status mgmt          │                         │
│            │  └─ Notifications        │                         │
│            └──────────────────────────┘                         │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CAPA ACCESO A DATOS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  admingadeServices (MODIFY)                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Métodos a añadir:                                      │   │
│  │  ├─ getSuppliers()       → GET admingade/proveedores      │   │
│  │  │    (ruta YA EXISTE en el Gateway, solo falta el       │   │
│  │  │     método frontend)                                  │   │
│  │  ├─ postSupplier(data)   → POST admingade/proveedores      │   │
│  │  │    (ruta A AÑADIR en el Gateway)                       │   │
│  │  ├─ putSupplier(data)    → PUT admingade/proveedores        │   │
│  │  │    (ruta A AÑADIR en el Gateway)                       │   │
│  │  └─ deleteSupplier(id)   → DELETE admingade/proveedores/:id │   │
│  │       (ruta A AÑADIR en el Gateway)                       │   │
│  │                                                          │   │
│  │  + Métodos existentes (conservar)                       │   │
│  └──────────────────────────────────────────────────────────┘   │
│              │                        │                         │
│              └────────────┬───────────┘                         │
│                           ▼                                      │
│              ┌──────────────────────────┐                       │
│              │   axiosCall()            │                       │
│              │ (HTTP client configurado)│                       │
│              └──────────────────────────┘                       │
│                           │                                      │
│                           ▼                                      │
│              ┌──────────────────────────┐                       │
│              │  PATH_SERVICES_ADMIN_GADE│                       │
│              │  SUPPLIERS/PROVEEDORES(NEW)│                       │
│              └──────────────────────────┘                       │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│         API GATEWAY — Composición (ADR-0604) — YA EXISTE         │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Ruta expuesta al frontend: admingade/proveedores (entre otras:  │
│  rsp_grp/, rsp_eqp/, super/proveedores)                          │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ProveedorController.java (Gateway) — YA EXISTE          │   │
│  │  ├─ GET    {rsp_grp,rsp_eqp,admingade,super}/proveedores │   │
│  │  ├─ GET    .../proveedores/nombre/{nombre}               │   │
│  │  ├─ POST   proveedores            (A AÑADIR)             │   │
│  │  ├─ PUT    proveedores            (A AÑADIR)             │   │
│  │  └─ DELETE proveedores/{id}       (A AÑADIR)             │   │
│  └──────────────────────────────────────────────────────────┘   │
│              │                        │                         │
│              └────────────┬───────────┘                         │
│                           ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ProveedorServiceImpl.java (Gateway) — YA EXISTE         │   │
│  │  ├─ findAll(): agrega GdaPersonaApiClient.getProveedores()│  │
│  │  │   + GdaAdmingadeApiClient.getEncargoVigente()          │   │
│  │  │   → filtra por encargo vigente (composición real,     │   │
│  │  │     confirma ADR-0604)                                 │   │
│  │  ├─ findByNombre(): mismo filtro                          │   │
│  │  ├─ save/update/delete            (A AÑADIR)              │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  Nota: existe una ruta de paso simple para /admingade/proveedores│
│  ya escrita pero COMENTADA en RouteLocatorConfig.java — fue       │
│  sustituida por esta capa de composición. NO reactivarla.         │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼  (Feign client GdaPersonaApiClient)
┌─────────────────────────────────────────────────────────────────┐
│           BACKEND / API — módulo gda-persona — YA EXISTE          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ProveedorController.java (gda-persona) — YA EXISTE      │   │
│  │  ├─ GET    /proveedores                                  │   │
│  │  ├─ GET    /proveedores?encargoId=                       │   │
│  │  ├─ GET    /proveedores/{id}                             │   │
│  │  ├─ GET    /proveedores/nombre/{nombre}                  │   │
│  │  ├─ GET    /proveedores/externo/encargo/{id}             │   │
│  │  ├─ POST   /proveedores           (A AÑADIR)             │   │
│  │  ├─ PUT    /proveedores           (A AÑADIR)             │   │
│  │  └─ DELETE /proveedores/{id}      (A AÑADIR)             │   │
│  └──────────────────────────────────────────────────────────┘   │
│              │                        │                         │
│              └────────────┬───────────┘                         │
│                           ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ProveedorService.java / Impl — YA EXISTE                │   │
│  │  ├─ findAll(), findByEncargoId(), findById(),            │   │
│  │  │  findByNombre(), findByExternoAndEncargoId()          │   │
│  │  └─ save/update/delete            (A AÑADIR)             │   │
│  └──────────────────────────────────────────────────────────┘   │
│              │                        │                         │
│              └────────────┬───────────┘                         │
│                           ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ProveedorRepository.java — YA EXISTE                    │   │
│  │  extends JpaRepository<Proveedor, Long>                  │   │
│  │  (save()/deleteById() ya heredados, solo falta exponerlos)│  │
│  └──────────────────────────────────────────────────────────┘   │
│              │                                                    │
│              └───────────────┬────────────────────────────────┐  │
│                              ▼                                 │  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Proveedor.java — YA EXISTE                              │   │
│  │  @Entity @Table(name = "proveedor")                       │   │
│  │  ├─ id                    │                            │   │
│  │  ├─ codigo                │                            │   │
│  │  ├─ nombre                │                            │   │
│  │  ├─ emailContacto         │                            │   │
│  │  ├─ personaContacto       │                            │   │
│  │  ├─ telefonoContacto      │                            │   │
│  │  ├─ observaciones         │                            │   │
│  │  └─ encargoId             │                            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  (módulo principal: gda-persona — NO gda-admingade)               │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼  (serializa/deserializa en el borde del controller)
┌─────────────────────────────────────────────────────────────────┐
│              MÓDULO COMMONS (gda-persona-commons) — YA EXISTE     │
│                         — ADR-0601 —                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ProveedorDto.java — YA EXISTE                            │   │
│  │  ├─ id                     │                           │   │
│  │  ├─ codigo                 │                           │   │
│  │  ├─ nombre                 │                           │   │
│  │  ├─ emailContacto          │                           │   │
│  │  ├─ personaContacto        │                           │   │
│  │  ├─ telefonoContacto       │                           │   │
│  │  ├─ observaciones          │                           │   │
│  │  └─ encargoId              │                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  Ya está en -commons, igual que el DTO real de Guards:           │
│  gda-admingade-commons/src/main/java/.../dto/GuardiaDto.java     │
│  → ProveedorDto ya cumple ADR-0601 de fábrica, nada que mover.   │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                 BASE DE DATOS — YA EXISTE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  PostgreSQL (instancia compartida gda_03_desa / GDA_REIR_02_PRO) │
│  └─ Schema: gda_persona (ADR-0603 — schema propio, no instancia  │
│     separada)                                                     │
│     └─ Tabla: proveedor — YA EXISTE                              │
│        ├─ id (PK, BigInt, identity)                              │
│        ├─ codigo (VARCHAR 30)                                    │
│        ├─ nombre (VARCHAR 255)                                   │
│        ├─ email_contacto (VARCHAR 255)                           │
│        ├─ persona_contacto (VARCHAR 255)                         │
│        ├─ telefono_contacto (VARCHAR 255)                        │
│        ├─ observaciones (VARCHAR 250)                            │
│        └─ encargo_id (BigInt)                                    │
│                                                                   │
│  No se requiere script de creación de tabla. Si el formulario    │
│  nuevo necesita columnas adicionales, sería una migración de     │
│  columnas, no de tabla completa.                                  │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Flujos de Datos CRUD

### 1. LECTURA (Fetch)

```
┌────────────────────────────────────────────────────────────────┐
│ USER ACTION: Página carga o refresco                           │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Suppliers.vue (mounted hook)                                     │
│ await fetchSuppliers()                                           │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ useFetchSuppliers()                                              │
│ - Set isLoading = true                                         │
│ - Set status = LOADING                                         │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ admingadeServices.getSuppliers()                                 │
│ axiosCall.get({ url: PATH_SERVICES.SUPPLIERS })                │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ HTTP GET admingade/proveedores  (ruta YA EXISTE)               │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ [API GATEWAY, YA EXISTE] ProveedorController.getProveedor()    │
│ ProveedorServiceImpl.findAll()                                 │
│  → agrega GdaPersonaApiClient.getProveedores()                 │
│  + GdaAdmingadeApiClient.getEncargoVigente() (ADR-0604)        │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ [gda-persona, YA EXISTE] ProveedorController.getProveedor()    │
│ ProveedorService.findAll()                                     │
│ ProveedorRepository.findAll()                                  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Database: SELECT * FROM proveedor (schema gda_persona)        │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Response: List<ProveedorDto>                                   │
│ HTTP 200 OK                                                    │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ useFetchSuppliers()                                              │
│ - Transform with modelSupplier()                                 │
│ - suppliers.value = transformedList                              │
│ - status = SUCCESS                                             │
│ - isLoading = false                                            │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Suppliers.vue                                                    │
│ - suppliersCustomized (computed) actualizado                    │
│ - render: suppliersList                                          │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ USER VE: Tabla con listado de suppliers                         │
└────────────────────────────────────────────────────────────────┘
```

### 2. CREACIÓN (Save)

```
┌────────────────────────────────────────────────────────────────┐
│ USER ACTION: Click en "Agregar Supplier"                         │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Suppliers.vue: onAdd()                                           │
│ onShowDialogAdd()                                              │
│ isDialogForm = true                                            │
│ formType = 'add'                                               │
│ formModel = {}                                                 │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ USER VE: Dialog abierto con supplierForm vacío                 │
│ Rellena campos del formulario                                  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ USER ACTION: Click en "Guardar"                               │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ supplierForm: emit('save')                                       │
│ Suppliers.vue: saveFormSupplier()                                  │
│ isFormTypeEdit(formType) = false                               │
│ _addFormSupplier()                                               │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ generateModelSaveAdd(formModel)                                │
│ Transforma datos al formato backend                            │
│ model = { name, code, ... }                                    │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ useSaveSupplier()                                                │
│ await saveSupplier(model)                                        │
│ - isLoading = true                                             │
│ - status = LOADING                                             │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ modelBackSupplier(model)                                         │
│ Transforma al formato backend (nombrePropio, etc.)             │
│ backModel = { nombrePropio: model.name, ... }                  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ admingadeServices.postSupplier(backModel)                        │
│ HTTP POST admingade/proveedores                                │
│ Body: { nombre: "...", codigo: "...", ... }                   │
│ ⚠️ Ruta NO EXISTE hoy — A AÑADIR en Gateway y gda-persona      │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ [API GATEWAY, A AÑADIR] ProveedorController.addProveedor()     │
│ ProveedorServiceImpl.save()                                    │
│  → Feign GdaPersonaApiClient.addProveedor() (A AÑADIR)         │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ [gda-persona, A AÑADIR] ProveedorController.addProveedor()     │
│ ProveedorService.save(proveedor)                               │
│ ProveedorRepository.save(proveedor) — método YA EXISTE         │
│ (heredado de JpaRepository, solo falta exponerlo)              │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Database: INSERT INTO proveedor VALUES (...) (schema gda_persona)│
│ Retorna: Proveedor con ID generado                             │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Response: ProveedorDto (con id asignado)                       │
│ HTTP 201 CREATED                                               │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ useSaveSupplier()                                                │
│ - modelSupplier(result)                                          │
│ - status = SUCCESS                                             │
│ - isLoading = false                                            │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Suppliers.vue: isStatusSuccess(statusSaveSupplier)                │
│ closeDialogForm()                                              │
│ showDialogInformative()                                        │
│ messageInformative = "Registro guardado exitosamente"         │
│ _initialize() - recarga lista                                  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ USER VE: Dialog cierra, notificación de éxito                 │
│ Lista actualizada con nuevo supplier                             │
└────────────────────────────────────────────────────────────────┘
```

### 3. ACTUALIZACIÓN (Update)

```
[Similar a CREATE pero:]
- User clicks edit en fila
- Dialog abre con datos rellenados
- formType = 'edit'
- saveFormSupplier → _editFormSupplier()
- generateModelSaveEdit(formModel) [incluye id]
- useUpdateSupplier() → putSupplier()
- HTTP PUT admingade/proveedores ⚠️ NO EXISTE hoy — A AÑADIR en Gateway y gda-persona
- ProveedorController/Service/Repository.save() (gda-persona, A AÑADIR el endpoint)
- Database UPDATE (schema gda_persona)
- Lista recargada
```

### 4. ELIMINACIÓN (Delete)

```
┌────────────────────────────────────────────────────────────────┐
│ USER ACTION: Click delete en fila                              │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ suppliersList: emit('delete', row)                              │
│ Suppliers.vue: showDialogOkCancelDelete()                       │
│ Almacena row.id en formModel                                   │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ USER VE: Dialog de confirmación                               │
│ "¿Eliminar este supplier?"                                      │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ USER ACTION: Click "Aceptar"                                   │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Suppliers.vue: deleteSupplier()                                    │
│ useDeleteSupplier()                                              │
│ await deleteSupplier(formModel._model.id)                       │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ admingadeServices.deleteSupplier(id)                            │
│ HTTP DELETE admingade/proveedores/123                          │
│ ⚠️ Ruta NO EXISTE hoy — A AÑADIR en Gateway y gda-persona      │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ [API GATEWAY, A AÑADIR] ProveedorController.deleteProveedor(id)│
│ ProveedorServiceImpl.delete(id)                                │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ [gda-persona, A AÑADIR] ProveedorController.deleteProveedor(id)│
│ ProveedorService.delete(id)                                    │
│ ProveedorRepository.deleteById(id) — método YA EXISTE          │
│ (heredado de JpaRepository, solo falta exponerlo)              │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Database: DELETE FROM proveedor WHERE id = 123 (gda_persona)  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Response: HTTP 204 NO CONTENT o 200 OK                        │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ Suppliers.vue                                                    │
│ closeDialogOkCancelDelete()                                    │
│ showDialogInformative()                                        │
│ _initialize() - recarga lista                                  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│ USER VE: Notificación de éxito                                │
│ Supplier eliminado de lista                                      │
└────────────────────────────────────────────────────────────────┘
```

---

## Leyenda

- **(SHR)** = Componente Compartido (Existing, reutilizable)
- **(NEW)** = Archivo Nuevo (debe ser creado)
- **(MODIFY)** = Archivo Existente (requiere modificaciones)
- **→** = Flujo de datos
- **├─** = Relación/jerarquía

---

## Resumen Ejecutivo

> **Corrección 2026-07-16:** el backend de Suppliers es la entidad `Proveedor`, ya existente en
> `gda-persona` (no un backend nuevo en `gda-admingade`). Ver detalle en `REQUISITOS_MODULO_SUPPLIERS.md` §4.

### Archivos Nuevos: 8 archivos (solo frontend)
- **Frontend Views:** 1 (Suppliers.vue)
- **Frontend Components:** 2 (suppliersList.vue, supplierForm.vue)
- **Frontend Composables:** 4 (useFetch, useSave, useUpdate, useDelete)
- **Frontend Models:** 1 (supplier.js)
- **Backend:** 0 archivos nuevos — todo (entidad, DTO, repositorio, controller, service) ya existe

### Archivos a Ampliar (backend, ya existen): 5 archivos
- `ProveedorController.java` y `ProveedorService(Impl).java` en `gda-persona` (añadir POST/PUT/DELETE)
- `ProveedorController.java` y `ProveedorServiceImpl.java` en el API Gateway (añadir POST/PUT/DELETE)
- `GdaPersonaApiClient.java` (añadir métodos Feign de escritura)

### Archivos Modificados: 5-7 archivos (frontend)
- admingade.js (servicio)
- endpoints (constantes)
- models/index.js
- i18n (traducciones)
- router (rutas)

### Archivos Reutilizados: 12+ componentes compartidos
- Componentes Quasar/GDA
- Composables comunes
- Utilidades

### Patrón de Arquitectura
- **MVVM** con Vue 3 Composition API
- **Service Locator** para composables
- **Repository** para acceso a datos
- **Layered Architecture** (Presentation → Business Logic → Data Access)
- **Reactive State Management** con refs/computed/reactive
- **Separation of Concerns** (Vista, Lógica, Datos)
