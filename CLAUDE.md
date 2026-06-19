# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-page portfolio site for Richard DeVeau — novelist, creative director, and painter. No build step. Two files own everything: `index.html` (layout + logic) and `data.json` (all content).

## Development

```bash
npx serve .        # preview at http://localhost:3000
node -e "JSON.parse(require('fs').readFileSync('data.json','utf8')); console.log('valid')"  # validate JSON
```

Deploy: push to GitHub → Netlify auto-deploys `index.html` from the root.

## Architecture

### Content rule
All text, image URLs, and data arrays live in `data.json`. Nothing is hardcoded in `index.html`. Alpine.js binds everything via `x-text`, `x-bind`, `x-for`.

### data.json structure
Five top-level keys:
- `profile` — name, tagline, email, bio_summary, profile_image, links
- `in_plain_sight` — hero_video, cover_image, synopsis, buy_links, reviews (array with full_text on [0]), saffron_review
- `lights_out` — hero_video, audio_preview, cover_image, synopsis, synopsis_detail, tagline, buy_links, chapter_videos (15 items), reviews (9 items), saffron_review
- `creative` — headline, summary, brands, notable_work (with optional gallery array), video_intro, video_assets (3 items with vimeo_id + thumbnail), linkedin
- `artist` — statement, series, solo_exhibits, publications, highlights, paintings (34 items with url + title)

### Alpine.js state (`site()` function)
```
d             — loaded from data.json via fetch('./data.json')
activeVideo   — vimeo ID string or null (video modal)
popup         — { open, title, body, image, credit }
contact       — { open, success, submitting }
gallery       — { open, title, images[], current }
```

Key methods: `openVideo(id)`, `openPopup(title, body, image, credit)`, `openContact()`, `openGallery(title, images, startIndex)`, `openPaintings(startIndex)`.

Escape key priority: gallery → popup → contact → video.

### Modal stack (z-index order)
- Contact form: `z-[120]`
- Gallery lightbox: `z-[125]`
- Video (Vimeo iframe): `z-[100]`
- Popup (reviews/synopsis): `z-[110]`

### Page sections
1. **NAV** — fixed header, links to `#in-plain-sight`, `#lights-out`, `#creative`, `#artist`
2. **HERO** — 2-col grid (`lg:grid-cols-[1fr_480px]`), text left + profile photo right
3. **#in-plain-sight** — full-viewport video hero → below-hero with cover + Saffron review + RT Booksterr + 2 Amazon reader reviews
4. **#lights-out** — full-viewport video hero → dark panel with chapter carousel (15 videos) + reviews horizontal scroller (Saffron first, then 9 reader quotes)
5. **#creative** — bio/brands/notable-work left, notable-work cards right, then AI video section (2-col: intro text left + 3 portrait video cards right)
6. **#artist** — statement, exhibits/publications/awards grid, horizontal paintings carousel (34 items)

## Critical Gotchas

### Tailwind CDN config
Must use `tailwind = { config: {...} }` (creates the object) NOT `tailwind.config = {...}` (fails because `tailwind` is undefined at script parse time).

```html
<script>
  tailwind = { config: { theme: { extend: { fontFamily: { ... } } } } }
</script>
<script src="https://cdn.tailwindcss.com"></script>
```

### Background video
Use hardcoded `src` on `<video>` tags — never Alpine `:src` binding. Dynamic src binding does not trigger browser buffering. Also never put `x-cloak` on `<body>` as it blocks video from loading.

```html
<video autoplay muted loop playsinline src="FULL_URL_HERE"></video>
```

### Vimeo progressive URLs
The `%20%28720p%29` fragment in Vimeo progressive redirect URLs is a required part of the filename. Never clean or remove it.

### JSON string safety
Never use ASCII double quotes (`"`) inside JSON string values (e.g. in painting titles or descriptions with inch marks). Use single quotes or rephrase. Curly/smart quotes (`"` `"`) are safe.

### Cloudinary URLs
Always apply transformations: insert `f_auto,q_auto,w_N` between `upload/` and the version segment. Choose width by context: `w_64` favicons, `w_200` avatars, `w_800` standard, `w_1000` hero photos, `w_1500` full-bleed.

```
https://res.cloudinary.com/CLOUD/image/upload/f_auto,q_auto,w_800/v.../image.jpg
```

### Netlify Forms
The hidden `<form name="contact" data-netlify="true">` in the DOM is required for Netlify's build bot to register the endpoint. The live form submits via `fetch('/', { method: 'POST', ... })` with `form-name=contact` in the URL-encoded body.

## Reusable CSS patterns

`.carousel-track` — horizontal scroll snap container (used for chapter videos, reviews, and paintings). Cards use `flex-none` + inline `style="scroll-snap-align: start"`. Arrow buttons call `scrollBy()` via vanilla JS `onclick`.

Gallery cards with background images use Alpine `:style` binding:
```js
{ backgroundImage: 'url(' + work.gallery[0] + ')', backgroundSize: 'cover', backgroundPosition: 'center' }
```

## Typography

- Headings / blockquotes: Playfair Display (serif) — applied via CSS `h1,h2,h3,blockquote,blockquote p { font-family: ... }`
- Body: Inter (sans-serif) — set on `body`
- Both loaded from Google Fonts in `<head>`
- Use `font-serif` Tailwind class on `<p>` elements that need Playfair (pull-quotes, subtitles) since the CSS rule only targets heading/blockquote elements

## Source files (read-only)
Content was extracted from these — do not modify:
- `information-files/RichardDeVeau-author-In-Plain-Sight.html`
- `information-files/RichardDeVeau-author-Lights-Out .html` (note space before `.html`)
- `information-files/RichardDeVeau-Copy-Writer.html`
- `information-files/RichardDeVeau-Art.html`
- `information-files/richard-notes.rtf`
- `information-files/reads-saffron-book-review-*.rtf`
- `information-files/in-plain-sight-Reviews.docx`
