<!-- Fuente oficial: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-mcp-server/ads-mcp-server-tools-ad-creation-and-management.md · descargada el 2026-11-03, sin cambios -->

# Ad creation and management


Create and edit ads, ad sets, and campaigns end-to-end — from looking up which accounts and Pages you have access to, through building campaign structures, uploading creatives, targeting audiences, and boosting Instagram content. Write tools create entities in a paused state; your AI client asks for confirmation before activation.

## Account and Page discovery

| Tool | Description |
|---|---|
| `ads_get_ad_accounts` | List the ad accounts the user has access to (name, status, currency, timezone). |
| `ads_get_ad_account_pages` | List Facebook Pages that have been used for ads on a specific ad account. |
| `ads_get_pages_for_business` | List Facebook Pages owned by a business. |
| `ads_get_user_pages` | List all Facebook Pages the user can use for advertising, across their businesses. |
| `ads_get_field_context` | Look up metadata for ad fields (type, filterability, enum values, supported entity levels) before calling `ads_get_ad_entities`. |

## Campaign, ad set, and ad lifecycle

| Tool | Description |
|---|---|
| `ads_create_campaign` | Create a new ad campaign (paused). |
| `ads_create_ad_set` | Create an ad set with targeting, budget, and optimization goal under a campaign (paused). |
| `ads_create_ad` | Create an ad with a creative under an ad set (paused). |
| `ads_update_entity` | Update fields on an existing campaign, ad set, or ad. |
| `ads_activate_entity` | Change a paused campaign, ad set, or ad to active — starts spending budget. |

## Creatives and media

| Tool | Description |
|---|---|
| `ads_create_creative` | Create a single-image link ad creative. Also supports placement customization or multi-media optimization using images, videos, or both (limited availability). |
| `ads_get_creatives` | List ad creatives in an ad account. |
| `ads_get_creative_ads` | List the ads that use a specific creative. |
| `ads_get_ad_images` | List ad images uploaded to an ad account. |
| `ads_get_ad_videos` | List ad videos uploaded to an ad account. |
| `ads_get_ad_preview` | Render a preview of an ad or creative in a specific placement (Feed, Story, Reels, and so on). |

## Query and discovery

| Tool | Description |
|---|---|
| `ads_get_ad_entities` | Search and retrieve campaigns, ad sets, and ads (also the primary reporting tool — see [Comprehensive reporting](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-mcp-server/ads-mcp-server-tools-comprehensive-reporting)). |
| `ads_library_search` | Search the public Meta Ad Library for competitive research and creative inspiration. |

## Instagram

| Tool | Description |
|---|---|
| `ads_get_ig_accounts` | List Instagram Business or Creator Accounts linked to an ad account. |
| `ads_get_ig_media` | List recent media (posts, reels, stories) on an Instagram Business or Creator Account, filtered to boost-eligible items. |
| `ads_boost_ig_post` | Turn an existing organic Instagram post into a boosted ad. |

## Custom audiences

| Tool | Description |
|---|---|
| `ads_get_ad_account_custom_audiences` | List custom audiences on an ad account (optionally filtered by subtype). |
| `ads_get_custom_audience` | Get details for a single custom audience (size, status, subtype). |
| `ads_get_custom_audience_adsets` | List the ad sets currently targeting a given custom audience. |
| `ads_create_custom_audience` | Create a new custom audience (customer file, website, lookalike, app, engagement, or offline conversion). |
| `ads_update_custom_audience` | Update the name, description, rule, or labels on an existing custom audience. |
| `ads_update_custom_audience_users` | Add or remove users on a customer list audience by uploading hashed personal data. |
| `ads_delete_custom_audience` | Delete a custom audience. |


---

Full documentation index for this product: https://developers.facebook.com/documentation/ads-commerce/llms.txt
