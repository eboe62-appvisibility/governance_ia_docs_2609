## 01 FRONT

Siguiendo el mismo modelo de desarrollo documentado en:
.vscode\docs\Documentacion_FrontEnd_GDA_2607_Suppliers.md
.vscode\docs\Documentacion_BackEnd_GDA_2607_Suppliers.md
.vscode\docs\Documentacion_BBDD_GDA_2607_Suppliers.md

Vamos a crear en la rama GDAV3-446 del microservicio gda-admingade una nueva pantalla en la aplicación denominada LICENCIAS con características similares a la desarrollada para PROVEEDORES (Suppliers) en la rama GDAV3-443 del microservicio gda-persona (aun no mergeada con Develop)

El trabajo que vayamos haciendo lo iremos documentando en:
.vscode\docs\Documentacion_GDA_2607_Licenses.md
siguiendo un esquema semejante al utilizado en los documentos que nos sirven de muestra.

Empezaremos con el Front:
En el menu principal se ubicará entre "Personas ETT" y "Consultas"
Empezaremos por dar forma a la tabla que tendrá las siguientes columnas:

	licencia
	localización
	estado
	expediente compra
	observaciones
	fecha modificación
	acciones (editar - consultar - eliminar)

Esta es la tabla en BBDD:

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

## 02

Como puedo comprobar el resultado el 404 me impide ver la pantalla

## 03

Correcciones:
Hay que crear un botón AÑADIR siguiendo el mismo procedimiento utilizado en Suppliers
Vamos a incorporar una columna FECHA BAJA entre FECHA MODIFICACION y ACCIONES

## 02b

Cambiamos el nombre del documento Documentacion_GDA_2607_Licenses.md a Documentacion_FrontEnd_GDA_2607_Licenses.md
e desarrollamos el BackEnd para poder ver los resultados del FrontEnd
seguimos los pasos utilizados en .vscode\docs\Documentacion_BackEnd_GDA_2607_Suppliers.md
y documentamos en .vscode\docs\Documentacion_BackEnd_GDA_2607_Licenses.md

## 03b

Correcciones:
- Fecha Alta debe de ser una de los imputs del formulario AÑADIR presentándose la fecha actual por defecto
- EDITAR y CONSULTAR no muestran el formulario
- DELETE no tiene funcionalidad
- En la tabla debe de mostrarse la columna FECHA BAJA entre FECHA EFECTO y ACCIONES

## 04

1.- EDITAR no permte guardar modificaciones:

axiosCall.js:66
 PUT http://localhost:9007/admingade/licencias 405 (Method Not Allowed)
Promise.then
(anonymous)	@	axiosCall.js:66
promiseAxios	@	axiosCall.js:64
put	@	axiosCall.js:26
putLicense	@	admingade.js?t=1784930304163:35
updateLicense	@	useUpdateLicense.js:22
(anonymous)	@	Licenses.vue:108
(anonymous)	@	Licenses.vue:115
(anonymous)	@	licenseForm.vue:124
await in (anonymous)
(anonymous)	@	formFooter.vue:53

utils.js?t=1784919704955:136
AxiosError {message: 'Request failed with status code 405', name: 'AxiosError', code: 'ERR_BAD_REQUEST', config: {…}, request: XMLHttpRequest, …}
cError	@	utils.js?t=1784919704955:136
updateLicense	@	useUpdateLicense.js:27
await in updateLicense
(anonymous)	@	Licenses.vue:108
(anonymous)	@	Licenses.vue:115
(anonymous)	@	licenseForm.vue:124
await in (anonymous)
(anonymous)	@	formFooter.vue:53

{
    "timestamp": "2026-07-24T22:07:49.452+00:00",
    "path": "/admingade/licencias",
    "status": 405,
    "error": "Method Not Allowed",
    "requestId": "2df7ac19"
}

2.- DELETE no completa la operacion
axiosCall.js:66
 DELETE http://localhost:9007/admingade/licencias/1739 404 (Not Found)
Promise.then
(anonymous)	@	axiosCall.js:66
promiseAxios	@	axiosCall.js:64
delete	@	axiosCall.js:24
deleteLicense	@	admingade.js?t=1784930304163:36
deleteLicense	@	useDeleteLicense.js:18
(anonymous)	@	Licenses.vue:119
(anonymous)	@	dialogOkCancel.vue:54
utils.js?t=1784919704955:136
AxiosError {message: 'Request failed with status code 404', name: 'AxiosError', code: 'ERR_BAD_REQUEST', config: {…}, request: XMLHttpRequest, …}
cError	@	utils.js?t=1784919704955:136
deleteLicense	@	useDeleteLicense.js:22
await in deleteLicense
(anonymous)	@	Licenses.vue:119
(anonymous)	@	dialogOkCancel.vue:54



# 10

Cuando una LICENCIA se le ha asignado una Fecha Baja los inputs del formulario EDITAR deberían quedar deshabilitados.

# 11

En EDITAR, una vez elegido el ESTADO Baja Definitiva aparece por defecto la fecha actual en el input FECHA BAJA pero no permite modificar la fecha. Hasta que no se Guarde la fecha de Baja en la tabla no deben deshabilitarse los inputs del formulario EDITAR
