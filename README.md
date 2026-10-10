<p align="center">
  <img src="logo.png" width="96" alt="Blawgy" />
</p>

# Blawgy MCP server

Run your blog from Claude, ChatGPT, Cursor or your own agent.

[Blawgy](https://blawgy.com) researches keywords, writes SEO articles and publishes them to your site (WordPress, Shopify, Webflow, Wix, Framer, Ghost, Duda or your own API) on a schedule. This MCP server gives an AI agent everything the dashboard can do. Read your Search Console traffic, run keyword research, build and reorder the content plan, write and edit articles, draft and approve updates to live posts, change images, products, locations and settings, manage guardrails, webhooks and your team.

This repo holds the public listing (README, `server.json`, logo). The server itself runs inside Blawgy and is not open source.

**Endpoint** `https://app.blawgy.com/mcp` (Streamable HTTP)
**Auth** OAuth 2.1 with PKCE and dynamic client registration, or an API key as a bearer token
**Plans** every paid Blawgy plan, including clients of white-label partners
**Docs** [blawgy.com/mcp](https://blawgy.com/mcp) and the [help article](https://blawgy.com/help/connecting-ai-agents)

## Connect

### Claude (web, desktop, mobile)

Customize, then Connectors, then Add custom connector. Name it Blawgy, paste the server URL, press Connect and sign in with your Blawgy account. Pick the sites and whether the connection can write. On Team and Enterprise plans an owner adds it under Organization settings. To use an API key instead, paste `Authorization` and `Bearer <key>` under Request headers.

### Claude Code

```bash
claude mcp add --transport http blawgy https://app.blawgy.com/mcp
claude mcp login blawgy     # or run /mcp inside a session
```

With an API key, no sign-in needed

```bash
claude mcp add --transport http blawgy https://app.blawgy.com/mcp \
  --header "Authorization: Bearer blk_live_..."
```

Shared with your repo in `.mcp.json` (`--scope project`)

```json
{
  "mcpServers": {
    "blawgy": {
      "type": "http",
      "url": "https://app.blawgy.com/mcp",
      "headers": { "Authorization": "Bearer ${BLAWGY_API_KEY}" }
    }
  }
}
```

### ChatGPT

Settings, then Connectors, then Advanced settings, turn on Developer mode, press Create. Name it Blawgy, paste the server URL, choose OAuth, sign in. ChatGPT only supports OAuth, so keys do not work there. Developer mode on Plus and Pro is read only. Writing, publishing and settings changes need a Business, Enterprise or Edu workspace with Developer mode enabled by an admin.

### Cursor

`.cursor/mcp.json` in a project, or `~/.cursor/mcp.json` for every project. Leave out `headers` and Cursor prompts you to sign in.

```json
{
  "mcpServers": {
    "blawgy": { "url": "https://app.blawgy.com/mcp" }
  }
}
```

With an API key

```json
{
  "mcpServers": {
    "blawgy": {
      "url": "https://app.blawgy.com/mcp",
      "headers": { "Authorization": "Bearer ${env:BLAWGY_API_KEY}" }
    }
  }
}
```

### VS Code and GitHub Copilot

`.vscode/mcp.json`, or run MCP: Add Server from the command palette.

```json
{
  "inputs": [
    { "type": "promptString", "id": "blawgy-key", "description": "Blawgy API key", "password": true }
  ],
  "servers": {
    "blawgy": {
      "type": "http",
      "url": "https://app.blawgy.com/mcp",
      "headers": { "Authorization": "Bearer ${input:blawgy-key}" }
    }
  }
}
```

Drop `inputs` and `headers` to sign in with OAuth instead.

### Windsurf (Devin Desktop)

`~/.config/devin/mcp_config.json` (Windows `%APPDATA%\devin\mcp_config.json`), or Cascade's menu, Open MCP config file.

```json
{
  "mcpServers": {
    "blawgy": {
      "serverUrl": "https://app.blawgy.com/mcp",
      "headers": { "Authorization": "Bearer blk_live_..." }
    }
  }
}
```

### Gemini CLI

```bash
gemini mcp add --transport http blawgy https://app.blawgy.com/mcp
/mcp auth blawgy
```

or with a key, `gemini mcp add --transport http --header "Authorization: Bearer blk_live_..." blawgy https://app.blawgy.com/mcp`.

### Codex CLI

```bash
codex mcp add blawgy --url https://app.blawgy.com/mcp --bearer-token-env-var BLAWGY_API_KEY
```

or `codex mcp add blawgy --url https://app.blawgy.com/mcp` followed by `codex mcp login blawgy` for OAuth.

### Gemini CLI extension

This repo is also a Gemini CLI extension (`gemini-extension.json`), so one command installs it with OAuth sign-in.

```bash
gemini extensions install https://github.com/adamgusky/blawgy-mcp
```

### Cline

```bash
cline mcp install blawgy --transport http https://app.blawgy.com/mcp
```

Clients that only speak stdio can bridge with `npx -y mcp-remote https://app.blawgy.com/mcp`. See `llms-install.md`.

### One-click links

- Cursor `cursor://anysphere.cursor-deeplink/mcp/install?name=blawgy&config=eyJ1cmwiOiAiaHR0cHM6Ly9hcHAuYmxhd2d5LmNvbS9tY3AifQ==`
- VS Code `vscode:mcp/install?%7B%22name%22%3A%22blawgy%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapp.blawgy.com%2Fmcp%22%7D`

### Your own agent

Create a key in Settings, AI agents. Choose its reach (one site, all your sites, or every client site if you are a partner) and whether it can write. The key is shown once.

TypeScript SDK

```ts
import { Client, StreamableHTTPClientTransport } from '@modelcontextprotocol/client';

const client = new Client({ name: 'my-agent', version: '1.0.0' });
const transport = new StreamableHTTPClientTransport(new URL('https://app.blawgy.com/mcp'), {
  requestInit: { headers: { Authorization: `Bearer ${process.env.BLAWGY_API_KEY}` } },
});
await client.connect(transport);
const { tools } = await client.listTools();
```

Claude API (MCP connector, beta header `anthropic-beta: mcp-client-2025-11-20`)

```json
"mcp_servers": [
  { "type": "url", "url": "https://app.blawgy.com/mcp", "name": "blawgy", "authorization_token": "blk_live_..." }
],
"tools": [{ "type": "mcp_toolset", "mcp_server_name": "blawgy" }]
```

OpenAI Agents SDK (Python)

```python
MCPServerStreamableHttp(params={
    "url": "https://app.blawgy.com/mcp",
    "headers": {"Authorization": f"Bearer {os.environ['BLAWGY_API_KEY']}"},
})
```

## What you can ask

- What grew and what slipped on my blog in the last 28 days, and why?
- Find low-difficulty keywords about soy candle care and add the best five to the plan.
- Write an article on fixing candle tunnelling and send it to WordPress as a draft.
- Which published posts are losing clicks? Draft an update for the worst one and show me the diff.
- Is yesterday's post indexed by Google and Bing yet? Recheck it.
- Add a guardrail that we never quote prices, and apply it to every site I run.
- Switch all my client sites to three posts a week and reschedule anything overdue.

## Tools

Every site tool takes a `site` argument (the domain). The server's instructions tell the agent which sites the credential can reach. Slow work (research, writing, scans, syncs) returns a job id immediately and a status tool reports progress.

<!-- tools:start -->
123 tools. "confirm" means the agent must confirm before the call runs. "Pro+" tools are hidden on lower plans.

### Content plan

| Tool | What it does | Access |
|---|---|---|
| `get_plan` | Get content plan | read |
| `generate_plan` | Generate or refill the plan | write |
| `add_to_plan` | Add to plan | write |
| `update_plan_entry` | Edit a plan entry | write |
| `remove_plan_entry` | Remove a plan entry | confirm |
| `reschedule_overdue` | Reschedule overdue entries | write |
| `pause_autopilot` | Pause the plan | write |
| `resume_autopilot` | Resume the plan | write |
| `update_plan_config` | Update plan settings | write |
| `dismiss_topic` | Hide a research topic | write |
| `exclude_topic` | Exclude a topic or keyword | write |
| `accept_plan_suggestion` | Accept a suggested topic | write |
| `dismiss_plan_suggestion` | Dismiss a suggested topic | write |

### Articles

| Tool | What it does | Access |
|---|---|---|
| `list_articles` | List articles | read |
| `get_article` | Get an article | read |
| `write_article_now` | Write article(s) now | write |
| `update_article` | Edit an article | write |
| `reschedule_article` | Change an article's publish date | write |
| `publish_draft` | Publish a CMS draft | confirm |
| `republish_article` | Republish an article | confirm |
| `delete_article` | Delete an article | confirm |
| `release_held_article` | Release a held article | confirm |
| `get_article_preview_url` | Get a preview link | read |

### Article builder

| Tool | What it does | Access |
|---|---|---|
| `generate_titles` | Builder step 1: titles | write |
| `generate_outline` | Builder step 2: outline | write |
| `regenerate_outline` | Builder: redo the outline | write |
| `write_from_outline` | Builder step 3: write the article | write |
| `regenerate_section` | Builder: rewrite one section | write |
| `remix_from_urls` | Remix an existing article | write |
| `list_builder_drafts` | List builder drafts | read |
| `get_builder_draft` | Get a builder draft | read |
| `save_builder_draft` | Save a builder draft | write |
| `delete_builder_draft` | Delete a builder draft | confirm |
| `publish_article_from_builder` | Save and publish a builder article | confirm |

### Smart updates

| Tool | What it does | Access |
|---|---|---|
| `list_update_opportunities` | List page updates | read |
| `check_for_updates` | Find new update opportunities | write, Pro+ |
| `draft_update` | Write an update draft | write, Pro+ |
| `edit_update_draft` | Edit an update draft | write, Pro+ |
| `approve_update` | Approve and publish an update | confirm, Pro+ |
| `dismiss_update` | Dismiss an update | write, Pro+ |
| `revert_update` | Revert a published update | confirm, Pro+ |
| `set_updates_autopilot` | Turn updates autopilot on or off | write, Pro+ |

### Images and photo bank

| Tool | What it does | Access |
|---|---|---|
| `list_image_posts` | List posts whose image can be changed | read |
| `get_post_image` | Get a post's image panel | read |
| `generate_post_image` | Generate a new image for a post | write, Pro |
| `upload_post_image` | Upload your own image for a post | write, Pro |
| `apply_post_image` | Apply a candidate as the post's image | write |
| `revert_post_image` | Revert a post's image | confirm |
| `get_image_preview_usage` | Image preview allowance and sample titles | read |
| `preview_image_style` | Render a sample image in a style | write |
| `extract_brand_colors` | Read brand colours off the website | write |
| `list_photos` | List the product photo bank | read |
| `get_photo_candidates` | Products that can be added to the photo bank | read |
| `add_photo_from_product` | Add a catalogue product to the photo bank | write |
| `add_photo` | Upload a product photo to the bank | write |
| `update_photo` | Edit a photo bank entry | write |
| `remove_photo_background` | Remove a photo's background | write |
| `restore_photo_original` | Restore a photo's original upload | write |
| `duplicate_photo` | Add another look of a product | write |
| `delete_photo` | Delete a photo from the bank | confirm |

### Products

| Tool | What it does | Access |
|---|---|---|
| `list_products` | List products | read |
| `get_product` | Get one product | read |
| `sync_products` | Sync products from Shopify now | write |
| `get_product_sync_status` | Product sync settings and status | read |
| `update_product_sync_settings` | Change automatic product sync | write |
| `add_product` | Add a product by hand | write |
| `update_product` | Edit a product | write |
| `hide_product` | Hide a product from articles | write |
| `unhide_product` | Unhide a product | write |
| `delete_product` | Delete (archive) a product | confirm |
| `get_dutchie_status` | Dutchie POS connection status | read |
| `connect_dutchie` | Connect a Dutchie location | write |
| `sync_dutchie` | Pull the Dutchie menu now | write |
| `remove_dutchie_location` | Disconnect a Dutchie location | confirm |

### Location pages

| Tool | What it does | Access |
|---|---|---|
| `list_business_locations` | List business locations | read |
| `detect_business_location` | Detect the business location from the website | write |
| `add_business_location` | Add a business location | write |
| `update_business_location` | Edit a business location | write |
| `delete_business_location` | Delete a business location | confirm |
| `set_primary_location` | Make a location the primary one | write |
| `list_page_templates` | List WordPress pages to use as templates | read |
| `get_nearby_towns` | Find nearby towns to target | read |
| `preview_location_pages` | Preview titles and slugs for a batch of pages | read |
| `preview_location_page_content` | Preview the full rewritten content of one page | write |
| `generate_location_page_drafts` | Generate location pages as WordPress drafts | write |
| `publish_location_pages` | Generate and PUBLISH location pages | confirm |

### Settings and integrations

| Tool | What it does | Access |
|---|---|---|
| `get_settings` | Get all site settings | read |
| `update_settings` | Update site settings | write |
| `find_competitors` | Suggest competitors | write |
| `generate_description` | Draft a business description from the website | write |
| `fetch_sitemap_links` | Read the site's sitemap | read |
| `get_site_guardrails` | Guardrails applied to this site | read |
| `update_site_guardrails` | Set which guardrails apply to this site | write |
| `list_guardrails` | List the guardrail library | read |
| `create_guardrail` | Create a guardrail | write |
| `update_guardrail` | Edit a guardrail | write |
| `delete_guardrail` | Delete a guardrail | confirm |
| `connect_platform` | Connect or re-test the CMS | write |
| `get_platform_options` | Options the connected CMS offers | read |
| `get_connect_link` | Get a Google Search Console or Bing connect link | write |
| `disconnect_gsc` | Disconnect Google Search Console | confirm |
| `disconnect_bing` | Disconnect Bing Webmaster Tools | confirm |
| `set_draft_mode` | Turn draft mode on or off | write |
| `get_pixel_snippet` | Engagement pixel status and install snippet | read, Pro+ |
| `set_pixel_enabled` | Turn the engagement pixel on or off | write, Pro+ |
| `list_webhooks` | List webhooks | read |
| `create_webhook` | Create a webhook | write |
| `update_webhook` | Edit a webhook | write |
| `delete_webhook` | Delete a webhook | confirm |
| `regenerate_webhook_token` | Regenerate a webhook's auth token | confirm |
| `test_webhook` | Send a test event to a webhook | write |
| `get_webhook_history` | Webhook delivery history | read |
| `retry_webhook_delivery` | Retry a failed webhook delivery | write |
| `get_notification_prefs` | Get email notification preferences | read |
| `update_notification_prefs` | Change email notification preferences | write |

### Team, billing and help

| Tool | What it does | Access |
|---|---|---|
| `list_team` | List teammates and invitations | read |
| `invite_teammate` | Invite a teammate | write |
| `resend_invite` | Resend an invitation | write |
| `revoke_invite` | Revoke an invitation | confirm |
| `get_billing` | Current plan and billing status | read |
| `get_upgrade_link` | Get a checkout or billing portal link | write |
| `search_help` | Ask the help centre | read |
| `contact_support` | Escalate to a human | write |
<!-- tools:end -->

Checkout, card entry, plan changes and account deletion stay in the browser. The relevant tools return a link for a human to open.

## Security

- **Scoped credentials.** A key or OAuth connection reaches one site, all your sites, or (for partners) every approved client site. Each is read only or read and write. Read-only credentials never see write tools.
- **Confirm step.** Delete, publish, revert, disconnect and cancel require the agent to confirm with a one-line description of exactly what will happen.
- **Same limits as the dashboard.** Posts per week, credits and rate limits apply as they do when you click. Per credential, 120 calls a minute and 3000 a day. Agents never get admin access.
- **Revoke any time.** Settings, AI agents lists every key and connected app with its last use. Revoking takes effect on the next call.
- **Stored as hashes.** Keys and tokens are stored hashed. The key is shown once at creation.
- **Logged.** Every call records the tool, the site, the result and the time. Argument values are not stored.
- **OAuth details.** Authorization code with PKCE S256, dynamic client registration (RFC 7591), exact redirect URI match, 1 hour access tokens, rotating refresh tokens with reuse detection, resource indicator bound to the `/mcp` URL. Metadata at `https://app.blawgy.com/.well-known/oauth-authorization-server` and `https://app.blawgy.com/.well-known/oauth-protected-resource`.

## Partners and agencies

A partner owner who connects reaches every approved client site from one connection, so a request like "switch every client to three posts a week" is a single call per site, driven by the agent. Clients can connect their own agents too, scoped to their own sites. Today the connection is made at `app.blawgy.com`, not on a white-label domain.

## Links

- Product page [blawgy.com/mcp](https://blawgy.com/mcp)
- Help article [blawgy.com/help/connecting-ai-agents](https://blawgy.com/help/connecting-ai-agents)
- Model Context Protocol [modelcontextprotocol.io](https://modelcontextprotocol.io)
- Support adam@blawgy.com

## License

The contents of this repository (README, `server.json`, logo) are MIT licensed. The Blawgy service and the MCP server behind `app.blawgy.com/mcp` are proprietary and governed by the [Blawgy terms](https://blawgy.com/terms).
