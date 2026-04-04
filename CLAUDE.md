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
- Service role key (for direct DB writes — never put in frontend code): `YOUR_SERVICE_KEY`
- Table: `camps` — 1003 rows as of Apr 3, 2026, RLS disabled (public read, intentional)

## Supabase tables
- `camps` — main camp listings (1003 rows as of Apr 3, 2026)
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

## How Claude works with Supabase directly

Claude can read and write the database without you touching Supabase at all. Just describe what you want in plain English.

### Reading data (uses publishable key — read-only, safe)
Claude runs `curl` commands against the Supabase REST API to query any table. Examples:
```
curl "https://ikhvkrbwwnapeofqpxdb.supabase.co/rest/v1/camps?select=id,camp_name&city_state=eq.Seattle, WA" \
  -H "apikey: sb_publishable_YPY0o2fRbGoDP1CBFoZb-A_11ucDenw" \
  -H "Authorization: Bearer sb_publishable_YPY0o2fRbGoDP1CBFoZb-A_11ucDenw"
```

### Writing data (uses service role key — required for INSERT/UPDATE/DELETE)
Claude runs `curl` commands with the service key for any change to the database:
```bash
# INSERT a new camp
curl -X POST "https://ikhvkrbwwnapeofqpxdb.supabase.co/rest/v1/camps" \
  -H "apikey: YOUR_SERVICE_KEY" \
  -H "Authorization: Bearer YOUR_SERVICE_KEY" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=representation" \
  -d '{"camp_name":"...", "city_state":"...", ...}'

# UPDATE a camp
curl -X PATCH "https://ikhvkrbwwnapeofqpxdb.supabase.co/rest/v1/camps?id=eq.123" \
  -H "apikey: YOUR_SERVICE_KEY" \
  -H "Authorization: Bearer YOUR_SERVICE_KEY" \
  -H "Content-Type: application/json" \
  -d '{"website_url":"https://..."}'

# DELETE rows
curl -X DELETE "https://ikhvkrbwwnapeofqpxdb.supabase.co/rest/v1/camps?id=in.(1,2,3)" \
  -H "apikey: YOUR_SERVICE_KEY" \
  -H "Authorization: Bearer YOUR_SERVICE_KEY"
```

### Supabase admin API (user management — service key required)
```bash
# List users
curl "https://ikhvkrbwwnapeofqpxdb.supabase.co/auth/v1/admin/users" \
  -H "apikey: sb_secret_..." -H "Authorization: Bearer sb_secret_..."

# Delete a user
curl -X DELETE "https://ikhvkrbwwnapeofqpxdb.supabase.co/auth/v1/admin/users/{user_id}" \
  -H "apikey: sb_secret_..." -H "Authorization: Bearer sb_secret_..."
```
Note: Admin API calls must come from the terminal (server-side). Supabase blocks service key use from the browser.

### What to just say to Claude
- "Add a new camp called X in Seattle for ages 6–12, runs Jun 9–13, website is..."
- "Update the website URL for Camp ABC to..."
- "Change the session dates for Pedalheads Bothell to..."
- "Delete camp ID 123"
- "Find all camps with no session dates"
Claude will run the curl command, confirm the result, and show you what changed.

### For bulk additions or schema changes
Claude will provide copy-paste SQL for the Supabase SQL Editor (supabase.com → project → SQL Editor).

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

## Current site state (as of Apr 3, 2026)
- Title: `popcamps | Washington Summer Camp Directory`
- H1: `Washington Summer Camp Directory`
- og:title / twitter:title: `PopCamps — Washington Summer Camp Directory`
- og:description: `Find the best summer camps in Washington state. Filter by age, type, and dates.`
- og:image: `https://popcamps.one/og-image.png` (1200×630px)
- SEO: meta description, canonical, Open Graph, Twitter Card, Schema.org, sitemap all in place
- Footer: dark green — "Listings updated regularly for summer 2026. · © 2026 PopCamps · Privacy Policy · Terms and Conditions" + disclaimer line
- My Calendar page: full summer toggle, Mon–Sun visible on both mobile and desktop, camp blocks stay within rows, icons removed from blocks, dates show in camp blocks on mobile
- City search: custom autocomplete dropdown (replaced native datalist — works on mobile)
- Camp owner portal: LIVE — magic link login, edit form, admin review queue (photo upload removed)
- Admin review page: LIVE — approve/reject edits before they go live
- "Updated" badge: appears on camp cards within 30 days of owner-approved edit
- Privacy Policy page: LIVE (`page-privacy`)
- Terms and Conditions page: LIVE (`page-terms`)
- Listing accuracy disclaimer: on every camp detail page and footer
- Page load: shows loading spinner (not dummy data) while Supabase loads
- Date filter: camps with no dates (null/TBD) are excluded from date searches; umbrella camps check sub-programs
- Auth/login (as of Apr 3, 2026):
  - Session restores immediately on refresh from stored token — no DB query needed to show logged-in state
  - Admin/owner/profile checks run in background after nav updates
  - Admin users also load personal profile name and calendar data (not just admin page)
  - logOut() clears UI instantly, awaits signOut with 5s cap to release auth lock
  - Login timeout: 30 seconds
  - Scheduled weekly agent (trig_01Q1ngAG9dHeZhxrcgjgmQha) runs Mondays 3pm UTC to research missing session dates

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
- Remaining session_dates cleanup: only 1 camp left (Redmond Ridge Junior Golf, ID 185 — "2026 dates not yet published")
- Facebook ads — launching soon

## Work log
Full history of all changes: `~/Documents/campquest/CAMPQUEST_WORK_LOG.md`
