# Connectors

## How tool references work

This plugin connects directly to a Ghost CMS instance through the `ghost-mcp` MCP server. Unlike plugins that use `~~category` placeholders for tool-agnostic design, Ghost Writer targets a single platform — Ghost CMS — and requires the specific MCP server described below.

## Required connector

| Connector | MCP Server | Transport | Purpose |
|-----------|------------|-----------|---------|
| Ghost CMS | `ghost-mcp` | stdio | Blog post CRUD operations via the Ghost Admin API |

## Setup

The ghost-mcp server must be installed and authenticated before the plugin can function:

1. Authenticate: `bunx @perezd/ghost-mcp auth` (requires Ghost site URL and Admin API key)
2. Configure MCP server definition for your Claude instance, like so:

```json
"mcpServers": {
    "ghost": {
        "command": "bunx",
        "args": ["@perezd/ghost-mcp", "serve"],
      "env": {
        "GHOST_MCP_CONFIG": "$HOME/.ghost-mcp/config.json"
      }
    }
}
```

## Tools provided

The ghost-mcp server exposes these tools:

- **list_posts** — Query posts with NQL filters, pagination, and sorting
- **get_post** — Retrieve a single post by ID or slug (includes full Lexical content)
- **create_post** — Create a new post in Lexical format (defaults to draft)
- **update_post** — Update a post with optimistic locking via `updated_at`
- **delete_post** — Permanently remove a post by ID
