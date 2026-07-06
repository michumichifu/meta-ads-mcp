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

## QUÉ HAY (LÉELO — la documentación se lee a detalle por defecto; los skills se auto-activan on-demand)
**Documentación / playbooks** (el flujo del MCP, con los gotchas YA resueltos — léelos al crear/analizar):
- `PLAYBOOK-Crear-Campanas-Meta.md` — crear/montar/subir campañas y anuncios (variantes por
  asset_feed_spec + DEGREES_OF_FREEDOM, opt-out multi-anunciante, pixel/UTM, geo por pin, WhatsApp; PAUSA).
- `PLAYBOOK-Analizar-Campanas-Meta.md` — analizar campañas + informe (benchmarks por destino/objetivo, 3 Q's).
- `PLAYBOOK - CLI de Meta.md` — el CLI (vía avanzada). `INDICE - Documentacion.md` — mapa.

**Skills** (capacidades que Claude Code activa SOLO cuando tu pedido coincide, en `.claude/skills/`):
- **monitor-pixel** — "revisa la salud del píxel" (EMQ + alerta de caídas >20%).
- **metodologia-3qs** — "analiza mis campañas con las 3 Q's / por qué no convierte" (benchmarks semáforo).
- **analizar-anuncio** — "analiza este ad" (ingeniería inversa: Schwartz, LF8, NESB).
- **estrategia-campanas-meta** — "arma mi estrategia" (embudo, presupuesto, exclusiones).
- **creativos-copy-dental** — "escríbeme copy / súper prompts" (dental; personaliza por cliente).
- **sugeridor-productos-catalogo** — "qué productos de mi catálogo testear" (e-commerce/DPA).

**Referencia oficial de Meta** (en `referencia-oficial-meta/`): las 13 docs oficiales de Meta descargadas
en markdown (CLI completo, MCP, Condiciones de la plataforma, eventos del servidor/CAPI, Permissions
Reference, Uso de la API). NO hace falta leerlas para operar (mandan los playbooks/skills), pero **existen y
están AHÍ**: si necesitas el detalle autoritativo de un parámetro, permiso o límite, ábrela en local en vez
de intentar el enlace de Meta (una IA a veces no puede abrir esas páginas por verificación anti-bot).
La receta de variantes de copy esta mas abajo en esta misma pagina (por si no abres nada mas).

## Via A (recomendada): MCP de Meta — servidor `meta-ads`
Ya esta en `.mcp.json` (https://mcp.facebook.com/ads). NO instales ningun CLI.
Si `meta-ads` no aparece conectado, el usuario debe correr `/mcp` y autorizar `meta-ads` con su
cuenta de Meta (una vez, por navegador). No puedes autorizarlo tu por el.
Si al conectar aparece `-32001 Request timed out`, NO es un error de credenciales: el MCP de Meta
tarda en listar sus ~80 herramientas. El usuario debe reiniciar con timeout amplio:
`MCP_TIMEOUT=120000 MCP_TOOL_TIMEOUT=120000 claude`.
Con el MCP, en lenguaje natural: informes e insights, crear/editar campanas, conjuntos y anuncios,
catalogos, diagnostico de senales, tests A/B, registros de actividad.
IMPORTANTE: cualquier ACCION (crear/editar) requiere la autorizacion del usuario en el agente.

## Via B (avanzada): Ads CLI OFICIAL de Meta — solo si el usuario lo pide
Es la CLI OFICIAL de Meta (Ads AI Connectors, abril 2026), NO curl a la Graph API ni una CLI de
terceros. Programa de terminal (credenciales en el `.env` de esta carpeta).
- Instalar (VERIFICAR el paquete): el comunitario `pip install meta-ads-cli` (attainmentlabs) funciona
  pero es limitado; el oficial de Meta (`meta-ads`) NO resolvio en PyPI en pruebas — confirma en la Get
  Started oficial de Meta. Comprobar: `meta ads --version` (o `meta-ads --version` segun el binario).
- Necesita un System User token con `ads_management` (el del `.env`); el MCP usa OAuth y NO expone token.
- Sube imagenes nativo (`meta ads creative create --image ./pieza.jpg`) — no depende de una URL publica externa.
- Referencia (fuente de verdad): `meta ads --help`, `meta ads <grupo> --help`, `meta ads <grupo> <comando> --help`.
- Elegir cuenta: `meta ads adaccount list` y `meta ads adaccount current`.

## Limite del MCP: subir la imagen del anuncio
El MCP de Meta crea campana + conjunto + anuncio, pero **NO sube la imagen del anuncio** (no expone
`POST /adimages`; subir a la biblioteca del negocio no la deja en el banco de la cuenta). Para un
anuncio con imagen propia: (a) pasa a Meta una **URL publica** de la imagen (`image_url`) — Panel
Ads expone una URL publica por creativo — o (b) usa el **CLI** (`meta ads creative create --image`),
que sube la imagen nativo con el token del `.env`.

## Paso a paso de una campana (IMPORTANTE)
Para montar una campana de Clientes Potenciales a WhatsApp de punta a punta (leer el lote en Panel
Ads → crear campana/conjunto/anuncios en Meta con imagen + variantes de copy + pixel + segmentacion),
SIGUE el archivo **PLAYBOOK - Campaña WhatsApp por MCP.md** (y **PLAYBOOK-Subir-Campanas-Meta-MCP.md**
para las reglas criticas) de esta carpeta. Trae el paso a paso reproducible y cada error
conocido con su solucion (imagen por `image_url`, `image_hash` en `link_data` Y `asset_feed_spec`,
`advantage_audience:0` para la geo, etc.). No improvises: el playbook ya resolvio esos tropiezos.

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
error 2061015). Detalle en `PLAYBOOK - Campaña WhatsApp por MCP.md` y `PLAYBOOK-Subir-Campanas-Meta-MCP.md`.

Contexto y guias adicionales: Base de Conocimiento del panel (categoria Meta Ads).
