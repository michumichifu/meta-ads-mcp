<!-- Fuente oficial: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-mcp-server/ads-mcp-server-get-started.md · descargada el 2026-11-03, sin cambios -->

# Get started


Owning a Meta app is not a prerequisite on using the ads MCP server. For access without owning a Meta app, follow [How to set up Meta ads AI connectors](https://www.facebook.com/business/help/1456422242197840) instead.

To use the ads MCP server with your own Meta app, you must follow some key steps to set up your environment and gain access.

## Create or reuse a Meta developer app
1. Go to [developers.facebook.com/apps](https://developers.facebook.com/apps) and create a new app, or open an existing app.
2. Add the **Create & manage ads with ads MCP server** use case.

## Authentication

The ads MCP server supports three authentication methods.

### OAuth

The MCP client redirects you to the Facebook Login for Business dialog. You sign in with your Facebook account or your Meta Managed Account (MMA) and approve the requested permissions. No manual token setup is required.

Make sure your Meta developer app's **redirect URL** is configured correspondingly with your choice of MCP client in the [**Facebook Login for Business**](https://developers.facebook.com/documentation/facebook-login/facebook-login-for-business) settings.

### User access token

To generate a user access token, use the [Graph API Explorer](https://developers.facebook.com/tools/explorer/) or your own OAuth flow, requesting the [permissions](#permissions) listed below.

### System user access token

You can also use a [system user access token](https://developers.facebook.com/docs/business-management-apis/system-users) to access the ads MCP server, for example for programmatic or server-to-server setups. See [System Users](https://developers.facebook.com/docs/business-management-apis/system-users) for how to create a system user and generate a token, requesting the [permissions](#permissions) listed below.

> **Note:** Only **Employee**-role system user access tokens are supported. Admin-role system user access tokens can't access the ads MCP server.

### Permissions

These permissions apply to both user access tokens and system user access tokens.

At a minimum, the token must be granted:

- `ads_mcp_management`
- `ads_read` or `ads_management`

Granting additional permissions enables access to more tools:

- `ads_management`
- `catalog_management`
- `business_management`
- `pages_show_list`
- `instagram_basic`


## Connect to ads MCP server
Sever URL of the ads MCP server
```text
https://mcp.facebook.com/ads
```

### Connect to the ads MCP server from Claude Code
To add the ads MCP server to [Claude Code](https://code.claude.com/docs/en/mcp#configure-mcp-servers), run the command below.
```bash
claude mcp add --transport http --client-id <META_APP_ID> meta-ads https://mcp.facebook.com/ads
```

### Connect to the ads MCP server from ChatGPT
Set the redirect URL of your Meta developer app according to [ChatGPT's specification](https://developers.openai.com/api/docs/actions/authentication).

Follow [ChatGPT](https://developers.openai.com/api/docs/guides/developer-mode) instruction:
- Select "OAuth" as Authentication method
- Input https://mcp.facebook.com/ads as the Server URL
- In Advanced OAuth Settings, input your Meta developer app ID as the OAuth Client ID.

### Connect to the ads MCP server with user access token
For programmatic setups, pass an user access token to in the request to the ads MCP server in the `Authorization: Bearer <ACCESS_TOKEN>` header.
```bash
curl -i -X POST "https://mcp.facebook.com/ads" \
    -H "Authorization: Bearer <ACCESS_TOKEN>" \
    --data-raw '{"jsonrpc":"2.0","method":"tools/list","id":1}'
```

## Run your first prompts

Once connected, try:

- "List my ad accounts."
- "Create a new traffic campaign in ad account `<AD_ACCOUNT_ID>`."
- "Draft an ad set for my `<CAMPAIGN_ID>` campaign that optimizes for link clicks."
- "Summarize the top-spending campaigns in `<AD_ACCOUNT_ID>` over the last 30 days."


---

Full documentation index for this product: https://developers.facebook.com/documentation/ads-commerce/llms.txt
