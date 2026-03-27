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
- **Formspree integration**: wire up "List Your Camp" and "Claim Your Camp" forms to actually send emails. User needs to create Formspree account → get two form endpoint URLs → paste them here.
- **Website rename**: site is currently called "CampQuest" — user plans to rename. When ready: find/replace the name in `index.html`, rename the GitHub repo, update Supabase redirect URLs.
- **Custom domain**: recommended to buy a domain (~$12/year on Namecheap) once the name is decided.
- **Google Analytics**: deferred — add once site has real traffic.
- **Resend SMTP**: for reliable password reset emails from a custom address — deferred.

---

## How to Start a New Session

Tell Claude:
> "I'm working on CampQuest, a Seattle-area summer camp finder. Single index.html file, Supabase database. See CAMPQUEST_WORK_LOG.md in ~/Documents/campquest/ for full context."

Then share this file or paste the relevant section.
