# Meta Ads (para el agente) — MCP + CLI

Puedes gestionar Meta Ads de DOS formas. PREFIERE el MCP (no instala nada). NO instales el CLI salvo
que el usuario lo pida.

## Principio rector — el criterio humano por encima de la IA (LÉELO PRIMERO)
Los fundamentos de por que una campana vende NO cambian con la tecnologia: detras de cada compra hay una
persona, y por que elige una empresa y no otra es un patron humano que la IA y Meta NO descifran al 100%.
Este MCP —y cualquier automatizacion de Meta— es una PALANCA que amplifica el criterio del operador, no
que lo reemplaza. Como agente, compórtate asi:
- Presenta datos, hipotesis y benchmarks, pero deja la LECTURA final (por que la gente respondio o no) al
  humano. No vendas la salida de la IA ni la optimizacion de Meta como verdad absoluta.
- Invita al operador a aportar su lectura del comportamiento (del producto, del anuncio, del lead): ahi
  esta la ventaja que la optimizacion automatica no ve. Si detectas una señal, plantéala como hipotesis a
  confirmar con su criterio, no como conclusion cerrada.
- Distingue principios (atemporales) de "hacks" recien salidos de Meta: se pueden probar, pero no se
  persiguen a ciegas sin entender como funciona una campana. Los resultados consistentes salen de los
  fundamentos + la tecnologia, nunca del truco de moda solo.

## La ubicacion del negocio: con el PIN del usuario, NUNCA con el nombre de la ciudad
Este kit esta pensado sobre todo para negocios LOCALES (atienden en una ciudad o en un radio), aunque sirve
igual para un pais entero o e-commerce. El MCP NO tiene busqueda de lugares, asi que:
- Antes de crear el PRIMER conjunto de una campana local, mira si existe `MI-NEGOCIO.md` en esta carpeta.
  Si existe, usa sus coordenadas y su radio sin volver a preguntar.
- Si NO existe, pidele al usuario las coordenadas del pin, explicandole como sacarlas (paso 3 de
  "Primeros pasos" del `README.md`: Administrador de anuncios → conjunto → Ubicaciones → "Colocar pin" →
  clic en el mapa → copiar el par de numeros; o clic derecho en Google Maps). Y el radio en km (minimo 4).
- NUNCA deduzcas coordenadas de un nombre de ciudad ni de tu memoria: hay ciudades homonimas y el anuncio
  saldria en otro sitio, pagado, sin que nadie lo note.
- Al recibirlas, comprueba que el primer numero es la latitud (-90 a 90) y el segundo la longitud
  (-180 a 180), y guardalas en `MI-NEGOCIO.md`: nombre del negocio, latitud, longitud, radio en km y la
  fecha. Varias sedes = una linea por sede. Desde ahi, se usan en `custom_locations` (receta exacta en la
  seccion "SEGMENTACION GEOGRAFICA" de `PLAYBOOK-Crear-Campanas-Meta.md`).
- Si vende a todo un pais o es tienda online, no hace falta pin: `geo_locations.countries`.

## QUÉ HAY (LÉELO — la documentación se lee a detalle por defecto; los skills se auto-activan on-demand)
**Documentación / playbooks** (el flujo del MCP, con los gotchas YA resueltos — léelos al crear/analizar):
- `PLAYBOOK-Crear-Campanas-Meta.md` — crear/montar/subir campañas y anuncios (variantes por
  asset_feed_spec + DEGREES_OF_FREEDOM, opt-out multi-anunciante, pixel/UTM, geo por pin, WhatsApp; PAUSA).
  **Su § 0 corrige lo que cambió desde julio 2026: leelo primero.**
- `PLAYBOOK-Analizar-Campanas-Meta.md` — analizar, leer y escalar campañas en marcha (metricas que enganan,
  aprendizaje, gasto semanal, conversiones de WhatsApp por API de conversiones, Biblioteca) + informe.
- `PLAYBOOK - CLI de Meta.md` — el CLI y la Graph API. `INDICE - Documentacion.md` — mapa.
- `GUIA - Crear la app de Meta y pasar el App Review (acceso completo al CLI).md` — la app propia.
- `REFERENCIA - Herramientas del MCP de Meta.md` — las ~100 herramientas y lo que el MCP aun no hace.
- `REFERENCIA - Condiciones de Meta del 20 nov 2026.md` — desde esa fecha, el anunciante es "el unico
  responsable" de lo que haga cualquier software de terceros (este agente incluido) y de revisar lo que
  generen las funciones automaticas de Meta. Por eso: todo en PAUSA y nada se activa sin el humano.

**Skills** (capacidades que Claude Code activa SOLO cuando tu pedido coincide, en `.claude/skills/`):
- **monitor-pixel** — "revisa la salud del píxel" (EMQ + alerta de caídas >20%).
- **metodologia-3qs** — "analiza mis campañas con las 3 Q's / por qué no convierte" (benchmarks semáforo).
- **analizar-anuncio** — "analiza este ad" (ingeniería inversa: Schwartz, LF8, NESB).
- **estrategia-campanas-meta** — "arma mi estrategia" (embudo, presupuesto, exclusiones).
- **creativos-copy-dental** — "escríbeme copy / súper prompts" (dental; personaliza por cliente).
- **sugeridor-productos-catalogo** — "qué productos de mi catálogo testear" (e-commerce/DPA).

**Referencia oficial de Meta** (en `referencia-oficial-meta/`): las 13 docs oficiales de Meta descargadas
en markdown (CLI completo, MCP, Condiciones de la plataforma, eventos del servidor/CAPI, Permissions
Reference, Uso de la API) y, en `en-2026-11/`, las 18 paginas actuales de Ads AI Connectors en ingles, tal
cual (el CLI oficial y las 10 del servidor MCP de anuncios). Si chocan, manda la de `en-2026-11/`. NO hace falta leerlas para operar (mandan los playbooks/skills), pero **existen y
están AHÍ**: si necesitas el detalle autoritativo de un parámetro, permiso o límite, ábrela en local en vez
de intentar el enlace de Meta (una IA a veces no puede abrir esas páginas por verificación anti-bot).
La receta de variantes de copy esta mas abajo en esta misma pagina (por si no abres nada mas).

## Via A (recomendada): MCP de Meta — servidor `meta-ads`
Ya esta en `.mcp.json` (https://mcp.facebook.com/ads). NO instales ningun CLI.
Si `meta-ads` no aparece conectado, el usuario debe correr `/mcp` y autorizar `meta-ads` con su
cuenta de Meta (una vez, por navegador). No puedes autorizarlo tu por el.
Si al conectar aparece `-32001 Request timed out`, NO es un error de credenciales: el MCP de Meta
tarda en listar sus ~100 herramientas. El timeout amplio ya va en `.claude/settings.json`; si aun asi
falla, que el usuario cierre y vuelva a abrir la carpeta.
Con el MCP, en lenguaje natural: informes e insights, crear/editar campanas, conjuntos y anuncios,
subir imagenes y videos, catalogos, publicos personalizados, eventos del pixel, diagnostico de senales,
tests A/B y de conversion lift, benchmarks del sector, registros de actividad, Biblioteca de anuncios
y la ayuda oficial de Meta. Inventario completo, por grupo: `REFERENCIA - Herramientas del MCP de Meta.md`.
IMPORTANTE: cualquier ACCION (crear/editar) requiere la autorizacion del usuario en el agente.

## Via B (avanzada): Ads CLI OFICIAL de Meta — solo si el usuario lo pide
Es la CLI OFICIAL de Meta (Ads AI Connectors, abril 2026), NO curl a la Graph API ni una CLI de
terceros. Programa de terminal (credenciales en el `.env` de esta carpeta).
- Instalar: el oficial de Meta ya esta en PyPI y Homebrew: `pip install meta-ads` (Python 3.12+) o
  `brew install facebook/fb/meta-ads`. Comprobar: `meta --version`. (El comunitario `meta-ads-cli` de
  attainmentlabs tambien funciona, con comandos distintos.)
- Necesita una app propia de Facebook Developer aprobada en el App Review (y publicarla exige el
  portafolio dueno de la app verificado) + un token de usuario del sistema con `ads_management`. Paso a
  paso: `GUIA - Crear la app de Meta y pasar el App Review (acceso completo al CLI).md`.
  El MCP usa OAuth y NO expone token.
- Sube imagenes nativo (`meta ads creative create --image ./pieza.jpg`) — no depende de una URL publica externa.
- Referencia (fuente de verdad): `meta ads --help`, `meta ads <grupo> --help`, `meta ads <grupo> <comando> --help`.
- Elegir cuenta: `meta ads adaccount list` y `meta ads adaccount current`.

## Subir la imagen o el video del anuncio (CAMBIO de nov 2026: el MCP YA sube)
Hasta mediados de 2026 el MCP no subia imagenes. **Hoy si**: `ads_creative_upload_media` sube
imagenes y videos al banco de la CUENTA publicitaria, de dos formas:
- `upload_source:"URL"` + `media_type:"IMAGE"|"VIDEO"` + `media_url` (URL publica y directa) → devuelve
  el `image_hash` (o el id del video). Enlaces de Drive, Dropbox o Canva que piden sesion o devuelven
  HTML NO sirven.
- `upload_source:"LOCAL_FILE"` → abre una mini-app para que el usuario elija el archivo de su equipo
  (solo en clientes que soportan "MCP Apps"; si el tuyo no, usa URL).
Un video queda en `processing`: NO lo uses en un creativo hasta que `ads_get_ad_videos` diga `ready`.
Sigue valiendo pasar `image_url` (URL publica) directo en el creativo, y el CLI sube nativo. Ojo: la
"Biblioteca multimedia" del PORTAFOLIO no es el banco de imagenes de la CUENTA (`image_hash` no la ve).
(Este cambio sale de la descripcion de la herramienta en vivo; la doc oficial aun no lo lista.)

## Paso a paso de una campana (IMPORTANTE)
Para montar una campana de punta a punta (campana → conjunto → anuncios con imagen + variantes de
copy + pixel + segmentacion), SIGUE **`PLAYBOOK-Crear-Campanas-Meta.md`**. Trae el orden correcto y
cada error conocido con su solucion (`image_hash` en `link_data` Y `asset_feed_spec`,
`advantage_audience:0` para la geo, el evento antes de los anuncios, etc.). No improvises: el playbook
ya resolvio esos tropiezos.

## Regla de oro
Todo se crea EN PAUSED. NUNCA actives (`--status ACTIVE`) sin confirmacion explicita del usuario.

## Variantes de copy en UN mismo anuncio — RECETA QUE FUNCIONA (usala, NO la re-descubras)
Meta SI admite varios textos por anuncio, pero SOLO en formato flexible ("opciones de texto"), NO con
un solo `message`. Si armas el creativo con `message`, Meta deja 1 copy por anuncio. Receta EXACTA:
en `ads_create_ad`, en `creative` pasa object_story_spec MAS asset_feed_spec:
- object_story_spec.link_data: `link` (wa.me/<NUMERO_WHATSAPP>) + `image_hash` + call_to_action WHATSAPP_MESSAGE.
- asset_feed_spec: `images:[{hash:...}]`, `bodies:[...N textos]`, `titles:[...]`, `descriptions:[...]`,
  `ad_formats:["SINGLE_IMAGE"]`, `optimization_type:"DEGREES_OF_FREEDOM"`.
LAS 3 CLAVES que rompen esto si faltan: (1) `optimization_type:"DEGREES_OF_FREEDOM"` (sin esto Meta no
lo trata como "opciones de texto"); (2) `image_hash` en link_data Y en asset_feed_spec.images (si va
solo en asset_feed_spec, el anuncio sale SIN imagen); (3) `link` (wa.me) dentro de link_data (si falta,
error 2061015). Detalle en `PLAYBOOK-Crear-Campanas-Meta.md`.
