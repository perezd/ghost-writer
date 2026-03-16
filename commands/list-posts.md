---
description: Browse and search Ghost blog posts
argument-hint: "<optional search terms or filters>"
---

# /list-posts

## Trigger

When the user runs `/list-posts` or asks to see, browse, find, or search their blog posts.

## Inputs

Optional filters from the user:

- **Status** — drafts, published, or all
- **Tag** — filter by tag name
- **Search terms** — title or topic keywords
- **Sort order** — newest, oldest, recently updated

## Process

1. Translate the user's request into NQL filter and order parameters:
   - "my drafts" → `filter: "status:draft"`
   - "published posts about X" → `filter: "tag:X+status:published"`
   - "recent posts" → `order: "published_at desc"`, `limit: 10`
2. Call `list_posts` with the appropriate parameters
3. Present the results in a readable format

## Output

A formatted list showing for each post:

- Title
- Status (draft / published / scheduled)
- Date (published or last updated)
- Tags (if any)
- Slug (for easy reference)

Include pagination info if there are more results available.

## Notes

- Default to showing the 15 most recent posts if no filters are specified
- Use `order: "updated_at desc"` for "recently edited" requests
- Offer to open a specific post for editing if the user seems to be looking for one
