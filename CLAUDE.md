# PopCamps — Project Context for Claude

## What this project is
PopCamps (popcamps.one) is a Washington state summer camp directory. Parents search for camps by city, age, and date range. The owner is not a developer — always explain changes clearly before making them.

## Rules — read these first
- **Single file only**: All code lives in `index.html`. No separate JS or CSS files, ever.
- **No build tools**: No npm, no webpack, no frameworks. CDN links only.
- **Always push to GitHub after every change** — don't wait to be asked.
- **Never use the Supabase secret key** in frontend code.

## Tech stack
- Frontend: `index.html` (single file, all inline CSS + JS)
- Database: Supabase (PostgreSQL)
- Deployment: GitHub Pages → custom domain `popcamps.one`
- GitHub repo: https://github.com/Camps2026/popcamps
- Local repo: `~/Documents/campquest/`

## Supabase (publishable — safe for frontend)
- URL: `https://ikhvkrbwwnapeofqpxdb.supabase.co`
- Publishable key: `sb_publishable_YPY0o2fRbGoDP1CBFoZb-A_11ucDenw`
- Table: `camps` — 995+ rows, RLS disabled (public read, intentional)

## Formspree form endpoints
- List Your Camp: `https://formspree.io/f/mdapagpg`
- Claim Your Camp: `https://formspree.io/f/xqegeqgl`

## Current site state
- Title: `popcamps | Washington Summer Camp Directory`
- H1: `Washington Summer Camp Directory`
- SEO: meta description, canonical, Open Graph, Twitter Card, Schema.org, sitemap all in place
- og:image: `https://popcamps.one/og-image.png` (1200×630px, already pushed)
- Footer: dark green, "Listings updated regularly for summer 2026. · © 2026 PopCamps"
- My Calendar page: full summer toggle + empty week click-to-search
- Mobile: Show Filters button visible, calendar scrolls horizontally
- No Best Match sort dropdown (removed)
- Image upload UI removed from List Your Camp form (not functional yet)
- Claim listing copy updated to be honest — no self-service portal exists yet

## Planned future work
- Camp owner self-service portal (Phase 1: owner login → Phase 2: edit listing → Phase 3: photo uploads)

## Work log
Full history of all changes: `~/Documents/campquest/CAMPQUEST_WORK_LOG.md`
