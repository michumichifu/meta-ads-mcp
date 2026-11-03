<!-- Fuente oficial: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/get-started.md · descargada el 2026-11-03, sin cambios -->

# Get started with Ads CLI


This guide walks you through installing Ads CLI, authenticating with a system user access token, and running your first commands.

## Requirements

- macOS or Linux
- A Meta system user access token with appropriate scopes (see [Authentication](#authentication))
- An ad account with the assets you want to manage

## Installation

### Option A: Homebrew (recommended)

Install from the `facebook/fb` tap:

```
brew tap facebook/fb
brew trust facebook/fb
brew install facebook/fb/meta-ads
```

This installs the `meta` command. Verify with:

```
meta --version
```

### Option B: pip

The [`meta-ads`](https://pypi.org/project/meta-ads/) package is also available on PyPI (requires Python 3.12 or later):

```
pip install meta-ads
```

For more about configuration, see [Configuration](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/configuration).

## Authentication

> **Important:** Ads CLI requires a system user access token to authenticate for programmatic access.

### Step 1: Create a system user

You can use either an **Admin** or an **Employee** system user:

1. Go to [Meta Business Suite](https://business.facebook.com/) > **Business Settings** > **Users** > **System Users**.
2. Click **Add** to create a new system user.
3. Choose the **Admin** or **Employee** role.
4. Give it a descriptive name (for example, "Ads CLI").

### Step 2: Add the system user to your app with a role

The system user must be added to your app — a system user must have this roll before you can generate a token:

1. Go to your app in [Meta Business Suite](https://business.facebook.com/) > **Business Settings** > **Apps** > **Assign Users**
2. Add the system user with a permissions (for example, **Full Access**).

### Step 3: Add the system user to the ad account

The system user must also be added to every ad account you want to operate on:

1. In Meta Business Suite > **Business Settings** > **Accounts** > **Ad Accounts**, select your ad account.
2. Click **Add People** (or **Add System Users**) and select your system user.
3. Grant it access to manage the account.

Also use **Assign Assets** on the system user to grant access to any other assets you need:

- The datasets (Meta Pixels) you want to use for conversion tracking
- The Business Page(s) you want to use for ad creatives
- The product catalog(s), if using catalog ads

### Step 4: Generate an access token

1. In Meta Business Suite > **Settings** > **Users** > **System Users**, select your system user.
2. In the top right, click **Generate Token**.
3. Select your app.
4. Grant the following scopes:
   - `business_management`
   - `ads_management`
   - `pages_show_list`
   - `pages_read_engagement`
   - `pages_manage_ads`
   - `read_insights`

   Add any additional scopes required for your use case like `catalog_management`

5. Click **Generate Token** and copy the resulting token.

#### Save your token

##### Option A: Set it in your .env file (recommended)

Create a `.env` file in the directory where you run Ads CLI — this persists the variables so you do not have to export them every session:

```
cat > .env << 'DOTENV'
ACCESS_TOKEN='<ACCESS_TOKEN>'
APP_SECRET='<APP_SECRET>'
APP_ID='<APP_ID>'
AD_ACCOUNT_ID='<AD_ACCOUNT_ID>'
BUSINESS_ID='<BUSINESS_ID>'
DOTENV
```

`BUSINESS_ID` is optional — Ads CLI resolves it from your ad account when omitted. See [Configuration](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/configuration) for the full list of variables and precedence rules.

##### Option B: Environment variable (CI/scripts)

```
export ACCESS_TOKEN=<ACCESS_TOKEN>
```

### Check if access token setup correctly status

```
meta auth status
```

## Configure your ad account ID

Most commands require an ad account ID. Set it once and it applies to all commands:

```
# Via .env (recommended)
AD_ACCOUNT_ID=<AD_ACCOUNT_ID>

# Via environment variable
export AD_ACCOUNT_ID=<AD_ACCOUNT_ID>

# Via flag (per-command)
meta ads --ad-account-id <AD_ACCOUNT_ID> campaign list
```

To retrieve your ad account IDs:

```
meta ads adaccount list
```

See [Find your IDs](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/tutorials-and-recipes#find-your-ids) for more information on finding any IDs needed for Ads CLI.

## Run your first commands

Once you have your token and ad account ID configured, here are some commands to start with:

```
# List your ad campaigns
meta ads campaign list

# Check performance for the last 30 days
meta ads insights get --fields spend,impressions,ctr,cpc

# List your Business Pages (needed for creating ad creatives)
meta ads page list
```

## Next steps

- [Configuration](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/configuration) -- Environment variables and config precedence
- [Command Reference](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/command-reference) -- Full documentation for every command
- [Tutorials and Recipes](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/tutorials-and-recipes) -- Step-by-step workflows for common tasks
- [Ad Creatives](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ad-creatives) -- Creating ads with images, videos, and Dynamic Creative Optimization
- [Datasets and Catalogs](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/datasets-and-catalogs) -- Conversion tracking and product catalog management
- [Insights](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/insights) -- Querying ad performance insights


---

Full documentation index for this product: https://developers.facebook.com/documentation/ads-commerce/llms.txt
