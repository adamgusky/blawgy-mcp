# Installing the Blawgy MCP server

Blawgy is a hosted (remote) MCP server. There is nothing to download or run.

Endpoint `https://app.blawgy.com/mcp`, transport Streamable HTTP.

## If the client supports remote servers with OAuth

Add the server by URL and sign in when prompted.

```json
{
  "mcpServers": {
    "blawgy": { "url": "https://app.blawgy.com/mcp" }
  }
}
```

Cline `cline mcp install blawgy --transport http https://app.blawgy.com/mcp`

## If the client only supports stdio

Use the `mcp-remote` bridge.

```json
{
  "mcpServers": {
    "blawgy": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://app.blawgy.com/mcp"]
    }
  }
}
```

## Using an API key instead of signing in

Create a key in the Blawgy dashboard under Settings, AI agents. Choose which sites it reaches and whether it can write. Send it as `Authorization: Bearer <key>`.

```json
{
  "mcpServers": {
    "blawgy": {
      "url": "https://app.blawgy.com/mcp",
      "headers": { "Authorization": "Bearer ${BLAWGY_API_KEY}" }
    }
  }
}
```

With `mcp-remote`, add `"--header", "Authorization: Bearer ${BLAWGY_API_KEY}"` to `args`.

## Requirements

A paid Blawgy plan. The free trial cannot connect. Clients of a white-label partner count as paid.
