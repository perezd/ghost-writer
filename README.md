# Ghost Writer

A Claude Cowork plugin for drafting and managing blog posts on [Ghost CMS](https://ghost.org/) using the Lexical document format.

## Prerequisites

You'll need to ensure [Bun](https://bun.com) is installed locally, it's neccessary to run the MCP server locally.

Additionally, this plugin requires the [@perezd/ghost-mcp](https://www.npmjs.com/package/@perezd/ghost-mcp) server to be installed and authenticated:

```bash
bunx @perezd/ghost-mcp auth
```

The `auth` command will prompt you for your Ghost site URL and Admin API key (found in Ghost Admin → Settings → Integrations).

## Commands

| Command | Description |
|---------|-------------|
| `/draft-post` | Draft a new blog post (creates as draft on Ghost) |
| `/edit-post` | Find and edit an existing post |
| `/list-posts` | Browse, search, and filter posts |
| `/publish-post` | Publish a draft post |
| `/delete-post` | Permanently delete a post |

## Skills

| Skill | Description |
|-------|-------------|
| `ghost-lexical` | Lexical JSON format construction and Ghost MCP tool reference |

## How It Works

The plugin connects to your Ghost blog through the ghost-mcp server, which exposes five tools for post management: listing, reading, creating, updating, and deleting posts. All content is handled exclusively in Ghost's Lexical JSON format.

The `ghost-lexical` skill teaches Claude how to construct valid Lexical documents and use the MCP tools correctly, including collision detection for safe updates and NQL query syntax for filtering posts.

## MCP Integration

This plugin uses the `ghost-mcp` server (stdio transport). See [CONNECTORS.md](CONNECTORS.md) for details.
