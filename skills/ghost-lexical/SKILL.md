---
name: ghost-lexical
description: >
  Construct and manage Ghost CMS blog posts using Lexical JSON format via the ghost-mcp server.
  Use this skill whenever creating, editing, reading, or managing content on a Ghost blog.
  Triggers include: any mention of Ghost, blog post, Lexical, draft, publish, or any request
  involving blog content management. This skill covers both the Lexical document format and
  the MCP tool API for interacting with Ghost.
---

# Ghost Lexical Content Skill

This skill teaches you how to construct valid Ghost Lexical JSON documents and interact with a Ghost blog through the ghost-mcp server tools.

> **Prerequisites**: The ghost-mcp server must be configured and running. If MCP tools are unavailable, remind the user to run `bunx @perezd/ghost-mcp auth` to set up their Ghost site URL and Admin API key.

## MCP Tools Reference

The ghost-mcp server exposes five tools. All content is exclusively in Lexical JSON format.

### list_posts

Query and filter posts. Returns summaries (id, title, slug, status, dates, excerpt) plus pagination metadata.

**Parameters:**

- `filter` (string, optional) — NQL filter string. Examples: `status:draft`, `tag:news+status:published`, `author:derek`
- `limit` (number, default 15) — Posts per page
- `page` (number, default 1) — Page number
- `order` (string, optional) — Sort order. Example: `published_at desc`, `updated_at asc`

**Common filter patterns:**

- All drafts: `status:draft`
- Published posts with a tag: `tag:engineering+status:published`
- Posts updated recently: use `order: "updated_at desc"` with a small limit

### get_post

Retrieve a single post with full Lexical content. Accepts either a 24-character hex ID or a slug.

**Parameters:**

- `id_or_slug` (string, required) — Post ID (24-char hex like `6721a3b8e4f5c6d7e8f90a1b`) or URL slug (like `my-first-post`)

**Returns:** Full post object including `lexical` field containing the JSON document as a string. Always parse the `lexical` field with `JSON.parse()` before reading or modifying content.

### create_post

Create a new post. Defaults to draft status.

**Parameters:**

- `title` (string, required) — Post title
- `lexical` (string, optional) — Content as a JSON string of a Lexical document. Must be `JSON.stringify()`'d.
- `status` (enum: `draft` | `published` | `scheduled`, default `draft`) — Post status
- `tags` (string[], optional) — Tag names to assign. Tags are created automatically if they don't exist.

**Important:** The `lexical` value must be a JSON *string*, not a raw object. Always wrap the Lexical document with `JSON.stringify()`.

### update_post

Modify an existing post. Requires `updated_at` for optimistic locking — Ghost rejects updates if the timestamp doesn't match, preventing overwrites of concurrent edits.

**Parameters:**

- `id` (string, required) — Post ID (24-char hex)
- `updated_at` (string, required) — The current `updated_at` value from the post. Obtain this from `get_post` or `list_posts` immediately before updating.
- `title` (string, optional) — New title
- `lexical` (string, optional) — New content as Lexical JSON string
- `status` (enum: `draft` | `published` | `scheduled`, optional) — New status
- `tags` (string[], optional) — New tag names. **Replaces** all existing tags.

**Collision detection workflow:**

1. Call `get_post` to get the current post (and its `updated_at`)
2. Make your modifications to the Lexical content
3. Call `update_post` with the `updated_at` value from step 1
4. If Ghost returns a conflict error, re-fetch and retry

### delete_post

Permanently delete a post.

**Parameters:**

- `id` (string, required) — Post ID (24-char hex)

**Warning:** This is irreversible. Always confirm with the user before deleting.

## Lexical Document Structure

All Ghost content uses a Lexical JSON document format. The complete reference is in `references/lexical-format-guide.md`. Below are the essential patterns for constructing documents.

### Root Structure

Every Lexical document has this wrapper:

```json
{
  "root": {
    "children": [],
    "direction": "ltr",
    "format": "",
    "indent": 0,
    "type": "root",
    "version": 1
  }
}
```

All content nodes go inside `root.children`.

### Node Types

#### Paragraphs

```json
{
  "type": "paragraph",
  "children": [
    {
      "type": "extended-text",
      "text": "Your paragraph text here.",
      "format": 0,
      "detail": 0,
      "mode": "normal",
      "style": "",
      "version": 1
    }
  ],
  "direction": "ltr",
  "format": "",
  "indent": 0,
  "version": 1
}
```

A paragraph can contain multiple `extended-text` and `link` children to mix formatting and links inline.

#### Headings

```json
{
  "type": "extended-heading",
  "tag": "h2",
  "children": [
    {
      "type": "extended-text",
      "text": "Section Title",
      "format": 0,
      "detail": 0,
      "mode": "normal",
      "style": "",
      "version": 1
    }
  ],
  "direction": "ltr",
  "format": "",
  "indent": 0,
  "version": 1
}
```

Tags: `h1` through `h6`. Use `h2` for main sections, `h3` for subsections.

#### Text Formatting (Bitwise Flags)

The `format` field on `extended-text` nodes uses bitwise flags:

| Value | Style |
|-------|-------|
| 0 | Plain |
| 1 | Italic |
| 2 | Bold |
| 3 | Bold + Italic |
| 4 | Underline |
| 8 | Strikethrough |
| 16 | Inline code |

Combine with addition: bold + code = `18`.

To mix formatting within a paragraph, use multiple `extended-text` children with different `format` values.

#### Links

Links are inline children within paragraphs:

```json
{
  "type": "link",
  "url": "https://example.com",
  "children": [
    {
      "type": "extended-text",
      "text": "link text",
      "format": 0,
      "detail": 0,
      "mode": "normal",
      "style": "",
      "version": 1
    }
  ],
  "direction": "ltr",
  "format": "",
  "indent": 0,
  "version": 1,
  "rel": null,
  "target": null,
  "title": null
}
```

For bare URLs (no custom text), use `"children": []`.

#### Lists

```json
{
  "type": "list",
  "listType": "bullet",
  "start": 1,
  "tag": "ul",
  "children": [
    {
      "type": "listitem",
      "value": 1,
      "children": [
        {
          "type": "extended-text",
          "text": "First item",
          "format": 0,
          "detail": 0,
          "mode": "normal",
          "style": "",
          "version": 1
        }
      ],
      "direction": "ltr",
      "format": "",
      "indent": 0,
      "version": 1
    }
  ],
  "direction": "ltr",
  "format": "",
  "indent": 0,
  "version": 1
}
```

For ordered lists: `"listType": "number"`, `"tag": "ol"`. List item `value` fields should be sequential (1, 2, 3...).

#### Images

```json
{
  "type": "image",
  "version": 1,
  "src": "https://example.com/photo.jpg",
  "width": 1200,
  "height": 800,
  "title": "",
  "alt": "Descriptive alt text",
  "caption": "",
  "cardWidth": "regular",
  "href": ""
}
```

`cardWidth` options: `regular`, `wide`, `full`. Captions accept HTML.

#### Code Blocks

```json
{
  "type": "codeblock",
  "version": 1,
  "code": "const greeting = 'hello';",
  "language": "javascript",
  "caption": ""
}
```

Use the appropriate language identifier for syntax highlighting.

#### Bookmark Cards

Rich link previews. Ghost auto-fetches metadata when you provide a URL:

```json
{
  "type": "bookmark",
  "version": 1,
  "url": "https://example.com/article",
  "metadata": {
    "icon": "",
    "title": "Article Title",
    "description": "Article description",
    "author": "",
    "publisher": "",
    "thumbnail": ""
  },
  "caption": ""
}
```

When creating bookmarks, you can provide just the `url` and minimal metadata — Ghost will enrich it.

#### Signup Cards

Newsletter subscription forms:

```json
{
  "type": "signup",
  "version": 1,
  "alignment": "left",
  "backgroundColor": "#F0F0F0",
  "backgroundImageSrc": "",
  "backgroundSize": "cover",
  "textColor": "#000000",
  "buttonColor": "accent",
  "buttonTextColor": "#FFFFFF",
  "buttonText": "Subscribe",
  "disclaimer": "<span style=\"white-space: pre-wrap;\">No spam. Unsubscribe anytime.</span>",
  "header": "<span style=\"white-space: pre-wrap;\">Sign up for updates</span>",
  "labels": [],
  "layout": "wide",
  "subheader": "",
  "successMessage": "Email sent! Check your inbox to complete your signup.",
  "swapped": false
}
```

#### Toggle / Accordion

Collapsible content sections. Both `heading` and `content` accept HTML:

```json
{
  "type": "toggle",
  "version": 1,
  "heading": "<span style=\"white-space: pre-wrap;\">Click to expand</span>",
  "content": "<p>Hidden content here</p>"
}
```

#### Blockquotes

```json
{
  "type": "extended-quote",
  "children": [
    {
      "type": "extended-text",
      "text": "Quoted text goes here.",
      "format": 0,
      "detail": 0,
      "mode": "normal",
      "style": "",
      "version": 1
    }
  ],
  "direction": "ltr",
  "format": "",
  "indent": 0,
  "version": 1
}
```

A container node like paragraphs — children can include `extended-text` and `link` nodes with mixed formatting.

#### Aside

A styled tangent or editorial aside:

```json
{
  "type": "aside",
  "children": [
    {
      "type": "extended-text",
      "text": "An editorial tangent or side note.",
      "format": 0,
      "detail": 0,
      "mode": "normal",
      "style": "",
      "version": 1
    }
  ],
  "direction": "ltr",
  "format": "",
  "indent": 0,
  "version": 1
}
```

Same structure as a paragraph but rendered with distinct visual styling.

#### Callout Cards

Highlighted callout boxes with optional emoji and background color. The `calloutText` field accepts HTML:

```json
{
  "type": "callout",
  "version": 1,
  "calloutText": "<p><strong>Note</strong> — This is important context.</p>",
  "calloutEmoji": "",
  "backgroundColor": "grey"
}
```

`backgroundColor` options include: `grey`, `white`, `blue`, `green`, `yellow`, `red`, `pink`, `purple`, and `accent`.

#### Embed Cards

oEmbed content (YouTube, Twitter, etc.). Ghost fetches the embed metadata from the URL:

```json
{
  "type": "embed",
  "version": 1,
  "url": "https://www.youtube.com/watch?v=VIDEO_ID",
  "embedType": "video",
  "html": "<iframe ...></iframe>",
  "metadata": {
    "title": "Video Title",
    "author_name": "Channel Name",
    "author_url": "https://www.youtube.com/@channel",
    "type": "video",
    "height": 113,
    "width": 200,
    "version": "1.0",
    "provider_name": "YouTube",
    "provider_url": "https://www.youtube.com/",
    "thumbnail_height": 360,
    "thumbnail_width": 480,
    "thumbnail_url": "https://i.ytimg.com/vi/VIDEO_ID/hqdefault.jpg",
    "html": "<iframe ...></iframe>"
  },
  "caption": ""
}
```

When creating embeds, provide the `url` and `embedType` at minimum — Ghost enriches the metadata.

#### HTML Cards

Raw HTML injection with optional visibility controls for member segmentation:

```json
{
  "type": "html",
  "version": 1,
  "html": "<div>Custom HTML content here</div>",
  "visibility": {
    "web": {
      "nonMember": true,
      "memberSegment": "status:free,status:-free"
    },
    "email": {
      "memberSegment": "status:free,status:-free"
    }
  }
}
```

The `visibility` field is optional. When present, it controls who sees the content:

- `web.nonMember` — whether non-members see this block on the website
- `web.memberSegment` / `email.memberSegment` — NQL segment filter (e.g., `status:free` for free members, `status:-free` for paid members)

If `visibility` is omitted, the HTML block is shown to everyone.

#### Line Breaks (Horizontal Rules)

```json
{
  "type": "linebreak",
  "version": 1
}
```

### Required Fields Checklist

Every container node (paragraph, heading, list, listitem, extended-quote, aside) requires all of these:

- `type` — node type identifier
- `version` — always `1`
- `children` — array of child nodes
- `direction` — `"ltr"` for left-to-right
- `format` — `""` (empty string for container nodes)
- `indent` — `0` for no indentation

Every `extended-text` node requires:

- `type` — `"extended-text"`
- `text` — the text content
- `format` — bitwise format flag (0 for plain)
- `detail` — `0`
- `mode` — `"normal"`
- `style` — `""`
- `version` — `1`

Card nodes (image, codeblock, bookmark, signup, toggle, callout, embed, html, linebreak) are simpler — they require `type`, `version`, and their specific fields.

## Content Construction Patterns

### Building a Complete Post

1. Start with the root wrapper
2. Add nodes to `root.children` in reading order
3. Stringify the entire document: `JSON.stringify(doc)`
4. Pass the string to `create_post` or `update_post` as the `lexical` parameter

### Editing an Existing Post

1. `get_post` to retrieve current content
2. Parse the `lexical` field: `JSON.parse(post.lexical)`
3. Navigate `root.children` to find and modify nodes
4. Re-stringify and call `update_post` with the post's current `updated_at`

### Typical Blog Post Structure

A well-formed blog post typically follows this node sequence in `root.children`:

1. Opening paragraph(s) — hook and context
2. `extended-heading` (h2) — first section
3. Paragraphs with mixed formatting and links
4. `extended-quote` for blockquotes, `aside` for editorial tangents
5. `list` nodes for enumerations
6. `image`, `embed`, or `codeblock` nodes as needed
7. `callout` cards for highlighted notes or warnings
8. `linebreak` between major sections
9. More h2/h3 sections as needed
10. Closing paragraph(s)
11. Optional `html` card for author bio or custom blocks (with visibility controls)
12. Optional `signup` card at the end

## NQL Filter Reference

The `list_posts` tool supports Ghost's NQL (Neon Query Language) for filtering:

- **Field match:** `status:draft`, `tag:news`
- **Combining filters:** `tag:news+status:published` (AND), `tag:news,tag:tech` (OR)
- **Negation:** `-status:draft` (NOT draft)
- **Comparison:** `created_at:>'2025-01-01'`
- **Author filter:** `author:derek`

Common queries:

- All drafts: `status:draft`
- Published with specific tag: `tag:my-tag+status:published`
- Recent posts: use `order: "published_at desc"` with `limit: 5`
