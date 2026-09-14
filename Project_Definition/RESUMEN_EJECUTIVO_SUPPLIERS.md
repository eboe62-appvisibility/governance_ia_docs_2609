# Resumen Ejecutivo - Análisis de Dependencias Módulo Suppliers

**Fecha:** 2026-07-16 (revisado 2026-07-16)  
**Basado en:** GuardiasIneco.vue (patrón de pantalla) — **Backend real: entidad `Proveedor` en `gda-persona`**  
**Objetivo:** Documento de requisitos para construcción del módulo Suppliers  

> **Corrección importante (2026-07-16):** el backend de Suppliers NO es un dominio nuevo en
> `gda-admingade`/`gda-finance`. La entidad `Proveedor` ya existe, con CRUD de solo lectura, en
> `gda-persona` (+ `gda-persona-commons` para el DTO), y el API Gateway ya tiene su propia capa de
> composición para ella (agrega `gda-persona` con `encargoVigente` de `gda-admingade`). El trabajo de
> backend pasa de "crear desde cero" a "ampliar con create/update/delete".

---

## 📊 ESTADÍSTICAS

| Categoría | Cantidad | Detalles |
|-----------|----------|---------|
| **Archivos a Crear (frontend)** | 8 | 1 Vista, 2 Componentes, 4 Composables, 1 Modelo — todo nuevo en frontend |
| **Archivos a Ampliar (backend, ya existen)** | 5 | `ProveedorController`/`Service` en gda-persona, `ProveedorController`/`Service` en el Gateway, `GdaPersonaApiClient` |
| **Archivos a Modificar (frontend)** | 5-7 | Servicios, Constantes, Modelos, i18n, Router |
| **Archivos Reutilizables** | 12+ | Componentes compartidos y composables |
| **Capas de Arquitectura** | 6 | Presentación, Lógica, Datos, Transversal, Composición Gateway, Backend |
| **Flujos CRUD** | 4 | Read (YA EXISTE end-to-end), Create/Update/Delete (A CREAR en todas las capas backend) |
| **Endpoints API** | 4 | GET (existente), POST/PUT/DELETE (a añadir) |
| **Patrones de Diseño** | 5 | MVVM, Service Locator, Repository, Layered, Backend-for-Frontend (ADR-0604) |

---

## 🏗️ ARQUITECTURA

### Capas

```
┌─────────────────────────────────────┐
│  Presentación (Vue Components)      │  ← Suppliers.vue, suppliersList, supplierForm (NUEVO)
├─────────────────────────────────────┤
│  Lógica de Negocio (Composables)    │  ← useFetch, useSave, useUpdate, useDelete (NUEVO)
├─────────────────────────────────────┤
│  Modelos (Transformación)           │  ← modelSupplier, modelBackSupplier (NUEVO)
├─────────────────────────────────────┤
│  Acceso a Datos (Services HTTP)     │  ← admingadeServices (NUEVO, llama a ruta ya existente)
├─────────────────────────────────────┤
│  API Gateway (Composición, ADR-0604)│  ← ProveedorController/ServiceImpl (YA EXISTE, solo GET)
├─────────────────────────────────────┤
│  Backend (Spring Boot + JPA)        │  ← ProveedorController/Service/Repository en gda-persona
│                                      │    (YA EXISTEN, solo GET — ampliar con CUD)
├─────────────────────────────────────┤
│  Base de Datos (PostgreSQL)         │  ← Tabla `proveedor`, schema gda_persona (YA EXISTE)
└─────────────────────────────────────┘
```

### Componentes Clave

| Componente | Tipo | Responsabilidad | Complejidad |
|-----------|------|------------------|------------|
| **Suppliers.vue** | Vista | Orquestación, estado CRUD, diálogos | Alta |
| **suppliersList.vue** | Componente | Presentación en tabla | Media |
| **supplierForm.vue** | Componente | Formulario con validación | Alta |
| **useFetchSuppliers** | Composable | Lectura de datos | Media |
| **useSaveSupplier** | Composable | Crear supplier | Media |
| **useUpdateSupplier** | Composable | Actualizar supplier | Media |
| **useDeleteSupplier** | Composable | Eliminar supplier | Baja |
| **modelSupplier** | Modelo | Transformación de datos | Media |

---

## 📁 ESTRUCTURA DE CARPETAS

```
GDA/
├── frontend/src/
│   ├── modules/gestionAdmingade/
│   │   ├── views/
│   │   │   └── Suppliers.vue ........................ NUEVA
│   │   ├── components/suppliers/
│   │   │   ├── suppliersList.vue ................... NUEVA
│   │   │   └── supplierForm.vue .................... NUEVA
│   │   └── composables/service/suppliers/
│   │       ├── useFetchSuppliers.js ................ NUEVA
│   │       ├── useSaveSupplier.js .................. NUEVA
│   │       ├── useUpdateSupplier.js ................ NUEVA
│   │       └── useDeleteSupplier.js ................ NUEVA
│   ├── models/
│   │   ├── supplier.js ............................ NUEVA
│   │   └── index.js ............................ MODIFICAR
│   ├── services/
│   │   └── admingade.js ........................ MODIFICAR
│   ├── constants/
│   │   ├── endpoints/ ......................... MODIFICAR
│   │   └── common.js ......................... VERIFICAR
│   ├── i18n/
│   │   ├── locales/es.json ................... MODIFICAR
│   │   └── locales/en.json ................... MODIFICAR
│   └── router/
│       └── index.js .......................... MODIFICAR
├── api-gateway/
│   └── src/main/java/com/ineco/gda/apigateway/
│       ├── web/controller/persona/ProveedorController.java ... YA EXISTE (solo GET, AMPLIAR)
│       ├── service/persona/ProveedorService(Impl).java ....... YA EXISTE (solo GET, AMPLIAR)
│       └── web/clients/GdaPersonaApiClient.java ............... YA EXISTE (AMPLIAR con POST/PUT/DELETE)
└── backend/
    └── gda-persona/
        ├── gda-persona/
        │   ├── ProveedorController.java ......... YA EXISTE (solo GET, AMPLIAR)
        │   ├── ProveedorService(Impl).java ....... YA EXISTE (solo GET, AMPLIAR)
        │   ├── ProveedorRepository.java .......... YA EXISTE (save/delete ya heredados)
        │   └── Proveedor.java (entidad) .......... YA EXISTE
        └── gda-persona-commons/
            └── dto/
                └── ProveedorDto.java .............. YA EXISTE (ya cumple ADR-0601)
```

---

## 🔗 DEPENDENCIAS PRINCIPALES

### Frontend

```javascript
// Imports en Suppliers.vue
import { useFetchSuppliers } from '@/modules/gestionAdmingade/composables/service/suppliers/useFetchSuppliers.js'
import { useSaveSupplier } from '@/modules/gestionAdmingade/composables/service/suppliers/useSaveSupplier.js'
import { useUpdateSupplier } from '@/modules/gestionAdmingade/composables/service/suppliers/useUpdateSupplier.js'
import { useDeleteSupplier } from '@/modules/gestionAdmingade/composables/service/suppliers/useDeleteSupplier.js'
import { useCrud } from '@/composables/utils/useCrud.js'  // ← Compartido
import { admingadeServices } from '@/services'
import { modelSupplier } from '@/models'
import { STATUS, TYPE_FORM } from '@/constants/common.js'
import { isFormTypeEdit, isStatusSuccess } from '@/utils/utils.js'
import { useI18n } from 'vue-i18n'

// Componentes locales
import suppliersList from '@/modules/gestionAdmingade/components/suppliers/suppliersList.vue'
import supplierForm from '@/modules/gestionAdmingade/components/suppliers/supplierForm.vue'
import crudHeader from '@/modules/common/components/crudHeader.vue'  // ← Compartido

// Componentes globales (Ya registrados)
// gda-dialog, gda-dialog-ok-cancel, gda-dialog-informative, gda-loading
```

### Backend (gda-persona) — código real ya existente, solo falta añadir lo marcado A AÑADIR

```java
// ProveedorController.java (real, gda-persona) — hoy solo GET
@RestController
public class ProveedorController {
    private final ProveedorService proveedorService;

    @GetMapping(value = "/proveedores")
    public ResponseEntity<List<ProveedorDto>> getProveedor() { /* YA EXISTE */ }

    @GetMapping(value = "/proveedores/{id}")
    public ResponseEntity<ProveedorDto> getProveedorById(@PathVariable Long id) { /* YA EXISTE */ }

    // A AÑADIR:
    // @PostMapping("/proveedores") addProveedor(@RequestBody ProveedorDto dto)
    // @PutMapping("/proveedores") editProveedor(@RequestBody ProveedorDto dto)
    // @DeleteMapping("/proveedores/{id}") deleteProveedor(@PathVariable Long id)
}

// ProveedorRepository.java (real, gda-persona) — ya extiende JpaRepository
public interface ProveedorRepository extends JpaRepository<Proveedor, Long> {
    List<Proveedor> findAll(); // YA EXISTE (con order by nombre)
    List<Proveedor> findByEncargoIdOrderByNombreAsc(Long encargoId); // YA EXISTE
    Optional<Proveedor> findByNombre(String nombre); // YA EXISTE
    // save() y deleteById() YA los provee JpaRepository — solo falta exponerlos en Service/Controller
}

// Proveedor.java (entidad real, gda-persona) — tabla `proveedor`, schema gda_persona
@Entity
@Table(name = "proveedor")
public class Proveedor implements Serializable {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String codigo;
    private String emailContacto;
    private String nombre;
    private String observaciones;
    private String personaContacto;
    private String telefonoContacto;
    private Long encargoId;
}
```

### API Gateway (composición, ADR-0604) — código real ya existente

```java
// ProveedorServiceImpl.java (real, api-gateway) — ya agrega gda-persona + gda-admingade
@Service
public class ProveedorServiceImpl implements ProveedorService {
    private final GdaPersonaApiClient gdaPersonaApiClient;
    private final GdaAdmingadeApiClient gdaAdmingadeApiClient; // dependencia cruzada YA EXISTE

    public Mono<List<ProveedorDto>> findAll() {
        // filtra los proveedores de gda-persona por el encargoVigente de gda-admingade
        // YA EXISTE — ver ADR-0604
    }
    // A AÑADIR: save(ProveedorDto), update(ProveedorDto), delete(Long id)
}
```

---

## 🔄 FLUJOS DE DATOS

### Read Flow — backend YA EXISTE end-to-end, falta el frontend
```
Vue Component → useFetchSuppliers() → admingadeServices.getSuppliers()
→ Axios GET → Gateway ProveedorController → ProveedorServiceImpl
  (agrega gda-persona + encargoVigente de gda-admingade, ADR-0604)
→ gda-persona ProveedorController → Service → Repository
→ Database (tabla proveedor) → DTO → Frontend Model → Computed Binding → View
```

### Create Flow — NO EXISTE en ninguna capa backend, hay que construirlo entero
```
Form Input → saveFormSupplier() → generateModelSaveAdd() → modelBackSupplier()
→ useSaveSupplier() → admingadeServices.postSupplier() → Axios POST
→ Gateway ProveedorController (A AÑADIR) → ProveedorServiceImpl (A AÑADIR)
→ gda-persona ProveedorController (A AÑADIR) → Service (A AÑADIR) → Repository (save() ya existe)
→ Database → Success → Reload List → Update View
```

### Update Flow — NO EXISTE en ninguna capa backend
```
Form Edit → saveFormSupplier() → generateModelSaveEdit() → modelBackSupplier()
→ useUpdateSupplier() → admingadeServices.putSupplier() → Axios PUT
→ Gateway ProveedorController (A AÑADIR) → ProveedorServiceImpl (A AÑADIR)
→ gda-persona ProveedorController (A AÑADIR) → Service (A AÑADIR) → Repository (save() ya existe)
→ Database → Success → Reload List → Update View
```

### Delete Flow — NO EXISTE en ninguna capa backend
```
Delete Button → showDialogOkCancel() → deleteSupplier() → useDeleteSupplier()
→ admingadeServices.deleteSupplier() → Axios DELETE
→ Gateway ProveedorController (A AÑADIR) → ProveedorServiceImpl (A AÑADIR)
→ gda-persona ProveedorController (A AÑADIR) → Service (A AÑADIR) → Repository (deleteById() ya existe)
→ Database → Success → Reload List → Update View
```

---

## 📋 CHECKLIST DE IMPLEMENTACIÓN

### Fase 1: Planning & Design ✓
- [x] Analizar GuardiasIneco.vue
- [x] Crear grafo de dependencias (JSON)
- [x] Documentar requisitos
- [x] Crear mapas visuales

### Fase 2: Backend Setup (gda-persona — ampliar lo existente, no crear desde cero)
- [x] ~~Crear Proveedor.java (entidad)~~ Ya existe en `gda-persona`
- [x] ~~Crear ProveedorDto.java~~ Ya existe en `gda-persona-commons`, ya cumple ADR-0601
- [x] ~~Crear ProveedorRepository.java~~ Ya existe (save/deleteById ya heredados de JpaRepository)
- [ ] Añadir POST/PUT/DELETE a ProveedorService/ProveedorServiceImpl (gda-persona)
- [ ] Añadir POST/PUT/DELETE a ProveedorController (gda-persona)
- [ ] Verificar si faltan columnas nuevas en la tabla `proveedor` (migración de columnas, no de tabla)
- [ ] Ampliar ProveedorController/ProveedorServiceImpl del API Gateway con POST/PUT/DELETE
- [ ] Ampliar GdaPersonaApiClient (Feign) con addProveedor/editProveedor/deleteProveedor
- [ ] Decidir si el Gateway introduce ProveedorRequest/ProveedorResponse propios o reutiliza ProveedorDto
- [ ] Validar endpoints

### Fase 3: Frontend Infrastructure
- [ ] Crear supplier.js (modelo)
- [ ] Actualizar models/index.js
- [ ] Añadir endpoint constante
- [ ] Actualizar admingade.js
- [ ] Añadir traducciones i18n
- [ ] Actualizar router

### Fase 4: Frontend Composables
- [ ] Crear useFetchSuppliers.js
- [ ] Crear useSaveSupplier.js
- [ ] Crear useUpdateSupplier.js
- [ ] Crear useDeleteSupplier.js
- [ ] Validar con mock data

### Fase 5: Frontend Components
- [ ] Crear suppliersList.vue
- [ ] Crear supplierForm.vue
- [ ] Crear Suppliers.vue
- [ ] Integrar composables
- [ ] Pruebas iniciales

### Fase 6: Integration Testing
- [ ] Tests unitarios backend
- [ ] Tests integracion backend
- [ ] Tests componentes frontend
- [ ] Tests E2E flujo CRUD
- [ ] Validación de permisos

### Fase 7: Documentation
- [ ] Documentar API (Swagger/OpenAPI)
- [ ] Crear guía de uso
- [ ] Documentar decisiones ADR
- [ ] Actualizar diagrama de arquitectura

---

## ⚠️ CONSIDERACIONES CRÍTICAS

### 1. Diferencias vs GuardiasIneco

| Aspecto | GuardiasIneco | Suppliers | Acción |
|--------|--------------|---------|--------|
| Microservicio backend | gda-admingade | **gda-persona** (resuelto 2026-07-16) | — |
| Filtro por supplier | Sí (ID_SUPPLIER_INECO) | Sí, por `encargoId`/`encargoVigente` (resuelto) | — |
| Relaciones complejas | Sí (person→group→initiative) | Baja (Proveedor solo referencia `encargoId`) | **Confirmar si basta** |
| Operaciones masivas | Sí (GuardiasInecoMasivo) | ¿? | **Evaluar necesidad** |
| Validaciones | Complejas | ¿? (código único, campos obligatorios) | **Definir reglas** |
| Autorización | Roles específicos | Ya cubierta a nivel Gateway (rsp_grp/rsp_eqp/admingade/super) | **Confirmar si basta para CUD** |

### 2. Validaciones de Negocio Requeridas

- [ ] ¿Todos los campos son obligatorios?
- [ ] ¿Hay validaciones de formato (código, etc.)?
- [ ] ¿Hay relaciones que deben validarse?
- [ ] ¿Hay restricciones de duplicidad?
- [ ] ¿Hay restricciones de eliminación?

### 3. Seguridad

- [ ] Definir roles que pueden CRUD
- [ ] Validar autorización en backend
- [x] ~~Encriptar datos sensibles si aplica~~ Resuelto 2026-07-16: `Supplier` no almacenará datos sensibles; no aplica cifrado en reposo.
- [ ] Auditar cambios (createdBy, updatedBy, timestamps)
- [ ] Validar CORS y CSRF

### 4. Performance

- [ ] ¿Hay paginación requerida?
- [ ] ¿Hay búsqueda/filtrado requerido?
- [ ] ¿Hay índices de BD necesarios?
- [ ] ¿Hay caché requerida?
- [ ] ¿Límites de resultados?

### 5. Observabilidad

- [ ] Logs en backend
- [ ] Logs en frontend
- [ ] Notificaciones de error
- [ ] Métricas de rendimiento
- [ ] Tracing distribuido

---

## 📚 REFERENCIAS

### Documentos Generados

1. **knowledge-graph-suppliers.json** - Grafo técnico detallado con nodes, edges, capas y flujos
2. **REQUISITOS_MODULO_SUPPLIERS.md** - Documento de requisitos con listas de archivos
3. **MAPA_VISUAL_SUPPLIERS.md** - Visualización ASCII de dependencias y flujos

### Archivos de Referencia Original

- [GuardiasIneco.vue](../../GDA/frontend/src/modules/gestionAdmingade/views/guardias/GuardiasIneco.vue)
- [guardiasInecoList.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/guardias/guardiasInecoList.vue)
- [guardiaInecoForm.vue](../../GDA/frontend/src/modules/gestionAdmingade/components/guardias/guardiaInecoForm.vue)
- [useFetchGuards.js](../../GDA/frontend/src/modules/gestionAdmingade/composables/service/guards/useFetchGuards.js)
- **Backend real de Suppliers (gda-persona):** [ProveedorController.java](../../GDA/backend/gda-persona/gda-persona/src/main/java/com/ineco/gda/persona/web/controller/ProveedorController.java), [ProveedorDto.java](../../GDA/backend/gda-persona/gda-persona-commons/src/main/java/com/ineco/gda/persona/dto/ProveedorDto.java)
- **Composición en el Gateway:** [ProveedorController.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/web/controller/persona/ProveedorController.java), [ProveedorServiceImpl.java (Gateway)](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/persona/impl/ProveedorServiceImpl.java)

### ADRs Relevantes

- [ADR-0601-desacoplamiento-modulo-commons.md](../../docs/decisiones/ADR-0601-desacoplamiento-modulo-commons.md) - DTO en `-commons`, entidad/repo/service/controller en el módulo principal
- [ADR-0604-patron-composicion-api-gateway.md](../../docs/decisiones/ADR-0604-patron-composicion-api-gateway.md) - Criterio de composición en el API Gateway
- [ADR-0802-integracion-front-back.md](../../docs/decisiones/ADR-0802-integracion-front-back.md) - Patrones de integración
- [ADR-0803-seguridad-secretos.md](../../docs/decisiones/ADR-0803-seguridad-secretos.md) - Seguridad y autenticación

---

## 🎯 PRÓXIMOS PASOS

### Recomendación

1. **Obtener aprobación** de este documento de requisitos
2. **Clarificar preguntas críticas** (filtro supplier, relaciones, etc.)
3. **Crear ADR específico** para decisiones de diseño de Suppliers
4. **Iniciar desarrollo backend** en paralelo con frontend
5. **Usar este documento** como guía de implementación

### Estimación Inicial (Rough) — revisada 2026-07-16

- **Backend:** 1 sprint (3-8 pts) — se reduce respecto a la estimación inicial porque entidad, DTO,
  repositorio y GET ya existen en `gda-persona` y en el Gateway; solo hay que añadir POST/PUT/DELETE
  en 4 archivos (2 en gda-persona, 2 en el Gateway) + el método Feign correspondiente.
- **Frontend:** 2-3 sprints (5-15 pts) — sin cambios, sigue siendo trabajo nuevo completo.
- **Testing:** 1 sprint (3-8 pts)
- **Total:** 2-4 sprints (11-31 pts)

*Nota: Depende de complejidad de validaciones definitivas y de si el Gateway necesita
ProveedorRequest/ProveedorResponse propios o reutiliza ProveedorDto.*

---

## 📞 AUTORIDADES DE DECISIÓN

- **Arquitectura:** Revisar con [Equipo de Arquitectura]
- **Seguridad:** Revisar con [Equipo de Seguridad]
- **BD:** Revisar con [DBA/Data Team]
- **Producto:** Requisitos con [Product Owner]

---

**Documento preparado:** 2026-07-16  
**Estado:** Listo para revisión  
**Siguiente revisión:** Tras clarificación de requisitos críticos
