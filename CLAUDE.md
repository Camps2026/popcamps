# PopCamps — Project Context for Claude

## What this project is
PopCamps (popcamps.one) is a Washington state summer camp directory. Parents search for camps by city, age, and date range. The owner is not a developer — always explain changes clearly before making them.

## Rules — read these first
- **Single file only**: All code lives in `index.html`. No separate JS or CSS files, ever.
- **No build tools**: No npm, no webpack, no frameworks. CDN links only.
- **Always push to GitHub after every change** — don't wait to be asked.
- **Never use the Supabase secret key** in frontend code.
- **Always read the file before editing** — never guess at line numbers or content.
- **Explain every change before making it** — owner is not a developer.

## Tech stack
- Frontend: `index.html` (single file, all inline CSS + JS)
- Database: Supabase (PostgreSQL)
- Deployment: GitHub Pages → custom domain `popcamps.one`
- GitHub repo: https://github.com/Camps2026/popcamps
- Local repo: `~/Documents/campquest/`

## Supabase (publishable — safe for frontend)
- URL: `https://ikhvkrbwwnapeofqpxdb.supabase.co`
- Publishable key: `sb_publishable_YPY0o2fRbGoDP1CBFoZb-A_11ucDenw`
- Table: `camps` — 994 rows, RLS disabled (public read, intentional)

## Supabase tables
- `camps` — main camp listings (994 rows as of Mar 31, 2026)
- `camp_owners` — links `auth.users.id` to `camps.id` (plain integer, no FK)
- `site_admins` — stores admin user_id; RLS disabled (publicly readable)
- `camp_edit_requests` — staging table for owner edits pending admin review
- `user_profiles` — stores parent user name, email, zip
- `user_children` — stores children profiles per parent user
- `user_schedules` — stores camps added to each child's calendar
- `user_saved_camps` — stores saved/favorited camps per user
- Storage bucket: `camp-photos` (public, unused — photo upload feature removed)

## camps table key columns
| Column | Description |
|---|---|
| `id` | Integer primary key |
| `camp_name` | Name of the camp |
| `camp_type` | Type: Sports, Arts, STEM, Outdoor, Academic, Music, Dance, Cooking, Chess, Variety |
| `city_state` | e.g. "Seattle, WA" |
| `display_type` | `standard`, `umbrella`, or `sub-program` |
| `parent_camp_id` | For sub-programs: stores the `camp_name` of the parent umbrella |
| `session_dates` | Comma-separated date ranges e.g. "Jun 22–26, Jun 29–Jul 3" |
| `hours` | e.g. "9am–3pm" |
| `age_min` / `age_max` | Integer age range |
| `website_url` | Camp website (all HTTPS, verified clean as of Mar 31, 2026) |
| `description` | Optional camp description (added for owner portal) |
| `owner_updated_at` | Timestamptz — set when owner edits are approved; drives "Updated" badge |

## How to add or edit camp listings
Claude can do this directly via Supabase REST API using the publishable key. Just describe what you want:
- "Add a new camp called X in Seattle for ages 6–12, runs Jun 9–13, website is..."
- "Update the website URL for Camp ABC to..."
- "Change the session dates for Pedalheads Bothell to..."
Claude will write and execute the SQL or REST call, confirm the result, and show you what changed.

For bulk additions or schema changes, Claude will provide copy-paste SQL for the Supabase SQL Editor.

## Formspree form endpoints
- List Your Camp: `https://formspree.io/f/mdapagpg`
- Claim Your Camp: `https://formspree.io/f/xqegeqgl`

## Admin user
- user_id: `db121d42-e139-4e4d-ab01-ab360c7010ed` (inserted into `site_admins`)
- Login: magic link via the "Camp Owner Login" section on the Portal page

## How to approve a camp owner claim
1. Claim arrives via Formspree → your email
2. Supabase → Authentication → Users → "Invite User" → enter their email
3. Copy their `user_id` from the users list
4. Table Editor → `camp_owners` → Insert Row: `user_id` + `camp_id`
5. Done — owner can log in and edit

## How to check registered users
- Supabase → Authentication → Users (shows email, signup date, last login)
- Table Editor → `user_profiles` (shows full name, zip code)

## Testing
- Use **MCP Chrome DevTools** to visually verify changes on mobile before and after edits
- Always test at 390x844 mobile viewport (iPhone size)
- Use `mcp__chrome-devtools__emulate` + `mcp__chrome-devtools__take_screenshot` to check layout
- Use `mcp__chrome-devtools__evaluate_script` to force-show UI states

## Current site state (as of Mar 31, 2026)
- Title: `popcamps | Washington Summer Camp Directory`
- H1: `Washington Summer Camp Directory`
- og:title / twitter:title: `PopCamps — Washington Summer Camp Directory`
- og:description: `Find the best summer camps in Washington state. Filter by age, type, and dates.`
- og:image: `https://popcamps.one/og-image.png` (1200×630px)
- SEO: meta description, canonical, Open Graph, Twitter Card, Schema.org, sitemap all in place
- Footer: dark green — "Listings updated regularly for summer 2026. · © 2026 PopCamps · Privacy Policy · Terms and Conditions" + disclaimer line
- My Calendar page: full summer toggle, Mon–Sun visible on both mobile and desktop, camp blocks stay within rows, icons removed from blocks
- City search: custom autocomplete dropdown (replaced native datalist — works on mobile)
- Camp owner portal: LIVE — magic link login, edit form, admin review queue (photo upload removed)
- Admin review page: LIVE — approve/reject edits before they go live
- "Updated" badge: appears on camp cards within 30 days of owner-approved edit
- Privacy Policy page: LIVE (`page-privacy`)
- Terms and Conditions page: LIVE (`page-terms`)
- Listing accuracy disclaimer: on every camp detail page and footer
- Login bug fixed: try/catch on login handler, regular users handled in onAuthStateChange
- Page load: shows loading spinner (not dummy data) while Supabase loads

## Camp type colors (imgClass)
| Type | Class | Color |
|---|---|---|
| Sports | img-sports | Green |
| Arts & Crafts | img-arts | Purple |
| STEM | img-stem | Blue |
| Outdoor | img-outdoor | Earth green |
| Academic | img-academic | Navy |
| Music | img-music | Pink |
| Dance | img-dance | Coral |
| Cooking | img-cooking | Orange |
| Chess | img-chess | Dark gray |
| Variety | img-variety | Teal |

## Planned future work
- Google Analytics (deferred until real traffic)
- Remaining session_dates cleanup (~66 camps still have non-date text)
- Facebook Page marketing (PopCamps — Washington Summer Camp Directory)
- Facebook ads (deferred — free organic posting first)

## Work log
Full history of all changes: `~/Documents/campquest/CAMPQUEST_WORK_LOG.md`
