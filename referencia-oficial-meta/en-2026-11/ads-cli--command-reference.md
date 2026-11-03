<!-- Fuente oficial: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/command-reference.md · descargada el 2026-11-03, sin cambios -->

# Command Reference


Complete reference for all Ads CLI commands.

- [Global option flags](#global-option-flags)
- [Authentication](#authentication)
- [Ad accounts](#ad-accounts)
- [Facebook Business Pages](#facebook-business-pages)
- [Ad campaigns](#ad-campaigns)
- [Ad sets](#ad-sets)
- [Ads](#ads)
- [Ad creatives](#ad-creatives)
- [Datasets (Meta Pixels)](#datasets-meta-pixels)
- [Product catalogs](#product-catalogs)
- [Product items](#product-items)
- [Product sets](#product-sets)
- [Product feeds](#product-feeds)
- [Guidance](#guidance)
- [Ad studies](#ad-studies)
- [Insights](#insights)

### Command syntax

- Ads commands follow the pattern `meta ads <resource> <action> [options]`
- Auth commands use `meta auth <action>`
- `<lowercase>` = fixed keyword (type exactly as shown)
- `<UPPER_CASE>` = placeholder (replace with your value)
- `[lowercase]` = optional keyword or group
- `--flag` = named option or parameter
- All `list` and `get` commands accept `--fields name,other,...` to fetch any Marketing API fields beyond the defaults (names are passed straight to the API), for example `meta ads campaign get <CAMPAIGN_ID> --fields name,objective,spend_cap,budget_remaining`
- `create` commands take their parent as a flag: `--campaign-id` on `adset create`, `--adset-id` on `ad create`, `--catalog-id` on `product-set create`, `product-item create`, and `product-feed` commands

---

## Global options

These options apply to all commands and must be placed before the subcommand when used:

```
meta [global options] ads <command> <subcommand> [options]
```

| Option | Short | Description |
|--------|-------|-------------|
| `--output <format>` | `-o` | Output format: `table` (default), `json`, `plain` |
| `--no-color` | | Disable colored output |
| `--no-input` | | Disable interactive prompts |
| `--debug` | | Enable debug output |
| `--help` | `-h` | Show help text |
| `--version` | `-v` | Show version |

#### Ads group options

These options are set on the `meta ads` group (after `ads`, before the resource) and apply to every ads command. They can also be set with environment variables:

```
meta ads --ad-account-id <AD_ACCOUNT_ID> campaign list
meta ads --business-id <BUSINESS_ID> dataset create --name "My Pixel"
```

| Option | Env var | Description |
|--------|---------|-------------|
| `--ad-account-id` | `AD_ACCOUNT_ID` | Meta ad account ID (with or without the `act_` prefix) |
| `--business-id` | `BUSINESS_ID` | Meta business ID. Used by dataset, catalog, and page commands; resolved from the ad account when omitted |

#### Output formats

- **table** (default) -- Human-readable table with headers. Best for interactive use.
- **json** -- Structured JSON array. Ideal for piping to `jq` or other tools.
- **plain** -- Tab-separated values, one record per line. Works well with `cut`, `awk`, and `sort`.

```
# JSON output, pipe to jq
meta --output json ads campaign list | jq '.[].name'

# Plain output, sort by column
meta --output plain ads campaign list | sort -t$'\t' -k5 -rn
```

---

## Commands

### Authentication

Manage authentication with Meta.

Ads CLI authenticates using a Meta admin system user access token. See [Get Started](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/get-started) for detailed setup instructions.

#### meta auth

Save your system user access token. Set the token in your `.env` file or as an environment variable:

```
# .env file
ACCESS_TOKEN=<ACCESS_TOKEN>

# Or export directly
export ACCESS_TOKEN=<ACCESS_TOKEN>
```

#### meta auth status

Check current authentication status. Shows a masked version of the stored token.

```
meta auth status
```

---

### Ad accounts

Manage ad accounts.

#### meta ads adaccount list

List all ad accounts the authenticated user has access to.

```
meta ads adaccount list
meta ads adaccount list --limit 50
meta --output json ads adaccount list
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 25 | Maximum number of accounts to return |

**Output columns:** `id`, `name`, `account_status`, `currency`, `timezone_name`

#### meta ads adaccount get

Get details for a specific ad account. If the ID is omitted, uses the currently configured ad account (`--ad-account-id` or `AD_ACCOUNT_ID`).

```
meta ads adaccount get act_123456
meta ads adaccount get                       # uses configured ad account
meta --output json ads adaccount get act_123456
```

The `act_` prefix is optional — `meta ads adaccount get 123456` works the same.

#### meta ads adaccount current

Show the currently configured ad account ID (from `--ad-account-id` or `AD_ACCOUNT_ID`).

```
meta ads adaccount current
```

---

### Facebook Business Pages

Manage Facebook Business Pages.

#### meta ads page list

List all Facebook Business Pages you have access to. Use Business Page IDs from this list with `--page-id` in ad creative commands.

```
meta ads page list
meta ads page list --limit 50
meta --output json ads page list
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 25 | Maximum number of Business Pages to return |

**Output columns:** `id`, `name`, `category`

#### meta ads page get

Get details for a specific Facebook Business Page.

```
meta ads page get <PAGE_ID>
meta --output json ads page get <PAGE_ID>
```

---

### Ad campaigns

Manage ad campaigns.

#### meta ads campaign list

List ad campaigns in the ad account.

```
meta ads campaign list
meta ads campaign list --limit 25
meta --output json ads campaign list
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 10 | Maximum number of ad campaigns to return |

**Output columns:** `name`, `effective_status`, `id`, `objective`, `daily_budget`, `lifetime_budget`, `start_time`

#### meta ads campaign create

Create an ad campaign.

```
meta ads campaign create --name "My Campaign" --objective OUTCOME_TRAFFIC
meta ads campaign create --name "Sales Campaign" --objective OUTCOME_SALES \
  --daily-budget 5000
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `--name` | Yes | | Name for the ad campaign |
| `--objective` | Yes | | Ad campaign objective |
| `--daily-budget` | No | | Daily budget in cents (for example, 5000 = $50.00). Mutually exclusive with `--lifetime-budget` and with any ad set budget |
| `--lifetime-budget` | No | | Lifetime budget in cents. Mutually exclusive with `--daily-budget` and with any ad set budget |
| `--status` | No | `PAUSED` | Initial status: `ACTIVE` or `PAUSED` |
| `--bid-strategy` | No | `LOWEST_COST_WITHOUT_CAP` (CBO/flex) | Campaign-level bid strategy for CBO or ad set budget sharing campaigns. Omit for plain ABO — set `--bid-strategy` on the ad set instead |
| `--buying-type` | No | | Buying type: `AUCTION` (default) or `RESERVED`. `RESERVED` does not support `--adset-budget-sharing` |
| `--adset-budget-sharing` | No | | Ad set budget sharing (flex): budget stays on the ad sets. Requires no campaign budget and a daily ad set budget |
| `--pacing-type` | No | | Budget delivery pacing (`standard`, `no_pacing`). Campaign-level pacing requires a campaign budget (CBO); `no_pacing` requires `--bid-strategy LOWEST_COST_WITH_BID_CAP` |
| `--special-ad-categories` | No | | Regulated categories (repeatable): `HOUSING`, `EMPLOYMENT`, `CREDIT`, `FINANCIAL_PRODUCTS_SERVICES`, `ISSUES_ELECTIONS_POLITICS`, `ONLINE_GAMBLING_AND_GAMING`. Omit when none apply |
| `--spend-cap` | No | | Lifetime spending limit for the whole campaign, in cents |
| `--start-time` | No | | Campaign start time (ISO 8601) |
| `--stop-time` | No | | Campaign stop time (ISO 8601) |
| `--execution-options` | No | | `validate_only` validates without creating anything (dry run); `include_recommendations` returns delivery recommendations |
| `--skadnetwork-attribution` | No | | Enable SKAdNetwork attribution (iOS 14+ app promotion campaigns) |
| `--smart-promotion-type` | No | | Smart promotion type for guided campaign flows (`GUIDED_CREATION`) |

**Objectives:** `OUTCOME_APP_PROMOTION`, `OUTCOME_AWARENESS`, `OUTCOME_ENGAGEMENT`, `OUTCOME_LEADS`, `OUTCOME_SALES`, `OUTCOME_TRAFFIC`

**Budget modes (pick one):** CBO sets `--daily-budget` or `--lifetime-budget` on the campaign and no budget on the ad sets; ABO sets the budget on each ad set instead; flex passes `--adset-budget-sharing` with no campaign budget and a daily ad set budget.

#### meta ads campaign get

Get details for a specific ad campaign.

```
meta ads campaign get <CAMPAIGN_ID>
```

#### meta ads campaign delete

Delete an ad campaign and all its child ad sets and ads.

```
meta ads campaign delete <CAMPAIGN_ID>
meta ads campaign delete <CAMPAIGN_ID> --force    # Skip confirmation
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

#### meta ads campaign update

Update an ad campaign.

```
meta ads campaign update <CAMPAIGN_ID> --name "New Name"
meta ads campaign update <CAMPAIGN_ID> --status ACTIVE
meta ads campaign update <CAMPAIGN_ID> --daily-budget 10000
```

| Option | Description |
|--------|-------------|
| `--name` | New name for the ad campaign |
| `--status` | New status: `ACTIVE`, `PAUSED`, `ARCHIVED` |
| `--daily-budget` | New daily budget in cents. Mutually exclusive with `--lifetime-budget` |
| `--lifetime-budget` | New lifetime budget in cents. Mutually exclusive with `--daily-budget` |
| `--bid-strategy` | New bid strategy for the campaign |
| `--adset-budget-sharing` | Turn ad set budget sharing (flex) on or off. Requires no campaign budget |
| `--pacing-type` | Budget delivery pacing (`standard`, `no_pacing`; `no_pacing` needs `--bid-strategy LOWEST_COST_WITH_BID_CAP`). Requires a campaign budget |
| `--spend-cap` | New lifetime spending limit for the whole campaign, in cents |
| `--start-time` | New start time (ISO 8601) |
| `--stop-time` | New stop time (ISO 8601) |
| `--execution-options` | `validate_only` validates the update without applying it (dry run); `include_recommendations` returns delivery recommendations |

**Note:** At least one option must be provided. Only the flags you pass are sent; everything else is left alone.

---

### Ad sets

Manage ad sets.

#### meta ads adset list

List ad sets. Optionally filter by ad campaign.

```
meta ads adset list                    # All ad sets in the ad account
meta ads adset list <CAMPAIGN_ID>        # Ad sets for a specific campaign
meta ads adset list --limit 25
```

| Argument | Required | Description |
|----------|----------|-------------|
| `<CAMPAIGN_ID>` | No | Filter to ad sets in this campaign |

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 10 | Maximum number of ad sets to return |

**Output columns:** `name`, `effective_status`, `id`, `optimization_goal`, `billing_event`, `daily_budget`, `lifetime_budget`, `start_time`

#### meta ads adset create

Create an ad set for an ad campaign.

```
meta ads adset create --campaign-id <CAMPAIGN_ID> --name "My Ad Set" \
  --optimization-goal LINK_CLICKS --billing-event IMPRESSIONS \
  --bid-amount 500 --targeting-countries US

# Conversion ad set (CBO campaign: no budget here)
meta ads adset create --campaign-id <CAMPAIGN_ID> --name "Conversions Set" \
  --optimization-goal OFFSITE_CONVERSIONS \
  --billing-event IMPRESSIONS --pixel-id <PIXEL_ID> \
  --custom-event-type PURCHASE --targeting-countries US
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `--campaign-id` | Yes | | ID of the campaign that will own this ad set |
| `--name` | Yes | | Name for the ad set |
| `--optimization-goal` | Yes | | Optimization goal |
| `--billing-event` | Yes | | Billing event |
| `--daily-budget` | No | | Daily budget in cents. Set for ABO or ad set budget sharing campaigns; omit for CBO. Mutually exclusive with `--lifetime-budget` |
| `--lifetime-budget` | No | | Lifetime budget in cents (ABO only, not CBO or budget sharing). Requires `--end-time` |
| `--bid-amount` | No | | Bid amount in cents. Required by `COST_CAP` and bid-cap bid strategies |
| `--bid-strategy` | No | | Ad set bid strategy (ABO and flex ad sets; CBO sets it on the campaign) |
| `--start-time` | No | | Start time (ISO 8601) |
| `--end-time` | No | | End time (ISO 8601). Required with `--lifetime-budget`; also required on each ad set when the parent campaign uses a lifetime budget |
| `--status` | No | `PAUSED` | Initial status: `ACTIVE` or `PAUSED` |
| `--pacing-type` | No | | Budget delivery pacing (`standard`, `no_pacing`, `day_parting`; `day_parting` requires an ad set schedule) |
| `--targeting-countries` | No | | Comma-separated country codes (for example, `US,CA,GB`) |
| `--targeting` | No | | Raw targeting spec as JSON, or `@path/to/file.json` (age, genders, geo_locations, interests, custom audiences, placements). Replaces `--targeting-countries` |
| `--advantage-audience` | No | | Advantage+ audience automation. Opt in to let Meta treat detailed targeting as signals rather than hard limits |
| `--pixel-id` | No | | Dataset (Pixel) ID for conversion tracking. Sets the promoted object |
| `--custom-event-type` | No | `PURCHASE` (with `--pixel-id`) | Conversion event type. Used with `--pixel-id` |
| `--promoted-object` | No | | Raw promoted object as JSON, or `@path/to/file.json` (for example, `application_id` for app installs, `product_set_id` for catalog ads). Replaces `--pixel-id` and `--custom-event-type` |
| `--attribution-spec` | No | | Attribution window as raw JSON. Do not combine with `--incremental-attribution` |
| `--incremental-attribution` | No | | Optimize delivery for incremental conversions. Do not also pass `--attribution-spec` |
| `--destination-type` | No | | Where conversions happen. Required for some objectives such as messaging |
| `--dynamic-creative` | No | | Enable Dynamic Creative on the ad set. Required to run a DCO creative |
| `--dsa-beneficiary` | No | | EU DSA: person or organization that benefits from the ads. Required for EU delivery |
| `--dsa-payor` | No | | EU DSA: person or organization that pays for the ads. Required for EU delivery |

**Optimization goals:** `APP_INSTALLS`, `CONVERSATIONS`, `EVENT_RESPONSES`, `IMPRESSIONS`, `LANDING_PAGE_VIEWS`, `LEAD_GENERATION`, `LINK_CLICKS`, `OFFSITE_CONVERSIONS`, `PAGE_LIKES`, `POST_ENGAGEMENT`, `REACH`, `THRUPLAY`, `VALUE`

**Billing events:** `APP_INSTALLS`, `CLICKS`, `IMPRESSIONS`, `LINK_CLICKS`, `PAGE_LIKES`, `POST_ENGAGEMENT`, `THRUPLAY`

**Custom event types:** `ADD_PAYMENT_INFO`, `ADD_TO_CART`, `ADD_TO_WISHLIST`, `COMPLETE_REGISTRATION`, `CONTACT`, `CONTENT_VIEW`, `CUSTOMIZE_PRODUCT`, `DONATE`, `FIND_LOCATION`, `INITIATED_CHECKOUT`, `LEAD`, `OTHER`, `PURCHASE`, `SCHEDULE`, `SEARCH`, `START_TRIAL`, `SUBMIT_APPLICATION`, `SUBSCRIBE`

#### meta ads adset get

Get details for a specific ad set.

```
meta ads adset get <AD_SET_ID>
```

#### meta ads adset delete

Delete an ad set and all its child ads.

```
meta ads adset delete <AD_SET_ID>
meta ads adset delete <AD_SET_ID> --force
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

#### meta ads adset update

Update an ad set.

```
meta ads adset update <AD_SET_ID> --name "New Name"
meta ads adset update <AD_SET_ID> --status ACTIVE
meta ads adset update <AD_SET_ID> --daily-budget 10000
```

| Option | Description |
|--------|-------------|
| `--name` | New name for the ad set |
| `--status` | New status: `ACTIVE`, `PAUSED`, `ARCHIVED` |
| `--daily-budget` | New daily budget in cents |
| `--lifetime-budget` | New lifetime budget in cents |
| `--bid-amount` | New bid amount in cents |
| `--end-time` | New end time (ISO 8601) |

**Note:** At least one option must be provided.

---

### Ads

Manage ads.

#### meta ads ad list

List ads. Optionally filter by ad set.

```
meta ads ad list                  # All ads in the ad account
meta ads ad list <AD_SET_ID>         # Ads for a specific ad set
meta ads ad list --limit 25
```

| Argument | Required | Description |
|----------|----------|-------------|
| `<ADSET_ID>` | No | Filter to ads in this ad set |

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 10 | Maximum number of ads to return |

**Output columns:** `name`, `effective_status`, `id`, `adset_id`, `campaign_id`, `created_time`

#### meta ads ad create

Create an ad in an ad set. An ad references an ad creative -- create the ad creative first with `meta ads creative create`.

```
meta ads ad create --adset-id <AD_SET_ID> --name "My Ad" --creative-id <CREATIVE_ID>
meta ads ad create --adset-id <AD_SET_ID> --name "Conversion Ad" --creative-id <CREATIVE_ID> \
  --pixel-id <PIXEL_ID> --conversion-domain example.com
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `--adset-id` | Yes | | ID of the ad set that will own this ad |
| `--name` | Yes | | Name for the ad |
| `--creative-id` | Yes | | ID of the ad creative to use |
| `--status` | No | `PAUSED` | Initial status: `ACTIVE`, `PAUSED` |
| `--pixel-id` | No | | Dataset (Pixel) ID for conversion tracking |
| `--tracking-specs` | No | | Tracking specs as raw JSON (a list), or `@path/to/file.json` (mutually exclusive with `--pixel-id`) |
| `--conversion-domain` | No | | Destination domain for conversions (for example, `example.com`). Required by Meta for `OUTCOME_SALES` and `OFFSITE_CONVERSIONS` ads |
| `--schedule-start-time` | No | | Ad-level schedule start (ISO 8601). Time-boxes when this ad runs, independent of the ad set schedule. Pairs with `--schedule-end-time`; only some campaign types allow ad-level scheduling |
| `--schedule-end-time` | No | | Ad-level schedule end (ISO 8601). Pairs with `--schedule-start-time` |

**Note:** Use `--pixel-id` **or** `--tracking-specs`, not both.

#### meta ads ad get

Get details for a specific ad.

```
meta ads ad get <AD_ID>
meta --output json ads ad get <AD_ID>
```

#### meta ads ad update

Update an ad.

```
meta ads ad update <AD_ID> --name "New Name"
meta ads ad update <AD_ID> --creative-id <CREATIVE_ID>
meta ads ad update <AD_ID> --status ACTIVE
```

| Option | Description |
|--------|-------------|
| `--name` | New name for the ad |
| `--creative-id` | New ad creative ID |
| `--status` | New status: `ACTIVE`, `PAUSED`, `ARCHIVED` |

**Note:** At least one option must be provided.

#### meta ads ad delete

Delete an ad.

```
meta ads ad delete <AD_ID>
meta ads ad delete <AD_ID> --force
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

---

### Ad creatives

Manage ad creatives. See the [Ad Creatives](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ad-creatives) for detailed usage.

#### meta ads creative list

List ad creatives in the ad account.

```
meta ads creative list
meta ads creative list --limit 25
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 10 | Maximum number of ad creatives to return |

**Output columns:** `name`, `id`, `status`, `title`, `body`

#### meta ads creative create

Create an ad creative. Supports two modes: **standard** and **Dynamic Creative Optimization**.

##### Standard ad creative

Provide a single image or video with ad copy:

```
meta ads creative create --name "Summer Sale" --image ./banner.jpg \
  --page-id <PAGE_ID> --body "50% off everything!" \
  --link-url https://example.com --title "Shop Now" \
  --call-to-action SHOP_NOW
```

| Option | Required | Description |
|--------|----------|-------------|
| `--name` | Yes | Ad creative name |
| `--page-id` | Yes | Facebook Business Page ID to use as the ad's identity |
| `--image` | No | Path to an image file |
| `--video` | No | Path to a video file |
| `--body` | No | Primary text / ad copy |
| `--title` | No | Headline text below the image |
| `--link-url` | No | Destination URL when clicked |
| `--description` | No | Link description below the headline |
| `--call-to-action` | No | Call-to-action button type |
| `--instagram-user-id` | No | Instagram account ID for Instagram placements |
| `--image-hash` | No | Hash of an already-uploaded image (skips upload) |
| `--video-id` | No | ID of an already-uploaded video (skips upload) |
| `--url-tags` | No | URL query params appended to the destination link (for example, `utm_source=facebook&utm_campaign=summer_sale`) |
| `--product-set-id` | No | Product set ID for catalog / dynamic product ads |
| `--ad-format` | No | DCO ad format. Normally omit — inferred from `--images`/`--videos`. Set explicitly for `CAROUSEL`/`AUTOMATIC_FORMAT`, or when passing both images and videos |
| `--status` | No | Initial creative status. Use `PAUSED` to stage a creative without it going live until an ad activates it |

Provide `--image` **or** `--video`, not both. `--image-hash` and `--video-id` reuse already-uploaded media instead of uploading a file.

##### Advanced creative options

| Option | Description |
|--------|-------------|
| `--object-story-id` | Boost an existing Page post as the creative (format: `PAGEID_POSTID`). No media upload or `--page-id` needed |
| `--source-instagram-media-id` | Boost an existing Instagram post by its Instagram media ID |
| `--instagram-permalink-url` | Boost an existing Instagram post by its public permalink URL |
| `--object-story-spec` | Raw `object_story_spec` as JSON, or `@path/to/file.json`. Escape hatch for advanced creatives the shortcut flags do not cover (carousels, app deep links, image overlays). Replaces the `--page-id`/`--image`/`--body`/… shortcuts |
| `--asset-feed-spec` | Raw `asset_feed_spec` as JSON, or `@path/to/file.json`. Full control of Advantage+/DCO multi-asset feeds beyond `--images`/`--titles`/… |
| `--degrees-of-freedom-spec` | Raw `degrees_of_freedom_spec` as JSON, or `@path/to/file.json` |
| `--authorization-category` | Declared category for ads about social issues, elections, or politics (`POLITICAL`, `POLITICAL_WITH_DIGITALLY_CREATED_MEDIA`) |
| `--contextual-multi-ads` / `--no-contextual-multi-ads` | Opt in/out of contextual multi-ads (Meta may show the ad alongside related ads in feed). Leave unset to inherit the account default |
| `--applink-treatment` | Fallback when a deep-linked ad opens without the app installed (`deeplink_with_appstore_fallback`, `deeplink_with_web_fallback`, `web_only`) |

##### Dynamic Creative Optimization

Supply multiple assets and Meta automatically tests combinations:

```
meta ads creative create --name "DCO Creative" --page-id <PAGE_ID> \
  --link-url https://example.com \
  --images ./img1.jpg --images ./img2.jpg --images ./img3.jpg \
  --titles "Title A" --titles "Title B" \
  --bodies "Body 1" --bodies "Body 2" \
  --call-to-actions SHOP_NOW --call-to-actions LEARN_MORE
```

| Option | Max | Description |
|--------|-----|-------------|
| `--images` | 10 | Image file paths (repeat for each) |
| `--videos` | 10 | Video file paths (repeat for each) |
| `--titles` | 5 | Headline variations (repeat for each) |
| `--bodies` | 5 | Primary text variations (repeat for each) |
| `--descriptions` | 5 | Description variations (repeat for each) |
| `--call-to-actions` | 5 | Call-to-action types (repeat for each) |

Dynamic Creative Optimization requires `--link-url` and at least one `--images` or `--videos`.

**Call-to-action types:** `APPLY_NOW`, `BOOK_TRAVEL`, `BUY_NOW`, `CONTACT_US`, `DOWNLOAD`, `GET_OFFER`, `GET_QUOTE`, `LEARN_MORE`, `NO_BUTTON`, `OPEN_LINK`, `SHOP_NOW`, `SIGN_UP`, `SUBSCRIBE`, `WATCH_MORE`

See the [Ad Creatives](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ad-creatives#dynamic-creative-optimization) for detailed examples and format selection logic.

#### meta ads creative get

Get details for a specific ad creative.

```
meta ads creative get <CREATIVE_ID>
meta --output json ads creative get <CREATIVE_ID>
```

#### meta ads creative update

Update an existing ad creative. Ads CLI changes only the fields you specify.

```
meta ads creative update <CREATIVE_ID> --name "New Name"
meta ads creative update <CREATIVE_ID> --body "Updated copy" --title "New Title"
meta ads creative update <CREATIVE_ID> --image ./new-banner.jpg
meta ads creative update <CREATIVE_ID> --status PAUSED
```

| Option | Description |
|--------|-------------|
| `--name` | New ad creative name |
| `--image` | New image file to upload |
| `--video` | New video file to upload |
| `--body` | New primary text |
| `--title` | New headline |
| `--link-url` | New destination URL |
| `--description` | New link description |
| `--call-to-action` | New call-to-action button type |
| `--instagram-user-id` | New Instagram account ID |
| `--image-hash` | Hash of an already-uploaded image (skips upload) |
| `--video-id` | ID of an already-uploaded video (skips upload) |
| `--status` | New status: `ACTIVE` or `PAUSED` |

**Note:** Some fields cannot be updated after creation due to Meta API restrictions. In those cases, create a new ad creative instead.

#### meta ads creative delete

Delete an ad creative.

**Note:** Ad creatives in use by active ads cannot be deleted.

```
meta ads creative delete <CREATIVE_ID>
meta ads creative delete <CREATIVE_ID> --force
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

---

### Datasets (Meta Pixels)

Manage datasets (Meta Pixels / Conversions API). See [Datasets and Catalogs](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/datasets-and-catalogs#datasets) for detailed usage.

#### meta ads dataset list

List datasets. Lists from the ad account by default, or from the business when one is configured (via the `meta ads --business-id` group flag or `BUSINESS_ID`).

```
meta ads dataset list
meta ads --business-id <BUSINESS_ID> dataset list
meta ads dataset list --limit 50
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 25 | Maximum number of datasets to return |

**Output columns:** `name`, `id`, `creation_time`, `last_fired_time`

#### meta ads dataset get

Get details for a specific dataset.

```
meta ads dataset get <PIXEL_ID>
```

#### meta ads dataset create

Create a dataset (Pixel) in a business. Ads CLI automatically assigns the authenticated user `ADVERTISE`, `ANALYZE`, and `EDIT` permissions.

```
meta ads dataset create --name "My Pixel"
meta ads --business-id <BUSINESS_ID> dataset create --name "My Pixel"
```

| Option | Required | Description |
|--------|----------|-------------|
| `--name` | Yes | Name for the dataset |

The business is taken from the `meta ads --business-id` group flag (or `BUSINESS_ID`), resolved from your ad account when omitted.

#### meta ads dataset connect

Connect a dataset to an ad account and/or product catalog.

```
meta ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
meta ads dataset connect <PIXEL_ID> --catalog-id <CATALOG_ID>
meta ads dataset connect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --catalog-id <CATALOG_ID>
```

| Option | Description |
|--------|-------------|
| `--ad-account-id` | Ad account to connect the dataset to |
| `--catalog-id` | Product catalog to connect the dataset to |
| `--business-id` | Business that owns the dataset (resolved if omitted) |

**Note:** At least one of `--ad-account-id` or `--catalog-id` is required.

#### meta ads dataset disconnect

Unshare a dataset from an ad account.

```
meta ads dataset disconnect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID>
meta ads dataset disconnect <PIXEL_ID> --ad-account-id <AD_ACCOUNT_ID> --force
```

| Option | Short | Required | Description |
|--------|-------|----------|-------------|
| `--ad-account-id` | | Yes | Ad account to unshare from |
| `--business-id` | | No (inferred from ad account) | Business that owns the dataset |
| `--force` | `-f` | No | Skip confirmation prompt |

#### meta ads dataset assign-user

Assign a user to a dataset with specified permissions.

```
meta ads dataset assign-user <PIXEL_ID>
meta ads dataset assign-user <PIXEL_ID> --user-id <USER_ID>
meta ads dataset assign-user <PIXEL_ID> --tasks ADVERTISE --tasks ANALYZE --tasks EDIT
```

| Option | Default | Description |
|--------|---------|-------------|
| `--user-id` | Authenticated user | User ID to assign |
| `--tasks` | `ADVERTISE`, `ANALYZE` | Permissions to grant (repeat for multiple) |

**Available tasks:** `ADVERTISE`, `ANALYZE`, `EDIT`, `UPLOAD`

---

### Product catalogs

Manage product catalogs. See [Datasets and Catalogs](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/datasets-and-catalogs#product-catalogs) for detailed usage.

#### Group-level option

All catalog commands accept:

| Option | Default | Description |
|--------|---------|-------------|
| `--business-id` | Resolved from ad account | Business ID. Also reads `BUSINESS_ID` env var. |

If `--business-id` is not provided, Ads CLI resolves it from the configured ad account.

#### meta ads catalog list

List product catalogs for a business.

```
meta ads catalog list
meta ads catalog list --business-id <BUSINESS_ID>
meta ads catalog list --limit 50
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--limit` | `-l` | 25 | Maximum number of catalogs to return |

**Output columns:** `name`, `id`, `vertical`, `product_count`, `feed_count`

#### meta ads catalog get

Get details for a specific product catalog.

```
meta ads catalog get <CATALOG_ID>
```

#### meta ads catalog create

Create a product catalog.

```
meta ads catalog create --name "My Catalog"
meta ads catalog create --name "Hotel Catalog" --vertical hotels
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `--name` | Yes | | Name for the catalog |
| `--vertical` | No | `commerce` | Catalog type |

**Verticals:** `adoptable_pets`, `commerce`, `destinations`, `flights`, `generic`, `home_listings`, `hotels`, `local_service_businesses`, `offer_items`, `offline_commerce`, `transactable_items`, `vehicles`

#### meta ads catalog update

Update a product catalog.

```
meta ads catalog update <CATALOG_ID> --name "Renamed Catalog"
```

| Option | Description |
|--------|-------------|
| `--name` | New name for the catalog |

#### meta ads catalog delete

Delete a product catalog.

```
meta ads catalog delete <CATALOG_ID>
meta ads catalog delete <CATALOG_ID> --force
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

---

### Product items

Manage individual products inside a catalog. `list` and `create` take the catalog as a `--catalog-id` flag; `get`, `update`, and `delete` take the product item ID positionally.

```
meta ads product-item list --catalog-id <CATALOG_ID>
meta ads product-item list --catalog-id <CATALOG_ID> --limit 50
meta ads product-item get <PRODUCT_ITEM_ID>
```

#### meta ads product-item create

```
meta ads product-item create --catalog-id <CATALOG_ID> --name "Running Shoes" \
  --retailer-id shoes-001 --price 7999 --currency USD \
  --image-url https://example.com/shoes.jpg --url https://example.com/shoes
```

| Option | Required | Description |
|--------|----------|-------------|
| `--catalog-id` | Yes | Product catalog to create the item in |
| `--name` | Yes | Product name |
| `--retailer-id` | Yes | Retailer-defined ID, unique within the catalog |
| `--price` | Yes | Price in minor units (for example, cents) |
| `--currency` | Yes | ISO 4217 currency code (for example, `USD`) |
| `--image-url` | Yes | URL of the product image |
| `--url` | Yes | Destination URL for the product |
| `--availability` | No | Stock availability |
| `--brand` | No | Brand name |
| `--category` | No | Product category |
| `--condition` | No | Item condition |
| `--description` | No | Product description |

#### meta ads product-item update

```
meta ads product-item update <PRODUCT_ITEM_ID> --price 6999
meta ads product-item update <PRODUCT_ITEM_ID> --availability in_stock --description "New description"
```

Accepts the same fields as `create` (all optional). At least one option must be provided.

#### meta ads product-item delete

```
meta ads product-item delete <PRODUCT_ITEM_ID>
meta ads product-item delete <PRODUCT_ITEM_ID> --force
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

---

### Product sets

Manage product sets (filtered subsets of a catalog, used for catalog ads). `list` and `create` take the catalog as a `--catalog-id` flag; `get`, `update`, and `delete` take the product set ID positionally.

```
meta ads product-set list --catalog-id <CATALOG_ID>
meta ads product-set get <PRODUCT_SET_ID>
```

#### meta ads product-set create

```
meta ads product-set create --catalog-id <CATALOG_ID> --name "Shoes" \
  --filter '{"product_type":{"eq":"shoes"}}'
```

| Option | Required | Description |
|--------|----------|-------------|
| `--catalog-id` | Yes | Product catalog to create the set in |
| `--name` | Yes | Name for the product set |
| `--filter` | No | Product filter as JSON, or `@path/to/file.json`. Selects which catalog products belong to the set |
| `--retailer-id` | No | Retailer-defined ID for the product set |

#### meta ads product-set update

```
meta ads product-set update <PRODUCT_SET_ID> --name "Renamed Set"
```

Accepts `--name`, `--filter`, and `--retailer-id` (all optional). At least one option must be provided.

#### meta ads product-set delete

```
meta ads product-set delete <PRODUCT_SET_ID>
meta ads product-set delete <PRODUCT_SET_ID> --force
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

---

### Product feeds

Manage product feeds (scheduled inventory uploads) inside a catalog. `list` and `create` take the catalog as a `--catalog-id` flag; `get`, `update`, and `delete` take the feed ID positionally.

```
meta ads product-feed list --catalog-id <CATALOG_ID>
meta ads product-feed list --catalog-id <CATALOG_ID> --limit 50
meta ads product-feed get <FEED_ID>
meta --output json ads product-feed list --catalog-id <CATALOG_ID>
```

#### meta ads product-feed create

```
meta ads product-feed create --catalog-id <CATALOG_ID> --name "Daily Feed"
meta ads product-feed create --catalog-id <CATALOG_ID> --name "Hotels Feed" \
  --feed-type HOTEL --default-currency USD --country US
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `--catalog-id` | Yes | | Product catalog to create the feed in |
| `--name` | Yes | | Name for the product feed |
| `--feed-type` | No | `PRODUCTS` | Feed type (`PRODUCTS`, `HOTEL`, `FLIGHT`, `DESTINATION`, `VEHICLES`, and others) |
| `--default-currency` | No | | Default ISO currency code (for example, `USD`) |
| `--country` | No | | Two-letter country code for the feed (for example, `US`) |
| `--encoding` | No | | Feed file encoding (`AUTODETECT`, `UTF8`, `LATIN1`, `UTF16BE`, `UTF16LE`, `UTF32BE`, `UTF32LE`) |
| `--file-name` | No | | Source file name to ingest |

#### meta ads product-feed update

```
meta ads product-feed update <FEED_ID> --name "Renamed Feed"
meta ads product-feed update <FEED_ID> --default-currency EUR
```

Accepts `--name`, `--default-currency`, `--country`, `--encoding`, and `--file-name` (all optional). At least one option must be provided.

#### meta ads product-feed delete

```
meta ads product-feed delete <FEED_ID>
meta ads product-feed delete <FEED_ID> --force
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

---

### Guidance

Read pre-flight and mid-flight ad recommendations (Unified Guidance Platform) for the configured ad account — useful before or after creating campaigns, ad sets, and ads.

#### meta ads guidance list

```
meta ads guidance list
meta --output json ads guidance list
```

**Output columns:** `type`, `recommendation_stage`, `opportunity_score_lift`, `lift_estimate`, `body`

---

### Ad studies

List ad studies (experiments and lift studies) impacting the configured ad account.

#### meta ads study list

```
meta ads study list
meta ads study list --study-types LIFT,SPLIT_TEST
meta ads study list --study-status ACTIVE --name-query "Q3 lift"
meta ads study list --max-creation-time 2025-01-15
```

| Option | Default | Description |
|--------|---------|-------------|
| `--fields` | `id,name,type,start_time,end_time` | Comma-separated list of fields to return |
| `--limit` / `-l` | 25 | Maximum number of studies to return |
| `--study-types` | | Comma-separated types: `BACKEND_AB_TESTING`, `CONTINUOUS_LIFT_CONFIG`, `GEO_LIFT`, `LIFT`, `SPLIT_TEST`, `SPLIT_TEST_V2` |
| `--study-status` | | Status filter: `ALL`, `PLANNED`, `ACTIVE`, `PLANNED_AND_ACTIVE`, `FINISHED`, `CANCELED`, `DRAFT`, `ON` |
| `--name-query` | | Substring match on study name |
| `--objective-types` | | Comma-separated objectives: `SALES`, `NONSALES`, `MAE`, `TELCO`, `FTL`, `MAI`, `PARTNER`, `BRANDLIFT`, `BRAND`, `MPC_CONVERSION`, `CONVERSIONS` |
| `--max-creation-time` | | Upper bound on study creation time, `YYYY-MM-DD` or ISO datetime (UTC) |
| `--min-cooldown-start-time` | | Lower bound on cooldown start time, `YYYY-MM-DD` or ISO datetime (UTC) |
| `--max-observation-end-time` | | Upper bound on observation end time, `YYYY-MM-DD` or ISO datetime (UTC) |

---

### Insights

Query ad performance data. See [Insights](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/insights) for detailed usage.

**Note:** `insights get` prints the raw Marketing API response body, so the global `--output` format does not reformat it — pipe to `jq` to reshape the JSON.

#### meta ads insights get

Query ad performance insights with full control over fields, date range and breakdowns.

```
meta ads insights get
meta ads insights get --fields spend,impressions,ctr
meta ads insights get --since 2024-01-01 --until 2024-01-31
meta ads insights get --breakdown age --breakdown gender
meta ads insights get --campaign-id <CAMPAIGN_ID>
```

See [Insights](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/insights) for the full options reference.


---

Full documentation index for this product: https://developers.facebook.com/documentation/ads-commerce/llms.txt
