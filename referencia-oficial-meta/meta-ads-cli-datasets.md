# Conjuntos de datos y catálogos — CLI de anuncios de Meta

> Fuente: Documentación oficial de Meta para desarrolladores — Ads CLI.
> URL original (inglés): https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/datasets-and-catalogs
> Actualizado: 29 abr. 2026

En esta guía se cubre la administración de conjuntos de datos (píxeles de Meta) y catálogos de productos con la CLI de anuncios.

## Resolución del identificador de empresa

Los comandos del catálogo y del conjunto de datos necesitan el identificador de empresa. La CLI de anuncios lo resuelve en este orden:

1. Marca `--business-id`
2. Variable de entorno `BUSINESS_ID`
3. Se deriva automáticamente de la cuenta publicitaria configurada.

Si ninguno está disponible, el comando te pedirá que proporciones uno.

## Conjuntos de datos

Un conjunto de datos representa un extremo del píxel de Meta o la API de conversiones. Usa conjuntos de datos para hacer un seguimiento de los eventos en tu sitio web, como las compras, los artículos añadidos al carrito y las visitas a páginas. Los conjuntos de datos son esenciales para las campañas publicitarias de conversión y la nueva segmentación.

### Enumerar conjuntos de datos

```bash
# List from the ad account (default and fallback)
meta ads dataset list

# List from a specific business
meta ads dataset list --business-id <BUSINESS_ID>
```

El identificador de empresa se resuelve automáticamente a partir de la cuenta publicitaria configurada. Establece la variable de entorno `BUSINESS_ID` o utiliza `--business-id` para especificar una de forma explícita.

### Crear un conjunto de datos

Antes de crear un conjunto de datos, un administrador de la empresa debe aceptar las Condiciones del servicio de las herramientas de Meta para empresas. La CLI de anuncios te pide que completes este paso si es necesario.

```bash
meta ads dataset create --name "My Website Pixel"
meta ads dataset create --name "My Pixel" --business-id <BUSINESS_ID>
```

Después de la creación, la CLI de anuncios asigna automáticamente el usuario autenticado al conjunto de datos con los permisos `ADVERTISE`, `ANALYZE` y `EDIT`.

### Ver los detalles del conjunto de datos

```bash
meta ads dataset get <PIXEL_ID>
meta --output json ads dataset get <PIXEL_ID>
```

### Conectar un conjunto de datos

Conecta los conjuntos de datos a las cuentas publicitarias o los catálogos de productos antes de usarlos.

#### Conectarse a una cuenta publicitaria

```bash
meta ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
```

#### Conectarse a un catálogo de productos (para el seguimiento de las conversiones de los productos del catálogo)

```bash
meta ads dataset connect <PIXEL_ID> --catalog-id <CATALOG_ID>
```

#### Conectar a ambos con un solo comando

```bash
meta ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --catalog-id <CATALOG_ID>
```

### Desconectar un conjunto de datos

Eliminar la conexión de un conjunto de datos de una cuenta publicitaria:

```bash
meta ads dataset disconnect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
meta ads dataset disconnect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --force
```

### Asignar usuarios

Concede permisos a un usuario en un conjunto de datos:

```bash
# Assign yourself (default: ADVERTISE + ANALYZE)
meta ads dataset assign-user <PIXEL_ID>

# Assign a specific user
meta ads dataset assign-user <PIXEL_ID> --user-id <USER_ID>

# Grant specific permissions
meta ads dataset assign-user <PIXEL_ID> --tasks ADVERTISE --tasks ANALYZE --tasks EDIT
```

Tareas disponibles: `ADVERTISE`, `ANALYZE`, `EDIT` y `UPLOAD`.

### Usar conjuntos de datos en campañas publicitarias

Cuando un conjunto de datos esté conectado a tu cuenta publicitaria, haz referencia a él al crear conjuntos de anuncios y anuncios de conversión:

```bash
# Create a conversion ad set with Pixel tracking
meta ads adset create <CAMPAIGN_ID> --name "Purchase Conversions" \
  --optimization-goal OFFSITE_CONVERSIONS \
  --billing-event IMPRESSIONS \
  --pixel-id <PIXEL_ID> \
  --custom-event-type PURCHASE \
  --targeting-countries US

# Create an ad with Pixel tracking
meta ads ad create <AD_SET_ID> --name "Conversion Ad" \
  --creative-id <CREATIVE_ID> --pixel-id <PIXEL_ID>
```

## Catálogos de productos

Los catálogos de productos contienen el inventario de productos para los anuncios de catálogo de Advantage+, las compras y el comercio. Los catálogos pertenecen a una empresa.

### Enumerar los catálogos

```bash
meta ads catalog list
meta ads catalog list --business-id <BUSINESS_ID>
meta ads catalog list --limit 50
```

### Crear un catálogo

```bash
meta ads catalog create --name "My Product Catalog"
meta ads catalog create --name "Hotel Inventory" --vertical hotels
```

La opción `--vertical` especifica el tipo de productos del catálogo.

**Sectores disponibles:** `adoptable_pets`, `commerce` (predeterminado), `destinations`, `flights`, `generic`, `home_listings`, `hotels`, `local_service_businesses`, `offer_items`, `offline_commerce`, `transactable_items` y `vehicles`.

### Ver detalles del catálogo

```bash
meta ads catalog get <CATALOG_ID>
meta --output json ads catalog get <CATALOG_ID>
```

### Actualizar un catálogo

```bash
meta ads catalog update <CATALOG_ID> --name "Renamed Catalog"
```

### Eliminar un catálogo

```bash
meta ads catalog delete <CATALOG_ID>
meta ads catalog delete <CATALOG_ID> --force
```

**Nota:** No puedes eliminar catálogos que tengan listas de productos activas o anuncios que hagan referencia a ellos.

## Proceso completo: configurar el seguimiento de las conversiones

```bash
# 1. Create a dataset (Pixel)
meta ads dataset create --name "Website Pixel"
# Output: Created dataset 'Website Pixel' (ID: <PIXEL_ID>)

# 2. Connect the dataset to your ad account
meta ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>

# 3. (Optional) Connect to a catalog for product-level tracking
meta ads dataset connect <PIXEL_ID> --catalog-id <CATALOG_ID>

# 4. Create a conversion ad campaign
meta ads campaign create --name "Sales Campaign" --objective OUTCOME_SALES

# 5. Create a conversion ad set using a Pixel
meta ads adset create <CAMPAIGN_ID> --name "Purchase Optimization" \
  --optimization-goal OFFSITE_CONVERSIONS \
  --billing-event IMPRESSIONS \
  --pixel-id <PIXEL_ID> \
  --custom-event-type PURCHASE \
  --targeting-countries US

# 6. Create an ad creative and ad
meta ads creative create --name "Product Ad" --page-id <PAGE_ID> \
  --image ./product.jpg --body "Buy now!" \
  --link-url https://example.com --call-to-action SHOP_NOW

meta ads ad create <AD_SET_ID> --name "Product Ad" \
  --creative-id <CREATIVE_ID> --pixel-id <PIXEL_ID>
```
