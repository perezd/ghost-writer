---
description: Draft a new blog post on Ghost
argument-hint: "<topic or title>"
---

# /draft-post

## Trigger

When the user runs `/draft-post` or asks to write, create, or draft a new blog post.

## Inputs

Gather from the user through conversation:

- **Topic or title** — what the post is about
- **Key points** — any specific points, arguments, or structure the user wants
- **Tags** — categories or tags for the post (optional)
- **Tone/audience** — who this is for and what voice to use (optional)

## Process

1. Read the **ghost-lexical** skill for Lexical format reference
2. Collaborate with the user on the content — outline first, then flesh out sections
3. Construct a valid Lexical JSON document following the skill's node structure
4. Use the `create_post` tool to create the post as a **draft** on Ghost
5. Share the post title and ID with the user for future reference

## Output

- Confirmation that the draft was created on Ghost
- The post ID and slug for future editing
- A brief summary of the post structure (sections, word count estimate)

## Notes

- Always create as `status: "draft"` — never publish directly from this command
- If the user provides a link to reference content, use Chrome MCP to read it first
- For long posts, build the Lexical document incrementally to avoid errors
- Ensure all Lexical nodes include required fields (see skill checklist)
