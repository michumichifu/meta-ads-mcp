# Referencia de comandos — Ads CLI

> Fuente: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/command-reference
> Actualizado: 29 abr. 2026

Referencia completa de todos los comandos de la CLI de anuncios.

## Sintaxis del comando

- Comandos de anuncios: `meta ads <resource> <action> [options]`
- Comandos de autenticación: `meta auth <action>`
- `<lowercase>` = palabra clave fija (escribe el texto tal cual)
- `<UPPER_CASE>` = marcador de posición (sustituye por tu valor)
- `[lowercase]` = palabra clave o grupo opcional
- `--flag` = opción o parámetro con nombre

## Opciones globales

Se aplican a todos los comandos y van **antes** del subcomando:

```
meta [global options] ads <command> <subcommand> [options]
```

| Opción | Alias | Descripción |
|---|---|---|
| `--output <format>` | `-o` | Formato de salida: `table` (predeterminado), `json`, `plain`. |
| `--no-color` | | Desactivar el color en la salida. |
| `--no-input` | | Desactivar peticiones interactivas. |
| `--debug` | | Activar salida de depuración. |
| `--help` | `-h` | Mostrar ayuda. |
| `--version` | `-v` | Mostrar la versión. |

### Formatos de salida

- **table** (predeterminado): tabla legible con encabezados. Mejor para uso interactivo.
- **json**: array JSON estructurado. Ideal para canalizar a `jq`.
- **plain**: valores separados por tabulaciones, un registro por línea. Funciona con `cut`, `awk`, `sort`.

```bash
# Salida JSON, pipe a jq
meta --output json ads campaign list | jq '.[].name'

# Salida plain, ordenar por columna
meta --output plain ads campaign list | sort -t$'\t' -k5 -rn
```

---

## Autenticación

Se autentica con un identificador de acceso de usuario del sistema.

### `meta auth` — guardar el token

```bash
# archivo .env
ACCESS_TOKEN=<ACCESS_TOKEN>

# o exportar directamente
export ACCESS_TOKEN=<ACCESS_TOKEN>
```

### `meta auth status` — comprobar estado

Muestra una versión enmascarada del identificador almacenado.

```bash
meta auth status
```

---

## Cuentas publicitarias

### `meta ads adaccount list`

Enumera todas las cuentas publicitarias accesibles.

```bash
meta ads adaccount list
meta ads adaccount list --limit 50
meta --output json ads adaccount list
```

| Opción | Alias | Predeterminado | Descripción |
|---|---|---|---|
| `--limit` | `-l` | 25 | Máximo de cuentas a devolver. |

Columnas de salida: `id`, `name`, `account_status`, `currency`, `timezone_name`

### `meta ads adaccount current`

Muestra el `AD_ACCOUNT_ID` configurado actualmente.

```bash
meta ads adaccount current
```

---

## Páginas empresariales de Facebook

### `meta ads page list`

Enumera las páginas de empresa accesibles. Usa estos IDs con `--page-id` en los comandos de creative.

```bash
meta ads page list
meta ads page list --limit 50
meta --output json ads page list
```

| Opción | Alias | Predeterminado | Descripción |
|---|---|---|---|
| `--limit` | `-l` | 25 | Máximo de páginas a devolver. |

Columnas de salida: `id`, `name`, `category`

---

## Campañas publicitarias

### `meta ads campaign list`

```bash
meta ads campaign list
meta ads campaign list --limit 25
meta --output json ads campaign list
```

| Opción | Alias | Predeterminado | Descripción |
|---|---|---|---|
| `--limit` | `-l` | 10 | Máximo de campañas a devolver. |

Columnas: `name`, `effective_status`, `id`, `objective`, `daily_budget`, `lifetime_budget`, `start_time`

### `meta ads campaign create`

```bash
meta ads campaign create --name "My Campaign" --objective OUTCOME_TRAFFIC
meta ads campaign create --name "Sales Campaign" --objective OUTCOME_SALES \
  --daily-budget 5000
```

| Opción | Obligatorio | Predeterminado | Descripción |
|---|---|---|---|
| `--name` | Sí | | Nombre de la campaña. |
| `--objective` | Sí | | Objetivo de la campaña. |
| `--daily-budget` | No | | Presupuesto diario en céntimos (5000 = 50,00 USD). |
| `--lifetime-budget` | No | | Presupuesto total en céntimos. |
| `--status` | No | `PAUSED` | Estado inicial: `ACTIVE` o `PAUSED`. |

**Objetivos:** `OUTCOME_APP_PROMOTION`, `OUTCOME_AWARENESS`, `OUTCOME_ENGAGEMENT`, `OUTCOME_LEADS`, `OUTCOME_SALES`, `OUTCOME_TRAFFIC`

### `meta ads campaign get`

```bash
meta ads campaign get <CAMPAIGN_ID>
```

### `meta ads campaign update`

```bash
meta ads campaign update <CAMPAIGN_ID> --name "New Name"
meta ads campaign update <CAMPAIGN_ID> --status ACTIVE
meta ads campaign update <CAMPAIGN_ID> --daily-budget 10000
```

| Opción | Descripción |
|---|---|
| `--name` | Nuevo nombre. |
| `--status` | Nuevo estado: `ACTIVE`, `PAUSED`, `ARCHIVED`. |
| `--daily-budget` | Nuevo presupuesto diario (céntimos). |
| `--lifetime-budget` | Nuevo presupuesto total (céntimos). |

Nota: se debe proporcionar al menos una opción.

### `meta ads campaign delete`

Elimina una campaña y todos sus conjuntos de anuncios y anuncios dependientes.

```bash
meta ads campaign delete <CAMPAIGN_ID>
meta ads campaign delete <CAMPAIGN_ID> --force   # omitir confirmación
```

| Opción | Alias | Descripción |
|---|---|---|
| `--force` | `-f` | Omitir el mensaje de confirmación. |

---

## Conjuntos de anuncios

### `meta ads adset list`

```bash
meta ads adset list                 # todos los adsets de la cuenta
meta ads adset list <CAMPAIGN_ID>   # adsets de una campaña
meta ads adset list --limit 25
```

| Argumento | Obligatorio | Descripción |
|---|---|---|
| `<CAMPAIGN_ID>` | No | Filtra por adsets de esta campaña. |

| Opción | Alias | Predeterminado | Descripción |
|---|---|---|---|
| `--limit` | `-l` | 10 | Máximo de adsets a devolver. |

Columnas: `name`, `effective_status`, `id`, `optimization_goal`, `billing_event`, `daily_budget`, `lifetime_budget`, `start_time`

### `meta ads adset create`

```bash
meta ads adset create <CAMPAIGN_ID> --name "My Ad Set" \
  --optimization-goal LINK_CLICKS --billing-event IMPRESSIONS \
  --bid-amount 500 --targeting-countries US

# Adset de conversiones
meta ads adset create <CAMPAIGN_ID> --name "Conversions Set" \
  --optimization-goal OFFSITE_CONVERSIONS \
  --billing-event IMPRESSIONS --pixel-id <PIXEL_ID> \
  --custom-event-type PURCHASE --targeting-countries US
```

| Opción | Obligatorio | Predeterminado | Descripción |
|---|---|---|---|
| `--name` | Sí | | Nombre del adset. |
| `--optimization-goal` | Sí | | Objetivo de optimización. |
| `--billing-event` | Sí | | Evento de facturación. |
| `--daily-budget` | No | | Presupuesto diario (céntimos). Omitir si la campaña usa optimización de presupuesto de campaña (CBO). |
| `--lifetime-budget` | No | | Presupuesto total (céntimos). Requiere `--end-time`. |
| `--bid-amount` | No | | Importe de puja (céntimos). |
| `--start-time` | No | | Hora de inicio (ISO 8601). |
| `--end-time` | No | | Hora de fin (ISO 8601). Obligatorio con `--lifetime-budget`. |
| `--status` | No | `PAUSED` | Estado inicial: `ACTIVE` o `PAUSED`. |
| `--targeting-countries` | No | | Códigos de país separados por comas (p. ej. `US,CA,GB`). |
| `--pixel-id` | No | | ID del conjunto de datos (píxel) para seguimiento de conversiones. |
| `--custom-event-type` | No | `PURCHASE` | Tipo de evento de conversión. Se usa con `--pixel-id`. |

**Objetivos de optimización:** `APP_INSTALLS`, `CONVERSATIONS`, `EVENT_RESPONSES`, `IMPRESSIONS`, `LANDING_PAGE_VIEWS`, `LEAD_GENERATION`, `LINK_CLICKS`, `OFFSITE_CONVERSIONS`, `PAGE_LIKES`, `POST_ENGAGEMENT`, `REACH`, `THRUPLAY`, `VALUE`

**Eventos de facturación:** `APP_INSTALLS`, `CLICKS`, `IMPRESSIONS`, `LINK_CLICKS`, `PAGE_LIKES`, `POST_ENGAGEMENT`, `THRUPLAY`

**Tipos de eventos personalizados:** `ADD_PAYMENT_INFO`, `ADD_TO_CART`, `ADD_TO_WISHLIST`, `COMPLETE_REGISTRATION`, `CONTACT`, `CONTENT_VIEW`, `CUSTOMIZE_PRODUCT`, `DONATE`, `FIND_LOCATION`, `INITIATED_CHECKOUT`, `LEAD`, `OTHER`, `PURCHASE`, `SCHEDULE`, `SEARCH`, `START_TRIAL`, `SUBMIT_APPLICATION`, `SUBSCRIBE`

### `meta ads adset get`

```bash
meta ads adset get <AD_SET_ID>
```

### `meta ads adset update`

```bash
meta ads adset update <AD_SET_ID> --name "New Name"
meta ads adset update <AD_SET_ID> --status ACTIVE
meta ads adset update <AD_SET_ID> --daily-budget 10000
```

| Opción | Descripción |
|---|---|
| `--name` | Nuevo nombre. |
| `--status` | Nuevo estado: `ACTIVE`, `PAUSED`, `ARCHIVED`. |
| `--daily-budget` | Nuevo presupuesto diario (céntimos). |
| `--lifetime-budget` | Nuevo presupuesto total (céntimos). |
| `--bid-amount` | Nuevo importe de puja (céntimos). |
| `--end-time` | Nueva hora de fin (ISO 8601). |

Nota: se debe proporcionar al menos una opción.

### `meta ads adset delete`

Elimina un adset y todos sus anuncios dependientes.

```bash
meta ads adset delete <AD_SET_ID>
meta ads adset delete <AD_SET_ID> --force
```

| Opción | Alias | Descripción |
|---|---|---|
| `--force` | `-f` | Omitir confirmación. |

---

## Anuncios

### `meta ads ad list`

```bash
meta ads ad list                # todos los anuncios de la cuenta
meta ads ad list <AD_SET_ID>    # anuncios de un adset
meta ads ad list --limit 25
```

| Argumento | Obligatorio | Descripción |
|---|---|---|
| `<ADSET_ID>` | No | Filtra por anuncios de este adset. |

| Opción | Alias | Predeterminado | Descripción |
|---|---|---|---|
| `--limit` | `-l` | 10 | Máximo de anuncios a devolver. |

Columnas: `name`, `effective_status`, `id`, `adset_id`, `campaign_id`, `created_time`

### `meta ads ad create`

Los anuncios hacen referencia a un creative. **Crea primero el creative** con `meta ads creative create`.

```bash
meta ads ad create <AD_SET_ID> --name "My Ad" --creative-id <CREATIVE_ID>
meta ads ad create <AD_SET_ID> --name "Conversion Ad" --creative-id <CREATIVE_ID> \
  --pixel-id <PIXEL_ID>
```

| Opción | Obligatorio | Predeterminado | Descripción |
|---|---|---|---|
| `--name` | Sí | | Nombre del anuncio. |
| `--creative-id` | Sí | | ID del creative a usar. |
| `--status` | No | `PAUSED` | Estado inicial: `ACTIVE`, `PAUSED`. |
| `--pixel-id` | No | | ID del conjunto de datos (píxel) para seguimiento de conversiones. |
| `--tracking-specs` | No | | Especificaciones de seguimiento como JSON (mutuamente excluyente con `--pixel-id`). |

Nota: usa `--pixel-id` **o** `--tracking-specs`, no ambos.

### `meta ads ad get`

```bash
meta ads ad get <AD_ID>
meta --output json ads ad get <AD_ID>
```

### `meta ads ad update`

```bash
meta ads ad update <AD_ID> --name "New Name"
meta ads ad update <AD_ID> --creative-id <CREATIVE_ID>
meta ads ad update <AD_ID> --status ACTIVE
```

| Opción | Descripción |
|---|---|
| `--name` | Nuevo nombre. |
| `--creative-id` | Nuevo ID de creative. |
| `--status` | Nuevo estado: `ACTIVE`, `PAUSED`, `ARCHIVED`. |

Nota: se debe proporcionar al menos una opción.

### `meta ads ad delete`

```bash
meta ads ad delete <AD_ID>
meta ads ad delete <AD_ID> --force
```

| Opción | Alias | Descripción |
|---|---|---|
| `--force` | `-f` | Omitir confirmación. |

---

## Contenido de los anuncios (creatives)

> Ver la doc dedicada: `.../ads-cli/ad-creatives`

### `meta ads creative list`

```bash
meta ads creative list
meta ads creative list --limit 25
```

| Opción | Alias | Predeterminado | Descripción |
|---|---|---|---|
| `--limit` | `-l` | 10 | Máximo de creatives a devolver. |

Columnas: `name`, `id`, `status`, `title`, `body`

### `meta ads creative create`

Dos modos: **estándar** y **optimización dinámica del contenido (DCO)**.

#### Creative estándar (una imagen o vídeo)

```bash
meta ads creative create --name "Summer Sale" --image ./banner.jpg \
  --page-id <PAGE_ID> --body "50% off everything!" \
  --link-url https://example.com --title "Shop Now" \
  --call-to-action SHOP_NOW
```

| Opción | Obligatorio | Descripción |
|---|---|---|
| `--name` | Sí | Nombre del creative. |
| `--page-id` | Sí | ID de la página de Facebook usada como identidad del anuncio. |
| `--image` | No | Ruta a archivo de imagen. |
| `--video` | No | Ruta a archivo de vídeo. |
| `--body` | No | Texto principal del anuncio. |
| `--title` | No | Título debajo de la imagen. |
| `--link-url` | No | URL de destino al hacer clic. |
| `--description` | No | Descripción del enlace debajo del título. |
| `--call-to-action` | No | Tipo de botón de CTA. |
| `--instagram-actor-id` | No | ID de la cuenta de Instagram para ubicaciones de IG. |

Proporciona `--image` **o** `--video`, no ambos.

#### Optimización dinámica del contenido (DCO)

Varias piezas y Meta prueba combinaciones automáticamente:

```bash
meta ads creative create --name "DCO Creative" --page-id <PAGE_ID> \
  --link-url https://example.com \
  --images ./img1.jpg --images ./img2.jpg --images ./img3.jpg \
  --titles "Title A" --titles "Title B" \
  --bodies "Body 1" --bodies "Body 2" \
  --call-to-actions SHOP_NOW --call-to-actions LEARN_MORE
```

| Opción | Máx. | Descripción |
|---|---|---|
| `--images` | 10 | Rutas de imágenes (repetir por cada una). |
| `--videos` | 10 | Rutas de vídeos (repetir por cada uno). |
| `--titles` | 5 | Variaciones de título. |
| `--bodies` | 5 | Variaciones de texto principal. |
| `--descriptions` | 5 | Variaciones de descripción. |
| `--call-to-actions` | 5 | Tipos de CTA. |

DCO requiere `--link-url` y al menos un `--images` o `--videos`.

**Tipos de CTA (call-to-action):** `APPLY_NOW`, `BOOK_TRAVEL`, `BUY_NOW`, `CONTACT_US`, `DOWNLOAD`, `GET_OFFER`, `GET_QUOTE`, `LEARN_MORE`, `NO_BUTTON`, `OPEN_LINK`, `SHOP_NOW`, `SIGN_UP`, `SUBSCRIBE`, `WATCH_MORE`

### `meta ads creative get`

```bash
meta ads creative get <CREATIVE_ID>
meta --output json ads creative get <CREATIVE_ID>
```

### `meta ads creative update`

Solo cambia los campos que especifiques.

```bash
meta ads creative update <CREATIVE_ID> --name "New Name"
meta ads creative update <CREATIVE_ID> --body "Updated copy" --title "New Title"
meta ads creative update <CREATIVE_ID> --image ./new-banner.jpg
meta ads creative update <CREATIVE_ID> --status PAUSED
```

Opciones: `--name`, `--image`, `--video`, `--body`, `--title`, `--link-url`, `--description`, `--call-to-action`, `--instagram-actor-id`, `--status` (`ACTIVE`/`PAUSED`).

Nota: algunos campos no se pueden actualizar tras la creación (restricciones de la API de Meta); en esos casos crea un creative nuevo.

### `meta ads creative delete`

No se puede eliminar un creative en uso por anuncios activos.

```bash
meta ads creative delete <CREATIVE_ID>
meta ads creative delete <CREATIVE_ID> --force
```

---

## Conjuntos de datos (píxeles de Meta)

> Ver: `.../ads-cli/datasets-and-catalogs#datasets`

### `meta ads dataset list`

```bash
meta ads dataset list
meta ads dataset list --business-id <BUSINESS_ID>
meta ads dataset list --limit 50
```

| Opción | Alias | Predeterminado | Descripción |
|---|---|---|---|
| `--business-id` | | | Listar los conjuntos de datos de esta empresa. |
| `--limit` | `-l` | 25 | Máximo a devolver. |

Columnas: `name`, `id`, `creation_time`, `last_fired_time`

### `meta ads dataset get`

```bash
meta ads dataset get <PIXEL_ID>
```

### `meta ads dataset create`

Crea un píxel en una empresa (asigna permisos automáticamente).

```bash
meta ads dataset create --name "My Pixel"
meta ads dataset create --name "My Pixel" --business-id <BUSINESS_ID>
```

| Opción | Obligatorio | Descripción |
|---|---|---|
| `--name` | Sí | Nombre del conjunto de datos. |
| `--business-id` | No | Empresa donde crearlo (se resuelve desde la cuenta si se omite). |

### `meta ads dataset connect`

Conecta un píxel a una cuenta publicitaria o catálogo.

```bash
meta ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
meta ads dataset connect <PIXEL_ID> --catalog-id <CATALOG_ID>
meta ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --catalog-id <CATALOG_ID>
```

Necesita al menos uno de `--ad-account-id` o `--catalog-id`. También acepta `--business-id`.

### `meta ads dataset disconnect`

```bash
meta ads dataset disconnect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
meta ads dataset disconnect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --force
```

`--ad-account-id` es obligatorio. Opcionales: `--business-id`, `--force`.

### `meta ads dataset assign-user`

```bash
meta ads dataset assign-user <PIXEL_ID>
meta ads dataset assign-user <PIXEL_ID> --user-id <USER_ID>
meta ads dataset assign-user <PIXEL_ID> --tasks ADVERTISE --tasks ANALYZE --tasks EDIT
```

| Opción | Predeterminado | Descripción |
|---|---|---|
| `--user-id` | Usuario autenticado | ID de usuario a asignar. |
| `--tasks` | `ADVERTISE`, `ANALYZE` | Permisos (repetir). |

Tareas disponibles: `ADVERTISE`, `ANALYZE`, `EDIT`, `UPLOAD`

---

## Catálogos de productos

> Ver: `.../ads-cli/datasets-and-catalogs#product-catalogs`

Todos los comandos de catálogo aceptan `--business-id` (o env `BUSINESS_ID`); si se omite, se resuelve desde la cuenta publicitaria.

### `meta ads catalog list`

```bash
meta ads catalog list
meta ads catalog list --business-id <BUSINESS_ID>
meta ads catalog list --limit 50
```

Columnas: `name`, `id`, `vertical`, `product_count`, `feed_count`

### `meta ads catalog get`

```bash
meta ads catalog get <CATALOG_ID>
```

### `meta ads catalog create`

```bash
meta ads catalog create --name "My Catalog"
meta ads catalog create --name "Hotel Catalog" --vertical hotels
```

| Opción | Obligatorio | Predeterminado | Descripción |
|---|---|---|---|
| `--name` | Sí | | Nombre del catálogo. |
| `--vertical` | No | `commerce` | Tipo de catálogo. |

**Sectores (vertical):** `adoptable_pets`, `commerce`, `destinations`, `flights`, `generic`, `home_listings`, `hotels`, `local_service_businesses`, `offer_items`, `offline_commerce`, `transactable_items`, `vehicles`

### `meta ads catalog update`

```bash
meta ads catalog update <CATALOG_ID> --name "Renamed Catalog"
```

### `meta ads catalog delete`

```bash
meta ads catalog delete <CATALOG_ID>
meta ads catalog delete <CATALOG_ID> --force
```

---

## Insights

> Ver la doc dedicada para la referencia completa de opciones: `.../ads-cli/insights`

### `meta ads insights get`

Control total sobre campos, intervalo de fechas y desgloses.

```bash
meta ads insights get
meta ads insights get --fields spend,impressions,ctr
meta ads insights get --since 2024-01-01 --until 2024-01-31
meta ads insights get --breakdown age --breakdown gender
meta ads insights get --campaign-id <CAMPAIGN_ID>
```

Opciones vistas: `--fields` (lista separada por comas), `--since` / `--until` (fechas), `--breakdown` (repetible), `--campaign-id`. La referencia completa de opciones está en la página de Insights.
