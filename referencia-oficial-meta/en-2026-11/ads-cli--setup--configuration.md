<!-- Fuente oficial: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/configuration.md · descargada el 2026-11-03, sin cambios -->

# Ads CLI configuration


This guide covers how Ads CLI resolves configuration settings, including environment variables, `.env` files, and the config directory.

## Environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ACCESS_TOKEN` | Yes | System user access token that you generate in the Meta Business Suite |
| `AD_ACCOUNT_ID` | Yes (for most commands) | Your Meta ad account ID, with or without the `act_` prefix |
| `BUSINESS_ID` | No | Default business ID (for catalog, dataset, and page commands; resolved from your ad account when omitted) |
| `APP_ID` | No | Facebook App ID (used by some login and app-scoped flows) |
| `APP_SECRET` | No | Facebook App Secret (used by some login and app-scoped flows) |

Shell environment variables take precedence over `.env` file values.

## Configuration precedence

Ads CLI resolves settings in this order (highest priority first):

1. Command-line flags (for example, `--ad-account-id`)
2. Environment variables
3. Project-level `.env` file

## Use a .env file

To persist your environment variables so you do not have to export them for each session, create a `.env` file in the directory where you run Ads CLI:

```
cat > .env << 'DOTENV'
ACCESS_TOKEN='<ACCESS_TOKEN>'
AD_ACCOUNT_ID='<AD_ACCOUNT_ID>'
BUSINESS_ID='<BUSINESS_ID>'
DOTENV
```

Omit `BUSINESS_ID` if you want Ads CLI to resolve it from your ad account automatically. To confirm what is configured, run:

```
meta auth status
meta ads adaccount current
```

**Note:** Environment variables set in your shell take precedence over `.env` values, so an exported variable overrides the same key in `.env`.


## See also

- [Find your IDs](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/tutorials-and-recipes#find-your-ids) -- Look up ad account, page, and other IDs
- [Get Started](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/get-started) -- Installation and authentication
- [Command Reference](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/command-reference) -- Every command, option, and argument


---

Full documentation index for this product: https://developers.facebook.com/documentation/ads-commerce/llms.txt
