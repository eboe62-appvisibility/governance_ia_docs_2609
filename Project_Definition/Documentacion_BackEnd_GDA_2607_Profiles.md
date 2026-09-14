# Documentación BackEnd GDA — Módulo Profiles (Perfil / TipoPerfil / Porcentaje)
Versión: 2607 (2026-07-28) · Basado en: `Documentacion_BBDD_GDA_2607_Profiles.md` (DDL real y hallazgo
crítico de esquema) · Backend real: `Perfil`/`TipoPerfil`/`Porcentaje` en **gda-admingade** (ya
desarrollado, no es módulo nuevo) · Ticket relacionado: **GDAV3-429** (autor: Ricardo González, en curso
en paralelo) · Estado: **Auditoría en curso, no desarrollo.** Gateway ya compila y arranca en local;
rutas de paso añadidas; 3 bugs reales corregidos localmente (sin commitear, pendiente de mandar diff a
Ricardo); pendiente confirmar end-to-end si `GET /admingade/perfiles` funciona dado el hallazgo de
`Documentacion_BBDD_GDA_2607_Profiles.md` §3 (desajuste entidad/tabla).

==========================================
1. INTRODUCCIÓN — CÓMO LLEGAMOS AQUÍ (para retomar en otra sesión)
==========================================

Este documento nació como un encargo de **auditoría/validación** (no desarrollo) de un módulo "Profiles"
que el usuario pensaba que era `gda_persona.perfil` (le dio ese DDL al principio). La investigación de
Fase 0 reveló que ese no es el módulo real: es un catálogo simple (`id`+`descripcion`) usado solo como
combo de solo lectura dentro del formulario de Persona, sin pantalla propia. **El módulo real** se
confirmó cuando el usuario dio los 3 endpoints que usa en la práctica:
```
http://172.16.22.114:9100/admingade/tipoPerfiles
http://172.16.22.114:9100/admingade/perfiles/perfilInecoTrue
http://172.16.22.114:9100/admingade/porcentajes/vigentes
```
Estos pertenecen al **Perfil "tarifario de licitación"** de `gda-admingade` (código/tipo de
perfil/porcentaje/tarifas), con pantalla ya construida en el frontend (`Perfiles.vue`,
`/administracionGade/perfiles`). Back y Front de este módulo concreto **ya estaban desarrollados** antes
de esta sesión.

Durante la auditoría, al intentar compilar/arrancar el stack para poder probar, aparecieron varios bugs
reales (no relacionados con el diseño del módulo en sí, sino con un refactor en curso de otro
desarrollador, ticket GDAV3-429) que hubo que corregir localmente para poder seguir.

==========================================
2. LOS 4 CONCEPTOS "PERFIL" EN ESTE CÓDIGO — NO CONFUNDIR
==========================================

Hallazgo central de la Fase 0, importante para cualquier sesión futura que toque esto:

| # | Clase / tabla | Microservicio | Qué es | Endpoint | Pantalla propia |
|---|---|---|---|---|---|
| 1 | `Perfil` → tabla `perfil` | **gda-persona** | Catálogo simple `id`+`descripcion`, FK opcional desde `persona.perfil_id` | `GET /perfilesUsuario`, `POST /addPerfil` (sin editar/borrar) | No — solo combo de solo lectura en `personaForm.vue` (`useFetchProfilesUser`) |
| 2 | **`Perfil` → tabla `perfil`** | **gda-admingade** | **Este es el módulo en auditoría** — perfil tarifario de licitación (código, tipo de perfil, porcentaje, tarifas) | `GET/POST/PUT /perfiles`, `GET /perfiles/perfilInecoTrue`, `DELETE /perfiles/{id}` | Sí — `Perfiles.vue`, `/administracionGade/perfiles` |
| 3 | `PerfilCandidato` → tabla `perfil_candidato` | gda-persona | Perfil profesional de un candidato en proceso de selección | `admingade/perfilesCandidatos` | No (combo dentro de `candidatosForm.vue`) |
| 4 | `PerfilLicitacion` → tabla `perfil_licitacion` | gda-admingade | Acción/histórico de licitación de un pedido, nada que ver con perfiles de persona | `perfilesLicitacion` | Sí — `PerfilLicitacionPedido.vue` |

Las tablas 1 y 2 comparten literalmente el mismo nombre físico `perfil` en dos schemas distintos
(`gda_persona.perfil` vs `gda_admingade.perfil`) — la colisión de nombres no es solo en el código Java,
es real en la BBDD. Ver `reference/lecciones-aprendidas.md` del skill `gda-modulo-crud` para la entrada
añadida sobre este hallazgo.

==========================================
3. ARQUITECTURA DEL MÓDULO EN AUDITORÍA (Perfil #2 + TipoPerfil + Porcentaje)
==========================================

```
Backend gda-admingade    PerfilController → PerfilServiceImpl → PerfilRepository → Perfil (entity)
                         TipoPerfilController → TipoPerfilServiceImpl → TipoPerfilRepository → TipoPerfil
                         PorcentajeController → PorcentajeServiceImpl → PorcentajeRepository → Porcentaje
                         Los 3 con CRUD (o GET, según el caso) ya implementado — YA EXISTE, sin cambios
                         de este módulo en esta sesión (salvo el bug de Recurso.java, ajeno al módulo,
                         ver sección 5)
                           │
API Gateway              Sin composición dedicada (no hace falta: ninguna de las 3 entidades depende de
                         otro microservicio) — solo rutas de paso simple, AUSENTES hasta esta sesión
                         (sección 7), ya añadidas.
                           │
Base de datos            `gda_admingade.perfil`, `tipo_perfil`, `porcentaje` — YA EXISTEN. Ver
                         `Documentacion_BBDD_GDA_2607_Profiles.md` §3 para el hallazgo grave: `Perfil.java`
                         no coincide con la tabla real `perfil`.
```

==========================================
4. VERIFICACIÓN DE COMPILACIÓN — HALLAZGO 1: pom.xml del Gateway desincronizado
==========================================

Al intentar `mvn clean install` en `api-gateway` (para poder levantar el stack en local y probar), la
build falló:
```
Could not resolve dependencies for project com.ineco:gda-apigateway:jar:4.7.0-SNAPSHOT
dependency: com.ineco:gda-admingade-commons:jar:4.6.0 (compile)
gda-admingade-commons:jar:4.6.0 was not found in https://repo.maven.apache.org/maven2
```
**Causa:** [api-gateway/pom.xml:49](../../GDA/api-gateway/pom.xml#L49) fijaba `gda-admingade-commons` a
`4.6.0` — versión nunca instalada localmente (solo había ficheros `.lastUpdated`, es decir, un intento
fallido de resolución remota cacheado). La versión real actual del módulo es `4.7.0-SNAPSHOT` (coincide
con `gda-admingade`/`gda-admingade-parent`), ya instalada en local `.m2` (compilada por el propio usuario
antes de esta sesión). No hay ningún repositorio Nexus interno configurado (ni en los `pom.xml` ni en
`settings.xml` de Maven) — todos los artefactos `com.ineco.*` se resuelven solo desde `.m2` local. Mismo
patrón exacto que **Hallazgo 2** de `Documentacion_BackEnd_GDA_2607_Suppliers.md` §14 (allí con
`gda-persona-commons`), ya documentado como deuda técnica en ADR-0600.

**Corregido (local, sin commitear):** `api-gateway/pom.xml:49` → `4.7.0-SNAPSHOT`.

==========================================
5. HALLAZGO 2: 4 errores de compilación reales — refactor GDAV3-429 sin propagar al Gateway
==========================================

Con el pom.xml corregido, la compilación avanzó y encontró 4 errores reales, todos derivados del mismo
origen: el ticket **GDAV3-429** (autor: Ricardo González, commits `1bb05b5` del 2026-07-23 y `fc507a7`
del 2026-07-28 en `gda-admingade`/`gda-admingade-commons`, confirmado con `git log`/`git show`) refactorizó
`RecursoDto` (quitó `idPersonal`/`personalId`, añadió `personaId` + 4 columnas de auditoría) y en algún
momento anterior `PerfilDto` (quitó `proveedorId`). El **API Gateway es un repositorio git independiente**
y nunca se actualizó para seguir ese cambio.

**Archivos corregidos (local, mínimo para compilar, con TODO marcado donde queda duda semántica):**

| Archivo | Línea | Cambio | Duda pendiente |
|---|---|---|---|
| [RecursoServiceImpl.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/RecursoServiceImpl.java) | 45 | Se elimina `recursoResponse.getPerfil().setProveedor(...)` (`PerfilDto` ya no tiene `proveedorId`) | ¿Debe resolverse "proveedor de un recurso" de otra forma ahora que Perfil no se relaciona con Proveedor? |
| RecursoServiceImpl.java | 71 | `getPersonalId()` → `getPersonaId()` | El `HashMap` de búsqueda sigue indexado por `PersonalDto.id` ("Personal Reir"). **No confirmado** si `personaId` ahora es `Persona.id` directamente (lo que invalidaría este lookup) |
| [ServicioSuministroServiceImpl.java:127](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/ServicioSuministroServiceImpl.java#L127) | 127 | ídem | ídem |
| [ServicioSuministroEstimadoServiceImpl.java:65](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/ServicioSuministroEstimadoServiceImpl.java#L65) | 65 | ídem | ídem |

**Verificación:** `mvn -o clean compile` en `api-gateway` → `BUILD SUCCESS` (221 archivos fuente).

**Pendiente:** enviar diff a Ricardo (el usuario decidió esperar, "no avasallarle", ver §8). Mensaje corto
ya redactado con el usuario, ver estilo en memoria de sesión (`feedback_mensajes_colegas_estilo`).

==========================================
6. HALLAZGO 3: bug real en `Recurso.java` — columna duplicada (bloqueaba el arranque de gda-admingade)
==========================================

Al arrancar `gda-admingade` (`mvn spring-boot:run`), Hibernate fallaba al construir el `SessionFactory`:
```
org.hibernate.MappingException: Column 'fecha_baja' is duplicated in mapping for entity
'com.ineco.gda.admingade.data.model.Recurso'
```
**Causa** (mismo commit GDAV3-429):
[Recurso.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Recurso.java)
tenía `fechaAlta` y `fechaModificacion` **ambas** mapeadas a la columna `fecha_baja` (copia-pega), dejando
además `usuario_modificacion_id` con `unique = true` (sin sentido: impediría que un mismo usuario
modificara más de un recurso).

**Corregido (local, sin commitear):**
```java
// antes
@Column(name="fecha_baja")
private LocalDate fechaAlta;
@Column(name="fecha_baja")
private LocalDate fechaModificacion;
@Column(name = "usuario_modificacion_id", unique = true)
private Long usuarioModificacionId;

// ahora
@Column(name="fecha_alta")
private LocalDate fechaAlta;
@Column(name="fecha_modificacion")
private LocalDate fechaModificacion;
@Column(name = "usuario_modificacion_id")
private Long usuarioModificacionId;
```
Verificado contra el DDL real de `recurso` (`Documentacion_BBDD_GDA_2607_Profiles.md` §2): las columnas
`fecha_alta`/`fecha_modificacion` sí existen (tipo `timestamp`, mapeadas como `LocalDate` — Hibernate lo
tolera, trunca la hora; no bloqueante, pero anotar como nota menor).

==========================================
7. HALLAZGO 4: rutas de Gateway ausentes para `perfiles`/`tipoPerfiles`/`porcentajes` (admingade)
==========================================

Confirmado primero por lectura de código (`RouteLocatorConfig.java`, 292 líneas, sin ninguna ruta para
estos 3 recursos) y luego **en vivo**: `GET /admingade/perfiles` devolvía 404 real desde el Gateway
desplegado. Añadidas en
[RouteLocatorConfig.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/config/RouteLocatorConfig.java)
(líneas 112-126), como ruta de paso simple (sin composición — ninguna de las 3 entidades depende de otro
microservicio):
```java
.route("perfilAdmingade", r -> r
        .path("/admingade/perfiles", "/admingade/perfiles/**")
        .filters(f -> f.rewritePath("/admingade/perfiles(?<segment>/?.*)", "/perfiles${segment}"))
        .uri(urlAdmingade))
.route("tipoPerfiles", r -> r
        .path("/admingade/tipoPerfiles")
        .filters(f -> f.rewritePath("/admingade/tipoPerfiles", "/tipoPerfiles"))
        .uri(urlAdmingade))
.route("porcentajesVigentes", r -> r
        .path("/admingade/porcentajes/vigentes")
        .filters(f -> f.rewritePath("/admingade/porcentajes/vigentes", "/porcentajes/vigentes"))
        .uri(urlAdmingade))
```
**Importante:** el nombre de ruta se eligió `perfilAdmingade` (no `perfiles`) porque **ya existe** una
ruta llamada `perfiles` en el mismo archivo (línea ~255) para el concepto #1 de la sección 2
(`gda_persona.perfil` / `perfilesUsuario`) — Spring Cloud Gateway exige nombres de ruta únicos, y habría
sido fácil chocar sin darse cuenta dada la colisión de nombres ya documentada.

Tras esto, `GET /admingade/perfiles` dejó de dar 404 y pasó a dar **500** (`Connection refused:
172.16.22.114:9102`) — corregido apuntando `gda-admingade` a `localhost:9102` en
`application-local.yml` (sección 8, cambio de entorno local, no de código).

==========================================
8. CAMBIOS DE ENTORNO LOCAL (NO forman parte de ningún commit)
==========================================

[application-local.yml](../../GDA/api-gateway/src/main/resources/config/application-local.yml) del
Gateway: bloque `gda-admingade` apuntado a `http://localhost:9102` (comentada la URL remota
`172.16.22.114:9102`) para poder probar contra el `gda-admingade` local ya arrancado. **Revertir antes de
cualquier commit/push** — mismo criterio que Suppliers (`Documentacion_BackEnd_GDA_2607_Suppliers.md` §9,
paso 3).

==========================================
9. ESTADO REAL Y PENDIENTES (para retomar en otra sesión)
==========================================

**Hecho:**
- ✅ `api-gateway` compila (`mvn clean compile` en verde).
- ✅ `gda-admingade` arranca (bug de columna duplicada corregido).
- ✅ Rutas de Gateway para `perfiles`/`tipoPerfiles`/`porcentajes` (admingade) añadidas.
- ✅ `gda-admingade` apuntado a local en el Gateway para poder probar.
- ⏳ Mensaje informal a Ricardo redactado pero **no enviado todavía** (el usuario decidió esperar,
  "vamos a dejarle trabajar, no quiero avasallarle" — no proponer reenviarlo sin que el usuario lo pida).

**Pendiente (bloqueante para cerrar la auditoría):**
1. **Confirmar si `GET /admingade/perfiles` funciona de verdad** ahora que ya tiene ruta — dado el
   hallazgo crítico de `Documentacion_BBDD_GDA_2607_Profiles.md` §3 (la entidad `Perfil.java` no coincide
   con la tabla real `perfil`), es probable que falle igualmente con un error 500 de SQL distinto (columna
   inexistente). **No asumir que ya funciona** solo porque la ruta ya existe.
2. Confirmar con Ricardo el punto semántico de `personaId` (sección 5) antes de dar por buenos los 4
   parches del Gateway.
3. Preparar y enviar el diff completo a Ricardo cuando el usuario lo pida (no proactivamente).
4. Una vez confirmado que el backend funciona de extremo a extremo, auditar el **frontend**
   (`Perfiles.vue`/`perfilForm.vue`/`models/profile.js`) contra el DDL real — todavía no se ha empezado
   esa parte. Older doc de frontend (`Documentacion_FrontEnd_GDA_2607_Profiles.md`) sigue vacío.
5. Revertir el cambio de `application-local.yml` (sección 8) antes de cualquier commit.

==========================================
10. REFERENCIAS
==========================================

- [Documentacion_BBDD_GDA_2607_Profiles.md](Documentacion_BBDD_GDA_2607_Profiles.md) — DDL real y
  hallazgo crítico de esquema.
- `Documentacion_FrontEnd_GDA_2607_Profiles.md` — todavía vacío, pendiente de iniciar auditoría frontend.
- Skill usado como guía: `.claude/skills/gda-modulo-crud` (`reference/lecciones-aprendidas.md` actualizado
  con la colisión de 4 conceptos "Perfil" y este caso de desincronización entidad/BBDD).
- Entidades: [Perfil.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Perfil.java), [TipoPerfil.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/TipoPerfil.java), [Porcentaje.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Porcentaje.java), [Recurso.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Recurso.java)
- Gateway: [RouteLocatorConfig.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/config/RouteLocatorConfig.java), [RecursoServiceImpl.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/RecursoServiceImpl.java), [ServicioSuministroServiceImpl.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/ServicioSuministroServiceImpl.java), [ServicioSuministroEstimadoServiceImpl.java](../../GDA/api-gateway/src/main/java/com/ineco/gda/apigateway/service/admingade/impl/ServicioSuministroEstimadoServiceImpl.java)
- Precedente del mismo tipo de bug (pom.xml desincronizado): `Documentacion_BackEnd_GDA_2607_Suppliers.md` §14, Hallazgo 2.
