# mateocerri.com

Personal site of Mateo Cerri. Static, English, zero client JavaScript.

## Stack

Astro 5, static output (`dist/`). Content in Markdown next to the code.
System serif body, Fraunces display, IBM Plex Mono labels. One accent
color (oxide `#C14A1F`) on paper `#FAF8F4`, sepia dark mode.

## Commands

| Command       | Action                        |
| ------------- | ----------------------------- |
| `npm install` | Install dependencies          |
| `npm run dev` | Dev server with live reload   |
| `npm run check` | Type + content check (Astro) |
| `npm run build` | Static build into `dist/`    |
| `npm run preview` | Preview the build locally  |

## Structure

- `src/pages/` — home (`index.astro`), `/contact`, `/blog`.
- `src/content/blog/` — posts as Markdown (filename = slug).
  Frontmatter: `title`, `date`, `description`.
  Posts close with a `## Recap` of concrete learnings; a `Takeaway`
  line only when the post earned one.
- `src/layouts/Base.astro` — shared head, fonts, favicon.
- `public/` — `styles.css`, `favicon.svg`, `llms.txt` (open summary
  for AI consumers; nothing hidden, nothing addressed to machines
  that a human cannot read).

## Deploy

Cloudflare Pages: build command `npm run build`, output directory
`dist/`, custom domain `mateocerri.com`.
