# Documentación BBDD GDA — Módulo Profiles (Perfil / TipoPerfil / Porcentaje)
Versión: 2607 (2026-07-28) · Basado en: `Documentacion_BackEnd_GDA_2607_Profiles.md` (mismo módulo, lado
backend/Gateway) · Tablas: `gda_admingade.perfil`, `gda_admingade.tipo_perfil`, `gda_admingade.porcentaje`
(+ `gda_admingade.recurso` como consumidor con FK) · Estado: **DDL real obtenido directamente de BBDD por
el usuario (2026-07-28) — confirma un desajuste grave entre la entidad JPA `Perfil.java` y la tabla real.
Pendiente de decidir con el equipo si se corrige la entidad o se migra la tabla.**

==========================================
1. INTRODUCCIÓN
==========================================

Esta auditoría partió de una confusión de nombres que conviene dejar escrita para no repetirla: el
usuario aportó primero el DDL de `gda_persona.perfil` (catálogo simple `id`+`descripcion`, usado como
combo de solo lectura dentro del formulario de Persona). **Ese no es el módulo real en auditoría.** El
módulo real — confirmado por los 3 endpoints que el usuario usa en la práctica
(`admingade/tipoPerfiles`, `admingade/perfiles/perfilInecoTrue`, `admingade/porcentajes/vigentes`) — es
el **Perfil "tarifario de licitación"** de `gda-admingade` (pantalla ya existente `Perfiles.vue`,
`/administracionGade/perfiles`), que nada tiene que ver con `gda_persona.perfil`. Ver
`Documentacion_BackEnd_GDA_2607_Profiles.md` §2 para el detalle completo de los 4 conceptos distintos
que comparten la palabra "Perfil" en este código.

Back y Front de este módulo ya estaban desarrollados antes de esta sesión — el trabajo aquí es de
**auditoría/validación**, no de construcción. El primer volcado de esquema disponible
(`.vscode/BBDD/gda_esquema_bbdd_251022.md`, octubre 2025) resultó estar desactualizado para este módulo
(ver sección 3); el DDL de esta sección 2 es el real, aportado directamente por el usuario contra la BBDD
el 2026-07-28.

==========================================
2. DDL REAL (aportado por el usuario, 2026-07-28)
==========================================

-- gda_admingade.perfil_ineco definition

-- Drop table

-- DROP TABLE gda_admingade.perfil_ineco;

CREATE TABLE gda_admingade.perfil_ineco (
	id bigserial NOT NULL,
	codigo varchar(255) NOT NULL,
	tipo_perfil_id int8 NOT NULL,
	porcentaje_id int8 NULL,
	tarifa float8 NOT NULL,
	tarifa_sin_tasa float8 NOT NULL,
	tarifa_tasa float8 NOT NULL,
	perfil_ineco bool NOT NULL,
	observaciones varchar(200) NULL,
	perfil_ineco_id int8 NULL,
	fecha_baja date NULL,
	fecha_alta timestamp NULL,
	fecha_modificacion timestamp NULL,
	usuario_modificacion_id int8 NULL,
	CONSTRAINT pk_perfil_ineco PRIMARY KEY (id),
	CONSTRAINT uq_tipo_perfil_ineco UNIQUE (codigo),
	CONSTRAINT fk_perfil_ineco_tipo_perfil FOREIGN KEY (tipo_perfil_id) REFERENCES gda_admingade.tipo_perfil(id),
	CONSTRAINT fk_perfil_ineco_usuario_modificacion_id FOREIGN KEY (usuario_modificacion_id) REFERENCES gda_persona.persona(id),
	CONSTRAINT perfil_ineco_perfil_ineco_fk FOREIGN KEY (perfil_ineco_id) REFERENCES gda_admingade.perfil_ineco(id),
	CONSTRAINT perfil_ineco_porcentaje_fk FOREIGN KEY (porcentaje_id) REFERENCES gda_admingade.porcentaje(id)
);

-- Permissions

ALTER TABLE gda_admingade.perfil_ineco OWNER TO postgres;
GRANT INSERT, SELECT, TRUNCATE, REFERENCES, DELETE, TRIGGER, UPDATE ON TABLE gda_admingade.perfil_ineco TO postgres;
GRANT INSERT, SELECT, DELETE, UPDATE ON TABLE gda_admingade.perfil_ineco TO userdesarrollo;


```sql
-- gda_admingade.perfil definition
CREATE TABLE gda_admingade.perfil (
	id bigserial NOT NULL,
	codigo varchar(255) NULL,
	id_perfil varchar(255) NULL,
	nombre varchar(255) NULL,
	observaciones varchar(200) NULL,
	origen varchar(255) NULL,
	porcentaje float8 NULL,
	tarifa float8 NULL,
	tarifa_tasa float8 NULL,
	tarifa_venta float8 NULL,
	proveedor_id int8 NULL,
	tarifa_sin_tasa float8 NULL,
	perfil_ineco_id int8 NULL,
	CONSTRAINT pk_perfil PRIMARY KEY (id),
	CONSTRAINT uk_perfil UNIQUE (id_perfil, codigo),
	CONSTRAINT fk_perfil_perfil_ineco FOREIGN KEY (perfil_ineco_id) REFERENCES gda_admingade.perfil(id),
	CONSTRAINT fk_perfil_proveedor FOREIGN KEY (proveedor_id) REFERENCES gda_persona.proveedor(id)
);

-- gda_admingade.tipo_perfil definition
CREATE TABLE gda_admingade.tipo_perfil (
	id bigserial NOT NULL,
	codigo varchar(50) NOT NULL,
	descripcion varchar(255) NOT NULL,
	CONSTRAINT tipo_perfil_pkey PRIMARY KEY (id),
	CONSTRAINT uq_tipo_perfil UNIQUE (codigo)
);

-- gda_admingade.porcentaje definition
CREATE TABLE gda_admingade.porcentaje (
	id bigserial NOT NULL,
	valor float8 NOT NULL,
	porcentaje_ineco bool NOT NULL,
	fecha_alta date NOT NULL,
	fecha_baja date NULL,
	CONSTRAINT porcentaje_pkey PRIMARY KEY (id)
);

-- gda_admingade.recurso definition (consumidor, FK a perfil)
CREATE TABLE gda_admingade.recurso (
	id bigserial NOT NULL,
	fecha_fin date NULL,
	fecha_inicio date NULL,
	id_personal varchar(255) NULL,
	id_recurso varchar(255) NULL,
	observaciones varchar(200) NULL,
	pedido_id int8 NULL,
	perfil_id int8 NULL,
	personal_id int8 NULL,
	recurso_id_origin int8 NULL,
	tarifa_guardia float8 NULL,
	perfil_licitacion_id int8 NULL,
	persona_id int8 NULL,
	usuario_modificacion_id int8 NULL,
	fecha_modificacion timestamp NULL,
	fecha_alta timestamp NULL,
	CONSTRAINT pk_recurso PRIMARY KEY (id),
	CONSTRAINT fk_recurso_pedido FOREIGN KEY (pedido_id) REFERENCES gda_admingade.pedido(id),
	CONSTRAINT fk_recurso_perfil FOREIGN KEY (perfil_id) REFERENCES gda_admingade.perfil(id),
	CONSTRAINT fk_recurso_perfil_licitacion FOREIGN KEY (perfil_licitacion_id) REFERENCES gda_admingade.perfil_licitacion(id),
	CONSTRAINT fk_recurso_persona FOREIGN KEY (persona_id) REFERENCES gda_persona.persona(id),
	CONSTRAINT fk_recurso_personal FOREIGN KEY (personal_id) REFERENCES gda_persona.personal(id),
	CONSTRAINT fk_recurso_usuario_modificacion_id FOREIGN KEY (usuario_modificacion_id) REFERENCES gda_persona.persona(id)
);
CREATE INDEX idx_recurso_persona_fecha ON gda_admingade.recurso USING btree (personal_id, fecha_inicio, fecha_fin);
```

Nótese que `recurso` conserva **ambas** columnas `personal_id` (FK a `gda_persona.personal`, legado) y
`persona_id` (FK a `gda_persona.persona`, nuevo) al mismo tiempo — la tabla ya soporta el modelo nuevo,
coherente con el refactor GDAV3-429 (ver `Documentacion_BackEnd_GDA_2607_Profiles.md` §5).

==========================================
3. HALLAZGO CRÍTICO: `Perfil.java` (entidad JPA) NO coincide con la tabla real
==========================================

Comparación campo a campo entre la entidad JPA real
([Perfil.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Perfil.java))
y el DDL real de la sección 2:

| Columna real en BBDD | ¿La mapea `Perfil.java`? |
|---|---|
| `codigo` | ✅ `private String codigo;` |
| `id_perfil` | ❌ no mapeado |
| `nombre` | ❌ no mapeado |
| `observaciones` | ✅ `private String observaciones;` |
| `origen` | ❌ no mapeado |
| `porcentaje` (float8, columna simple) | ❌ — la entidad espera `porcentaje_id` como FK a una tabla `Porcentaje` (`@ManyToOne @JoinColumn(name="porcentaje_id") private Porcentaje porcentaje;`), columna que **no existe** en la tabla real |
| `tarifa` / `tarifa_tasa` / `tarifa_sin_tasa` | ✅ los 3 mapeados |
| `tarifa_venta` | ❌ no mapeado |
| `proveedor_id` | ❌ no mapeado (la entidad no tiene ningún campo de proveedor) |
| `perfil_ineco_id` | ✅ mapeado, pero como `perfilInecoId` tipado `Perfil` (autorreferencia) — coincide en nombre de columna |
| — (no existe) | ❌ la entidad espera `tipo_perfil_id` (`@ManyToOne @JoinColumn(name="tipo_perfil_id") private TipoPerfil tipoPerfil;`) — **columna inexistente en la tabla real** |
| — (no existe) | ❌ la entidad espera `perfil_ineco` (booleano) — **columna inexistente** |
| — (no existe) | ❌ la entidad espera `fecha_alta`, `fecha_baja`, `fecha_modificacion`, `usuario_modificacion_id` (patrón de auditoría ADR-0606) — **ninguna existe en la tabla real** |

**Conclusión:** `Perfil.java` fue escrita para un esquema de tabla que todavía no existe en la BBDD real —
parece el resultado de una refactorización de código adelantada a su migración de esquema (añadir
`TipoPerfil`/`Porcentaje` como catálogos propios, quitar la relación directa con `Proveedor`, añadir
auditoría), migración que nunca se ejecutó sobre `gda_admingade.perfil`. `TipoPerfil.java` y
`Porcentaje.java`, en cambio, **sí coinciden exactamente** con sus tablas reales (`tipo_perfil`,
`porcentaje`) — solo `perfil` quedó a medias.

**Efecto práctico:** cualquier operación JPA sobre `Perfil` (`findAll`, `save`, etc., vía
`PerfilRepository`) generará SQL contra columnas (`tipo_perfil_id`, `porcentaje_id`, `perfil_ineco`,
`fecha_alta`...) que no existen — fallará en tiempo de ejecución con un error SQL (no se detecta al
arrancar porque no hay `ddl-auto: validate` configurado en `gda-admingade`, ver
`Documentacion_BackEnd_GDA_2607_Profiles.md` §4). Esto es, muy probablemente, la causa real por la que
este módulo necesita auditoría: **`GET /admingade/perfiles` probablemente falla en tiempo de ejecución**,
no solo por la ruta de Gateway que faltaba (ya corregida) sino por este desajuste de esquema, que sigue
sin corregir. **Pendiente de probar** end-to-end para confirmarlo.

==========================================
4. TABLA `tipo_perfil` Y `porcentaje` — SIN discrepancias
==========================================

A diferencia de `perfil`, estas dos tablas coinciden exactamente con sus entidades:
- `tipo_perfil` ↔ [TipoPerfil.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/TipoPerfil.java): `id`, `codigo` (`UNIQUE`), `descripcion`. Sin auditoría (no la necesita, ADR-0606 aplica a tablas nuevas de negocio con ciclo de vida, no a catálogos estáticos como este — a confirmar criterio).
- `porcentaje` ↔ [Porcentaje.java](../../GDA/backend/gda-finance/gda-admingade/src/main/java/com/ineco/gda/admingade/data/model/Porcentaje.java): `id`, `valor`, `porcentaje_ineco`, `fecha_alta` (`NOT NULL`), `fecha_baja`. Coincide exactamente.

==========================================
5. `.vscode/BBDD/gda_esquema_bbdd_251022.md` (octubre 2025) — desactualizado para este módulo
==========================================

Ese volcado (el único disponible antes de que el usuario aportara el DDL real de la sección 2) sí
mostraba la forma **antigua** de `gda_admingade.perfil` — coincide de hecho con el DDL real actual de la
sección 2 (`codigo`/`nombre`/`origen`/`proveedor_id`/`tarifa_venta`, `porcentaje` como columna simple),
confirmando que **la tabla real nunca cambió** desde octubre 2025. Lo que sí quedó desactualizado (o
adelantado) es el código Java (`Perfil.java`), no el volcado de esquema. `tipo_perfil` y `porcentaje`
como tablas propias **tampoco existían** en el volcado de octubre — se crearon después, ya con forma
correcta y ya reflejada en su entidad respectiva.

==========================================
6. PENDIENTE
==========================================

1. Confirmar con Ricardo (autor de los cambios de `Perfil.java`/`TipoPerfil.java`/`Porcentaje.java`, ver
   `Documentacion_BackEnd_GDA_2607_Profiles.md` §5) si la migración de `gda_admingade.perfil` (añadir
   `tipo_perfil_id`, `porcentaje_id`, `perfil_ineco`, columnas de auditoría; quitar/migrar `proveedor_id`,
   `nombre`, `origen`, `tarifa_venta`, `id_perfil`) está prevista y simplemente no se ha ejecutado
   todavía, o si `Perfil.java` se adelantó por error.
2. Probar `GET /admingade/perfiles` de extremo a extremo (ya con la ruta de Gateway añadida, ver
   `Documentacion_BackEnd_GDA_2607_Profiles.md` §7) para confirmar si efectivamente falla en tiempo de
   ejecución por este desajuste.
3. Decidir si se documenta como deuda técnica (ADR-0603) o se prioriza la migración real de la tabla.
