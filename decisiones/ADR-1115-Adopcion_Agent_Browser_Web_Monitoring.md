# ADR-1115: Adopción de agent-browser para la monitorización automatizada de fuentes documentales

Status: APPROVED

Date: 2026-08-08

Scope:
Knowledge

Category:
ARCHITECTURE

Domain: (Optional)
Documentation, Research

Tags:
web-monitoring, agent-browser, scraping, fuentes-documentales

Related ADRs:
ADR-1108 (patrón de tabla + identificador de bloque que la columna `Estado` de los índices de fuentes actualiza), ADR-1111 (criterios de interés — contraste de fuentes contradictorias), ADR-1113 (generalización del índice de fuentes documentales curadas que este mecanismo mantiene actualizado), ADR-1114 (Understand-Anything como Knowledge Provider — sistema independiente, no relacionado operativamente con este), ADR-1116 (metodología de evaluación comparativa de fuentes — usa este mecanismo como motor de acceso, aunque el método es agnóstico de herramienta), ADR-1119 (exige confirmar por lectura directa cualquier resumen o extracción automática que produzca este mecanismo antes de citar un hecho concreto — punto 5 de esa Decision)

Supersedes:
NONE

Superseded By:
NONE

---

## Context

El proyecto necesita un mecanismo sistemático para detectar novedades en las fuentes ya catalogadas en `conocimiento_index/260806-index-documentation-fuentes-curadas.md`. Muchas de esas fuentes son dinámicas (enciclopedias colaborativas, bases de datos académicas, medios digitales), por lo que el seguimiento manual es ineficiente y propenso a lagunas de información.

La prioridad de esta monitorización no es genérica: debe favorecer las fuentes que reflejan la perspectiva propia del ámbito de estudio sobre el resto del mundo, no solo la mirada externa hacia ese ámbito. En el índice actual esa asimetría ya es visible — la prensa que observa China desde una mirada occidental (ABC, El País, The New York Times) está íntegramente transcrita (`Estado: Incorporada`), mientras que las fuentes genuinamente chinas (prensa de la diáspora, enciclopedias, bases de datos académicas, plataformas sociales) siguen mayoritariamente en `Estado: Referenciada`, sin contenido propio incorporado.

Esta decisión estuvo motivada, en parte, por una fuente externa concreta: un vídeo de youtube (ver URL en el Anexo).

Inicialmente, esta decisión y su procedimiento operativo se redactaron como documento `Project_Definition/07_WEB_MONITORING.md`. Se reubican aquí porque `Project_Definition/00`-`06` están explícitamente diseñados para ser independientes de tecnología concreta (`02_MASTER_ARCHITECTURE.md`: *"Multiple implementations may coexist"*; `03_ONTOLOGY.md`: *"Implementation technologies may evolve. The ontology shall remain stable"*), mientras que la elección de una herramienta concreta de scraping, con sus alternativas descartadas y su procedimiento operativo específico, es exactamente el tipo de decisión revisable y con fecha para la que existen los ADR — mismo criterio ya aplicado en `ADR-1109` para Obsidian y en `ADR-1114` para Understand-Anything.

---

## Decision

1. **Motor de interacción principal: agent-browser (Vercel Labs)**, un CLI en Rust que se comunica con un daemon persistente, también en Rust, vía Chrome DevTools Protocol directo — sin Node.js ni Playwright de por medio (verificado contra el README del proyecto). El daemon arranca solo al primer comando y se cierra tras una hora de inactividad. Se elige por: (a) ser gratuito y ejecutarse en local, coherente con que el proyecto no tiene CI y las novedades se invocan manualmente desde el entorno de desarrollo habitual; (b) gestión real de sesión/autenticación (`state save/load`), imprescindible para acceder a CNKI, la Enciclopedia de China, o el Instituto Confucio de Madrid/GISEC con las credenciales institucionales propias del usuario — ningún servicio de solo lectura (Firecrawl, Jina Reader) resuelve esto; (c) integración nativa como skill de Claude Code. Esta reutilización de sesión está limitada al acceso legítimo ya autorizado del usuario — nunca como mecanismo para eludir el muro de pago de fuentes como CNKI o Wanfang.

2. **Servicio de soporte para extracción pasiva: Jina Reader**, reservado para fuentes estáticas simples (prensa, artículos sin JavaScript pesado) que no requieren sesión ni interacción — por tener un nivel gratuito real, a diferencia de Firecrawl.

3. **Herramientas descartadas para esta fase**, por redundancia con agent-browser en el caso de "interacción compleja": Firecrawl (API de pago sin ventaja real sobre Jina Reader a este volumen de fuentes), MultiOn (pivotó hacia una app móvil de consumo, AGI-0, alejándose de este caso de uso), Skyvern y Crawl4AI (quedan documentadas como alternativa futura si se necesitara un backend Python a medida, no como opción activa hoy). Ver Anexo para la comparativa completa.

4. **Procedimiento operativo**:
   1. *Detección de novedades*: `agent-browser snapshot` compara el árbol de accesibilidad actual con el almacenado. Es un diff **estructural**, no de contenido — detecta que *algo* cambió, no basta por sí solo para saber si el texto de una entrada ha cambiado.
   2. *Extracción y contraste de contenido*: ante una señal de cambio, `agent-browser read [url] --require-md` obtiene el Markdown actualizado; el contraste real de novedad se hace diferenciando ese Markdown contra la versión guardada anteriormente (p. ej. versionada en el propio repositorio), no contra el snapshot estructural del paso 1.
   3. *Validación de fuentes (ADR-1111)*: si la novedad procede de una fuente de la diáspora como China Minutes, el sistema dispara automáticamente un `agent-browser open` hacia fuentes oficiales o académicas para confirmar el dato antes de actualizar el índice, manteniendo el nivel de confianza.
   4. *Evidencia visual en discrepancias*: cuando el paso 3 no logra confirmar el dato, `agent-browser screenshot --annotate` captura una imagen anotada (etiquetas numeradas sobre los elementos interactivos, mapeadas a los mismos refs `@eN` del snapshot) que se adjunta al log de la novedad, para que la revisión humana no dependa solo de la descripción textual.
   5. *Trazabilidad de cambios*: cada actualización genera un log —incluyendo la evidencia visual del paso 4 cuando exista— que permite revisar la evolución de la fuente antes de consolidarla en el activo de conocimiento y de actualizar la columna `Estado` (ADR-1113). **Ubicación (decidido 2026-08-09)**: `logs/`, carpeta de primer nivel hermana de `fuentes/` (no hija) — un log de monitorización no es una fuente primaria de conocimiento, es un artefacto operativo del propio mecanismo de vigilancia, y mezclarlo dentro de `fuentes/` confundiría ambos conceptos.

5. **Principios técnicos que rigen la implementación**: Markdown-First (toda extracción se convierte a Markdown antes de procesarse); Veracidad Automatizada (no se inventan metadatos — fecha, autor; en caso de duda, se marca como evidencia insuficiente); Eficiencia sobre Volumen (se prioriza metadatos/resúmenes precisos sobre descarga masiva sin estructurar); Integridad del Índice (toda novedad detectada sigue las normas de inserción ya fijadas por `ADR-1108`).

6. **Fuera de alcance**: este mecanismo no debe vulnerar términos de servicio de bases de pago (CNKI, Wanfang), automatizar decisiones críticas sin supervisión humana, ni sustituir el juicio de veracidad sobre fuentes contradictorias — ese juicio sigue correspondiendo al criterio de confianza ya fijado en `ADR-1111`.

---

## Instalación, alcance y desinstalación (añadido 2026-08-08)

**Alcance de la instalación — al margen del proyecto, no embebida**: agent-browser se instala como herramienta de sistema (`npm install -g agent-browser` + `agent-browser install`, o vía Homebrew/Cargo), no como código versionado de este repositorio — mismo patrón ya establecido por `understand-anything` (`ADR-1114`): el binario/plugin vive en el entorno del usuario, y lo único que se versiona aquí es la decisión (este ADR), el resultado de cada ejecución (Markdown extraído hacia `fuentes/`, columna `Estado` actualizada en el índice) y, cuando exista, el log/evidencia visual de discrepancias. Este proyecto no tiene `package.json` ni lo necesita para esto — una instalación "de proyecto" vía `npm install agent-browser` (sin `-g`) exigiría crear ese fichero solo para declarar una dependencia de herramienta, introduciendo un patrón de tooling Node ajeno al resto del repositorio (100% Markdown/JSON/config); se descarta por ese motivo.

**Verificación de no-duplicación (comprobado 2026-08-08 contra el sistema real, no asumido)**: antes de instalar, se auditó si ya existía algo reutilizable o que fuera a duplicarse.
- El `node_modules` más grande del sistema (`~/.understand-anything/repo/node_modules`, 652 MB) pertenece a las dependencias de *desarrollo* del propio plugin de Understand-Anything (TypeScript, vitest, eslint) — verificado que no contiene Playwright ni Puppeteer ni nada relacionado con navegadores.
- No existe ningún binario de Chrome/Chromium ni caché de Playwright/Puppeteer en el sistema (`~/.cache/ms-playwright`, `~/.cache/puppeteer` u homólogos) — la descarga de Chrome for Testing que hace `agent-browser install` no duplica nada ya presente.
- Una instalación global (`-g`) no comparte árbol de dependencias con ningún proyecto — no hay mecanismo de duplicación posible con el `node_modules` de otra herramienta.

**Desinstalación — parcialmente documentada por el proveedor, no garantizada "sin huella" solo con los comandos oficiales**:
- Binario: `npm uninstall -g agent-browser` (o `brew uninstall agent-browser` / `cargo uninstall agent-browser`, según el método de instalación usado).
- Datos persistentes conocidos (documentados en el README): `~/.agent-browser/config.json`, `~/.agent-browser/.encryption-key`, `~/.agent-browser/sessions/`.
- Limpieza de estado en caliente, antes de desinstalar el binario: `agent-browser state clear --all` (borra estados de sesión guardados), `agent-browser cookies clear` y `agent-browser storage local clear`, `agent-browser close --all` (cierra el daemon y sesiones activas).
- `agent-browser doctor --fix` ejecuta "reparaciones destructivas (reinstalar Chrome, purgar estado antiguo...)" — el comando de diagnóstico más cercano a una limpieza completa que ofrece la propia herramienta.
- **Ruta del binario de Chrome — confirmada por ejecución real (2026-08-08)**: la documentación pública (README, sitio de instalación) no especificaba esta ruta; se confirmó ejecutando `agent-browser install` directamente, no por documentación de terceros. Chrome for Testing queda en `~/.agent-browser/browsers/chrome-<versión>/` (p. ej. `~/.agent-browser/browsers/chrome-151.0.7922.77`, 184 MB, instalado el 2026-08-08) — dentro del mismo directorio `~/.agent-browser/` ya documentado más arriba (`config.json`, `.encryption-key`, `sessions/`), no en una ruta de sistema separada. Consecuencia práctica: `rm -rf ~/.agent-browser/` elimina de una sola vez configuración, sesiones **y** el navegador descargado — no hacen falta pasos adicionales para una desinstalación completa, más allá de `npm uninstall -g agent-browser` para el propio binario CLI.
- **Detalle versionado**: al incluir el número de versión en el nombre de la carpeta (`chrome-<versión>/`), un `agent-browser upgrade` que traiga una versión más reciente de Chrome probablemente cree una carpeta nueva junto a la anterior en vez de sustituirla — no verificado todavía (no se ha ejecutado `upgrade` en este proyecto); revisar `~/.agent-browser/browsers/` tras cualquier actualización futura para confirmar si quedan versiones antiguas huérfanas.
- **Nota operativa (Linux/WSL)**: la instalación avisó explícitamente que, si el navegador falla al lanzarse por bibliotecas compartidas ausentes, el remedio es `agent-browser install --with-deps` — relevante para este entorno WSL, donde es más probable que falten paquetes del sistema que en una distribución Linux de escritorio completa.

**Ubicación de scripts de automatización (decidido 2026-08-09)**: si el procedimiento operativo (punto 4 de la Decision) llega a justificar un script que encadene los comandos de `agent-browser` en vez de invocarlos a mano, vivirá en `scripts/`, carpeta de primer nivel nueva, coherente con la convención de nombres en minúscula ya usada por el resto de carpetas raíz del proyecto (`fuentes/`, `conocimiento/`, `conocimiento_index/`, `vocabulary/`). No se crea todavía — Principio 03 de este mismo ADR (Eficiencia sobre Volumen) desaconseja automatizar antes de validar el patrón manual sobre fuentes reales.

**Deslinde con Obsidian Web Clipper (aclarado 2026-08-09)**: el proyecto ya usa la extensión Obsidian Web Clipper para captura manual de páginas hacia `fuentes/` (evidencia: frontmatter `tags: [clippings]` presente en varios ficheros de `fuentes/260801_youtube/` y `fuentes/260101_calendario/`). agent-browser no la sustituye — cubre un caso de uso distinto: Web Clipper es captura puntual de una página que el usuario ya está viendo en su propio navegador y sesión; agent-browser es vigilancia repetible y programable de fuentes ya catalogadas, con sesión persistente reutilizable entre ejecuciones. Mismo criterio de deslinde de herramientas ya aplicado en `ADR-1109` (Obsidian vs. Understand-Anything).

---

## Experimento de viabilidad por fuente y política de sesión (añadido 2026-08-09)

**Método del experimento**: se probó `agent-browser open <url>` contra las 16 fuentes con `Estado: Referenciada` de `conocimiento_index/260806-index-documentation-fuentes-curadas.md` (origen concreto de la prueba: la extracción comparativa de 腊八节/Laba Festival desde varias fuentes chinas). Cada fuente se abrió en una sesión aislada (`--session <nombre-único>`) para no interferir con el uso manual del usuario. **Regla de verificación aplicada**: el mensaje de estado del comando `open` (incluido "✗ Operation timed out") no se tomó nunca como resultado final por sí solo — se confirmó siempre con `get url` (¿navegó realmente?) y `screenshot` (¿qué contenido hay?), y con `eval`/`read` cuando la captura era ambigua. Esta regla resultó imprescindible: dos fuentes (zgbk.com y, en el experimento previo, Baidu Baike) reportaron "timeout" pero habían cargado contenido real — sin esa verificación se habrían descartado erróneamente como no viables.

**Resultado — 12 de 16 fuentes viables sin intervención**: acceso público, sin CAPTCHA ni muro de login bloqueante — baike.baidu.com, www.zgbk.com, www.baike.com, www.douban.com, chinaminutes.com, www.xihua.es, www.sixthtone.com, www.thinkchina.sg, www.wanfangdata.cn, www.ccchinamadrid.org, www.ucm.es/gisec, www.confuciomadrid.es. Detalle por fuente en la columna "Especialización / Fiabilidad" del índice.

**Resultado — 3 de 16 fuentes no monitorizables vía agent-browser**, cada una por un motivo distinto (no tratar como un único patrón):
1. **CNKI (oversea.cnki.net)** — CAPTCHA de deslizar pieza ("拖动下方拼图完成验证") tras cualquier búsqueda automatizada, incluso con sesión ya autenticada. No se intentó sortear — cruzaría el `OUT OF SCOPE` (punto 6 de la Decision).
2. **Zhihu (www.zhihu.com)** — muro de login obligatorio (código SMS +86 o QR/WeChat) incluso en páginas de búsqueda; no expone ningún contenido a un visitante no autenticado, con o sin agent-browser de por medio. No es un problema de la herramienta, es un límite estructural de la fuente.
3. **qikan.chaoxing.com** — acceso denegado ("对不起，您没有权限请求该地址"), probablemente restricción geográfica o de red institucional; no confirmado si un acceso autenticado lo resolvería.

**Hallazgo adicional — ouhua.info**: falla en ambos protocolos (HTTPS con certificado inválido, HTTP bloqueado por el propio navegador) — posible problema real del sitio en el momento de la prueba, a confirmar manualmente antes de descartarlo de forma permanente.

**Hallazgo adicional — nssd.cn**: no era un fallo de agent-browser sino una URL obsoleta en el índice (el dominio mostraba desde 2024-07-11 un aviso oficial de cese de servicio). Verificada la nueva URL (`www.ncpssd.cn`): accesible, con buscador funcional y 19 resultados académicos reales para 腊八节 — URL actualizada en el índice.

**Advertencias críticas para el uso futuro de este mecanismo**:
1. Nunca concluir "fuente no viable" a partir solo del código de salida o el mensaje de texto de `open` — confirmar siempre con `get url` + `screenshot` (falso negativo confirmado dos veces).
2. Un CAPTCHA (CNKI) y un muro de login estructural (Zhihu) son ambos fallos definitivos para este mecanismo — no se contemplan bypasses (anti-detección, resolución automática de CAPTCHA, credenciales de terceros), conforme al punto 6 de la Decision.
3. Un error de acceso o de certificado (qikan.chaoxing.com, ouhua.info) no es necesariamente permanente ni indica bloqueo intencional — requiere una segunda verificación manual antes de descartar la fuente del índice.

### Segundo experimento: acceso a contenido real, no solo a portada (añadido 2026-08-09)

El experimento inicial (arriba) solo verificó que la portada de cada fuente cargara sin CAPTCHA/login. Un segundo experimento, con el objetivo de extraer contenido real sobre 腊八节 de cada una de las 12 fuentes "viables", reveló que **ese criterio era insuficiente**: tres fuentes que habían pasado la prueba de portada fallaron al intentar buscar o acceder a un artículo concreto:

1. **www.baike.com** — portada y buscador cargan limpios, pero al entrar en un artículo concreto salta un CAPTCHA de puzzle deslizante ("请完成下列验证后继续") — mismo patrón que CNKI.
2. **www.douban.com** — portada limpia, pero toda búsqueda (por URL directa o interactuando con el cuadro) devuelve "获取搜索结果失败: 错误103" — fallo de API, posible bloqueo a nivel de backend.
3. **www.wanfangdata.cn** — portada y buscador visibles, pero ejecutar una búsqueda concreta devuelve "fault filter abort" — mensaje típico de un WAF bloqueando la petición.

Además, dos fuentes fallaron por causas ajenas a bloqueos anti-bot: **www.thinkchina.sg** sufrió una caída real del sitio en el momento de la prueba (aviso propio "Website outage"); **www.ucm.es/gisec** devolvió un 404 real de la propia UCM (URL retirada, corregida a `www.ucm.es/sinologiaespanolacomplutense/` con ayuda del usuario).

**Consecuencia práctica — reclasificación en cuatro categorías** (ver `conocimiento_index/260806-index-documentation-fuentes-curadas.md`, sección Description): "accesible" ya no significa solo "portada sin CAPTCHA" — significa portada **y** contenido/búsqueda verificados con éxito, con un archivo fuente generado en `fuentes/260101_calendario/` como evidencia. Las fuentes que fallan de forma no estructural (error de API, WAF puntual, caída temporal, URL obsoleta) se separan en una tabla "segunda oportunidad" — hay una posibilidad razonable de que otra entrada (otra URL, otro momento, otro método de búsqueda) tenga éxito. Las que fallan por una medida anti-bot deliberada y confirmada (CAPTCHA, muro de login) quedan en "no accesible" — no se espera que un reintento cambie el resultado.

**Advertencia crítica adicional**: antes de clasificar una fuente como "accesible" para consulta automatizada, verificar no solo la portada sino al menos una búsqueda o navegación a contenido real — la portada limpia de baike.com, douban.com y wanfangdata.cn habría llevado a una clasificación optimista incorrecta si el experimento se hubiera detenido ahí.

### Política de advertencia por colisión de sesión compartida

agent-browser usa una sesión `default` compartida por todas las invocaciones del mismo usuario del sistema que no especifiquen `--session`. Si el usuario abre manualmente el navegador de agent-browser (p. ej. `agent-browser open <url> --headed`) mientras el asistente de IA está operando sobre la misma sesión, ambos actores chocan sobre la misma pestaña — confirmado en este proyecto durante el experimento con CNKI (2026-08-09).

Reglas adoptadas a partir de este incidente:
1. **El asistente de IA usa siempre una sesión con nombre único** (`--session <nombre-descriptivo>`) para cualquier operación propia — nunca la sesión `default`, que se reserva para el uso manual/interactivo del usuario.
2. Antes de operar sobre una sesión potencialmente compartida, verificar con `agent-browser session info --json` si ya está `active`.
3. **Las sesiones son efímeras salvo persistencia explícita**: sin `--restore`, `--profile` o `state save`, el perfil de Chrome de una sesión (cookies, login) vive solo en memoria y se pierde al cerrarla (`close`) — incluida una sesión `default` con un login manual reciente del usuario. `close --all` cierra literalmente todas las sesiones activas, propias y del usuario. Antes de ejecutarlo, confirmar que ninguna sesión ajena a la tarea en curso contiene un estado de login que el usuario quiera conservar.

---

## Comandos para consultas puntuales (añadido 2026-08-09)

A diferencia de un experimento comparativo (`ADR-1116`, que recorre las 11 fuentes "accesibles" para un mismo tema), esta sección documenta cómo consultar **una única fuente ya elegida** para un término concreto — el caso de uso más frecuente una vez se conoce qué fuentes rinden mejor para qué tipo de contenido. Recetas verificadas para las 4 fuentes de mejor comportamiento hasta ahora (ver `Valoración` en `conocimiento_index/260806-index-documentation-fuentes-curadas.md`):

```bash
TERM="tu_termino_en_chino"   # p. ej. TERM="故宫"

# Wikipedia — un solo comando, lectura directa
agent-browser read "https://zh.wikipedia.org/wiki/${TERM}" > "fichero_destino.md"

# Baidu Baike — open + read (el read directo sobre la URL suele dar 403)
agent-browser open "https://baike.baidu.com/item/${TERM}" --session lookup
agent-browser read --session lookup > "fichero_destino.md"
agent-browser close --session lookup

# Douban — vía subdominio móvil (el buscador de escritorio falla con "错误103")
agent-browser open "https://m.douban.com/search/?query=${TERM}" --session lookup
agent-browser read --session lookup > "fichero_destino.md"
agent-browser close --session lookup

# Xihua — requiere interacción con el buscador del foro (mismo patrón que NCPSSD)
agent-browser open "https://www.xihua.es" --session lookup
agent-browser snapshot --session lookup | grep "搜索内容"   # localizar el ref del cuadro
agent-browser fill @eNN "${TERM}" --session lookup           # sustituir @eNN por el ref real
agent-browser click @eNN2 --session lookup                   # botón "搜索"
sleep 2
agent-browser read --session lookup > "fichero_destino.md"
agent-browser close --session lookup
```

Notas prácticas:
- Usar siempre `--session <nombre>` propio, nunca la sesión `default` (punto anterior de esta misma sección).
- Un "Operation timed out" en `open` no es necesariamente un fallo — confirmar con `agent-browser eval "document.readyState" --session lookup` antes de descartar (regla ya fijada en el Experimento de viabilidad, más arriba).
- Fuentes cuya búsqueda requiere interacción con formulario (NCPSSD, Xihua) necesitan un `sleep` explícito antes de `read` — los resultados cargan por AJAX y un `read` prematuro devuelve un fichero vacío (incidencia documentada por primera vez en `260809-engineering-beipanjiang-46-ncpssd.md`).

### Recuperación de entregas faltantes de una obra serializada sin PDF de origen (añadido 2026-09-03)

Caso excepcional de uso puntual, no un flujo habitual: recuperar el texto completo de una entrega numerada concreta de una obra serializada ya catalogada en el proyecto (p. ej. `fuentes/260828_abc_reverte/`, la columna "Una historia de Europa" de Arturo Pérez-Reverte en *XL Semanal*), republicada íntegra en un sitio de terceros. Motivado por una tanda real de 39 entregas así, todas ellas ausentes de cualquier otro soporte propio del proyecto (ni escaneo físico ni copia previa). Previsible que se repita un caso similar a corto plazo, de ahí documentarlo aquí aunque el caso de origen sea puntual.

Fuente usada: republicación web íntegra de la columna en Zenda Libros (`zendalibros.com`, sección "El bar de Zenda"), que reproduce el texto completo sin necesidad de sesión ni autenticación.

```bash
# Patrón de URL con fallback verificado — no asumir cuál funciona sin probar
agent-browser open "https://www.zendalibros.com/perez-reverte-una-historia-de-europa-${numeral_romano_minuscula}/" --session lookup
# Si el título/contenido devuelve "Página no encontrada", reintentar sin el prefijo de autor:
agent-browser open "https://www.zendalibros.com/una-historia-de-europa-${numeral_romano_minuscula}/" --session lookup
sleep 2
agent-browser read --session lookup
agent-browser close --session lookup
```

Hallazgos operativos verificados en esta tanda, no documentados hasta ahora en este ADR:

- **El slug de URL no sigue un único patrón**: piezas antiguas (p. ej. la I, publicada en 2021) a veces requieren la forma sin prefijo de autor y otras la forma con prefijo, sin relación aparente con la antigüedad de la pieza — hay que probar ambas, nunca asumir por precedente de una pieza cercana. La última entrega de toda la serie sigue además un tercer patrón, con una "y" literal antes del numeral (`una-historia-de-europa-y-cxxvi`), reflejando la convención literaria española de cerrar una enumeración con "y último".
- **Artefacto de caché verificado**: en varias lecturas, `agent-browser read` devolvió el cuerpo de un artículo completamente ajeno (reconocible por tratar de la novela *Bajo el cielo infernal*, de Alicia Giménez Bartlett) en vez del artículo pedido, pese a que el título de la página sí era el correcto — condición de carrera de la propia página, no un fallo del comando. Se resuelve reintentando `read` (a veces hace falta un `sleep` algo mayor); nunca se debe aceptar como bueno un cuerpo cuyo tema no coincide con el título ya confirmado.
- **La fecha de republicación web no es la fecha real de portada impresa**: Zenda indica al pie de cada artículo "Publicado el [fecha] en XL Semanal", pero esa fecha es la de subida a la web, no la de la revista física (que en este caso se distribuye siempre en domingo). La fecha real de portada se obtiene redondeando la fecha de Zenda al domingo siguiente inclusive — regla verificada de forma independiente contra tres entregas cuya fecha real ya se conocía por el escaneado físico (coincidencia exacta en los tres casos, incluida una republicación cuya fecha de Zenda caía en sábado en vez del viernes habitual). Esta regla es específica de publicaciones con día de distribución fijo conocido; para otra fuente serializada habría que confirmar antes cuál es su cadencia real en vez de asumir el mismo desfase.
- **La fecha indicada por la propia fuente puede estar mal y hay que poder detectarlo**: en dos entregas de esta misma tanda, el pie de fecha de Zenda repetía, literal y erróneamente, la fecha de la entrega inmediatamente anterior (mismo texto, mismo numeral duplicado). Se detectó por no encajar con la cadencia regular ya consolidada de entregas vecinas del proyecto (contraste de consistencia interna, no conocimiento externo), y se documentó explícitamente la inferencia en el Markdown resultante en vez de aceptar la fecha literal o corregirla en silencio.

#### Segundo caso: "Una historia de España" (añadido 2026-09-11)

Pieza XI (única faltante de la serie, ya compilada en libro por Alfaguara en 2019): Zenda no la tenía — solo republica columnas aún no compiladas. Recuperada de `perezreverte.com`, que devuelve `403 Forbidden` a agent-browser (bloqueo de origen, confirmado con `curl`); resuelto con Jina Reader. Sin fecha explícita en origen: se dedujo cruzando el timestamp de una réplica en blog con el índice CDX de Wayback Machine sobre la sindicación original en finanzas.com (fecha codificada en la propia URL, `AAAAMMDD/slug.html`), confirmando por lectura directa del snapshot. Nota: Jina Reader bloquea `web.archive.org`; ese tramo requiere `curl` directo.

### Plantilla de prompt para consulta de una sola fuente

Al pedir una consulta puntual en lenguaje natural (no en terminal), "busca X" sin más contexto es indistinguible de "monta un experimento comparativo" (`ADR-1116`). Tres elementos hacen la petición inequívoca: nombrar la fuente exacta, decir explícitamente que no es comparativo, y decir qué hacer con el resultado.

> Busca "**‹término›**" en **‹zh.wikipedia.org / baike.baidu.com / www.douban.com / www.xihua.es›** vía agent-browser. No es un experimento comparativo — no lo valores con la rúbrica de `ADR-1116`, no generes ficha `00-comparativa-fuentes`, no toques la columna `Valoración` del índice. Guarda el resultado en `fuentes/‹carpeta›/` y **‹cataloga como KA en `conocimiento/`› / ‹solo guarda el fichero fuente, sin KA›**.

Ejemplo real:

> Busca "颐和园" en baike.baidu.com vía agent-browser. No es un experimento comparativo. Guarda en `fuentes/260810_palacios/` y cataloga como KA en `conocimiento/`.

---

## Consequences

### Positive

* Sustituye el seguimiento manual, propenso a lagunas, por un mecanismo repetible que además resuelve el caso — antes sin solución — de fuentes con sesión/autenticación (CNKI, Zhihu, Douban).
* El paso 4 (evidencia visual) da a la supervisión humana una referencia objetiva del estado real de la página en el momento de una discrepancia, sin depender solo de lo que el propio sistema describe en texto.
* Mantiene el proyecto sin coste recurrente de API (agent-browser y Jina Reader gratuitos para este volumen), coherente con la ausencia de presupuesto de infraestructura del proyecto.

### Negative

* Requiere instalación y mantenimiento local del binario de agent-browser y de Chrome (descargado por `agent-browser install` vía Chrome for Testing) — más fricción operativa que una API de un solo request (Jina Reader).
* `agent-browser snapshot` por sí solo no detecta cambios de contenido, solo estructurales — si el paso 2 del procedimiento operativo (diff de Markdown) se omite por error, una novedad real de contenido puede pasar inadvertida.

### Risks Mitigated

* Que la reutilización de sesión/autenticación de agent-browser se use, por error o por comodidad, para eludir el muro de pago de fuentes académicas — mitigado por la restricción explícita del punto 1 y por `OUT OF SCOPE`/punto 6.
* Que una herramienta descartada (MultiOn) se recomendara de nuevo sin comprobar su vigencia — mitigado documentando explícitamente en el punto 3 por qué se descartó (pivote de producto verificado en el momento de esta decisión, ver Anexo).

---

## References

* conocimiento_index/260806-index-documentation-fuentes-curadas.md (índice que este mecanismo mantiene actualizado; contiene el detalle de viabilidad verificado por fuente el 2026-08-09, resumido en el Experimento de este ADR)
* docs/decisiones/ADR-1108-Vocabulario_Especifico_por_Knowledge_Asset.md
* docs/decisiones/ADR-1111-Criterios_Interes_Curacion_Conocimiento.md
* docs/decisiones/ADR-1113-Generalizacion_Indices_de_Fuentes_Documentales.md
* docs/decisiones/ADR-1109-Distincion_Grafo_Obsidian_vs_Grafo_Understand_Anything.md (mismo criterio de "herramienta concreta va en ADR, no en Project_Definition")
* docs/decisiones/ADR-1114-Adopcion_Understand_Anything_Knowledge_Provider.md (mismo criterio, aplicado al Knowledge Provider)
* fuentes/260828_abc_reverte/ (2026-09-03) — 39 entregas de "Una historia de Europa" recuperadas con este procedimiento (piezas I, II, V-VIII, X, XII, XIV, XXXI, XXXIV, XXXIX, L, LIII, LXXXIII, LXXXVI y CIV-CXXVI), incluida la corrección por consistencia interna de dos fechas erróneas en la fuente (piezas LIII y CXXIII)
* fuentes/260910_abc_reverte/131006_HRR_historia_espana_011.md (2026-09-11) — pieza XI de "Una historia de España" (única faltante), recuperada vía Jina Reader tras bloqueo 403 de agent-browser en perezreverte.com (ver Segundo caso, más abajo)
* Anexo de este mismo documento (fuente de inspiración — la transcripción local que la documentaba se retiró del repositorio el 2026-08-08; la URL original del vídeo queda preservada)

---

## Anexo: Comparativa de herramientas y fuente de inspiración

*Fuente de inspiración: vídeo de YouTube https://www.youtube.com/shorts/V3fc_Gghjcw ("5 skills gratis de Claude Code que nadie conoce", canal Nexum AI), que nombra Agent Browser entre cinco skills poco conocidas de Claude Code.*

No es parte de la Decision en sí, sino el camino que condujo a ella:

| Herramienta | Pros | Contras | Contexto ideal |
| :--- | :--- | :--- | :--- |
| **agent-browser** (elegida) | Ultra-rápido (Rust). Determinista mediante "refs". Persistencia de sesión y estados de auth. Skill nativa para Claude Code. | Requiere instalación local/CLI y gestión del binario de Chrome. | Interacción dinámica, acceso a bases de datos con login, flujo diario con Claude Code. |
| **Jina Reader** (elegida, soporte) | La forma más rápida y sencilla de obtener Markdown de una URL vía API simple. | Sin capacidad de interacción, clics o gestión de sesión. | Lectura rápida de artículos de prensa o blogs sin interacción. |
| **Firecrawl** (descartada) | Excelente manejo de proxies y bypass de bloqueos. Convierte sitios enteros a Markdown/JSON de forma robusta. | Requiere suscripción/API de pago sin ventaja real sobre Jina Reader a este volumen de fuentes. | Se documenta por si el volumen de fuentes estáticas creciera lo suficiente para justificar el coste. |
| **MultiOn** (descartada) | Navegación basada en visión; "entiende" la web como un humano. | Verificado en el momento de esta decisión: pivotó hacia AGI-0, una app móvil de consumo — ya no encaja en automatización vía API/CLI. | — |
| **Skyvern** (alternativa futura, no activa) | Navegación basada en visión; ideal para flujos impredecibles. | Mayor latencia y coste por ejecución. Redundante con agent-browser para el uso actual. | Fuentes con interfaces hostiles, si agent-browser resultara insuficiente. |
| **Crawl4AI** (alternativa futura, no activa) | Framework de Python de código abierto, totalmente personalizable. | Requiere desarrollo de scripts específicos por fuente. Curva de aprendizaje superior. | Backend Python a medida, si llegara a necesitarse. |

---

## Glosario de comandos agent-browser (añadido 2026-08-10)

Recopilación de todos los subcomandos y flags de `agent-browser` citados en este ADR, en el orden en que aparecen documentados a lo largo del texto — no es una referencia exhaustiva del CLI (ver README oficial de la herramienta para el listado completo), sino un glosario de lo que este proyecto ya ha usado y verificado.

| Comando | Descripción | Flags / opciones relevantes | Sección de este ADR |
|---|---|---|---|
| `agent-browser open <url>` | Navega hacia una URL en la sesión indicada. El código de salida (incl. "✗ Operation timed out") nunca se toma como resultado final por sí solo — se confirma con `get url` y `screenshot`. | `--session <nombre>` (sesión aislada, evita colisión con `default`); `--headed` (navegador visible, uso manual interactivo) | Procedimiento operativo; Experimento de viabilidad; Comandos para consultas puntuales |
| `agent-browser snapshot` | Captura el árbol de accesibilidad de la página y lo compara con el guardado. Diff **estructural**, no de contenido — detecta que *algo* cambió, no qué cambió. También expone los refs (`@eN`) de los elementos interactivos, usados por `fill`/`click`. | — | Procedimiento operativo; Comandos para consultas puntuales |
| `agent-browser read [url]` | Extrae el contenido de la página (o de una URL directa) convertido a Markdown. | `--require-md` (exige extracción en Markdown válido); `--session <nombre>` | Procedimiento operativo; Comandos para consultas puntuales |
| `agent-browser screenshot` | Captura una imagen de la página actual, usada como evidencia visual cuando la validación de fuentes no logra confirmar un dato. | `--annotate` (superpone etiquetas numeradas sobre elementos interactivos, mapeadas a los mismos refs `@eN` del `snapshot`) | Procedimiento operativo; Experimento de viabilidad |
| `agent-browser get url` | Devuelve la URL real en la que se encuentra la sesión — confirma que un `open` navegó de verdad, no solo que devolvió éxito aparente. | — | Experimento de viabilidad |
| `agent-browser eval "<js>"` | Ejecuta una expresión JavaScript arbitraria en la página (p. ej. `document.readyState`) para verificar el estado de carga cuando `open`/`read` resultan ambiguos. | `--session <nombre>` | Experimento de viabilidad; Comandos para consultas puntuales |
| `agent-browser fill @eNN "<texto>"` | Rellena un campo de formulario identificado por su ref (`@eNN`, obtenido previamente con `snapshot`). | `--session <nombre>` | Comandos para consultas puntuales |
| `agent-browser click @eNN` | Hace clic en el elemento identificado por su ref. | `--session <nombre>` | Comandos para consultas puntuales |
| `agent-browser close` | Cierra sesiones activas. | `--session <nombre>` (cierra solo esa sesión); `--all` (cierra el daemon y **todas** las sesiones activas, propias y del usuario — usar con precaución, ver política de colisión de sesión) | Comandos para consultas puntuales; Política de advertencia por colisión de sesión compartida |
| `agent-browser session info` | Devuelve el estado de una sesión — usado para comprobar si `default` u otra sesión ya está `active` antes de operar sobre ella. | `--json` (salida estructurada) | Política de advertencia por colisión de sesión compartida |
| `agent-browser state save` / `agent-browser state load` | Guarda/restaura el estado de sesión (cookies, login) entre ejecuciones — imprescindible para fuentes con muro de login legítimo ya autorizado (CNKI, Enciclopedia de China, Instituto Confucio de Madrid/GISEC). | `--restore`, `--profile` (persistencia explícita; sin ellos el perfil de Chrome vive solo en memoria y se pierde al cerrar la sesión) | Decision (punto 1); Política de advertencia por colisión de sesión compartida |
| `agent-browser state clear --all` | Borra todos los estados de sesión guardados. | `--all` | Instalación, alcance y desinstalación |
| `agent-browser cookies clear` | Limpia las cookies almacenadas por el navegador gestionado. | — | Instalación, alcance y desinstalación |
| `agent-browser storage local clear` | Limpia el local storage del navegador gestionado. | — | Instalación, alcance y desinstalación |
| `agent-browser install` | Instala el binario y descarga Chrome for Testing (`~/.agent-browser/browsers/chrome-<versión>/`). | `--with-deps` (instala también bibliotecas compartidas del sistema; relevante en WSL si el navegador falla al lanzarse) | Instalación, alcance y desinstalación |
| `agent-browser doctor --fix` | Diagnóstico con reparaciones destructivas (reinstala Chrome, purga estado antiguo) — el comando más cercano a una limpieza completa sin desinstalar el binario. | `--fix` | Instalación, alcance y desinstalación |
| `agent-browser upgrade` | Actualiza el binario/Chrome a una versión más reciente. No verificado en este proyecto si sustituye o duplica la carpeta `chrome-<versión>/` anterior — revisar `~/.agent-browser/browsers/` tras cualquier uso futuro. | — | Instalación, alcance y desinstalación |
