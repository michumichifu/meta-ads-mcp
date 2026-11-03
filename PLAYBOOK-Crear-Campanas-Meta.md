# Crear campañas en Meta por MCP (reglas críticas + receta WhatsApp)

> Todos los valores de este playbook (IDs de cuenta/página/píxel, números de teléfono, nombres como "Clínica Dental Aurora" o "Dra. Aurora Peña") son **ejemplos ficticios**. Reemplázalos por los de tu propia cuenta.

Cómo crear campañas correctas por el MCP de Meta, evitando los errores del primer intento.
Aplica a cualquier cliente. Todo se crea **en PAUSA** para revisión; nada se activa sin confirmación.

> **Alcance:** este playbook cubre **crear la campaña en Meta** (campaña, conjuntos, anuncios,
> segmentación, píxel) a partir de creativos ya listos (imagen + copy). **Generar los creativos** —
> las imágenes y el copy — es un paso PREVIO y aparte, con tu propia herramienta o generador de
> creativos; no es parte de este flujo. Aquí ya llegas con la imagen y los textos en mano.

Todas las tools de Meta son `mcp__meta-ads__ads_*`. Cargarlas con ToolSearch antes de usarlas.
Si `tools/list` da timeout, el timeout amplio ya va en `.claude/settings.json`: cierra y vuelve a abrir la carpeta.

## 0. LO QUE CAMBIÓ DESDE JULIO DE 2026 (léelo antes que el resto)

Las secciones de abajo se escribieron en julio. Esto las corrige donde chocan:

1. **El MCP ya sube imágenes y videos**: `ads_creative_upload_media` (por URL pública o archivo local).
   Su despliegue es gradual: si responde *"This tool is new and is being gradually rolled out"*, usa el
   rodeo de siempre (`ads_create_creative` con `image_url` y luego leer el hash) o el CLI.
2. 🔴 **Por MCP, el creativo de un anuncio YA NO se edita.** `ads_update_entity(entity_type:"ad",
   fields:{creative:{...}})` responde *"Ad creatives are immutable and cannot be edited in place"*. Es una
   regla del MCP, no de Meta: desde el Administrador y por la Graph API sí se puede. Consecuencias:
   - El "método alterno (crear simple → editar)" y el arreglo de multi-anunciante "editando con el
     creativo inline" **ya no funcionan por MCP**. Crea el anuncio **directo** con la receta
     `DEGREES_OF_FREEDOM` y con todo puesto a la primera.
   - **UTM (`url_tags`), botón (CTA), imagen y destino se deciden ANTES de crear.** Rehacer diez
     anuncios cuesta una sesión entera.
   - `tracking_specs` tampoco se pone después por MCP: va en el `ads_create_ad`.
3. 🔴 **No leas el `image_hash` mientras el creativo está `IN_PROCESS`.** Se midió que Meta devuelve, sin
   error, el hash de **otra imagen de la cuenta**. Espera a `ACTIVE`, comprueba que todos los hashes del
   lote son distintos y, si puedes, compara la imagen (un hash perceptual tipo dHash: distancia 0-1 = la
   misma; 10 o más = otra). Y no pongas `image_url` dentro de `ads_create_ad`: asocia mal el hash para
   siempre. Sube con `ads_create_creative` (o `ads_creative_upload_media`).
4. **`ads_create_creative` exige `link_url`** aunque el esquema no lo marque obligatorio (*"link_url is
   required for image ads"*). Meta añade al nombre un sufijo con fecha y hash. La `image_url` firmada que
   devuelve Meta **caduca** (403): no la guardes como dato estable.
5. **Mejoras automáticas (Advantage+ creative):** por Graph v25/v26, `standard_enhancements: OPT_OUT`
   da el error **3858504**. Hay que apagarlas **una a una** dentro de `creative_features_spec`:
   `image_touchups`, `text_optimizations`, `image_brightness_and_contrast`, `enhance_cta`,
   `image_templates`, `add_text_overlay`, `description_automation`, `adapt_to_placement`. `music` (es de
   video) tumba la llamada en un anuncio de imagen. Después de crear, **relee `creative_features_spec`**:
   haber mandado `OPT_OUT` no prueba que quedara apagado; lo que importa es que **ninguna** quede en
   `OPT_IN` (Meta devuelve decenas en `OPT_OUT`, no hay que contarlas). Por MCP, `standard_enhancements`
   todavía se aceptaba en septiembre de 2026.
6. **Un conjunto admite 50 anuncios como máximo, contando los pausados** (subcódigo 1487809, "Demasiados
   anuncios"). Los borrados liberan cupo al instante: para rehacer anuncios, primero borra los viejos.
7. **`"creative_summary": "Dynamic creative"` en la respuesta del MCP no significa que el conjunto sea
   dinámico.** La prueba real es que el segundo anuncio entre sin el error 1885553. El `spec` que devuelve
   el MCP es el eco de lo que mandaste, no una lectura de lo guardado.
8. **El `link` de destino sale de UNA sola fuente por lote** (tu hoja, tu brief), nunca se deduce ni se
   copia de un anuncio hermano. Antes de publicar, haz una tabla anuncio por anuncio con la URL real en
   Meta frente a la esperada.
9. **`start_time` no se edita** si ya quedó en el pasado, aunque el conjunto esté en pausa y sin gasto
   (error 100, subcódigo **1487057**). Si la campaña tiene que arrancar a una hora, se pone **al crear el
   conjunto**; si no, hay que rehacer el conjunto (la campaña se reaprovecha).
10. **Geografía:** `custom_locations` va **dentro** de `geo_locations` (si va fuera: 2446385, "A location
    is missing"). Meta añade `frequently_in` por su cuenta. Un conjunto **nuevo** con `location_types` sin
    `recent` pasa el POST pero **no se publica** ("segmentación por lugar suprimida", #1870194). Si dudas,
    quita `location_types` y deja que Meta ponga `["frequently_in","home","recent"]`.
11. **Edad con Público Advantage+:** con `advantage_audience: 1` la edad es solo una sugerencia (vuelve
    como `age_min_suggestion`). Para que el mínimo sea un suelo duro:
    `targeting_automation.individual_setting: {age: 1}`. Se puede cambiar aunque ya haya anuncios, sin
    mandarlos a revisión. Al auditar se mira `age_range`, no `age_min`. 🔴 Una regla de edad puesta en la
    cuenta **no corrige** lo que llega por API: si mandas `age_min: 18`, sale con 18.
12. **Emplazamientos obsoletos que tumban el conjunto entero:** `video_feeds` de Facebook (2490562) y
    `explore` de Instagram (2490589).
13. **Interacción → Sitio web exige `promoted_object` (píxel + evento) aunque optimices por clics.** Sin
    él, cualquier objetivo de rendimiento da 100/2490408. Con evento `CONTACT` pasan `LINK_CLICKS`,
    `LANDING_PAGE_VIEWS`, `REACH` e `IMPRESSIONS`; con `LEAD`, 2446814. Meta acepta un enlace `wa.me` como
    "Sitio web". Regla de fondo: **si la interfaz ofrece una combinación, a tu llamada le falta un campo.**
14. **Clic a WhatsApp que optimiza por clientes potenciales reales:** `OUTCOME_LEADS` +
    `destination_type: WHATSAPP` + `optimization_goal: LEAD_GENERATION` + `promoted_object {page_id,
    whatsapp_phone_number}`, sin píxel, atribución de **1 día tras el clic** (con 7: 1885423). Se alimenta
    con eventos `LeadSubmitted` por la API de conversiones. El paso a paso, en el § 1.7 de
    `PLAYBOOK-Analizar-Campanas-Meta.md`. Si el número no está vinculado a la página: 1487246.
15. **Duplicar un conjunto puede encoger la atribución** de 7 días de clic a 1. Revisa `attribution_spec`.
16. **Para publicar, la cuenta pide un teléfono verificado** (Configuración de la cuenta publicitaria →
    Información de contacto). El enlace del aviso de Meta lleva a otra pantalla donde esa opción no está.
17. **`url_tags` admite macros**: `{{site_source_name}}` (fb/ig/an/msg), `{{placement}}`,
    `{{campaign.id}}`. Ponlas al crear si quieres saber de qué emplazamiento llega cada visita.

## Reglas OBLIGATORIAS (no romper — aprendidas en uso real)

1. **DESACTIVAR "anuncio multi-anunciante" (multi-advertiser ads).** A veces es contraproducente.
   Se hace en el objeto `creative` del anuncio:
   `"contextual_multi_ads": {"enroll_status": "OPT_OUT"}`
   (además, opt-out de mejoras automáticas en `degrees_of_freedom_spec.creative_features_spec`: por MCP
   se aceptaba `standard_enhancements`; por Graph v25+ hay que apagarlas una a una, ver § 0 punto 5).

2. **SIEMPRE cargar las VARIANTES de copy** (opciones de texto), no una sola. Es CLAVE: ahorra la
   hora de copiar/pegar manual. Ten por creativo sus 3 primary texts, 5 headlines y 5 descriptions.
   Hay que subir TODAS.

3. **Destino web/landing → SIEMPRE activar eventos del sitio web (píxel) + parámetros UTM.**
   - Píxel: en el ADSET, `optimization_goal: "OFFSITE_CONVERSIONS"` +
     `promoted_object: {"pixel_id":"1122334455667788","custom_event_type":"LEAD"}`.
     (OJO: con `LANDING_PAGE_VIEWS` el píxel da error "combinación no válida"; usar OFFSITE_CONVERSIONS.)
   - UTM: en el `creative` del anuncio, `"url_tags":"utm_source=facebook&utm_medium=paid_social&utm_campaign=07-julio-brackets&utm_content=ad01-img01"`.
     El `utm_content` sale del identificador que asignes por creativo (ver convención de nombres).

4. **Preferencia de destino: WhatsApp (más simple).** Salvo que se pida web. WhatsApp no necesita
   píxel ni UTM.

5. **Prefijo 🤖 recomendado en los nombres** (campaña, conjunto y anuncios) para distinguir lo
   hecho por IA (Claude Code / MCP / CLI) de lo hecho por humanos. Ej.: `🤖 07.2 Julio - Clientes
   Potenciales a Web - Prótesis - MM/DD/YYYY`, `🤖 01 TF/SI - ...`, `🤖 01 Imagen 1 - estético`.

6. **CTA por defecto en web = `LEARN_MORE` ("Más información")**, el botón estándar de Meta. NO usar
   `BOOK_NOW` (en la GUI del cliente sale como "desconocido"/raro). Va en `link_data.call_to_action.type`
   Y en `asset_feed_spec.call_to_action_types`.

### ⚠️ ORDEN QUE SÍ FUNCIONA para crear el conjunto WEB (verificado 07/2026 — evita horas de vueltas)
El `create` de un ad_set con `OFFSITE_CONVERSIONS` directo, o el `update` de targeting a
`custom_locations`, fallan con `INTERNAL`. Y el MCP crea los conjuntos **"publicados en pausa"** (no
borrador): una vez que el conjunto se consolida (tiene anuncios), Meta **bloquea** cambiar
píxel/evento/optimización (subcode 3260011). Por eso el orden es:
1. `ads_create_ad_set` con **`optimization_goal: LANDING_PAGE_VIEWS`** y **el targeting definitivo YA en
   el create**: `custom_locations` (pin lat/long + radio) + `location_types:["home","recent"]` +
   `targeting_automation.advantage_audience:0`. Meterlo aquí evita el **#1870194** (Advantage+ audience
   re-inyecta al publicar el tipo de lugar descontinuado).
2. `ads_update_entity` del ad_set a **`OFFSITE_CONVERSIONS` + `promoted_object`** con el evento. El
   update SÍ funciona (a diferencia del create) y no pisa el targeting.
   - Evento **personalizado** (no estándar) por API: `{"pixel_id":"...","custom_event_type":"OTHER",
     "custom_event_str":"Registro_Exitoso"}`. El MCP no permite crear conversiones personalizadas.
3. **RECIÉN AHORA** crear los N anuncios (antes de esto el evento aún se puede cambiar; después no).
Si `custom_locations` hay que corregirlo luego y da `INTERNAL`, NO se arregla por update: se crea un
ad_set nuevo con el pin correcto en el create y se re-montan los anuncios.

## Concepto clave que hay que entender antes de empezar (imágenes)

- **Desde noviembre de 2026 el MCP SÍ sube imágenes y videos** con `ads_creative_upload_media`
  (`upload_source:"URL"` + `media_type` + `media_url`, o `"LOCAL_FILE"`), que devuelve el `image_hash`.
  En julio no existía (`ads_get_ad_images` / `ads_get_ad_videos` decían "upload not yet supported") y en
  algunas cuentas sigue en despliegue gradual. Para poner una imagen en un anuncio se usa:
  - `image_hash` → una imagen que **ya esté en el banco de imágenes de la CUENTA publicitaria**
    (`act_1234567890123456/adimages`), **o**
  - `image_url` → una **URL pública** que Meta descarga sola (y ahí Meta genera su propio `image_hash`).
- **La biblioteca de medios del portafolio/negocio ≠ el banco de imágenes de la cuenta.** Subir la
  imagen a la "Biblioteca multimedia" del portafolio NO la hace aparecer en `ads_get_ad_images` de la
  cuenta. No sirve para `image_hash`.
- **La solución que funciona:** ten cada imagen del creativo en **una URL pública** (hospédala donde
  sea; Meta la descarga sola por `image_url`). Se le pasa esa URL a Meta como `image_url` y listo —
  flujo 100% MCP, sin subir nada a mano.

## Activos / requisitos (rellenar una vez por cliente)

| Dato | Cómo obtenerlo por MCP |
|---|---|
| Cuenta publicitaria Meta (sin `act_`) | `ads_get_ad_accounts` → usar la que tenga `is_ads_mcp_enabled:true` **y** `is_queryable:true` |
| Negocio/portafolio dueño de la cuenta | — |
| Moneda de la cuenta | Revisar en la cuenta (¡ojo! puede ser **USD** aunque los precios del cliente sean DOP; mínimo diario según moneda — en **USD son 100 centavos = US$1/día**) |
| Página de Facebook | `ads_get_ad_account_pages` (ad_account_id) |
| Número WhatsApp Business (conectado a la página) | El de la página → link `https://wa.me/18095551234` |
| Pixel / dataset | `ads_get_datasets` (ad_account_id) |

## FASE A — Tener listos los creativos (imágenes + copy con variantes)

Antes de tocar Meta, ten tus creativos ya producidos: por cada uno, la **imagen** y su **copy** con
variantes (primary texts, headlines, descriptions y el mensaje de WhatsApp).

1. **Sube cada imagen a una URL pública.** Hospédala donde quieras (tu propio servidor, un bucket,
   etc.); lo único que importa es que la URL responda `200 image/jpeg` **sin autenticación**, porque es
   la que Meta descargará vía `image_url`. Evita rutas que exijan token (dan `401`) o que devuelvan HTML.
2. **Ten el copy a mano por creativo:** los **primary texts** (con sus variantes), los **headlines**,
   las **descriptions** y el **mensaje de WhatsApp**. Los usarás en el `asset_feed_spec`.
3. Si tienes una fuente propia de contexto de marca (colores, ofertas, doctores), úsala para afinar
   copys y validar; si no, sáltate esta parte.

**Verificar las URLs públicas antes de usarlas** (evita el 404 intermitente, ver catálogo de errores):
```bash
curl -sS -o /dev/null -w "HTTP %{http_code} | %{content_type} | %{size_download}b\n" "https://cdn.miagencia.do/aurora/brackets-01.jpg"
# Esperar: HTTP 200 | image/jpeg | >0 bytes.  Probar también con UA de Meta:
curl -sS -A "facebookexternalhit/1.1" -o /dev/null -w "%{http_code} %{content_type}\n" "https://cdn.miagencia.do/aurora/brackets-01.jpg"
```

## VARIANTES de copy: el `asset_feed_spec` (lo que más falla)

Esta es la pieza central y la que más errores dio. Léela una vez y aplícala en WhatsApp o en web.

- `ads_create_creative` (creativo suelto) **NO** soporta variantes de texto, ni `url_tags`, ni
  opt-out de multi-anunciante. Sirve solo para 1 texto. **No usarlo para esto.**
- Las variantes van en un **`asset_feed_spec`** dentro del parámetro `creative` de **`ads_create_ad`**.
- Requiere el **`image_hash`** (no la URL). Como al crear un creative por `image_url` Meta ya generó el
  hash, se puede **leer el hash**:
  `ads_get_creatives { ad_account_id, creative_ids:[...], fields:["id","name","image_hash"] }`
  (En `ads_get_ad_images` aparecen como "untitled_..." sin distinción; no sirve para mapear.)

### ⭐ La clave: `optimization_type: "DEGREES_OF_FREEDOM"` + imagen en los DOS lados

**Por qué fallaba antes:** con `asset_feed_spec` "a la brava" (imágenes solo en `asset_feed_spec`, sin
`link_data`), Meta lo trata como **Creativo Dinámico** → marca el conjunto `is_dynamic_creative:true` →
**máx 1 anuncio** (errores 1885553 al crear el 2º, 1885274/1885392/INTERNAL al editar). El error era
usar `optimization_type` mal (REGULAR / omitido) y NO poner la imagen en `link_data`.

**La receta que lo trata como "opciones de texto" ESTÁNDAR (NO dinámico) y deja N anuncios/conjunto:**
en `ads_create_ad`, `creative` = objeto con **object_story_spec.link_data COMPLETO** + **asset_feed_spec**:
```json
{
  "object_story_spec": {
    "page_id": "100200300400500",
    "link_data": {
      "link": "https://wa.me/18095551234",          
      "image_hash": "a1b2c3d4e5f60718293a4b5c6d7e8f90",                     
      "message": "Sonríe con confianza. Evaluación de ortodoncia sin costo esta semana.",
      "call_to_action": { "type": "WHATSAPP_MESSAGE", "value": { "app_destination": "WHATSAPP" } }
    }
  },
  "asset_feed_spec": {
    "images": [ { "hash": "a1b2c3d4e5f60718293a4b5c6d7e8f90" } ],  
    "bodies":  [ {"text":"v1"}, {"text":"v2"}, {"text":"v3"} ],   
    "titles":  [ {"text":"..."}, "... hasta 5" ],
    "descriptions": [ {"text":"..."}, "... hasta 5" ],
    "ad_formats": ["SINGLE_IMAGE"],
    "optimization_type": "DEGREES_OF_FREEDOM"    
  },
  "contextual_multi_ads": { "enroll_status": "OPT_OUT" }
}
```

**Los tres puntos que lo hacen funcionar (no negociables):**
1. `optimization_type: "DEGREES_OF_FREEDOM"` — ⭐ LA CLAVE (no `REGULAR`, no omitido).
2. El `link` es OBLIGATORIO dentro de `object_story_spec.link_data`. Si falta, falla con *"El campo
   link es obligatorio"* (subcode **2061015**). El `link` de WhatsApp va ahí.
3. El `image_hash` va en **ambos** lados: `link_data` **y** `asset_feed_spec.images`. Si va solo en
   `asset_feed_spec`, **la imagen no se renderiza** (el texto rota pero el anuncio sale sin imagen).

Con esto, crear los N anuncios DIRECTO así (no hace falta crear simple y editar). El conjunto queda
`is_dynamic_creative:false` y acepta los 10 anuncios, cada uno con sus variantes. VERIFICADO end-to-end
07/05/2026 (Ventas→WhatsApp): adset único, 10 anuncios `01 Imagen 1…10`, cada uno 3 bodies + 5 titles +
5 descriptions, multi-anunciante OFF.

Verificar con `ads_get_ad_preview { ad_id, ad_format:"MOBILE_FEED_STANDARD" }` que se vea imagen +
texto + botón de WhatsApp. (El preview a veces da timeout — reintentar; es lentitud del servidor de
Meta, no error.)

### ESTRUCTURA CORRECTA (criterio de operación que funcionó, no regla de Meta)
- **UN solo conjunto por servicio/estrategia**, y **dentro, TODOS los anuncios** (uno por imagen/ángulo),
  y **dentro de cada anuncio, sus variantes de copy** (3 textos principales, 5 títulos, 5 descripciones).
  Varios conjuntos SOLO si se quiere testear anuncios con **presupuesto separado** (no es lo normal).
- **Subir las imágenes Y todas las variantes de texto es TRABAJO DEL MODELO por MCP, no del usuario.**
  Ese es el propósito del flujo: evitarle el copy/pega manual entre pestañas. No se delega al usuario.
- **"Rechazo" de anuncio:** en pausa NADA se rechaza por política; los errores eran de **validación de
  la API** al armar la estructura, no revisión de Meta. La imagen es **estática y completa** (sin recorte).

### Método alterno (crear simple → editar para agregar variantes) — ⛔ YA NO FUNCIONA POR MCP
> Desde el 29 jul 2026 el MCP responde *"Ad creatives are immutable and cannot be edited in place"* al
> paso 2. Se deja como referencia porque **por la Graph API** el cambio de creativo de un anuncio sí
> funciona. Por MCP, crea el anuncio directo con la receta `DEGREES_OF_FREEDOM`.

Si trabajas por la Graph API y prefieres crear simple y luego editar, funcionaba así:
1. **Crear** cada anuncio SIMPLE con `object_story_spec.link_data` (imagen estática completa + 1 copy).
   Así conviven los 10 anuncios en el mismo conjunto (no dinámico):
   ```json
   {"object_story_spec":{"page_id":"100200300400500","link_data":{
      "image_hash":"a1b2c3d4e5f60718293a4b5c6d7e8f90","link":"https://...","message":"Sonríe con confianza. Evaluación de ortodoncia sin costo esta semana.","name":"Reserva tu evaluación",
      "description":"Brackets en Santiago con la Dra. Aurora Peña.","call_to_action":{"type":"LEARN_MORE","value":{"link":"https://..."}}}},
    "url_tags":"utm_...","contextual_multi_ads":{"enroll_status":"OPT_OUT"},
    "degrees_of_freedom_spec":{"creative_features_spec":{"standard_enhancements":{"enroll_status":"OPT_OUT"}}}}
   ```
2. **Editar** cada anuncio con `ads_update_entity(entity_type:"ad", fields:{creative:{...}})`, pasando el
   `creative` con **`asset_feed_spec`** (`optimization_type:"REGULAR"`, `images[{hash}]`, `bodies[]`,
   `titles[]`, `descriptions[]`, `link_urls`, `call_to_action_types`, `ad_formats:["SINGLE_IMAGE"]`) +
   `url_tags` + `contextual_multi_ads OPT_OUT`. **Editar NO vuelve dinámico el conjunto** → los 10
   anuncios quedan con sus variantes conviviendo en un solo conjunto. ✅ Confirmado jul-2026.
   (Si un update da `INTERNAL error`, reintentar: es transitorio.)

### Sin variantes (1 copy por anuncio)
- Crear el anuncio con el creativo **inline** (`object_story_spec` + `contextual_multi_ads:{enroll_status:
  "OPT_OUT"}`), no con un `creative_id` compartido: un `creative_id` deja el multi-anunciante MARCADO y,
  por MCP, ya no se puede editar después para desmarcarlo. Varios de estos anuncios caben en 1 conjunto.
- Crear ese **creativo simple**: `ads_create_creative` con `image_url` (URL pública) **o** `image_hash`,
  más `message`, `headline`, `description`, `link_url` y `call_to_action_type` → devuelve el `creative_id`.

## Secuencia recomendada (destino WEB) — 1 conjunto + N anuncios + variantes
> Sigue el **ORDEN QUE SÍ FUNCIONA** de la sección de Reglas Obligatorias (create LANDING con targeting
> definitivo → update a OFFSITE+evento → recién ahí los anuncios). Si no, aparecen INTERNAL / #1870194 /
> evento bloqueado.
1. `ads_create_campaign` — objetivo `OUTCOME_LEADS` (o SALES), `buying_type AUCTION`,
   `special_ad_categories "[]"`. Queda PAUSED. (Presupuesto: ABO, ver más abajo.)
2. **UN** `ads_create_ad_set` con `LANDING_PAGE_VIEWS` + `billing_event: IMPRESSIONS` + targeting
   definitivo (custom_locations + home/recent + advantage_audience 0) → **update** a `OFFSITE_CONVERSIONS` + `destination_type WEBSITE`
   + `promoted_object {pixel_id, custom_event_type}` (evento custom: OTHER + custom_event_str).
3. **N** `ads_create_ad` en ESE conjunto, cada uno con la receta `asset_feed_spec` DEGREES_OF_FREEDOM
   (o simple + variantes) + `url_tags` + `call_to_action LEARN_MORE` + opt-out multi-anunciante.
4. Tracking del píxel en cada anuncio (`tracking_specs`), **en el mismo `ads_create_ad`** (por MCP no se
   añade después).
5. Todo PAUSED → avisar. **No editar la geo en la GUI** antes de publicar (re-inyecta Advantage+ → #1870194).

## Destino WHATSAPP — VERDAD DEFINITIVA (verificada end-to-end, 07/04/2026)
> Esta sección reemplaza teorías equivocadas anteriores (que era la ToS, que Meta cambió reglas, que
> solo Ventas admitía variantes, que las de otros clientes eran anuncios separados). NADA de eso era
> correcto. Lo real es lo de abajo.

**Base del conjunto WhatsApp:** campaña `OUTCOME_LEADS`, **ABO** (presupuesto en el conjunto). Adset:
`destination_type WHATSAPP` + `promoted_object {page_id}` + `daily_budget`. **Sin píxel ni UTM** (el
destino es el chat). El `advantage_audience` hay que declararlo (0 o 1) o Graph pide la marca (1870227).

### Crear la campaña y el conjunto WhatsApp
`ads_create_campaign`:
```
ad_account_id: 1234567890123456
campaign_name: "🤖 07 Julio - Clientes Potenciales a WhatsApp - Brackets (MCP)"
objective: OUTCOME_LEADS          # objetivo Clientes Potenciales
buying_type: AUCTION
special_ad_categories: "[]"       # dental NO es categoría especial
# ABO (presupuesto por conjunto): NO poner campaign_daily_budget aquí
```
La respuesta trae `valid_optimization_goals` — para WhatsApp usaremos `CONVERSATIONS` (está en la lista).
> Convención de nombres: usar el mismo estilo que las campañas del cliente + un emoji para marcar el
> autor. Aquí `🤖` = hecha por Claude Code vía MCP. (Para un test con CLI usar otro emoji/prefijo, ej.
> `⚙️CLI`.)

`ads_create_ad_set`:
```
ad_account_id: 1234567890123456
campaign_id: 120210987654321000
ad_set_name: "🤖 CJ Único - Brackets WhatsApp"
billing_event: IMPRESSIONS
optimization_goal: CONVERSATIONS
destination_type: WHATSAPP
promoted_object: {"page_id":"100200300400500"}
daily_budget: 200                 # en centavos USD (ABO)
targeting: (ver Segmentación — usar YA la buena para no arreglarla después)
```
> Decisión de estructura: **1 conjunto con varios anuncios** (no un conjunto por creativo). Con 1
> conjunto, los anuncios **comparten** ese presupuesto (Meta reparte). Si se quisiera presupuesto
> independiente por creativo, sería un conjunto por creativo (más conjuntos).

Luego crea los N anuncios con la receta `asset_feed_spec` DEGREES_OF_FREEDOM de arriba. El `wa.me/18095551234`
es el WhatsApp de la página.

### Variantes de texto en WhatsApp — cuándo SÍ y cuándo NO (contexto de goals)
- **SÍ existen en Clientes Potenciales** (verificado: anuncios con 3 textos principales + 5 títulos + 5
  descripciones en UN anuncio, vistos en Ads Manager). NO son anuncios separados; son opciones de texto
  dentro del mismo anuncio.
- Requieren el conjunto en **`optimization_goal REPLIES`**. Bajo `CONVERSATIONS` + `OUTCOME_LEADS`/
  `OUTCOME_ENGAGEMENT`, meter `asset_feed_spec` con varios textos puede dar error **1885392** salvo con
  la receta DEGREES_OF_FREEDOM (que sí funciona bajo CONVERSATIONS, verificado).
- **`REPLIES` no se puede FIJAR si la WhatsApp Business Account (WABA) de la página no está sana:**
  - Por MCP → error **1815089** (el MCP mapea `REPLIES`→`LEAD_GENERATION`).
  - Por Graph/CLI → `success` pero al releer **se degradó a `CONVERSATIONS`** (create y update).
  - **Causa raíz:** si la WABA está "revisión no iniciada" + error de acceso (glitch de Meta), Meta no
    habilita `REPLIES`. Con la WABA funcional, `REPLIES` pega y las variantes entran solas. **NO es
    cambio de reglas ni ToS.**
  - **La ToS es un falso positivo:** `ads_get_ad_account_pages` reporta `leadgen_tos_accepted:false`
    VIEJO/erróneo. Verificar la PÁGINA directo por Graph: `GET /{page_id}?fields=leadgen_tos_accepted`.
    No perseguir la ToS.

### Ruta de respaldo para rotación de copy (WABA en mal estado): `OUTCOME_SALES`
- `OUTCOME_SALES` + destino WhatsApp SÍ admite `asset_feed_spec` con variantes (create directo).
- Receta: campaña `OUTCOME_SALES` (ABO) → conjunto `is_dynamic_creative:true`, `CONVERSATIONS`,
  `destination_type WHATSAPP`, `promoted_object {page_id}` → **1 anuncio** con `object_story_spec{page_id}`
  + `asset_feed_spec` (`bodies`/`titles`/`descriptions` hasta 5, `images[{hash}]`,
  `call_to_action_types:["WHATSAPP_MESSAGE"]`, `ad_formats:["SINGLE_IMAGE"]`) + `contextual_multi_ads OPT_OUT`.
- Dinámico = 1 anuncio/conjunto → N imágenes con su copy = N conjuntos. (Da el mismo resultado de
  llevar gente a WhatsApp; solo cambia la etiqueta del objetivo. Optimiza por conversaciones.)

## SEGMENTACIÓN GEOGRÁFICA — la parte más delicada (leer con cuidado)

Esto causó los errores más molestos. Reglas que quedaron claras:

1. **El MCP no tiene búsqueda de geo** (`ads_targeting_search` no está expuesto). No se puede obtener
   la "clave" de una ciudad/región. **Solución:** usar `custom_locations` con **latitud/longitud +
   radio** (las coordenadas del pin del mapa). Ventaja extra: por ciudad el radio mínimo de la GUI es
   ~16 km (demasiado para una ciudad chica); con `custom_locations` se puede bajar a **4 km**.
2. **Desactivar Advantage+ Audience** para campañas locales: `targeting_automation.advantage_audience: 0`.
   Con Advantage+ ON, Meta trata la ubicación como *señal* y **re-inyecta al publicar** el tipo de
   lugar "personas interesadas / que viajan a" (que Meta ya **descontinuó**) → provoca el error
   **#1870194** JUSTO al activar (en el editor no aparece, pero al darle activar/publicar sí).
3. **Fijar `location_types` a `["home","recent"]`** ("viven o estuvieron recientemente"; Meta añade
   `frequently_in` por su cuenta, y un conjunto nuevo sin `recent` no se publica), sin
   `travel_in`.

**Targeting recomendado (usarlo desde la creación del conjunto para no tener que arreglarlo):**
```json
{
  "geo_locations": {
    "custom_locations": [
      { "latitude": 19.4545, "longitude": -70.6876, "radius": 4, "distance_unit": "kilometer" }
    ],
    "location_types": ["home", "recent"]
  },
  "targeting_automation": { "advantage_audience": 0 }
}
```
(Coordenadas 19.4545, -70.6876 = Santiago de los Caballeros. Para otro cliente, tomar el lat/long de su
pin.)

Aplicar/actualizar con `ads_update_entity { entity_type:"ad_set", fields:{"targeting": {...}} }`.
> Si el usuario ya tocó la GUI, tras cambiarlo por API debe **"Descartar borrador" o recargar** la
> página para que la GUI cargue el estado guardado (si publica con el borrador encima, la GUI pisa el
> cambio del API y el error vuelve). El usuario cierra su edición antes de que el modelo escriba, así
> no se pisan.

## Tracking — eventos web (pixel) y offline (destino web)
En cada anuncio, el pixel va en `tracking_specs` **al crearlo** (`ads_create_ad`): el MCP ya no deja
añadirlo después. Por la Graph API sí se puede con un update. La forma del campo:
```
ads_update_entity {
  entity_type:"ad", entity_id:120210000000000001,
  fields: {"tracking_specs":[{"action.type":["offsite_conversion"],"fb_pixel":["1122334455667788"]}]}
}
```
- **Eventos offline:** Meta **unificó** pixel + offline en un mismo *dataset*. La clave
  `offline_event_set` en `tracking_specs` **da error** (subcode 1634011) si no hay un conjunto de
  eventos offline separado. Si solo existe el Pixel (que ya cubre offline en el modelo unificado),
  solo se enlaza el pixel. Si el cliente tuviera un *offline event set* aparte, se enlazaría con su ID.

## Image hashes (necesarios para asset_feed_spec)
- Las imágenes subidas por `image_url` quedan en la cuenta con el hash que Meta les asignó.
- Mapeo fiable creativo→hash: `ads_get_creatives(creative_ids=[...], fields=["image_hash"])`.
  (En `ads_get_ad_images` aparecen como "untitled_..." sin distinción; no sirve para mapear.)

## PRESUPUESTO: SIEMPRE ABO (en el conjunto), NO CBO (campaña)
- Criterio de operación (no regla de Meta): en estas campañas el **presupuesto va en el CONJUNTO de anuncios (ABO)**, nunca a
  nivel campaña. → En `ads_create_campaign` NO pasar `campaign_daily_budget`/`campaign_lifetime_budget`;
  pasar `daily_budget` en `ads_create_ad_set`. Con ABO, la opción "compartir presupuesto entre
  conjuntos" queda deshabilitada por naturaleza (no se activa).
- **Multi-anunciante OFF — GOTCHA IMPORTANTE (verificado 07/04):** el opt-out `contextual_multi_ads:
  {enroll_status:"OPT_OUT"}` SOLO pega si el anuncio tiene **creativo propio inline** (`object_story_spec`).
  Si el anuncio referencia un **`creative_id` compartido**, editar `{creative:{creative_id,
  contextual_multi_ads:OPT_OUT}}` devuelve `success:true` pero **NO cambia nada** (el anuncio sigue con
  el creativo compartido sin opt-out → queda MARCADO). ⛔ Por MCP ya no se arregla editando (los
  creativos son inmutables por esa vía): **créalo bien a la primera** con el creativo inline. Por la Graph
  API, el arreglo era editar con el creativo **inline completo**:
  `{creative:{object_story_spec:{page_id, link_data:{link, message, name, image_hash,
  call_to_action:{type:"WHATSAPP_MESSAGE", value:{link}}}}, contextual_multi_ads:{enroll_status:"OPT_OUT"}}}`.
  Eso crea un creativo nuevo propio del anuncio CON el opt-out. **Verificar SIEMPRE por Graph:**
  `GET /{ad_id}?fields=creative{contextual_multi_ads}` (debe decir `OPT_OUT`; el MCP no lo expone bien).
  Para obtener el `object_story_spec` de un creativo existente: `GET /{creative_id}?fields=object_story_spec`.

## Orden de activación / publicación
Entrega solo si **los 3 niveles están ACTIVOS + publicado**: campaña ✅ + conjunto ✅ + anuncios ✅.
- Activar por MCP (requiere confirmación explícita del usuario, porque **empieza a gastar**):
  `ads_activate_entity { entity_type: campaign|ad_set|ad, entity_id }`.
- O el usuario activa los toggles en la GUI y da **"Revisar y publicar"**.
- El botón **"Publicar"** se ve gris/bloqueado cuando hay **errores de validación pendientes** o cuando
  **no hay cambios por publicar** — NO es que "solo el modelo pueda publicar".

## CLI/Graph vs MCP (cuál puede qué)
- **CLI/Graph** (token `.env`, app en **modo DEV**): crea campañas y conjuntos, pero **NO anuncios ni
  creativos** → error **1885183** ("app en modo de desarrollo").
- **MCP** (usa la app de Meta, en **Live**): SÍ crea anuncios/creativos.
- **Ventaja del CLI:** sube la imagen del anuncio de forma **nativa** (`POST /adimages`), así que —a
  diferencia del MCP— NO depende de una URL pública. Por eso puede convenir pese al muro de modo DEV.
- Combinable: crear el conjunto por Graph (p.ej. para forzar un goal) + el anuncio por MCP.

## Catálogo de errores encontrados y su solución (referencia rápida)

| Síntoma / error | Causa real | Solución |
|---|---|---|
| No hay cómo subir la imagen por MCP | Hasta nov 2026 el MCP no subía imágenes; hoy `ads_creative_upload_media` sí, con despliegue gradual ("gradually rolled out") | `ads_creative_upload_media`; si no está activa en tu cuenta, `image_url` = una URL pública de tu imagen |
| La imagen subida a "Biblioteca multimedia" no aparece en la cuenta | Biblioteca de portafolio ≠ `adimages` de la cuenta | Usar `image_url` pública (no la biblioteca) |
| Google Drive no da URL pública | El MCP de Drive no crea/edita permisos (solo lee) | No sirve para hospedar; usar tu propio hosting |
| La URL pública da **404 intermitente** | Imagen recién generada; tu servidor de imágenes aún no la sirve estable | Esperar a que responda `200` fijo (probar con `curl`) y reintentar |
| Meta: *"no se descargó la imagen ... robots.txt"* | La URL de esa imagen respondía 404 en ese momento | Reintentar cuando la URL dé 200 |
| Flexible: *"El campo link es obligatorio"* (2061015) | Falta `link` en `link_data` | Poner `link` (wa.me) dentro de `object_story_spec.link_data` |
| Flexible: texto rota pero **sin imagen** | `image_hash` solo en `asset_feed_spec` | Poner `image_hash` **también** en `link_data` |
| **1885553** al crear el 2º anuncio / **1885274 / 1885392** al editar | Conjunto quedó `is_dynamic_creative:true` (asset_feed_spec sin link_data, `optimization_type` mal) | Usar la receta `DEGREES_OF_FREEDOM` + imagen en los DOS lados → conjunto NO dinámico, N anuncios |
| `tracking_specs` offline: *"clave offline_event_set no válida"* (1634011) | Dataset unificado; no hay offline set separado | Solo enlazar el pixel (`offsite_conversion` + `fb_pixel`) |
| Campos bloqueados en GUI (conversión, objetivo, página, WhatsApp) | Normal en conjuntos de **mensajería** tras crearse | No es bug; no hace falta editarlos |
| **#2923012** "Necesitas WhatsApp Business" | **Cascada** del error de ubicación (el número ya estaba conectado — Meta lo confirma en el diálogo "Instalar") | Se cae solo al arreglar la ubicación |
| **#1870194** "opción de segmentación por lugar suprimida" — vuelve **al activar** | **Advantage+ Audience** re-inyecta el tipo de lugar descontinuado al publicar, **o** un conjunto nuevo con `location_types` sin `recent` | `advantage_audience: 0` + `location_types:["home","recent"]` + `custom_locations` |
| **1815089** al fijar `REPLIES` por MCP | El MCP mapea `REPLIES`→`LEAD_GENERATION` y la WABA no está sana | Usar `CONVERSATIONS` + receta DEGREES_OF_FREEDOM, o ruta de respaldo `OUTCOME_SALES` |
| UTM / parámetros de URL en WhatsApp | No aplican a WhatsApp (sí a campañas web) | No poner UTM en WhatsApp |
| "Mejoras de contenido" / Advantage+ creative | Meta las activa por defecto | Dejarlas **OFF**: por MCP `standard_enhancements` OPT_OUT; por Graph v25+ una a una (§ 0 punto 5), y releer `creative_features_spec` |
| Con `LANDING_PAGE_VIEWS` el píxel da "combinación no válida" | Goal incompatible con el píxel | Usar `OFFSITE_CONVERSIONS` |
| `INTERNAL error` en un update | A veces transitorio | Reintentar 1-2 veces; PERO ver los casos NO transitorios de abajo |
| `INTERNAL` al **crear** ad_set con `OFFSITE_CONVERSIONS` | Bug del create con ese goal | Crear con `LANDING_PAGE_VIEWS` y **editar** a OFFSITE + promoted_object |
| `INTERNAL` al **actualizar** targeting a `custom_locations` (pin) | El update no acepta custom_locations (el create sí) | Poner el pin YA en el `create` del ad_set; si el conjunto ya existe, recrearlo con el pin |
| **3260011** "no puedes editar píxel/evento/optimización de un conjunto publicado" | El MCP crea el ad_set "publicado en pausa"; se bloquea al consolidarse con anuncios | Fijar el evento/optimización correctos ANTES de crear anuncios; si ya está bloqueado, crear un ad_set nuevo |
| CTA `BOOK_NOW` sale como "desconocido" en la GUI | No es el CTA estándar para ese anuncio | Usar `LEARN_MORE` ("Más información") por defecto |

## CONVENCIÓN DE NOMBRES (la que usamos; adáptala a la tuya)
- **🤖 al INICIO de TODO nombre** (campaña, conjunto y anuncios), antes del resto: marca lo hecho por
  IA (MCP/CLI) vs humano, en los tres niveles.
- **Campaña:** `MM.n Mes - [Objetivo] a [Destino] - [servicio/detalle] - MM/DD/YYYY`
  - `MM` = número del mes; `.n` = correlativo si hay más de una campaña del mes (07.1, 07.2, 07.3…).
  - `Mes` en palabra (Julio). Objetivo: Clientes Potenciales / Interacción / Ventas / Tráfico…
  - Destino: a WhatsApp / a Landing / a Web / a Perfil…
  - Ej.: `07.2 Julio - Clientes Potenciales a WhatsApp - Páginas web y Landing - 07/04/2026`
- **Conjunto:** `NN [TF|TC]/[SI|CI][ /PS | /PP ] - [detalle a destino] - [ciudad/zona] - MM/DD/YYYY`
  - `NN` = número de conjunto (01, 02…). **TF** = tráfico frío · **TC** = tráfico caliente.
  - **SI** = sin intereses (segmentación abierta, lo que usamos) · **CI** = con intereses.
  - **PS** = público similar (lookalike) · **PP** = público personalizado. (Si es abierto: solo `TF/SI`.)
  - Ej.: `01 TF/SI - Páginas web a WhatsApp - RD - 07/04/2026`
- **Anuncio:** `[NN del conjunto] Imagen [N de la imagen] - [ángulo]`. **REGLA IRREMPLAZABLE:** el PRIMER
  número es el **prefijo del CONJUNTO** donde vive el anuncio — **todos** los anuncios de un mismo
  conjunto empiezan con ese mismo prefijo. Luego `Imagen` + el número de la imagen/creativo.
  - Ej. conjunto `01` con 10 imágenes: `01 Imagen 1`, `01 Imagen 2`, `01 Imagen 3` … `01 Imagen 10`.
  - Ej. conjunto `05` con 3 imágenes: `05 Imagen 1`, `05 Imagen 2`, `05 Imagen 3`.
  - ❌ NO `01 Imagen 01`, `02 Imagen 02`… (eso pone el número del conjunto donde va el de la imagen).
- **UTM (solo destino web):** debe dejar claro el origen — campaña, conjunto, servicio, nº de anuncio y TIPO.
  - `utm_source=meta&utm_medium=paid_social&utm_campaign=<MM.n-objetivo-destino>&utm_content=cj<NN>-<servicio>-ad<NN>-<tipo>`
  - Ej.: `...&utm_campaign=07.2-clientes-potenciales-web&utm_content=cj01-protesis-ad08-precio`.
  - `<tipo>` = ángulo/clasificación del anuncio: estetico, autoridad, funcion, precio,
    durabilidad, bajo-riesgo, comparacion.

## Paso a paso resumido (checklist reproducible)
1. Ten listos tus creativos: por cada uno, el copy (primary texts, headlines, descriptions) y su imagen
   subida a **una URL pública**. Verificar cada URL con `curl` (200 image/jpeg).
2. `ads_get_ad_accounts` (cuenta queryable) · `ads_get_ad_account_pages` (page_id) ·
   `ads_get_datasets` (pixel).
3. `ads_create_campaign` (WhatsApp: OUTCOME_LEADS; web: TRAFFIC/SALES/LEADS; AUCTION, `[]`, ABO).
4. `ads_create_ad_set` (WhatsApp: CONVERSATIONS, WHATSAPP, promoted_object page_id; web:
   OFFSITE_CONVERSIONS, WEBSITE, promoted_object pixel; `daily_budget`, `billing_event IMPRESSIONS`,
   **targeting bueno desde ya**).
5. Por creativo: `ads_creative_upload_media` (o `ads_create_creative` con la `image_url` pública) → espera a
   `ACTIVE` y lee el `image_hash` (todos distintos, § 0 punto 3) → `ads_create_ad` **flexible** con hash en `link_data` **y**
   `asset_feed_spec` (`optimization_type: DEGREES_OF_FREEDOM`, bodies/titles/descriptions) + opt-out.
6. Web: `tracking_specs` con el pixel + `url_tags` UTM **dentro del mismo `ads_create_ad`** (por MCP no se
   añaden después).
7. `ads_get_ad_preview` de cada anuncio para confirmar imagen + variantes + botón/CTA.
8. Usuario recarga la GUI, **Revisar y publicar**; luego activar (campaña→conjunto→anuncios) —
   confirmación explícita porque gasta.
