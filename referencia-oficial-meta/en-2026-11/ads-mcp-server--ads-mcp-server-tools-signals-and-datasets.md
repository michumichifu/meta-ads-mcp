<!-- Fuente oficial: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-mcp-server/ads-mcp-server-tools-signals-and-datasets.md · descargada el 2026-11-03, sin cambios -->

# Signals and datasets


Manage your conversion tracking datasets (Meta Pixels), Conversions API events, and pixel schema. Access signal health and quality information to prioritize where to invest in your signals setup.

## Datasets

| Tool | Description |
|---|---|
| `ads_get_datasets` | List datasets (pixels or applications) on a business or ad account. |
| `ads_get_dataset_details` | Get metadata and configuration for a dataset, including Conversions API setup. |
| `ads_get_dataset_stats` | Report event volume received by a dataset over a configurable window (up to 28 days), with optional breakdown by device or event source. |
| `ads_get_dataset_quality` | Report Event Match Quality (EMQ) scores, per-match-key coverage, and data freshness by channel. |

## Event Setup Tool
### Pixel events

Configure additional events (for example, Purchase, AddToCart, Lead) for your pixel based on website interactions

| Tool | Description |
|---|---|
| `ads_pixel_event_read` | List event rules on a pixel, or fetch a single rule by ID. |
| `ads_pixel_event_create` | Create a new event rule on a pixel (starts inactive). |
| `ads_pixel_event_update` | Activate or deactivate an existing event rule. |
| `ads_pixel_event_delete` | Delete an event rule. |

### Pixel parameters

Configure which parameters (for example, `value`, `currency`, `content_ids`) an event extracts.

| Tool | Description |
|---|---|
| `ads_pixel_parameter_read` | List parameters on a pixel, or fetch a single parameter by ID. |
| `ads_pixel_parameter_create` | Create a new parameter extractor. |
| `ads_pixel_parameter_update` | Update an existing parameter. |
| `ads_pixel_parameter_delete` | Delete a parameter. |

## Custom conversions

| Tool | Description |
|---|---|
| `ads_get_customconversions` | List custom conversions defined on an ad account. |


---

Full documentation index for this product: https://developers.facebook.com/documentation/ads-commerce/llms.txt
