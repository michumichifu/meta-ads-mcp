# Insights — CLI de anuncios de Meta

> Fuente: Documentación oficial de Meta para desarrolladores — Ads CLI.
> URL original (inglés): https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/insights
> Actualizado: 29 abr. 2026

En esta guía se explica cómo consultar los datos sobre el rendimiento de los anuncios con la CLI de anuncios, incluidos los intervalos de fechas, los desgloses, las métricas personalizadas y la ordenación.

## Opciones de la consulta

### Uso básico

```bash
# Account-level metrics for the last 30 days (default)
meta ads insights get

# Specific metrics
meta ads insights get --adset-id <AD_SET_ID> --fields spend,impressions,ctr,cpc
```

### Intervalos de fechas

#### Intervalos predefinidos

```bash
meta ads insights get --date-preset last_30d
meta ads insights get --date-preset yesterday
```

Valores predefinidos disponibles: `today`, `yesterday`, `last_3d`, `last_7d`, `last_14d`, `last_30d`, `last_90d`, `this_month`, `last_month` (igual que la API de marketing).

#### Intervalo de fechas personalizado

```bash
meta ads insights get --since 2024-01-01 --until 2024-01-31
```

**Nota:** `--since` y `--until` deben utilizarse juntos y anulan `--date-preset`.

### Granularidad de tiempo

Controla cómo se desglosan los resultados a lo largo del tiempo con `--time-increment`:

| Valor | Descripción |
|---|---|
| `all_days` (predeterminado) | Agrega los valores de todo el intervalo de fechas. |
| `daily` | Una fila por día. |
| `weekly` | Una fila por semana. |
| `monthly` | Una fila por mes. |

```bash
# Daily spend for the last 30 days
meta ads insights get --date-preset last_30d --time-increment daily --fields spend

# Weekly campaign performance
meta ads insights get --campaign-id <CAMPAIGN_ID> --time-increment weekly
```

### Desgloses

Añade dimensiones de ubicación o datos demográficos con `--breakdown` (repetible):

| Desglose | Descripción |
|---|---|
| `age` | Intervalos de edad |
| `gender` | Género |
| `country` | País |
| `publisher_platform` | Facebook, Instagram, Audience Network |
| `device_platform` | Móvil y ordenador |
| `platform_position` | Feed, historias y reels |
| `impression_device` | Tipo de dispositivo |

```bash
# Performance by age and gender
meta ads insights get --breakdown age --breakdown gender

# Platform comparison
meta ads insights get --breakdown publisher_platform --fields spend,impressions,ctr
```

### Filtración

Filtra los resultados para una campaña publicitaria, un conjunto de anuncios o un anuncio específicos:

```bash
meta ads insights get --campaign-id <CAMPAIGN_ID>
meta ads insights get --adset-id <AD_SET_ID>
meta ads insights get --ad-id <AD_ID>
```

### Ordenación

Ordena los resultados según una métrica:

```bash
meta ads insights get --adset-id <AD_SET_ID> --sort spend_descending
meta ads insights get --adset-id <AD_SET_ID> --sort impressions_ascending

# Last 30 days, ad campaign level, sorted by spend
meta ads insights get --campaign-id <CAMPAIGN_ID> --date-preset last_30d --sort spend_descending
```

El formato es `<metric>_ascending` o `<metric>_descending`.

### Campos personalizados

Especifica qué métricas quieres incluir con `--fields` (separadas por comas):

```bash
meta ads insights get --fields spend,impressions,clicks,ctr,cpc,reach
meta ads insights get --fields spend,conversions,cost_per_conversion,purchase_roas
```

Campos predeterminados: `spend`, `impressions`, `clicks`, `ctr`, `cpc`, `reach`.

Se puede usar cualquier campo válido de la API de insights de Meta. Estos son algunos de los campos comunes:

| Campo | Descripción |
|---|---|
| `spend` | Importe total gastado |
| `impressions` | Número de veces que se han mostrado los anuncios. |
| `reach` | Número de usuarios únicos que han visto el anuncio. |
| `clicks` | Clics totales |
| `ctr` | Tasa de clics |
| `cpc` | Coste por clic |
| `cpm` | Coste por 1000 impresiones |
| `frequency` | Número medio de veces que cada persona ha visto el anuncio. |
| `conversions` | Número de conversiones. |
| `cost_per_conversion` | Coste por conversión |
| `purchase_roas` | Retorno del gasto publicitario |

### Límite de resultados

```bash
meta ads insights get --campaign-id <CAMPAIGN_ID> --limit 100    # Default: 50
```

## Referencia completa de opciones

| Opción | Valor predeterminado | Descripción |
|---|---|---|
| `--date-preset` | `last_30d` | Intervalo de fechas predefinido |
| `--since` | | Fecha de inicio (AAAA-MM-DD). Requiere `--until`. |
| `--until` | | Fecha de finalización (AAAA-MM-DD). Requiere `--since`. |
| `--time-increment` | `all_days` | Granularidad de tiempo: `all_days`, `daily`, `weekly`, `monthly`. |
| `--breakdown` | | Dimensión del desglose (repetible) |
| `--fields` | `spend,impressions,...` | Lista separada por comas de métricas |
| `--campaign-id` | | Aplica un filtro a una campaña publicitaria. |
| `--adset-id` | | Filtro para un conjunto de anuncios |
| `--ad-id` | | Filtrar por anuncio |
| `--sort` | | Orden (por ejemplo, `spend_descending`) |
| `--limit` / `-l` | 50 | Número máximo de filas que se devolverán. |
