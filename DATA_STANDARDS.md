# PopCamps Data Standards

The rulebook for every data task: research, weekly checks, batches, owner edits.
If a rule here and a habit disagree, this file wins. Changes to this file need P's yes.

Adopted Nov 24, 2026.

## 0. Ground rules
- **Never change 2026 data in `camps`** (`session_dates`, `hours`, ages…). 2027+ information goes in `camp_seasons` and `camp_sessions`.
- **Every change ships as a reviewed SQL batch** (`~/Documents/popcamps-backups/<date>/sql/`). Each batch has a check before, runs as one unit, and has a check after. P pastes it after approving.
- **Identify camps by `id` only.** Never by name: different camps share names.
- **Clean text:** no leading or trailing spaces, no line breaks inside a value, single spaces only.
- **Ranges use an en dash `–`**, not a hyphen `-`, with no spaces around it: "Jun 21–25", "9am–3pm", "Ages 6–12".
- **Never log in, create an account or start a registration** on a camp's site to see information.
- **Respect robots.txt.** If a site's robots.txt asks automated visitors not to read a page, the sweep doesn't read it; the page is listed as routine "check by hand".

## 1. Session dates
**Stored:** one `camp_sessions` row per session, with real dates in `start_date` / `end_date`.
- The year comes from the camp's own page, never guessed.
- `start_date` must fall in the row's `season_year`.
- A continuous program (e.g. drop-in all summer) is one session covering the whole span, with `notes` "Drop-in" or similar.
- A program offered on specific weekdays only ("Wed/Sat") is not a session. It goes in `camp_seasons.schedule_note`.
- Different age groups, times or prices in the same week are **separate sessions**, each with its own `label`, ages, hours and price.
- `label` holds only the name the camp uses: "Week 1 – Passport to Adventures", "Session 3a (Mini)". No dates in the label.

**Displayed:** generated from the sessions, never typed by hand:
- Sorted by start date, separated by ", ".
- Same month: "Jun 21–25". Across months: "Jun 28–Jul 2". Single day: "Jun 21".
- Month names: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec. No year, no weekday.
- Example: "Jun 21–25, Jun 28–Jul 2, Jul 6–9"

## 2. Hours
- Format "9am–3pm". Lowercase am/pm, no spaces, no ":00". Minutes only when needed: "8:30am–3:30pm".
- Both ends carry am/pm ("9am–12pm", not "9–12pm"). Noon is "12pm".
- More than one option: label each, separated by ", ": "Half day 9am–12pm, Full day 9am–3pm".
- Extended care is **not** part of hours. Put it in `notes`: "Extended care 7:30am–9am and 3pm–5:30pm".
- Fill `camp_seasons.hours` or `camp_sessions.hours` only if different from `camps.hours`. Unknown means empty, never "See website".

## 3. Ages
- In years. Whole numbers, or one decimal when the camp states half years ("3½" → 3.5). Allowed range 0–21.
- "Ages 6–12" → 6 and 12. "8+" → minimum 8, maximum empty.
- Grades: minimum = lowest grade + 5, maximum = highest grade + 6 (K = grade 0). "Grades 1–5" → 6–11. Keep the camp's wording in `notes`, e.g. "Grades 1–5".
- Teens / adults without numbers → yellow (section 10).
- Fill ages on a session only if different from the camp's ages.

## 4. Camp types (`camps.camp_type`)
Allowed values, the 9 search filter types: **Sports, Arts, STEM, Outdoor / Nature, Music, Dance, Cooking, Academic, Variety**.
- At most two, most important first, separated by ", ": "Sports, Outdoor / Nature".
- How current values map:

  | Current value | Becomes |
  |---|---|
  | "Outdoor" | "Outdoor / Nature" |
  | "Arts & Crafts" | "Arts" |
  | "Chess", "Language", "Leadership" | "Academic" (e.g. "Language, Variety" → "Academic, Variety") |
  | "Neurodiversity" | **Not a type.** Each of these camps keeps its real type (Sports, Arts…), decided camp by camp. See "Open questions" |

- **Changes to `camp_type` in `camps` happen later, as a separate batch, only with P's yes.** Until then, new findings use the allowed values only.

## 5. Season status (`camp_seasons.status`)
| Status | Means | Site wording |
|---|---|---|
| `not_posted` | The camp hasn't published dates for that year (including: page still shows 2026, or dates only behind a login) | "2027 dates not yet posted" |
| `partial` | Some sessions posted, others still TBD | Show the posted dates, plus "More 2027 dates not yet posted" |
| `confirmed` | All sessions found on an allowed source (section 8) | Show dates |
| `needs_review` | Red-flagged information waiting for P | Treat as not_posted on the site until resolved |
| `not_running` | The camp says it won't run that year (P confirms first) | "Not running in 2027" |

Exact wording: **"2027 dates not yet posted"**. Use the season year, capital first letter, no trailing period on cards. For a date search with no results, keep the current message: "Most camps haven't posted 2027 dates yet. We're adding them as they're announced. Search without dates to browse all camps."

## 6. Price (`price_text`)
- Only if found on an allowed source (section 8); otherwise leave empty. Never "varies" or a guess.
- Format: "$350/week", "$95/day", "$1,250/session", "$350–$425/week", "Free". Use a comma for thousands, no cents unless they're non-zero.
- Units: /day, /week, /session, /month, /camp.
- Member pricing: "$350/week ($300 members)".
- Different prices for different sessions go on each session.

## 7. Registration
| Field | Rule |
|---|---|
| `registration_status` | `not_open`, `open`, `waitlist`, `full` or `closed`, as stated on an allowed source. Empty if unknown. |
| `registration_opens` | Exact date only, when the camp states one ("Registration opens Feb 1, 2027"). |
| `registration_opens_text` | When there's no exact date: "Early February 2027", "Winter 2027". Capital first letter, include the year. |
| `registration_url` | The page where parents actually sign up (may be a registration system; see section 8). |

## 8. Sources (`source_url`)
**Allowed:**
1. **The camp's own site:** the same domain as `camps.website_url`, or a subdomain of it.
2. **A registration or booking system** (ActiveNet, CampBrain, CampMinder, UltraCamp, Sawyer registration, RegFox, Eventbrite and similar), **only when the camp's own website links to it.** Record the linking page in `notes` as "Linked from https://…". If the camp's site doesn't link to it, it doesn't count.

**Never allowed:** aggregators and listings such as ActivityHero, Yelp, Google, social media, news articles or PopCamps itself, even if their information looks right.

**Format:**
- The exact page or file where the information appears, not the home page.
- `https://` only, with no tracking parts (`utm_…`, `fbclid`).
- If the page moved, record the new URL and say so in the batch notes.

## 9. Check timestamps
- `last_checked_at`: every time someone looks, even if nothing changed.
- `last_changed_at`: only when stored information actually changes.
- `checked_by`: "P" or "weekly check".

## 10. Weekly check: proof, then green / yellow / red

### 10.1 Proof rule (applies to every finding, before any colour)
- **Every finding includes the exact quoted text from the page it came from**, copied word for word, plus **a saved copy of that page's text** from the moment it was checked (section 11 says where).
- **A script, not the AI, runs two checks:**
  1. **The quote is in the saved page.** The comparison ignores only invisible differences: spacing and line breaks, non-breaking spaces, soft hyphens, zero-width characters, and curly vs straight quotation marks. Dashes, digits and words must match exactly.
  2. **Each stored value is in its own quote.**
     - **Dates:** the script reads the dates out of the quote, and they must match the stored `start_date` and `end_date` exactly.
     - **Hours, ages and price:** the script reads them out of the quote and compares them with the stored value after converting both to the standard formats in sections 2, 3 and 6. So "9:00 AM - 3:00 PM" supports "9am–3pm", and "Grades 1–5" supports ages 6–11.
- **Every stored value** (each session's dates, hours, ages, price, registration date) **must be backed by a quote.**
- **If a quote isn't found in the saved page, or a quote is found but doesn't contain the value it is meant to support, the finding is 🔴 red and nothing from it is stored.**

### 10.2 Routine: not on the review list
These go in the batch only as routine updates (`last_checked_at` and `notes`). P approves the batch as a whole, but they aren't listed for review.
- **Page still shows only 2026 (past) dates:** status stays `not_posted`. Add the note "page still shows 2026 dates" and re-check next week.
- **Dates visible only behind a login or account:** never log in. Status stays `not_posted`, `schedule_note` = "See camp website for dates", plus a note "Dates behind login".
- **Nothing changed** since the last check.
- Dates that start on the wrong weekday in 2027 but fit 2026's calendar (e.g. "Jun 22–26" is Mon–Fri in 2026 but Tue–Sat in 2027), with no 2027-specific dates on the page → treated as "page still shows 2026 dates".

### 10.3 🟢 Green: goes in the batch as-is
- The proof check passed.
- On an allowed source (section 8), with the year stated (or the camp says "2027 summer camps").
- Weekday test (always run): every 2027 session starts on the weekday the camp's pattern uses: Monday for weekly day camps, Sunday allowed for overnight camps. A "2027" somewhere on the page isn't enough on its own.
- Each session lasts 1–14 days, ages are within 0–21, and nothing contradicts other pages or earlier findings.

### 10.4 🟡 Yellow: P decides, case by case
- Only some dates posted (→ `partial`).
- No year on the page, but the weekdays only fit 2027.
- Sessions outside Jun–Aug (spring, winter or mid-term breaks are fine, but flag them).
- Sessions longer than 14 days or a single day.
- Short weeks around Jul 5 (Independence Day observed) and Sep 6 (Labor Day) are normal; note them, don't flag them.
- Hours, ages or price differ from `camps`.
- Registration info is vague.
- Previously stored dates have changed.
- The page is on the camp's site but different from `website_url`.
- **Two pages on the camp's site disagree.** The registration page wins; the review shows both quotes side by side.
- **Dates found only in an image or a PDF.** Save the file itself as proof. For a PDF, its text is also checked by the script; an image is checked by P.
- **One page lists several locations or age groups, and it isn't clear which part matches our camp.**
- The weekday pattern is mixed or unclear (some sessions fit 2027, some fit 2026).
- Teens / adults without numeric ages.

### 10.5 🔴 Red: not stored; status → `needs_review`
- **The proof check failed** (section 10.1).
- The page can't be reached twice in a row, or returns 404.
- The information isn't from an allowed source (section 8).
- The text can't be read as dates, or end comes before start.
- Ages outside 0–21.
- The camp appears closed or not running. Propose `not_running`; P confirms.
- The camp looks like a duplicate of another listing.

## 11. Remembering where each camp's dates live
So the next check goes straight to the right place instead of searching again.

**Stored in Supabase, `camp_seasons` (one row per camp per year):**
- `source_url` holds the **exact location** of the dates: the specific page, the PDF's address, or the registration-system page.
- `notes` starts with one line saying what kind of location it is:
  `Dates at: page` · `Dates at: PDF` · `Dates at: image` · `Dates at: registration system (linked from https://…)` · `Dates at: behind login`
- When next year's rows are created, `source_url` and the "Dates at" line are copied forward as the starting point.

**Weekly check order:** `source_url` first → the "linked from" page → `camps.website_url`. If the location moved, the new one replaces it (section 8) and the change is noted in the batch.

**Saved proof copies, on P's Mac only** (outside the repo, never uploaded):
`~/Documents/popcamps-backups/proofs/<season year>/_pages/<page key>/<YYYY-MM-DD>-page.<txt|html.gz|jpg|pdf>`, plus `-meta.json` listing the camps the page covers. Full HTML and screenshots are kept only when a page changed or produced a finding; unchanged pages keep only a text fingerprint in `sweep-state/state.json`.
For PDFs, keep both the PDF and its extracted text. Old copies are kept, so any stored value can be traced back to what the page said on that day.

## Open questions for P
1. **Inclusive tag later:** camps focused on neurodiverse or disabled kids (currently typed "Neurodiversity") need a separate "Inclusive" tag, not a camp type. To design later.
2. **Optional:** if the "Dates at" line in `notes` gets messy, a dedicated column could replace it. That would be a small schema change, as its own batch with P's yes.
