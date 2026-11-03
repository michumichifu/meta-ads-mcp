<!-- Fuente oficial: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-mcp-server/ads-mcp-server-tools-catalog-creation-and-management.md · descargada el 2026-11-03, sin cambios -->

# Catalog creation and management


Create product catalogs, upload and edit product data, and troubleshoot data feed and item visibility issues for Advantage+ catalog ads.

## Catalogs

| Tool | Description |
|---|---|
| `ads_catalog_create` | Create a new product catalog and optionally upload initial product data via a feed URL, inline items, or file upload. |
| `ads_catalog_update_catalog` | Update settings on an existing catalog (for example, rename it). |
| `ads_catalog_list_catalogs` | List accessible catalogs, list catalogs for a business or ad account, or retrieve one catalog with its counts. |
| `ads_catalog_list_dpa_eligible_catalogs` | Find and rank catalogs eligible for dynamic ads or Advantage+ catalog ads for an ad account, and return a recommended catalog when available. Supports optional landing-page URL, pixel, Page, and ad-format context, plus optional catalog health signals. |
| `ads_catalog_get_diagnostics` | List issues on a catalog (errors, warnings, opportunities) that may affect ad delivery. |
| `ads_catalog_get_dynamic_ads_health` | Check whether a catalog (or product set) is healthy enough to serve Advantage+ catalog ads. |
| `ads_catalog_get_data_sources` | List all data sources associated with a catalog (feed URLs, uploads, API sources). |

## Products

| Tool | Description |
|---|---|
| `ads_catalog_list_products` | List products in a catalog or product set, search with a structured filter, or retrieve one product. |
| `ads_catalog_product_create` | Create a product in a catalog. |
| `ads_catalog_update_product` | Update a product's fields (name, price, availability, brand, and so on). |
| `ads_catalog_delete_product` | Delete a product from a catalog. |

## Product sets

| Tool | Description |
|---|---|
| `ads_catalog_get_suggested_filter` | Convert a natural-language product selection, such as "red dresses under $50 that are in stock," into a catalog-specific product-set filter. Use the returned filter to list matching products or create or update a product set. This tool only generates the filter; it does not retrieve products or modify the catalog. |
| `ads_catalog_create_product_set` | Create a dynamic product set from a filter rule. |
| `ads_catalog_update_product_set` | Update a product set's name, filter, or retailer ID. |
| `ads_catalog_list_product_sets` | List product sets in a catalog, retrieve one product set, or list sets that contain a product. |
| `ads_catalog_product_set_delete` | Delete a product set and its filter rule. Products themselves are not deleted; only the set that groups them. |

## Product feeds

| Tool | Description |
|---|---|
| `ads_catalog_create_product_feed` | Register a new product feed on a catalog (optionally scheduled to fetch from a URL). |
| `ads_catalog_update_product_feed` | Update a feed's name, currency defaults, parsing options, or schedules. |
| `ads_catalog_list_product_feeds` | List feeds in a catalog or retrieve one feed with its configuration, schedule, and status. |
| `ads_catalog_create_product_feed_upload_session` | Start a new upload session for a feed (used for chunked uploads). |
| `ads_catalog_get_product_feed_upload_sessions` | List upload sessions for a feed, including status and error counts. |
| `ads_catalog_product_feed_delete` | Delete a product feed and its schedule. Products ingested by the feed are also removed asynchronously; products also supplied by another data source are kept. |

## Feed rules

| Tool | Description |
|---|---|
| `ads_catalog_create_feed_rule` | Create a supplementary rule that maps or transforms an attribute on a feed. |
| `ads_catalog_get_feed_rules` | List the supplementary rules applied to a feed. |
| `ads_catalog_product_feed_delete_rule` | Delete a transformation rule from a product feed. The feed and its products are not affected. |

## Event sources

| Tool | Description |
|---|---|
| `ads_catalog_event_source_get` | List the event sources (pixels, apps, or offline conversion data sets) connected to a catalog. |
| `ads_catalog_event_source_get_catalogs` | Get the catalogs connected to a given event source (pixel, app, or offline conversion data set). |
| `ads_catalog_event_source_get_health` | Report the match rate and setup issues for a catalog's connected event sources, including 28-day historical match rate stats. |
| `ads_catalog_event_source_get_recommendations` | Recommend event sources to connect to a catalog with weak or missing signal coverage, based on which sources' conversion events best match the catalog's products. |
| `ads_catalog_event_source_connect` | Connect an event source (pixel, app, or offline conversion data set) to a catalog so its signals can be matched against catalog products for dynamic ads and Advantage+ catalog ads. |
| `ads_catalog_event_source_disconnect` | Disconnect an event source from a catalog, so its signals are no longer matched against catalog products. |


---

Full documentation index for this product: https://developers.facebook.com/documentation/ads-commerce/llms.txt
