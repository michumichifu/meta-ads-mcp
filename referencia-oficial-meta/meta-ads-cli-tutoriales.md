# Tutoriales y recetas — CLI de anuncios de Meta

> Fuente: Documentación oficial de Meta para desarrolladores — Ads CLI.
> URL original (inglés): https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/tutorials-and-recipes
> Actualizado: 29 abr. 2026

Aprende a buscar los identificadores que necesitas, crear tu primera campaña publicitaria completa, automatizar los flujos de trabajo con scripts y borrar los recursos.

- Buscar los identificadores
- Tu primera campaña publicitaria (de extremo a extremo)
- Scripts y automatización
- Limpieza: eliminar recursos

## Buscar los identificadores

### Identificador de la cuenta publicitaria

```bash
meta ads adaccount list
```

Busca la columna `id`. Establécelo con:

```bash
export AD_ACCOUNT_ID=<AD_ACCOUNT_ID>
```

### Identificador de página de empresa de Facebook

Usa el identificador de página de empresa cuando crees contenido de anuncios con la marca `--page-id`. También puedes encontrar los identificadores de página de empresa en la configuración de la empresa. Solo pueden ser páginas asociadas a una empresa a la que el usuario del sistema tenga acceso.

```bash
meta ads page list
```

### Identificador de la empresa

La CLI de anuncios normalmente resuelve el identificador de empresa automáticamente a partir de la cuenta publicitaria. Si es necesario, establécelo explícitamente:

```bash
export BUSINESS_ID=<BUSINESS_ID>
```

## Tu primera campaña publicitaria (de extremo a extremo)

En este tutorial se crea una configuración de anuncios completa: campaña publicitaria, conjunto de anuncios, contenido del anuncio y anuncio.

### 1. Autenticarse y configurar

Genera un identificador de usuario del sistema (consulta la guía "Introducción") y, a continuación, configúralo:

```bash
# Set your access token
export ACCESS_TOKEN=<ACCESS_TOKEN>

# Find your ad account
meta ads adaccount list

# Set it for the session
export AD_ACCOUNT_ID=<AD_ACCOUNT_ID>
```

### 2. Encontrar el identificador de la página de empresa

El contenido de cada anuncio necesita una página de empresa como identidad:

```bash
meta ads page list
```

Guarda el valor de `id` devuelto para la página de empresa que quieras usar.

### 3. Crear una campaña publicitaria

```bash
meta ads campaign create \
  --name "My First Campaign" \
  --objective OUTCOME_TRAFFIC \
  --daily-budget 5000
```

Guarda el identificador de la campaña publicitaria devuelto para crear el conjunto de anuncios.

### 4. Crear un conjunto de anuncios

```bash
meta ads adset create <CAMPAIGN_ID> \
  --name "US Audience" \
  --optimization-goal LINK_CLICKS \
  --billing-event IMPRESSIONS \
  --targeting-countries US
```

Aquí puedes omitir el presupuesto porque la campaña publicitaria ya tiene un presupuesto diario (optimización del presupuesto de la campaña). Guarda el identificador del conjunto de anuncios devuelto para crear el anuncio.

### 5. Creación del contenido de un anuncio

```bash
meta ads creative create \
  --name "Hero Banner" \
  --image ./banner.jpg \
  --page-id <PAGE_ID> \
  --body "Check out our latest deals!" \
  --title "Shop Now" \
  --link-url https://example.com/sale \
  --call-to-action SHOP_NOW
```

Guarda el identificador de contenido del anuncio devuelto para crear el anuncio.

### 6. Crear un anuncio

```bash
meta ads ad create <AD_SET_ID> \
  --name "Hero Banner Ad" \
  --creative-id <CREATIVE_ID>
```

### 7. Activar el anuncio

La CLI de anuncios crea todo con el estado `PAUSED` de forma predeterminada. Cuando esté todo listo para la publicación, ejecuta lo siguiente:

```bash
meta ads campaign update <CAMPAIGN_ID> --status ACTIVE
meta ads adset update <AD_SET_ID> --status ACTIVE
meta ads ad update <AD_ID> --status ACTIVE
```

## Scripts y automatización

La CLI de anuncios admite el uso de scripts con `--no-input`, `--force` y la salida estructurada.

### Configuración basada en el entorno

Para más información sobre variables de entorno y prioridad de configuración, consulta la guía "Configuración".

```bash
# CI/CD pipeline example
export ACCESS_TOKEN="$META_SYSTEM_USER_TOKEN"
export AD_ACCOUNT_ID=<AD_ACCOUNT_ID>
export BUSINESS_ID=<BUSINESS_ID>  # Inferred from ad account if missing

meta --output json ads campaign list
```

### Modo no interactivo

Usa `--no-input` y `--force` para suprimir todas las invitaciones a compartir:

```bash
meta --no-input ads campaign delete <CAMPAIGN_ID> --force
```

### Salida JSON para scripts

```bash
# Retrieve ad campaign IDs
CAMPAIGN_IDS=$(meta --output json ads campaign list | jq -r '.[].id')

# Iterate over ad campaigns
for id in $CAMPAIGN_IDS; do
  echo "Processing ad campaign $id"
  meta ads insights get --campaign-id "$id" --fields conversions,purchase_roas
done
```

### Comprobar los códigos de salida

```bash
meta ads campaign list
if [ $? -eq 3 ]; then
  echo "Not authenticated -- add token to .env"
fi
```

| Código de salida | Significado |
|---|---|
| 0 | Correcto |
| 1 | Error general |
| 2 | Error de uso o argumentos. |
| 3 | Error de autenticación. |
| 4 | Error de la API |
| 5 | Recurso no encontrado |

## Limpieza: eliminar los recursos

### Eliminar recursos

La eliminación de una campaña publicitaria se propaga de forma automática a sus conjuntos de anuncios y anuncios dependientes. Si eliminas un conjunto de anuncios, también se eliminarán los anuncios dependientes. También puedes eliminar recursos individuales directamente.

```bash
# Delete a single ad
meta ads ad delete <AD_ID> --force

# Delete an ad set (and its ads)
meta ads adset delete <AD_SET_ID> --force

# Delete an ad campaign (and all its ad sets and ads)
meta ads campaign delete <CAMPAIGN_ID> --force

# Delete an ad creative
meta ads creative delete <CREATIVE_ID> --force
```

### Desconectar un conjunto de datos de una cuenta publicitaria

```bash
meta ads dataset disconnect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --force
```

### Eliminar un catálogo

```bash
meta ads catalog delete <CATALOG_ID> --force
```
