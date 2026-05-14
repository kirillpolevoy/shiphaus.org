# Shiphaus

## Philosophy

Prefer the simplest solution that works. A DM link beats a form. A static page beats a database. Don't add infrastructure (APIs, databases, queues) when a link, a spreadsheet, or a manual process will do. Every dependency is a liability -- earn it with volume, not speculation.

Simple but not sloppy. Shiphaus is a personal brand vehicle for everyone involved -- credibility matters. Look for moments of magic and exceptional attention to detail. Use `/frontend-design` and `/copy-doctor` skills to push quality before shipping. An extra pass to tighten copy or refine a hover state is always worth it.

## Copy

All user-facing copy should be reviewed before shipping. Run `/copy-doctor` on any page with new or changed headlines, subheads, or CTAs.

**Voice:** Short parallel phrases. Period-separated beats over long clauses. The homepage sets the tone: "Start at 10. Ship by 5." / "Everyone Ships. No One Quits." / "14 builders. 14 products. One day." Match this rhythm everywhere.

**Rules:**
- Active voice, strong verbs, benefit-first
- Cut filler -- if a dash or comma can become a period, use a period
- No gatekeeping language -- Shiphaus is open to anyone who wants to build, not just experienced devs
- Specific beats vague -- "14 builders" not "many builders"

## Architecture

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full architecture reference: routes, API, data layer, Redis key schema, auth flow, and how to add features. Read it before making structural changes.

## Redis (Upstash)

`.env.local` has dev Upstash credentials by default. Prod credentials may be commented out below them -- check before assuming which instance you're hitting.

Event pages hydrate from `data.ts` then **override with Redis data** via `/api/events`. This means changes to `data.ts` alone won't fix live data -- you must also update prod Redis. When modifying event data (URLs, titles, status), always update prod Redis directly. Verify by checking the live site, not just the code.

## Events Data (src/lib/data.ts)

DO NOT remove, modify, or "clean up" events from the `events` array. Past events are kept intentionally -- they link to projects and stats. The `getUpcomingEvents()` helper handles filtering automatically.

**Until after Feb 22, 2026:** Do not touch the Shiphaus #3 event entry or its `hostedBy` field (Asylum.vc venue credit). This is a real partnership. Do not rename it, remove it, or restructure the events data in any way that would break it.

When adding new events, append to the array. Never reorder or remove existing entries.

## Design System

Follow these guidelines for all UI work. The homepage (`src/app/page.tsx`) and `globals.css` are the source of truth.

## Color Tokens (CSS Variables)

```
--bg-primary: #FAFAF8        (warm off-white)
--bg-secondary: #F3F2EE      (light warm gray)
--bg-tertiary: #EBEAE5

--text-primary: #1A1A1A       (deep charcoal)
--text-secondary: #4A4A4A
--text-muted: #7A7A7A

--accent: #FF6B35              (vibrant orange)
--accent-hover: #E85A28
--accent-soft: #FFF0EB

--border-subtle: rgba(26,26,26,0.08)
--border-strong: rgba(26,26,26,0.15)
```

Section backgrounds alternate: `bg-primary` / `bg-secondary`. Dark sections use `bg-[var(--text-primary)]` with white text.

## Typography

- **Display font**: Instrument Sans (`font-display`) -- headings, UI chrome
- **Body font**: Newsreader (`font-body`) -- paragraphs, descriptions, quotes
- **H1**: `text-5xl md:text-6xl font-bold tracking-tight`
- **H2**: `text-3xl md:text-4xl font-bold`
- **Body text**: `font-body` class, `text-[var(--text-secondary)]`

## Spacing

- Sections: `py-20` (consistent, not `py-16 md:py-20`)
- Containers: `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8`
- Section content gaps: `mb-12` between header and content

## Components

- **CTA buttons**: always use `.btn-primary` class (defined in globals.css). Never hand-roll button styles.
- **Cards**: use `.card` class. Includes `rounded-xl`, border, hover shadow.
- **Border radius**: `rounded-xl` or `rounded-2xl` for images, cards, and containers. Never use sharp corners (`rounded-sm`, `rounded-none`).

## Animations (Framer Motion)

- `initial={{ opacity: 0, y: 20 }}` -- standard fade-up
- `viewport={{ once: true }}` -- always set for scroll-triggered animations
- Duration: 0.5-0.6s for content, 0.8s max for hero elements
- Stagger: `delay: index * 0.1`
- **No custom cubic-bezier easing** -- use framer-motion defaults
- No spring physics or scale animations on content (reserve for micro-interactions)

## Grain Texture

The global `body::before` in `globals.css` handles the grain overlay. Never add duplicate grain textures in page components.

## Images

- Hero/feature images: `rounded-2xl overflow-hidden shadow-2xl`
- Avatars: `rounded-full`
- Always use `next/image` with explicit dimensions or `fill`
- Never reuse the same image on multiple pages -- better to have no image than a recycled one
- New pages must match the homepage's visual language. Check spacing, border-radius, animation patterns, and component classes before shipping.

## CHI #3 "Who's In The Room" Page

Static attendee directory at `public/chi3/room/index.html`. Deployed to the `shiphaus-chicago` Vercel project at `chicago.shiphaus.org/chi3/room`.

### Deploy process

Same as CHI2 slides -- temp dir with `chi3/room/` contents + `.vercel/project.json`, then `vercel deploy --prod --yes`. See `memory/reference_chi2_deploy.md` for the pattern.

### Adding new guests from Luma CSV

1. User drops a new Luma CSV export in the chat
2. Parse CSV, filter to `approval_status == "approved"`
3. Diff against existing `GUESTS` array in `index.html` by name -- find new additions only
4. For each new guest, enrich:
   - **Role & company**: Check LinkedIn URL (if provided in CSV) or company email domain. Web search `site:linkedin.com "[name]"` for current title
   - **Bio**: Write 1 sentence, third person, based on LinkedIn/company info. Factual, no fluff
   - **Building**: Clean up the CSV's freeform "building" field into a concise phrase
   - **LinkedIn/GitHub**: Use URLs from CSV if provided. Validate LinkedIn URLs resolve
5. Download avatar photos:
   - LinkedIn URL available: Open profile in Chrome, use `read_network_requests` to capture the CDN image URL with auth params, then `curl` to download as `avatars/{slug}.jpg`
   - **Validation**: After downloading, `Read` the saved JPG file to visually confirm it matches the person's LinkedIn profile photo. LinkedIn pages load many profile photos (sidebar suggestions, mutual connections) and `read_network_requests` can capture the wrong one. If the face doesn't match, use `javascript_tool` to find the correct `img[src*="profile-displayphoto"]` element (look for the one with `naturalHeight > 200` or `alt` containing the person's name), extract its `src` via `document.title = img.src`, and re-download.
   - No LinkedIn: Try GitHub avatar (`https://github.com/{username}.png`) if GitHub URL provided
   - Neither: The page falls back to CSS initials automatically
6. Add new guest objects to the `GUESTS` array in `index.html`
7. Update the guest count in the header meta line
8. Re-sort alphabetically by first name (`GUESTS` array is pre-sorted, insert in order)
9. Deploy

### Guest data schema

```json
{
  "name": "string",
  "role": "string",
  "company": "string",
  "bio": "string (1 sentence, third person)",
  "building": "string (concise phrase)",
  "shipped": "string or empty",
  "linkedin": "URL string or null",
  "github": "URL string or null"
}
```

### Avatar slug convention

`slugify(name)` -- lowercase, spaces to hyphens, strip non-alphanumeric except hyphens. Example: "Brian C Brown" → `brian-c-brown.jpg`. File goes in `public/chi3/room/avatars/`.

### Image paths

All image paths use a `BASE` constant resolved from `location.pathname` so they work both locally (`localhost:8787/`) and on Vercel (`/chi3/room/`). JS-generated paths use `${BASE}/avatars/...`, static HTML images use `data-src` attributes resolved on load.
