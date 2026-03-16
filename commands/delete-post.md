---
description: Delete a Ghost blog post
argument-hint: "<post title, slug, or ID>"
---

# /delete-post

## Trigger

When the user runs `/delete-post` or asks to remove or delete a blog post.

## Inputs

- **Post identifier** — title, slug, or ID of the post to delete

## Process

1. Find the post using `list_posts` or `get_post`
2. Present the post title, status, and publication date to the user
3. **Require explicit confirmation** — state clearly that deletion is permanent
4. After confirmation, call `delete_post` with the post ID
5. Confirm deletion

## Output

- Confirmation that the post has been permanently deleted

## Notes

- **Deletion is irreversible** — always require explicit user confirmation
- Show the post title and status before deleting so the user can verify it's the right post
- If the post is published, warn that it will immediately become unavailable to readers
