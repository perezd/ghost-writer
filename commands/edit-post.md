---
description: Edit an existing Ghost blog post
argument-hint: "<post title, slug, or ID>"
---

# /edit-post

## Trigger

When the user runs `/edit-post` or asks to modify, update, revise, or edit an existing blog post.

## Inputs

Gather from the user:

- **Post identifier** — title, slug, or ID of the post to edit
- **Changes requested** — what to add, remove, rewrite, or restructure

## Process

1. Read the **ghost-lexical** skill for Lexical format reference
2. Use `list_posts` to find the post if the user gave a title or partial match, or use `get_post` directly if they provided a slug or ID
3. Retrieve the full post with `get_post` and parse its `lexical` content
4. Present a summary of the current post structure to the user (sections, length)
5. Apply the requested edits to the Lexical document
6. Call `update_post` with the modified content and the post's current `updated_at` timestamp
7. Confirm the update was successful

## Output

- Summary of changes made
- The updated post ID and status
- Note if the post is still in draft or already published

## Notes

- Always fetch the latest `updated_at` immediately before calling `update_post` to avoid collision errors
- If updating tags, remember that the `tags` parameter **replaces** all existing tags — include the ones you want to keep
- When restructuring content, preserve existing Lexical node formatting
- For large edits, describe the planned changes to the user before applying them
