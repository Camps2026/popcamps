# CampQuest Work Log

## Project Overview
CampQuest is a Seattle/PNW summer camp directory for parents. Single `index.html` file, no build tools. Database is Supabase (PostgreSQL), deployed on GitHub Pages.

- **Repo:** https://github.com/Camps2026/campquest
- **Supabase project:** CampsQuest (production)
- **Table:** `camps` — 828+ rows as of Mar 2026
- **Supabase URL:** `https://ikhvkrbwwnapeofqpxdb.supabase.co`
- **Publishable key:** `sb_publishable_YPY0o2fRbGoDP1CBFoZb-A_11ucDenw` (safe for frontend)
- **Local file:** `~/Documents/campquest/index.html`

---

## Database Structure

### `camps` table key columns:
| Column | Description |
|---|---|
| `camp_name` | Name of the camp |
| `camp_type` | Type (Sports, Arts, STEM, etc.) — can be compound e.g. "Academic, Variety" |
| `city_state` | e.g. "Seattle, WA" |
| `display_type` | `standard`, `umbrella`, or `sub-program` |
| `parent_camp_id` | For sub-programs: stores the `camp_name` of the parent umbrella |
| `session_dates` | Comma-separated date ranges e.g. "Jun 22–26, Jun 29–Jul 3" |
| `hours` | e.g. "9am–3pm" |
| `subject_tag` | Used for tagging (Bike, etc.) |

### Display type counts (as of Mar 23, 2026):
- `standard`: 431
- `sub-program`: 399
- `umbrella`: 32

---

## Umbrella / Sub-Program Structure

Some camps have multiple venues. These are structured as:
- One **umbrella** row (`display_type = 'umbrella'`)
- One or more **sub-program** rows (`display_type = 'sub-program'`, `parent_camp_id` = umbrella's `camp_name`)

### SQL pattern to add a new umbrella + sub-program:
```sql
-- Step 1: Convert existing row to umbrella
UPDATE camps
SET display_type = 'umbrella'
WHERE camp_name = 'EXACT CAMP NAME HERE';

-- Step 2: Insert venue as sub-program
INSERT INTO camps (camp_name, camp_type, city_state, website_url, age_min, age_max,
  registration_type, overnight_camp, before_after_care, meals_available,
  listing_status, priority, display_type, parent_camp_id, subject_tag, session_dates, hours)
VALUES (
  'Venue Name',
  'Sports', 'City, WA',
  'https://example.com',
  '2', '12', 'External Website', 'No', 'No', 'No',
  'Unclaimed', 'High', 'sub-program',
  'EXACT PARENT CAMP NAME',
  'Tag',
  'Jun 22–26, Jun 29–Jul 3',
  '9am–3pm'
);
```

### Useful Supabase queries:
```sql
-- Check all umbrella camps and their sub-program counts
SELECT u.camp_name, u.city_state, COUNT(s.id) AS sub_program_count
FROM camps u
LEFT JOIN camps s ON s.parent_camp_id = u.camp_name AND s.display_type = 'sub-program'
WHERE u.display_type = 'umbrella'
GROUP BY u.camp_name, u.city_state
ORDER BY sub_program_count ASC, u.camp_name;

-- Check sub-programs for a specific camp
SELECT camp_name, display_type, parent_camp_id, session_dates
FROM camps
WHERE parent_camp_id = 'EXACT CAMP NAME HERE';

-- Check all distinct display_type values
SELECT DISTINCT display_type, COUNT(*) as count
FROM camps GROUP BY display_type ORDER BY count DESC;
```

---

## Pedalheads Bike Camps — Data Entry (Mar 2026)

SQL was generated for these — **do not re-run**, already in database:

### Bothell ✅
- Umbrella: `Pedalheads Bike Camp – Bothell`
- Venue: UW Bothell Campus (Discovery Hall)
- Sessions: Jun 22–26, Jun 29–Jul 3, Jul 6–10, Jul 13–17, Jul 20–24, Jul 27–31, Aug 3–7, Aug 10–14, Aug 17–21, Aug 24–28
- Hours: 9am–3pm

### Vancouver ✅
- Umbrella: `Pedalheads Bike Camp – Vancouver`
- Venue: East Vancouver Community Church
- Sessions: Jun 23–26, Jun 29–Jul 3, Jul 6–10, Jul 13–17
- Note: North Vancouver venue was intentionally excluded.

### Woodinville ✅
- Umbrella: `Pedalheads Bike Camp – Woodinville`
- Venue: Bear Creek United Methodist
- Sessions: Jun 22–26, Jun 29–Jul 3, Jul 6–10, Jul 13–17, Jul 20–24

---

## Front Page UI Changes (Mar 23, 2026)

All changes made to `~/Documents/campquest/index.html`.

### Hero / Search Bar
- Removed subheadline "Browse camps. Find the perfect fit and connect directly with camp staff."
- Removed emoji icons from all search field labels (📍 🎂 📅)
- Location field: now uses `<datalist>` autocomplete — cities populate from Supabase on load
- Location field: placeholder changed to "City"
- Location field: added ✕ clear button (appears when user types, clears field on click)
- Age field: changed from age ranges (4–6 yrs) to exact ages 1–17
- Start Date: when user selects a start date, End Date auto-sets to the next day (only if end date is empty or earlier)

### Filters Panel
- Filter panel is now scrollable (`max-height` + `overflow-y:auto`) — "Options" section is reachable
- Added **Search by Name** input at the top of filters
- **Camp Type** chips updated:
  - Removed: Day Camp, Swimming, Science, Writing, Media, Health
  - Added: Academic, Variety
  - Changed: "Outdoor" → "Outdoor / Nature"
- Removed **Registration Method** filter entirely
- Added **Location** filter — chips auto-populate from database cities; "Multiple locations, WA" excluded
- Added **Date Range** filter (Start + End date inputs)
- Renamed **Amenities** → **Options**

### Camp Cards
- Removed "Camp site" button from regular camp cards
- Fixed umbrella label overlap with camp type badge
- Card top colors are now type-specific:

| Camp Type | Color |
|---|---|
| Sports | #185FA5 (blue) |
| Arts | #534AB7 (purple) |
| STEM | #BA7517 (amber) |
| Outdoor / Nature | #2D6A4F (forest green) |
| Music | #C1440E (red) |
| Dance | #D4537E (pink) |
| Cooking | #3B9E6A (teal) |
| Academic | #5F5E5A (slate) |
| Variety | #7B3FA0 (purple) |
| Neurodiversity | #0F7E9E (teal-blue) |

- Compound camp types (e.g. "Academic, Variety") now use the **first type's color** instead of falling back to default

### Data Cleanup
- Deleted duplicate: `UW Youth & Teen Programs` (kept `UW Robinson Center – Summer Programs`)

---

## Camp Detail Page Changes (Mar 23, 2026)

### Layout Redesign
- Replaced tall hero + sidebar layout with compact banner + facts strip + scrollable body
- New structure: `.detail-banner` (160px color bar) → `.detail-header` (camp name) → `.detail-facts-row` (Ages, Hours) → `.detail-body` (sessions + CTA card)
- Removed duplicate website links from detail page

### Session Dates Display
- Sessions now render as individual lines (one per date entry), not a clump
- Each line styled with left border in moss green, sky blue background
- Price info stripped from display using `stripPrice()` function (e.g. "$585" removed)
- `isLikelySessionItem()` filter added — removes junk text (addresses, financial aid notes, early bird promos) while keeping real session entries; falls back to showing all if filter removes everything

### CTA Button
- Button text changed from "Visit Camp Website" → **"Check Availability & Register"**
- Sub-program button also updated to "Check Availability & Register ↗"
- Explainer text updated: "Spots may fill up — click below to check current availability and register on camp website."

### Hours Default
- Camps with NULL hours in Supabase previously showed "9am–3pm" as a default
- Changed fallback to **"See website"** so parents aren't misled

### Search by Name Filter
- Added ✕ clear button inside the Search by Name input in the filters panel
- Button appears when user types, clears field and re-runs filter on click

---

## Database Updates (Mar 23, 2026)

### Session Dates Added / Fixed
| Camp | Action |
|---|---|
| Alki Adventure Camp – SUP (West Seattle) | Added 13 weekly sessions: Jun 8–12 through Aug 31–Sep 4 |
| Camp Koinonia | Added 9 sessions: Jun 22–26 through Aug 17–21 |
| Woodland Park Zoo Camps | Added 9 sessions Jun 22–Aug 28 (Jul 13–15 is 3-day week); hours set to 9am–3:30pm |
| Camp Solomon Schechter | Reformatted session_dates from pipe-separated to comma-separated for proper display |
| Bellevue Children's Academy Summer Programs | Added full 8-session schedule (Jun 15–Aug 7); hours set to 8:30am–3pm |
| Pacific Northwest Ballet Summer Programs | Updated URL, ages (12–18), session (Jul 6–Aug 7, 5-week intensive), overnight=Yes |

### Umbrella Conversions
**Camp Kirby** — converted from `standard` to `umbrella` with two sub-programs:
- `Camp Kirby – Overnight Camp` (ages 6–18, Jun 28–Aug 14, 7 sessions + weekend stays, overnight=Yes)
  - Website: https://www.campfiresamish.org/camp-kirby-resident-camp/
- `Camp Kirby – Day Camp` (ages 6–14, Jun 29–Aug 14, 7 sessions, 9am–4pm, overnight=No)
  - Website: https://www.campfiresamish.org/camp-kirby-day-camp/

---

## Database Updates (Mar 25, 2026)

### New Capability: Direct Supabase API Access
Claude can now read and write directly to the Supabase database via the REST API using curl — no more copy/pasting SQL. All database edits below were made this way.

### Camps Removed
| Camp | Reason |
|---|---|
| Camp Cispus | Removed per owner |
| Cispus Learning Center | Removed per owner |
| Seattle Country Day School Summer Programs | Removed per owner |
| Seattle Art Museum Summer Workshops | Removed per owner |
| Kirkland Tennis Center Summer Camps | Removed per owner |
| Seattle Youth Ballet Summer Intensive | Removed per owner |
| Seattle Adaptive Sports Youth Camps | Removed per owner |
| Woodland Park Zoo ZooCrew | Duplicate/superseded by main Woodland Park Zoo Camps entry |

### Camps Added (New)
| Camp | City | Notes |
|---|---|---|
| Camp Galileo – Bellevue | Bellevue | Ages 5–14, Jun 22–Jul 24, before/after care |
| Camp Galileo – Burien | Burien | Ages 5–14, Jun 22–Jul 24, before/after care |
| Camp Galileo – North Seattle | Seattle | Ages 5–14, Jun 22–Jul 24, before/after care |
| Camp Galileo – Seattle (Madison Valley) | Seattle | Renamed from "Camp Galileo", same details |
| Seattle ReCreative Summer Camps – Greenwood | Seattle | Arts, Jun 15–Aug 28, 9am–1pm (aftercare to 3pm), Ages 4–10 |
| Seattle ReCreative Summer Camps – Georgetown | Seattle | Arts, Jun 22–Aug 28 (no camp weeks 1 & 3), same details |
| Seattle Gymnastics Academy – Summer Fun Camps (Ballard) | Seattle | Sports, Jul 6–Aug 28, Ages 3–10 |
| Seattle Gymnastics Academy – Summer Fun Camps (Burien) | Burien | Same details |
| Seattle Gymnastics Academy – Summer Fun Camps (Columbia City) | Seattle | Same details |
| Seattle Gymnastics Academy – Summer Fun Camps (Lake City) | Seattle | Jun 22–Aug 21 (different dates) |
| Seattle Gymnastics Academy – Summer Fun Camps (Mill Creek) | Mill Creek | Same as Ballard |
| Tennis Center Sand Point – Junior Summer Camps | Seattle | Sports, Jun 22–Aug 21, Ages 5–18, 9:30am–12:30pm or 1–4pm |
| TOPs Summer Camps at Eastside Tennis Center | Kirkland | Sports, 10 weeks Mon–Thu Jun 22–Aug 27, Ages 4–14 |
| ARC School of Ballet – Student Summer Intensive (Seattle) | Seattle | Dance, Ages 11–23, Jul 6–Aug 21 |
| ARC School of Ballet – Summer Intensive (Leavenworth) | Leavenworth | Dance, Ages 13–17, Jul 27–31 |
| Mariners Diamond Camps – Magnuson Park (Seattle) | Seattle | Sports, Jul 6–9 |
| Mariners Diamond Camps – Starfire Sports Complex (Renton) | Renton | Sports, Jul 6–9 |
| Mariners Diamond Camps – Puyallup Valley Sports Complex (Puyallup) | Puyallup | Sports, Jul 13–16 |
| Mariners Diamond Camps – Moshier Memorial Park (Burien) | Burien | Sports, Jul 13–16 |
| Mariners Diamond Camps – Capitol Little League (Olympia) | Olympia | Sports, Jul 27–30 |
| Mariners Diamond Camps – Rainier Playfield (Seattle) | Seattle | Sports, Jul 27–30 |
| Mariners Diamond Camps – Union High School (Vancouver) | Vancouver | Sports, Aug 3–5 |
| Mariners Diamond Camps – Meadowdale (Lynnwood) | Lynnwood | Sports, Aug 10–13 |
| Mariners Diamond Camps – Marymoor Park (Redmond) | Redmond | Sports, TBD |
| Mariners Diamond Camps – Northshore Athletic Fields (Lake Stevens) | Lake Stevens | Sports, Jul 20–23 |

### Camps Updated
| Camp | Changes |
|---|---|
| Camp Hamilton | Removed time info from session dates; hours → "See website" |
| Camp Hamilton | Removed all ($1,240) pricing from session dates |
| Villa Academy Summer Camps | Session dates → Jun 22–Jul 31; before/after care → After Care Only; Ages → 4–17 |
| Wing Luke Museum Youth Camps | Hours → 9:30am–4pm |
| Emerald City Karate Camps | Hours → 9am–4pm |
| Redmond Parks & Recreation Summer Camps | URL updated; session dates → Jun–Aug; Ages → 6–12 |
| Kirkland Parks – Junior Summer Day Camp | URL updated to PDF; session dates → Jun 23–Aug 29; Ages → 3–18 |
| Seattle Prep Panther Summer Camps | Hours → 8:30–11am or 12–2:30pm; before/after care → After Care Only |
| Bush School Summer Programs | Session dates → Jun 15–Aug 14; Ages → 7–18; hours → Varies; overnight → Yes; meals → Yes |
| Rain City Fencing Center Camps | URL updated; session dates → Jul 13–Aug 21; hours → 9am–12pm or 1–4pm; Ages → 8–18 |
| Moss Bay Kids Camps | URL updated; session dates → Jun 8–Aug 28; hours → 8:45am–3:15pm; before/after care → After Care Only |
| Mariners Diamond Camps (all 6 existing) | Hours → 9am–3pm; age_min → 4 |
| Sound FC Soccer Camps | Session dates cleared |
| Northwest Boychoir Summer Camps | URL updated; session dates → Jul 20–24; hours → 10am–2pm; Ages → 5–7 |
| Seattle ReCreative Summer Camps – Greenwood | Full themed session schedule added (11 weeks) |
| Seattle ReCreative Summer Camps – Georgetown | Full themed session schedule added (9 weeks) |
| School of Rock Bellevue Summer Camps | Full 10-camp schedule added; hours → 9am–3pm (Rookies 10am–2pm); Ages → 6–18 |
| Kirkland Arts Center Youth Camps | Full 9-week themed schedule added; hours → 9am–3pm |
| Bricks 4 Kidz Camps Thurston County | URL updated; "(reg opens 5/11)" removed from session dates |
| Seattle Area German American School Summer Camps | City corrected to Seattle; session dates updated (4 themed summer weeks); hours → 9am–1pm or 9am–3pm; Ages → 2–10 |
| Lakeside School Summer Programs | Ages → 8–18; full schedule for all 5 program tracks added; hours → Varies |
| Seattle Girls' Choir Summer Camps | Cleaned up session dates (removed prices, grades, times) |
| Seattle Youth Symphony Orchestra Summer Music | City corrected to Shoreline; Ages → 6–15; session dates updated with both sessions; hours added |
| Seattle Humane Youth Camps | URL updated; Ages → 7–14; hours → 8:30am–3:30pm Mon–Thu; full session schedule for Kids and Teen programs |
| SAMBICA | Removed all "(FULL)" from session dates; reordered chronologically |
| Camp Edward | URL updated to Cub Resident Camp page; session dates → 5 sessions Jul 23–Aug 23; Ages → 6–12 |
| Black Lake Bible Camp | Session dates updated with Pathfinders (Gr 1–6) and Explorers (Gr 7–9) programs; Ages → 6–15 |
| Miracle Ranch | URL updated; session dates → 3 programs (Teen Weekend, Middle School, Junior Camp); Ages → 9–18 |
| Lazy F Camp & Retreat Center | URL updated; full day camp schedule (8 themed weeks) + overnight camps (7 programs) added; Ages → 6–19 |
| Village Theatre KIDSTAGE – Issaquah | URL updated; financial aid → Yes; full session-by-session schedule Jun 22–Aug 14 added |
| Gage Academy of Art Youth Camps | URL updated; session dates → Jun 26–Aug 28; hours → 9:30am–3:30pm; before/after care → Yes; Ages → 6–18 |
| Seward Park Audubon Nature Camps | URL updated; Ages → 6–9; hours → 9am–3pm; 7 named sessions Jun 22–Aug 14 |
| Seattle Mountaineers – Summer Camps | overnight → Yes; Ages → 6–15; full day camp + overnight camp schedule added; meals → Yes |
| Woodland Park Zoo Camps | URL updated; before/after care → Yes; hours updated |
| Seattle Girls' School – Beginner Volleyball Camp | Hours → 9am–12pm |
| Seattle Girls' School – SGS Rock Camp | Hours → 9am–4pm |
| Seattle Girls' School – Beginner/Intermediate Volleyball Camp | Hours → 1–4pm |
| Seattle Girls' School – STEAM Camp | Hours → 9am–3pm; before/after care → Yes |
| Northwest Boychoir Summer Camps | URL, dates, hours, ages corrected |
| Play-Well TEKnologies – Shoreline | Full session schedule with camp names added; hours added; age max → 14 |
| Play-Well TEKnologies – Mercer Island | Full session schedule with camp names added; hours added |
| Play-Well TEKnologies – Kent | Full session schedule with camp names added; hours added |
| Play-Well TEKnologies – Issaquah | Full session schedule with camp names added; hours added |
| Play-Well TEKnologies – North Bend | Full session schedule with camp names added; hours added |
| Play-Well TEKnologies – Covington | Full session schedule added; hours added; age max → 8 |
| Play-Well TEKnologies – Federal Way | Session details added; hours → 9am–12pm |
| Play-Well TEKnologies – Friday Harbor | Reformatted to consistent style; hours → 9am–12pm or 1–4pm |
| TOPs Summer Camps at Eastside Tennis Center | URL corrected to topskirkland.org/summercamps |
| Seattle Gymnastics Academy (all 5 locations) | Renamed from "Seattle Gymnastics Boosters Camps" to consistent naming format |
| Lakeside School Summer Programs | Ages corrected; full multi-program schedule added |

### Data Cleanup
- Removed 10 duplicate Pedalheads – Sand Point Elementary School entries (kept id 891)
- Noted multi-location naming convention: same-city locations use "Camp Name – Neighborhood" format (consistent with iD Tech, Arena Sports, Code Ninjas pattern)

---

## Database Updates (Mar 25, 2026) – Session 2

### New Entries Added
| Camp | Details |
|------|---------|
| Village Theatre KIDSTAGE – Everett (id:1332) | New entry; Arts; Everett WA; ages 5–18; 8 weeks Jun 22–Aug 14 with full session listing |
| Wise Camps – Bellevue (id:1333) | New entry; STEM; Bellevue WA; ages 4–14; Jul 13–17, 20–24, 27–31; 9am–3:30pm |
| IncrediCamps – Bothell (id:1334) | New entry; Variety (filmmaking, robotics, arts, music); Bothell WA; ages 7–13; 6 weeks Jun 22–Jul 31; Morning 9am–12pm / Afternoon 1–4pm |

### Removed
- City of Bellevue Parks – Summer Camps (id:180)
- WSU Cougar Baseball – Youth Development Camp (id:215)
- Nike Soccer Camp – Tacoma / Meeker Middle School / Tacoma Stars (id:203)
- Kirkland Parks – Yoga Camps (id:357)
- Kirkland Parks – Girls Flag Football League (id:367)

### Updated
| Camp | Changes |
|------|---------|
| Village Theatre KIDSTAGE – Issaquah (id:174) | Full session listing added from user-pasted data |
| Wise Camps – Issaquah (id:175) | Hours → 9am–3:30pm; age min → 4; URL → wisecamps.com/issaquah-summer-camps/ |
| FISH Thrills & Gills Nature Camp (id:176) | URL updated; 3 age-specific session camps added; ages → 4–10; financial aid → Yes; hours → See website |
| PRO Club Bellevue – Youth Camps (id:178) | Confirmed accurate; no changes needed |
| Stroum Jewish Community Center (SJCC) – Summer Camps (id:179) | URL → sjcc.org/j-camp; all 9 weekly sessions added; hours → 9am–4pm (extended care 7:30am–6pm); ages → 4–16; financial aid → Yes |
| Nike Golf Camp – Seattle University (id:184) | Ages → 10–18; meals → Yes; overnight → Yes; hours → See website |
| Redmond Ridge Junior Golf Camps (id:185) | Ages → 7–16; session dates simplified to "2026 dates not yet published" |
| Snohomish Valley Golf Center – Junior Golf Camps (id:186) | URL updated; all 10 Tue–Thu sessions added (Jun 23–Aug 27); hours by age group added |
| Nike Golf Camp – Battle Creek Golf Course (id:187) | Added Jul 13–16 session; hours by age group; ages → 7–17; meals → Yes |
| Woodinville Sports Club – Golf & Adventure Summer Camps (id:189) | Ages → 3–18; session dates cleaned up; hours → Half-day or full-day |
| Nike Golf Camp – Hawks Prairie Golf Club (id:190) | Sessions corrected to Jun 22–25, Jul 13–16, Aug 3–6; hours → 9am–12pm; ages → 8–15 |
| PGA Junior Golf Camp – Esmeralda Golf Course (id:191) | Session → Aug 11–13; hours → 8am–12pm; ages → 7–13 |
| PGA Junior Golf Camp – Club Green Meadows (id:192) | 4 sessions added; hours → AM 9am–12pm / PM 1–4pm; ages → 7–13 |
| PGA Junior Golf Camp – Tri-Mountain Golf Course (id:193) | 6 sessions added with age groups; hours → Half Day 9am–12pm / Advanced Full Day 9am–3pm |
| PGA Junior Golf Camp – Sun Willows Golf Course (id:194) | 4 sessions added; hours → 9am–12pm; ages → 7–13 |
| PGA Junior Golf Camp – Red Wolf Golf Club (id:195) | Sessions → Jun 22–25, Jul 6–9; hours → 9am–12pm |
| PGA Junior Golf Camp – Meadow Park Golf Course (id:196) | Sessions → Jul 13–17, Aug 10–14; hours → 9am–12pm; ages → 7–13 |
| PGA Junior Golf Camp – Chambers Bay Golf Course (id:197) | Hours → Half Day 12–3pm / Full Day 12–7pm by age group |
| Nike Cross Country Camp – University of Puget Sound (id:198) | Added Aug 3–6 session; hours → Day camp 9am–4pm / Overnight available; ages → 10–17; overnight → Yes; meals → Yes |
| Nike Soccer Camp – Luke Jensen Field / Vancouver (id:199) | Sessions → Jul 6–9, Jul 20–23; hours → Half Day 9am–12pm / Full Day 9am–3pm |
| Nike Soccer Camp – Western Washington Surf / Puyallup (id:200) | Hours → Half Day 9am–12pm / Full Day 9am–3pm |
| Revolution Soccer Camp – University of Washington (id:207) | Session → Jul 26–29 (Overnight & Extended Day only); hours added |
| Revolution Soccer Camp – University of Puget Sound (id:208) | Session dates and hours cleaned up to consistent format |
| Revolution Soccer Camp – Wilson Playfields / Kent (id:209) | Session → Jul 27–30 (Half Day only); hours → 9am–12pm; overnight → No |
| Mariners Diamond Camps – Funko Field / Everett (id:211) | Ages → 4–6 |
| Mariners Diamond Camps – Sehmel Homestead Park / Gig Harbor (id:212) | Ages → 7–14 |
| YMCA Camp Orkila (id:1) | URL updated; ages → 8–18; financial aid → Yes |
| NxtGen Baseball Camp – Seattle (id:216) | Hours → 9am–12pm (extended care available) |
| NxtGen Baseball Camp – Bellevue (id:217) | Hours → 9am–12pm (extended care available) |
| NxtGen Baseball Camp – Edmonds (id:218) | Hours → 9am–12pm (extended care available) |
| NxtGen Baseball Camp – Funko Field / Everett (id:219) | URL updated; hours added |
| NxtGen Baseball Camp – Cheney Stadium / Tacoma (id:220) | Hours → 9am–12pm |
| Nike Baseball Camp – Seattle University (id:221) | All 6 sessions added; hours → 9am–3pm; ages → 7–13; after care noted |
| Nike Baseball Camp – North Creek High School / Bothell (id:222) | All sessions added; hours by program type; ages → 7–18; after care → Yes |
| Nike Baseball Camp – Whitworth University (id:223) | Hours by age group; ages → 5–13; meals → Yes |
| Nike Baseball Camp – University of Puget Sound (id:224) | Added Aug 3–6 session; hours → 9am–4pm; ages → 8–17 |
| Nike Swim Camp – Seattle University (id:225) | Added Session II Jun 28–Jul 1; hours → 9am–4pm (last day 9am–12pm) |
| Nike Swim Clinic – Seattle University (id:226) | All 3 April clinic dates and hours added |
| Nike Lutes Swim Clinic – PLU Starts & Turns (id:227) | Hours → 9am–3pm; age min → 9 |
| Nike Lutes Swim Camp – PLU Summer Camp (id:228) | Hours → 9am–3pm; age min → 9 |
| Samena Swim & Recreation Club – Day Camp (id:230) | Ages → 5–12; all 9 themed weekly sessions added; before/after care → Yes |
| adidas Tennis Camp – University of Washington (id:232) | All 3 sessions added; hours added; ages → 5–18; overnight → Yes |
| Nike Tennis Camp – Seattle University (id:234) | All 4 sessions added; hours → 9am–4pm |
| Nike Tennis Clinic – Mercer Island (id:235) | URL updated; session → Jun 22–24; hours added; ages → 5–18 |
| Nike Tennis Camp – Sammamish High School (id:236) | URL updated; sessions added; hours → 9am–4pm |
| Nike Tennis Camp – Lake Stevens High School (id:237) | Hours → 9am–4pm (Thu 9am–3:30pm); ages → 5–18 |
| Seattle Shaolin Kungfu Academy – Summer Camp Bellevue (id:238) | Hours updated with all session options and extended care |
| Seattle Shaolin Kungfu Academy – Summer Camp Bothell (id:239) | Same hours update as Bellevue |
| Premier Martial Arts – Summer Camp Bothell (id:240) | Session dates → TBD |
| Combat Arts Academy – Summer Camp West Seattle (id:241) | All 10 weekly sessions added; hours → 9am–3pm; ages → 5–12; before/after care → Yes |
| Tenzan Aikido – Kids Summer Camps (id:242) | All 5 camps added with age groups and times; before/after care → Yes |
| Avid4 Adventure – Mountain Biking Camp Bellevue (id:243) | All 7 sessions with grade groups added |
| Avid4 Adventure – SUP Camp Bellevue (id:244) | All 5 sessions with grade groups added |
| YMCA Camp Colman (id:2) | All program types summarized with full date ranges; ages → 6–18; financial aid → Yes |
| Code Ninjas – Queen Anne (id:252) | URL updated |
| Deerfield Farm – Horse Camps (id:257) | All 5 youth sessions with program names and hours; ages max → 17 |
| Camp Invention – all 25 entries | Hours bulk updated to 9am–3:30pm |
| Camp Invention – North Seattle (id:266) | Hours + ages → 5–14 |
| Camp Invention – Stanwood Camano (id:267) | Hours + ages → 5–14 |
| Camp Invention – Shoreline (id:268) | Session → TBD; ages → 5–14 |
| Camp Invention – Benjamin Franklin Elementary (id:269) | Hours + ages → 5–14 |
| Camp Invention – Redmond Elementary (id:270) | Hours + ages → 5–14 |
| Camp Invention – Fryelands Elementary (id:271) | Hours + ages → 5–14 |
| Camp Invention – Eastside Catholic High School (id:272) | Hours + ages → 5–14 |
| Camp Invention – Marvista Elementary (id:273) | Hours + ages → 5–14 |
| Camp Invention – Hidden Creek Elementary (id:274) | Hours + ages → 5–14 |
| Camp Invention – Opstad Elementary (id:275) | Hours + ages → 5–14 |
| Camp Invention – Purdy Elementary (id:276) | Hours + ages → 5–14 |
| Camp Invention – Browns Point Elementary (id:277) | Hours + ages → 5–14 |
| Camp Invention – Sherman Elementary (id:278) | Hours + ages → 5–14 |
| Camp Invention – Bowman Creek Elementary (id:279) | Hours + ages → 5–14 |
| Camp Invention – Hawkins Middle School (id:280) | Hours + ages → 5–14 |
| Camp Invention – Summit Public Schools: Atlas (id:265) | Ages → 5–14 |

---

## Known Issues / To-Do
- Date filtering logic is implemented but complex (parses freeform session_dates strings) — verify it works correctly in testing
- Camp type filter uses `.includes()` match — "Sports" will match "Sports, Outdoor / Nature" camps too (intentional)
- Vancouver Pedalheads camps are Vancouver, BC (Canada) — included by owner's decision
- Hours not confirmed for Vancouver and Woodinville Pedalheads sub-programs
- Formspree integration pending — "List Your Camp" and "Claim Your Camp" forms currently show success UI but do not send emails. User needs to create a Formspree account and provide two endpoint URLs.

---

## UI / Feature Work (Mar 23–27, 2026)

All changes are in `~/Documents/campquest/index.html` and deployed to GitHub Pages via SSH push.

### Infrastructure
- **SSH key** set up (`~/.ssh/id_ed25519`) and added to GitHub — Claude can now push directly from terminal
- **Chrome DevTools MCP** configured — Claude can take screenshots and interact with the live site in Chrome for visual debugging

### Navigation & Branding
- "Sign Up" nav button renamed to **"Sign Up / Log In"**
- **"PHASE 1"** label removed from logo
- Hero section: replaced plus-sign background pattern with **small triangle pattern** at low opacity
- **"Search Camps" button** color changed to logo yellow (`#f5c842`) with dark green text

### Search Bar (Front Page)
- **End date auto-sets** to the day after start date when user picks a start date (same behavior as the Add to Schedule modal)

### Camp Detail Page
- **"Already listed?" / "Claim your camp"** section width fixed to match the "List Your Camp" section above it

### Add Child Modal (Profile Page)
- **Age field** changed from free text to a dropdown (ages 1–17)
- **Emoji/icon field** removed

### My Schedule Page — Core Fixes
- **Date picker added to all camps**: instead of selecting a session from a list, parents always enter their specific start and end dates. This prevents camps like MoPOP (Jul–Aug range) from blocking the entire summer on the calendar.
- **Schedule persists after logout**: camp schedule items (including custom dates) are saved to Supabase `user_schedules` table. Requires SQL: `ALTER TABLE user_schedules ADD COLUMN IF NOT EXISTS custom_dates TEXT;`
- **Second child's schedule** now shows correctly after adding a camp
- **"Yes / No" buttons** in the Add to Schedule modal were silently broken (a `ReferenceError` on `hasSessions`). Fixed by removing the stale variable reference.
- **Date filter false positives** fixed: sessions formatted as "Week 1: Jun 21–27" were being parsed incorrectly because `parseSessionDate` expected bare month names. Fixed by stripping the prefix and changing the fallback from `return true` to `return false`.

### My Schedule Page — Calendar Improvements
- **Saturday and Sunday** columns added (calendar now shows Mon–Sun)
- **Exact dates shown on each camp block** (e.g. "Aug 24–28") so parents can see at a glance when each camp is
- **Registration progress bar** removed from the right panel
- **Clipboard emoji** removed from "My Camps" panel title
- **"Click any block to update"** behavior: tapping a camp block opens a popup with:
  - Editable start/end dates (pre-filled, end date auto-advances)
  - Notes field (freeform text — requires SQL: `ALTER TABLE user_schedules ADD COLUMN IF NOT EXISTS notes TEXT;`)
  - "Add to Google / Apple Calendar" button (downloads `.ics` file)
  - Mark Registered / Not Yet Registered toggle
  - View camp details link
  - Remove from schedule button
- **Multiple sessions of same camp**: parents can now add the same camp multiple times for the same child (e.g., two different weeks at MoPOP). Previously the child was grayed out and unclickable. Each entry gets a unique `localId` so they can be independently edited and removed.

### My Schedule Page — Conflict Detection
- **Overlapping camps are flagged**: if two camps overlap in dates for the same child, both blocks turn **red with diagonal white stripes**
- A warning banner appears at the top of the calendar
- The popup for a conflicting camp lists which other camp(s) it overlaps with
- Conflicts are only detected within the same child's schedule (two kids at different camps at the same time is fine)

### My Schedule Page — All Kids View
- **"All Kids" tab** appears automatically when 2 or more children are added
- Shows all children's camps on one shared calendar, each child assigned a distinct color
- Faded blocks = not yet registered; full color = registered
- Legend updates to show each child's color
- Right panel groups camps by child with a matching color dot
- All popup actions work correctly from the All Kids view

### My Schedule Page — Lane Layout for Overlapping Camps
- When two camps overlap in the same week, they each get their **own row (lane)** within that week rather than one hiding behind the other
- Non-overlapping camps in the same week share a row efficiently
- A subtle dashed line separates lanes within the same week

### Mobile Responsiveness Fixes (Mar 27, 2026)
- **Hero / search bar**: horizontal padding reduced so date and age inputs no longer get cut off on narrow screens; all inputs use `box-sizing:border-box`
- **Camp detail header**: "Is this your camp? Claim listing" button now stacks below camp name on mobile instead of overflowing off-screen
- **Schedule calendar**: wraps in a horizontal scroll container on mobile so the 7-column grid is fully accessible
- **List your camp / Claim your camp pages**: horizontal padding reduced to 1rem on mobile
- **Detail and schedule pages**: overall page padding reduced on mobile

---

## Supabase SQL Reference

### Add columns required for new features:
```sql
-- Required for schedule date saving (done if schedule persistence works)
ALTER TABLE user_schedules ADD COLUMN IF NOT EXISTS custom_dates TEXT;

-- Required for per-camp notes feature
ALTER TABLE user_schedules ADD COLUMN IF NOT EXISTS notes TEXT;
```

---

## Pending / Future Work
- ~~**Formspree integration**~~ ✅ Done Mar 27, 2026
- ~~**Website rename to PopCamps**~~ ✅ Done Mar 27, 2026
- ~~**Custom domain**~~ ✅ popcamps.one purchased and configured Mar 27, 2026 — HTTPS enforcement pending
- **Google Analytics**: deferred — add once site has real traffic.
- **Resend SMTP**: for reliable password reset emails from a custom address — deferred.
- **Enforce HTTPS on popcamps.one**: check GitHub Pages Settings once SSL cert is issued.
- **session_dates cleanup**: ~66 camps still have non-date text in their session dates.

---

## Website Testing & Major Changes (Mar 27, 2026)

### Full Site Test — Results
Ran automated browser testing using Chrome DevTools MCP against the live site. All core flows tested.

**Bugs Found & Fixed:**

| Bug | Fix |
|---|---|
| Event popup (camp calendar block click) had no close button and no Escape key support | Added ✕ button (top-right, absolute positioned) and Escape key listener to the popup. Both call `closeOverlay('event-popup')` |
| After logging out, the empty schedule message showed the previously active child's name (e.g. "No camps added yet for Lily") instead of generic text | Added reset of `#sched-empty h3` text to "No camps added yet" in the `logOut()` function, before `renderProfileChildren()` is called |

---

### Website Rename: CampQuest → PopCamps
All changes made in `~/Documents/campquest/index.html`:

| Location | Before | After |
|---|---|---|
| Browser tab title | `CampQuest — Find the Perfect Camp` | `PopCamps — Find the Perfect Camp` |
| Nav logo | `Camp<span>Quest</span>` | `Pop<span>Camps</span>` |
| Registration type short code | `short:"CQ"` | `short:"PC"` |

**GitHub repo renamed:** `campquest` → `popcamps`
- New repo URL: `https://github.com/Camps2026/popcamps`
- Git remote updated locally: `git remote set-url origin git@github.com:Camps2026/popcamps.git`

**Supabase project rename:** Still named "CampsQuest" — cosmetic only, low priority.

---

### Custom Domain: popcamps.one
Domain purchased from Porkbun. DNS configured as follows:

**Porkbun DNS Records Added:**
| Type | Host | Answer |
|---|---|---|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | camps2026.github.io |

**GitHub Pages configured:**
- Custom domain field set to `popcamps.one`
- `CNAME` file committed to repo with `popcamps.one`
- **Enforce HTTPS**: not yet available — waiting for GitHub to issue SSL certificate after DNS propagates (can take up to 24 hours). User needs to return to GitHub Pages Settings and check the "Enforce HTTPS" box once it becomes clickable.

---

### Formspree Integration
Both contact forms now send real emails via Formspree.

**List Your Camp form** (`submitListing()` function):
- Endpoint: `https://formspree.io/f/mdapagpg`
- Fields sent: Camp Name, Camp Type, Contact Email (_replyto), Location, Description
- IDs added to form fields: `list-name`, `list-type`, `list-email`, `list-location`, `list-description`, `list-submit-btn`
- On success: button hides, toast shows "Camp listing submitted! You'll hear from us within 24 hours."

**Claim Your Camp form** (`submitClaim()` function):
- Endpoint: `https://formspree.io/f/xqegeqgl`
- Fields sent: Camp Name (from `claimTargetCamp`), Your Name, Role, Work Email (_replyto), Phone, Verification
- On success: shows success HTML inside `#claim-form-section`
- Replaced the old fake `submitClaim()` that only showed success UI without sending anything

---

### Supabase session_dates Cleanup
Scanned all 995 camp records and found 73 camps with non-date text in `session_dates` (addresses, URLs, prices, program names, times).

**7 camps updated via Supabase REST API** — URLs removed, leaving "TBD":

| Camp | ID |
|---|---|
| Camp Sealth | 3 |
| Echo Falls Golf Club – Junior Golf Camps | 188 |
| Nike Soccer Camp – Renton / Ron Regis Park (Tacoma Stars) | 204 |
| Snapology STEAM Camps – Issaquah | 509 |
| Snapology STEAM Camps – Kirkland | 510 |
| Snapology STEAM Camps – Redmond | 511 |
| Snapology STEAM Camps – Sammamish | 512 |

All 7 confirmed updated (HTTP 204) and verified by re-fetching from Supabase.

**Remaining:** ~66 more camps have non-date text (program names, prices, addresses, times) — not yet cleaned. These were identified but no action taken yet.

---

## Session Dates Cleanup — Continued (Mar 28, 2026)

Removed addresses, prices, time info, and other non-date text from additional camps:

### Physical Addresses Removed
| Camp | What was removed |
|---|---|
| Camp Killoqua (id:4) | `15207 E Lake Goodwin Rd, Stanwood WA` |
| SGA Ballard (id:43) | `Ballard location: 1415 NW 52nd St, Seattle WA 98107` |
| SGA Columbia City (id:44) | `Columbia City location: 5034 37th Ave South Suite 200` |
| SGA Mill Creek (id:46) | `Mill Creek location: 15311 Main St, Mill Creek WA 98012` |
| SGA Burien (id:47) | `Burien location: 15840 1st Ave S Suite 103` |
| SGA Lake City 28th (id:48) | `Lake City Campus – 28th gym: 12739 28th Ave NE` |
| Premier Golf – Interbay (id:181) | `Located at Interbay Golf Center, 2501 15th Ave W` + phone number |
| Kirkland Parks – Martial Arts (id:365) | `Finn Hill Middle School` from location list |

### Prices Removed
| Camp | What was removed |
|---|---|
| Redmond Tennis Center (id:107) | All pack pricing ($300/$350, $550, etc.) from each program |
| Premier Golf – Bellevue (id:183) | `$140` and `$550` prices |
| Northwest Arts Center (id:91) | All `($xxx)` prices + removed `Fit & Fun Camp ($225)` (no date) |
| Camp Firwood (id:74) | Prices `$665/$689/$499` — reformatted with `\|` separators |
| Camp Lutherwood (id:71) | Prices `$310/$635/$720/$650/$350`, `(FULL)` tags, time info, `application required` |
| St. Thomas School Summer Camps (id:163) | All `($xxx)` prices from all 90 session entries |

### Other Cleanup
| Camp | What was removed |
|---|---|
| Camp Korey (id:17) | Medical category labels (e.g. `— Cardiac, Respiratory & Transplant`) — commas inside were causing split artifacts |
| Camp Killoqua (id:4) | `Financial aid available` |
| Bricks 4 Kidz Seattle (id:131) | `, waitlist` from Valley School entry |
| Bricks 4 Kidz South Sound/Eastside (id:132) | `, 4/8 all` from PenMet entry |
| Seattle Parks Specialized (id:300) | `Ravenna Park` and `Seward Park` from week labels |
| City of Olympia – Fish, Forage, Fire! (id:453) | `Fish, Forage, Fire!` prefix from each entry (commas inside were splitting it) |
| Camp Fire Mountain (id:75) | `, 2026 (Sun 1pm–Sat 10am)` from all week entries |
| Lakeside School Summer Programs (id:150) | Grades, hours, subject lists — kept program names + dates only |
| Village Theatre KIDSTAGE – Issaquah (id:174) | All time info (9am–12pm, etc.) from every session entry |
| Village Theatre KIDSTAGE – Everett (id:1332) | All time info from every session entry |

### Removed Entirely
- City of Olympia – Olywahoo Jr. Music Classes (Jefferson Middle School) (id:447)

---

## SEO Work (Mar 28, 2026)

All changes in `index.html`, pushed to GitHub.

### Meta Tags Added to `<head>`
- **Title**: `popcamps | Washington Summer Camp Directory`
- **Meta description**: "Browse hundreds of summer camps for kids across Seattle, the Eastside, and communities throughout Washington state. Find the right camp for your family on popcamps."
- **Canonical tag**: `<link rel="canonical" href="https://popcamps.one/">`
- **Open Graph tags**: og:title, og:description, og:url, og:type
- **Twitter Card tags**: twitter:card, twitter:title, twitter:description
- **Google Search Console verification tag**: `UpzpVcPfNDCPpKCylyfHxS3nozoHgnAhgIpDpE0A6jo`

### Files Added to Repo
- **sitemap.xml**: Points to `https://popcamps.one/`, weekly changefreq, priority 1.0
- **robots.txt**: Allow all crawlers, points to sitemap
- Sitemap link also added to `<head>`

### Google Search Console
- Property added: `https://popcamps.one/` (URL prefix method)
- Verified via HTML tag method
- Sitemap submitted: `https://popcamps.one/sitemap.xml` — Status: Success, 1 page discovered

### Schema.org Structured Data
Added two JSON-LD blocks before `</body>`:
- **WebSite schema** with SearchAction (tells Google the site has a search box)
- **LocalBusiness schema** with areaServed: Washington state

### Hero Section
- `<h1>` updated to: `Washington Summer Camp Directory`
- Subheadline `<p>` added: "Browse hundreds of summer camps for kids across Seattle, the Eastside, and communities throughout Washington state."

### HTTPS
- Enforce HTTPS checkbox enabled in GitHub Pages Settings — popcamps.one is now fully HTTPS ✅

---

## UI / Feature Changes (Mar 28, 2026)

### My Calendar (formerly My Schedule)
- Renamed "My Schedule" → **"My Calendar"** everywhere: nav, mobile menu, page title, buttons, empty state
- **"Show Full Summer" toggle button**: appears top-right of calendar. Shows all weeks Jun 8 – Sep 7, 2026. Button label flips to "Hide Empty Weeks" when active.
- **Empty week → search**: in full summer view, empty weeks show `+ Find a camp`. Clicking navigates to search page with Monday of that week pre-filled as start date and Friday as end date, then auto-runs search.
- **First child tab highlighted by default**: when logged in, the first child's tab is now active (dark green) on load so it's clear whose calendar is shown.

### Search Results
- Removed **"Best Match" sort dropdown** — it had only one option and looked broken. Removed entirely.

### Footer
- Added site footer (dark green, matches nav):
  > "Listings updated regularly for summer 2026. · © 2026 PopCamps"

---

## Known Issues / Pending Work (as of Mar 28, 2026)

- **og:image**: needs a 1200×630px branded image created in Canva — one line of code to add once ready.
- **Remaining session_dates cleanup**: some camps still have non-date text.
- **Supabase project rename**: still shows as "CampsQuest" — cosmetic only, no functional impact.
- **Google Analytics**: deferred — add once site has real traffic.

---

## Camp Owner Self-Service Portal (Mar 29, 2026)

All code changes in `~/Documents/campquest/index.html`. All Supabase changes made via API or SQL editor.

### Overview
Built a full camp owner portal so owners can log in with a magic link, submit edits to their listing, and upload photos — all of which go into a review queue for the admin (site owner) to approve before anything goes live.

### Supabase Schema Changes

**New columns on `camps` table:**
```sql
ALTER TABLE camps
  ADD COLUMN IF NOT EXISTS description text,
  ADD COLUMN IF NOT EXISTS photos text[],
  ADD COLUMN IF NOT EXISTS owner_updated_at timestamptz;
```

**New `camp_owners` table** (links auth users to their camps):
```sql
CREATE TABLE IF NOT EXISTS camp_owners (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL,
  camp_id integer NOT NULL,
  created_at timestamptz DEFAULT now(),
  UNIQUE(user_id, camp_id)
);
ALTER TABLE camp_owners ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users see own ownership" ON camp_owners
  FOR SELECT USING (auth.uid() = user_id);
```
Note: `camp_id` is a plain `integer NOT NULL` (no FK constraint) because `camps.id` is not a formal primary key.

**New `site_admins` table** (used to identify the admin user without a secret key):
```sql
CREATE TABLE IF NOT EXISTS site_admins (
  user_id uuid PRIMARY KEY
);
-- RLS disabled (publicly readable so publishable key can check it)
INSERT INTO site_admins (user_id) VALUES ('db121d42-e139-4e4d-ab01-ab360c7010ed');
```

**New `camp_edit_requests` table** (staging layer — edits sit here until admin approves):
```sql
CREATE TABLE IF NOT EXISTS camp_edit_requests (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  camp_id integer NOT NULL,
  user_id uuid NOT NULL,
  status text DEFAULT 'pending',
  edits jsonb,
  photos_to_add text[],
  photos_to_remove text[],
  created_at timestamptz DEFAULT now()
);
ALTER TABLE camp_edit_requests ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Owners can insert edit requests" ON camp_edit_requests
  FOR INSERT TO authenticated WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owners can read own edit requests" ON camp_edit_requests
  FOR SELECT TO authenticated USING (auth.uid() = user_id);
CREATE POLICY "Admin can read all edit requests" ON camp_edit_requests
  FOR SELECT USING (
    auth.uid() IN (SELECT user_id FROM site_admins)
  );
CREATE POLICY "Admin can update edit requests" ON camp_edit_requests
  FOR UPDATE USING (
    auth.uid() IN (SELECT user_id FROM site_admins)
  );
```

**RLS on camps table** (keep public read, allow admin to update):
```sql
ALTER TABLE camps ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public can read camps" ON camps FOR SELECT USING (true);
CREATE POLICY "Admin can update camps" ON camps
  FOR UPDATE USING (
    auth.uid() IN (SELECT user_id FROM site_admins)
  );
```

**Supabase Storage bucket:** `camp-photos` (Public: YES)
```sql
CREATE POLICY "Owners can upload to their camp folder" ON storage.objects
  FOR INSERT TO authenticated
  WITH CHECK (bucket_id = 'camp-photos');
CREATE POLICY "Public can view camp photos" ON storage.objects
  FOR SELECT USING (bucket_id = 'camp-photos');
CREATE POLICY "Owners can delete their photos" ON storage.objects
  FOR DELETE TO authenticated
  USING (bucket_id = 'camp-photos');
```

**Supabase Auth:**
- Email provider enabled with Magic Link (passwordless)
- Site URL set to: `https://popcamps.one`
- Redirect URL added: `https://popcamps.one`

### Admin User
- Admin email: (site owner's email)
- Admin user_id: `db121d42-e139-4e4d-ab01-ab360c7010ed`
- Inserted into `site_admins` table

### How the Approval Flow Works
1. Camp owner fills out "Claim Your Camp" form → Formspree emails the admin
2. Admin goes to Supabase → Authentication → Users → "Invite User" → enters owner's email
3. Admin copies the new user's `user_id`, inserts into `camp_owners`: `{ user_id, camp_id }`
4. Owner clicks magic link in email → lands on owner dashboard
5. Owner edits their listing + uploads photos → clicks "Submit for Review →"
6. Edits go into `camp_edit_requests` table as `status: 'pending'`
7. Admin logs in → sees "Review Pending Edits" page → approves or rejects each request
8. On approval: edits merged into `camps` table, `owner_updated_at` set → "Updated" badge appears on card

### New Pages Added to index.html

**`page-owner`** — Camp Owner Dashboard:
- Header: "Camp Owner Dashboard" + camp name + Log Out button
- Yellow notice banner when a review is pending (shows submission date)
- Edit form: Camp Name, Camp Type, Website, City/State, Description (new), Hours, Age Min, Age Max, Session Dates
- Photos section: shows current live photos + pending new photos with "NEW" badge; photos can be marked for removal
- Submit button: "Submit for Review →" (goes to `camp_edit_requests`, NOT directly to `camps`)

**`page-admin`** — Admin Review Dashboard:
- Header: "PopCamps Admin — Review Pending Edits" + Log Out button
- Each pending request shows as a review card:
  - Camp name + submission date
  - Before/After diff table (Current in red, Proposed in green)
  - Photo previews (new photos to add, existing photos marked for removal)
  - Approve (green) and Reject (red) buttons

### New JS Functions Added

| Function | What it does |
|---|---|
| `sendOwnerLoginLink()` | Sends magic link via `signInWithOtp({ shouldCreateUser: false })`; shows error if no account found |
| `checkOwnerSession()` | Checks `site_admins` → admin page; `camp_owners` → owner dashboard; else shows "pending" state |
| `populateOwnerForm(camp)` | Fills edit form with current camp data; checks for existing pending request |
| `renderOwnerPhotos(livePhotos)` | Shows live photos with remove/unmark controls; shows pending new photos |
| `markPhotoForRemoval(i)` / `unmarkPhotoForRemoval(i)` | Toggle pending removal state on existing photos |
| `removePendingPhoto(i)` | Remove a newly uploaded photo from the pending queue |
| `uploadOwnerPhoto(input)` | Uploads to `pending/{campId}/{timestamp}.ext` in Storage; adds URL to `pendingPhotosToAdd[]` |
| `submitOwnerEdits()` | Inserts into `camp_edit_requests` with all edits + photo lists |
| `loadAdminRequests()` | Fetches all pending requests, builds diff tables, renders review cards |
| `approveEditRequest(id, campId, btn)` | Merges edits + photos into `camps`, marks request approved, sets `owner_updated_at` |
| `rejectEditRequest(id, btn)` | Sets request status to `rejected` |
| `ownerLogout()` | Signs out, resets all state variables, returns to portal page |

### Key Design Decisions
- **No service role key**: admin authentication is solved by a `site_admins` table (RLS disabled, publicly readable) — the publishable key can check if a user's ID is in it
- **Edits never go live immediately**: the `camp_edit_requests` table acts as a staging layer
- **Photos staged in `pending/` folder**: uploaded to Storage but not added to `camps.photos` until admin approves
- **Three-tier auth check**: `site_admins` → `camp_owners` → regular user (in both `onAuthStateChange` and `checkExistingSession`)

### "Updated" Badge
Camps with `owner_updated_at` within the last 30 days show a green "Updated" pill badge on their card. Badge logic in `renderCamps()`:
```javascript
c.ownerUpdatedAt && ((new Date()-new Date(c.ownerUpdatedAt))/86400000)<30
  ? '<span class="updated-badge">Updated</span>' : ''
```

### Bugs Fixed During Build
| Bug | Fix |
|---|---|
| `camp_owners` FK on `camps(id)` failed ("no unique constraint") | Changed `camp_id` from FK reference to plain `integer NOT NULL` |
| Magic link redirected to `camps2026.github.io` (404) | Updated Supabase Site URL and Redirect URLs to `https://popcamps.one` |
| Supabase showed "destructive operations" warning on DROP POLICY | Just a confirmation dialog, not an error — clicked "Run this query" to proceed |

### Automated Tests Run (Mar 29, 2026)
All 11 tests passed using Chrome DevTools MCP against the live site (`https://popcamps.one`):

| # | Test | Result |
|---|------|--------|
| 1 | Site loads, camps display | ✅ 471 camps showing |
| 2 | Portal page shows "Camp Owner Login" section | ✅ Email input + Send Login Link button present |
| 3 | Invalid email format validation | ✅ Toast: "Please enter a valid email address." |
| 4 | Non-existent email error message | ✅ Red error: "No owner account found for this email." |
| 5 | Owner dashboard DOM structure | ✅ All 9 form fields + photo grid present |
| 6 | Submit button text | ✅ "Submit for Review →" |
| 7 | Owner dashboard with pending notice | ✅ Yellow banner + edit form render correctly |
| 8 | Admin review page renders | ✅ Diff table + Approve/Reject buttons display correctly |
| 9 | "Updated" badge logic | ✅ Shows for <30 days, hidden for 35+ days |
| 10 | State clears on logout | ✅ All variables reset to null/empty |
| 11 | Claim form success message | ✅ References "1–2 business days" and login link |

---

## Session: Mar 29–30, 2026 — Legal Pages

### Privacy Policy Page
- Helped user answer Termly's Privacy Policy questionnaire (product description, data collected, third-party services, etc.)
- Built a full `page-privacy` page in `index.html` with the complete Privacy Policy content from Termly
- Added "Privacy Policy" link to the footer
- Added a privacy hint below the "Child's Name" field in the Add Child modal: *"You don't need to use your child's real name — a nickname is fine."*
- Added "Delete My Account" feature to the Profile page (clears all local data: children, saved camps, booked camps)

### Terms and Conditions Page
- Helped user answer Termly's Terms and Conditions questionnaire
- Termly's free plan does not allow embedding or copying T&C text
- User captured 16 screenshots of the Termly T&C preview — Claude transcribed all 16 into full page text
- Built `page-terms` page in `index.html` with all 24 sections of the Terms and Conditions
- Added "Terms and Conditions" link to the footer next to "Privacy Policy"
- Pushed to GitHub: commit `275c6bc`

### Footer (after these changes)
```
Listings updated regularly for summer 2026. · © 2026 PopCamps · Privacy Policy · Terms and Conditions
```

---

## Session: Mar 30, 2026 — Legal & Privacy Cleanup

### Photo Upload Removed
- Decided to remove the camp owner photo upload feature due to child consent concerns — no way to verify parents consented to photos of their kids being published
- Removed: photo upload UI from owner dashboard, all photo JS functions (`uploadOwnerPhoto`, `renderOwnerPhotos`, `markPhotoForRemoval`, `unmarkPhotoForRemoval`, `removePendingPhoto`), photo diff from admin review cards, photo merging from `approveEditRequest`
- Database `photos` column and `camp-photos` storage bucket left in place (harmless, no action needed)
- Commit: `c924c48`

### "List Your Camp" Cleanup
- Removed fake social proof line: *"Join 12,400+ camps on PopCamps. Reach thousands of parents actively searching."*
- Commit: `ce1e405`

### Listing Accuracy Disclaimer
- Added a gray disclaimer box at the bottom of every camp detail page: *"Listing information is based on publicly available data and may not reflect current availability, pricing, or scheduling. Always confirm details directly with the camp before registering."*
- Added a second line to the footer in very light text: *"Camp listing details are for informational purposes only. Verify all information directly with the camp."*
- Commit: `4878b61`

### Privacy Law Improvements (WA My Health MY Data, COPPA, CCPA)
- Added sentence to Privacy Policy Section 6 (Minors) clarifying that non-logged-in users' child data stays only in browser localStorage and is never sent to PopCamps servers; logged-in users' data is deleted when they clear their account
- Renamed "Delete My Account" button → "Clear My Data" (more accurate since most users don't have a traditional account)
- Updated confirm dialog and success toast to match new wording
- Commit: `d0171ff`

### Neurodiversity Filter Removed
- Removed Neurodiversity from the camp type filter chips, camp owner edit dropdown, and CSS/JS type definitions
- Commit: `faf8c94`

### Footer (current state)
```
Listings updated regularly for summer 2026. · © 2026 PopCamps · Privacy Policy · Terms and Conditions
Camp listing details are for informational purposes only. Verify all information directly with the camp.
```

---

## Session: Mar 30–31, 2026 — UX, Bug Fixes & Marketing

### My Calendar UX Improvements
- Improved empty state: now explains the calendar is for tracking camps and what "registered" means
- Added toast after adding a camp: "Added to calendar! Remember to register directly with the camp."
- Legend updated: "Registered" → "Registered with camp" + explanation note
- Sidebar: added persistent reminder "Adding a camp here doesn't sign you up..."
- Removed icons from calendar camp blocks (cleaner, more room for text)
- Mobile calendar: Sat/Sun now visible (was Mon–Fri only) — grid updated to 7 columns
- Mobile calendar: camp blocks now stay within their rows (overflow:hidden, max-height, text truncation)

### City Search Autocomplete Fix (Mobile)
- Replaced native HTML `<datalist>` (broken on iOS/Android) with a custom dropdown
- Dropdown appears as you type, shows up to 8 matching cities, works on both touch and mouse
- Cities stored in `_allCities` global variable populated from Supabase data

### Login Bug Fix (Mobile + Desktop)
- **Root cause 1**: No try/catch on login handler — network errors left button stuck at "Logging in..." forever
- **Root cause 2**: `onAuthStateChange` never handled regular parent users — only admins and camp owners
- **Root cause 3**: `checkExistingSession()` only ran if camp loading succeeded
- **Fixes**: Added try/catch to login handler, added regular user handling in `onAuthStateChange` with `if (!currentUser)` guard, added `checkExistingSession()` to catch block of `loadCampsFromSupabase()`

### Page Load Fix
- Page was showing prototype/dummy camp data for 1–2 seconds before real data loaded
- Fixed: replaced `applyFilters()` on page load with `showLoadingState()` — now shows spinner instead

### Open Graph / Social Sharing Fix
- og:title and twitter:title updated: "Seattle & PNW Summer Camp Directory" → "Washington Summer Camp Directory"
- og:description and twitter:description updated to match

### URL Security Audit
- Checked all 994 camp website URLs for suspicious patterns
- Result: completely clean — all HTTPS, no shorteners, no suspicious TLDs, no malformed URLs
- Only note: 52 City of Olympia camps share one generic ActiveCommunities URL (not a security issue)

### Marketing — Facebook
- Created PopCamps Facebook Page: "PopCamps — Washington Summer Camp Directory"
- Wrote tailored posts for Seattle, Eastside, and general WA parent groups
- Strategy: post in groups one or two per day, reply to every comment, message admins if group doesn't allow self-promotion
- Facebook ads discussed — deferred, free organic posting first

### CLAUDE.md Updated
- Fully rewrote CLAUDE.md with current site state, all table columns, camp type colors, how to add/edit camps, how to check users, and planned future work

---

## How We Work Together

### Roles
- **You (site owner)**: Make decisions, approve changes, do manual Supabase steps (SQL, creating users, storage), and test on the live site from your side
- **Claude**: Writes all code, pushes to GitHub, queries/updates the database via REST API, runs automated browser tests, explains every change before making it

### Communication Style
- Claude explains what it's about to do and why before doing it — especially for database changes or anything that affects the live site
- If something requires your action in Supabase or elsewhere, Claude gives you exact copy-paste SQL or step-by-step instructions
- Claude asks for confirmation on risky/destructive actions (deleting records, dropping policies, etc.)
- You don't need to ask Claude to push to GitHub — it does it automatically after every change

### How Changes Get Made
1. You describe what you want (doesn't need to be technical)
2. Claude reads the current code before making any changes
3. Claude explains the plan, then makes the changes in `index.html`
4. Claude pushes to GitHub → GitHub Pages deploys automatically (usually within 1–2 minutes)
5. Claude takes a screenshot using Chrome DevTools MCP to verify it looks right on mobile

### Database Workflow
- Claude can read from and write to Supabase directly via REST API (using the publishable key)
- For operations that require a service role or admin access (creating users, running schema changes), Claude provides SQL to paste into the Supabase SQL Editor
- For schema changes, Claude provides the exact SQL and waits for you to confirm it ran successfully

### Testing Workflow
- Chrome DevTools MCP is connected to your local Chrome instance
- Claude can navigate to the live site, take screenshots, run JavaScript, fill forms, and check element state
- Standard test viewport: 390×844 (iPhone size)
- For UI states that are hard to reach manually (e.g., logged-in owner dashboard), Claude uses `evaluate_script` to force-show them with dummy data

### Single-File Rule
Everything lives in `index.html`. No separate CSS or JS files, no build tools, no npm. All libraries come from CDN links. This keeps the project simple enough to manage without a developer.

### GitHub Push Workflow
- Git remote: `git@github.com:Camps2026/popcamps.git`
- SSH key: `~/.ssh/id_ed25519` (already added to GitHub)
- Claude pushes directly via terminal after every change — you never need to do this manually

---

## How to Start a New Session

Tell Claude:
> "I'm working on PopCamps, a Washington state summer camp directory. Single index.html file, Supabase database, deployed at popcamps.one. See CAMPQUEST_WORK_LOG.md in ~/Documents/campquest/ for full context."

Then share this file or paste the relevant section.
