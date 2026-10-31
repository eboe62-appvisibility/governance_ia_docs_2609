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
