---
name: new-blog-post
description: Create a new blog post for this Hugo site. Use when the user wants to write a new blog post, start a draft, or brain-dump ideas for a post.
---

# New Blog Post

Help the user create and collaboratively draft a new blog post.

## Process

1. **Get the slug** — ask the user for a short slug (or derive one from their topic/title if they've given enough context). Use kebab-case, e.g. `my-post-topic`.

2. **Create the file** — run `hugo new content/blog/$(date +%Y-%m-%d)-SLUG.md` using the current date and the slug. This uses the archetype and sets the timestamp correctly.

3. **Set front matter** — open the generated file and update:
   - `title` — a proper title (confirm with user if unsure)
   - `categories` — pick one from: `"Build Log"`, `"Deep Dive"`, `"Honest Review"`, `"Project Jumps"`, `"Quick Tip"`
   - `tags` — relevant lowercase tags
   - `description` — one sentence for previews/SEO
   - Leave `draft: true` and `image: ""` until the user is ready

4. **Collaborative drafting** — the user's voice is the priority. They may brain-dump raw notes, stream-of-consciousness, or rough sentences. Your job is to shape that into a structured post with `##` section headings while preserving:
   - Short, punchy sentences
   - Honest/self-deprecating tone
   - Em-dashes for asides
   - Their specific word choices and phrasings

5. **Iterate** — edit section by section based on user feedback. Flag anything that sounds more like "AI words" than their words and offer a blend.

6. **Hero image** — remind the user that hero images go in `/static/images/uploads/` at **1200×675px** (16:9). Once they provide a filename, add it to the `image:` front matter field.

7. **Publish** — when the user is ready, use the **Publish Blog Post** VSCode task (Terminal → Run Task → Publish Blog Post) with the draft file open and active. It will: unset the draft flag, commit, push, and run `netlify watch`. Stop the Hugo debug session manually first — there's no way to stop it from within a VSCode task.

## Notes

- Use plain markdown links (`[text](/blog/slug)`) for internal cross-links — never Hugo `relref` shortcodes. The blog has moved platforms before and may again.
- The callout shortcode (`{{< callout type="note" >}}` or `type="warning"`) is available for asides.
- Hugo won't render posts dated in the future unless the server is started with `--buildFuture`. The archetype uses `hugo new` which stamps the current time, so this should not be an issue — but double-check if a post isn't appearing locally.
