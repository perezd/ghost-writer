---
description: Publish a draft Ghost blog post
argument-hint: "<post title, slug, or ID>"
---

# /publish-post

## Trigger

When the user runs `/publish-post` or asks to publish, go live with, or make a post public.

## Inputs

- **Post identifier** — title, slug, or ID of the draft to publish

## Process

1. Find the post using `list_posts` or `get_post`
2. Verify the post is currently in `draft` status
3. Present the post title and a brief content summary to the user for confirmation
4. After explicit user confirmation, call `update_post` with `status: "published"` and the current `updated_at`
5. Confirm publication

## Output

- Confirmation that the post is now live
- The post's slug (which forms its URL)

## Notes

- **Always confirm with the user before publishing** — this makes the post visible to readers
- If the post is already published, inform the user rather than re-publishing
- If the user wants to schedule instead, use `status: "scheduled"` (note: Ghost requires a `published_at` date in the future for scheduled posts)
