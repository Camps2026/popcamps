# PopCamps Data Standards

The rulebook for every data task: research, weekly checks, batches, owner edits.
If a rule here and a habit disagree, this file wins. Changes to this file need P's yes.

Adopted Nov 24, 2026.

## 0. Ground rules
- PopCamps lists only camps located in Washington state.
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
- Different age groups or times in the same week are separate sessions, each with its own ages and hours. Each label uses only the camp's own words from its page (see EXTRACT.md); the sweep never makes one up.
- Large camps with many programs (e.g. YMCA Camp Orkila): every youth program session is stored as its own session, with its own `label` (the camp's program name) and ages, even when several programs share the same week. Counselor-in-Training (CIT) programs are kept as sessions. Internships, staff training and other staff programs, family camps and adult programs are left out. Holdover nights between sessions are not sessions.
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
- For 2027, always store the verified hours: in `camp_seasons.hours` when they're the same for every session, otherwise on each session. (`camps.hours` is 2026 data.) Unknown means empty, never "See website".
- Overnight camps (camps marked overnight in our listing): hours are stored as "Overnight", backed by a quote showing the camp is overnight or resident (e.g. "Overnight Camp", "resident camp"). This counts as filled for completeness (section 12).

## 3. Ages
- In years. Whole numbers, or one decimal when the camp states half years ("3½" → 3.5). Allowed range 0–21.
- "Ages 6–12" → 6 and 12. "8+" → minimum 8, maximum empty.
- Grades: minimum = lowest grade + 5, maximum = highest grade + 6 (K = grade 0). "Grades 1–5" → 6–11. Keep the camp's wording in `notes`, e.g. "Grades 1–5".
- Teens / adults without numbers → yellow (section 10).
- For 2027, store the verified ages on every session. (`camps.age_min` / `age_max` are 2026 data.)

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
| `partial` | Some or all 2027 dates are posted, but the listing isn't complete yet (section 12) | Show the posted dates. Add "More 2027 dates not yet posted" only when some 2027 dates are still missing (the full schedule isn't posted yet, section 12), not when only hours or ages are missing |
| `confirmed` | Complete (section 12) and approved in a batch. No longer checked, except the optional May check | Show dates |
| `needs_review` | Red-flagged information waiting for P | Treat as not_posted on the site until resolved |
| `not_running` | The camp says it won't run that year (P confirms first) | "Not running in 2027" |

Exact wording: **"2027 dates not yet posted"**. Use the season year, capital first letter, no trailing period on cards. For a date search with no results, keep the current message: "Most camps haven't posted 2027 dates yet. We're adding them as they're announced. Search without dates to browse all camps."

## 6. Price (`price_text`)
**Not collected for now.** Leave `price_text` in `camp_seasons` and `camp_sessions` empty; the columns stay in place, unused. *(For later, when prices are collected: "$350/week", "$95/day", "$1,250/session", "$350–$425/week", "Free"; comma for thousands; no cents unless non-zero; units /day, /week, /session, /month, /camp; member pricing "$350/week ($300 members)"; only from an allowed source.)*

## 7. Registration
| Field | Rule |
|---|---|
| `registration_opens` | Exact date only, when the camp states one ("Registration opens Feb 1, 2027"). |
| `registration_opens_text` | When there's no exact date: "Early February 2027", "Winter 2027". Capital first letter, include the year. |
| `registration_url` | The page where parents actually sign up (may be a registration system; see section 8). |

**Availability is not collected** (open / waitlist / full / sold out). PopCamps links parents to the camp for availability. The `registration_status` column stays in the database, unused. The registration opening date is still collected; it's optional and never blocks completion (section 12).

## 8. Sources (`source_url`)
**Allowed:**
1. **The camp's own site:** the same domain as `camps.website_url`, or a subdomain of it.
2. **A registration or booking system** (ActiveNet, CampBrain, CampMinder, UltraCamp, Sawyer registration, RegFox, Eventbrite and similar), **only when the camp's own website links to it.** Record the linking page in `notes` as "Linked from https://…". If the camp's site doesn't link to it, it doesn't count.

**Approved registration accounts.** For camps whose dates live on an outside registration system, P confirms once per organization that the account is the camp's official one (e.g. `anc.apm.activecommunities.com/olyparksartsrec` for the City of Olympia). An approved account counts as an allowed source for all of that organization's camps without a "linked from" check, and isn't asked about again. The list of approved accounts is kept in `approved-accounts.json` in the popcamps-sweep repository; additions need P's yes.

**Never allowed:** aggregators and listings such as ActivityHero, Yelp, Google, social media, news articles or PopCamps itself, even if their information looks right. They may point the finder to the camp's own site (section 13), but never count as a source.

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
     - **Hours and ages:** the script reads them out of the quote and compares them with the stored value after converting both to the standard formats in sections 2 and 3. So "9:00 AM - 3:00 PM" supports "9am–3pm", and "Grades 1–5" supports ages 6–11.
- **Every stored value** (each session's dates, hours, ages, registration opening date) **must be backed by a quote.**
- **If a quote isn't found in the saved page, or a quote is found but doesn't contain the value it is meant to support, the finding is 🔴 red and nothing from it is stored.**

### 10.2 Routine: not on the review list
These go in the batch only as routine updates (`last_checked_at` and `notes`). P approves the batch as a whole, but they aren't listed for review.
- **Page still shows only 2026 (past) dates:** status stays `not_posted`. Add the note "page still shows 2026 dates" and re-check next week.
- **Dates visible only behind a login or account:** never log in. Status stays `not_posted`, `schedule_note` = "See camp website for dates", plus a note "Dates behind login".
- **Can't be read automatically** (robots.txt says no, the site blocks automated reading, it shows a CAPTCHA, or it needs a login): never bypassed, never logged in. These camps appear in a separate **"Check by hand"** section of the review, apart from the normal review list.
- **Nothing changed** since the last check.
- Dates that start on the wrong weekday in 2027 but fit 2026's calendar (e.g. "Jun 22–26" is Mon–Fri in 2026 but Tue–Sat in 2027), with no 2027-specific dates on the page → treated as "page still shows 2026 dates".

### 10.3 🟢 Green: goes in the batch as-is
- The proof check passed.
- On an allowed source (section 8), with the year stated (or the camp says "2027 summer camps").
- Weekday test (always run): every 2027 session starts on the weekday the camp's pattern uses: Monday for weekly day camps; Saturday, Sunday or Monday for overnight camps (camps marked overnight in our listing). In a week whose Monday is a holiday (Memorial Day, Independence Day observed, Labor Day), a Tuesday start is also fine. The weekly check and the database audit use exactly the same rule. A "2027" somewhere on the page isn't enough on its own.
- Each session lasts 1–14 days, ages are within 0–21, and nothing contradicts other pages or earlier findings.

### 10.4 🟡 Yellow: P decides, case by case
- No year on the page, but the weekdays only fit 2027.
- Sessions outside Jun–Aug (spring, winter or mid-term breaks are fine, but flag them).
- Sessions longer than 14 days or a single day.
- Short weeks around Jul 5 (Independence Day observed) and Sep 6 (Labor Day) are normal; note them, don't flag them.
- Hours differ from our 2026 listing in `camps`.
- The camp's overall 2027 age range (the youngest minimum to the oldest maximum across all its 2027 sessions) differs from our 2026 listing in `camps`. Different ages on different sessions are not a flag on their own.
- Registration info is vague.
- Previously stored dates have changed.
- **Same calendar dates as the 2026 listing.** A 2027 session starts and ends on the same month and day as one of the camp's 2026 sessions in `camps.session_dates` (e.g. 2027 "Jun 22–26" when 2026 had "Jun 22–26"). Shown as "same calendar dates as the 2026 listing; may be last year's dates". Weekdays move by one day each year, so real 2027 dates almost never repeat 2026's exactly. The weekly check and the database audit both apply it.
- **Two pages on the camp's site disagree.** The registration page wins; the review shows both quotes side by side.
- **Dates found only in an image or a PDF.** Save the file itself as proof. For a PDF, its text is also checked by the script; an image is checked by P.
- **One page lists several locations or age groups, and it isn't clear which part matches our camp.**
- The weekday pattern is mixed or unclear (some sessions fit 2027, some fit 2026).
- Teens / adults without numeric ages.
- A required field is proposed as "not published by camp" after 3 checks in a row without it (section 12).

### 10.5 🔴 Red: not stored; status → `needs_review`
- **The proof check failed** (section 10.1).
- The page can't be reached twice in a row, or returns 404.
- The information isn't from an allowed source (section 8).
- The text can't be read as dates, or end comes before start.
- Ages outside 0–21.
- The camp appears closed or not running. Propose `not_running`; P confirms.
- The camp looks like a duplicate of another listing.
- **Outside Washington.** The camp's own page shows its location outside Washington: red ("outside Washington"), and its dates are not stored. Existing listings outside Washington go to P for a decision; nothing is removed without P's yes.

### 10.6 Approved exceptions
When P approves an item in a batch, the yellow reasons on that item are recorded as accepted for that camp: the batch, its date, and each reason. They're kept in `accepted-exceptions.json` in the popcamps-sweep repo. Only an approved batch adds to it, and only once that batch has run (the sweep sees the batch's timestamp in the database). Nothing is added by hand or automatically.

The database audit doesn't report an accepted reason again for that camp while the camp's stored 2027 sessions and hours are unchanged. If they change (e.g. new or different sessions), the reason is reported again. An accepted exception never changes a value and never marks a camp confirmed.

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

## 12. When a 2027 listing is complete, and when to check
**Required for a complete listing:** all 2027 session dates (the page shows the full 2027 schedule, with no sessions marked TBD or "coming soon"; see "Full schedule" below), ages (on every session), and hours (in `camp_seasons.hours`, or on every session; "Overnight" for overnight camps, section 2).
**Optional:** the registration opening date; it never blocks completion. **Not collected:** price and availability.

**Full schedule.** The 2027 dates count as complete when either:
- the camp's own page says the schedule is complete, with a quote under the proof rule (e.g. "Our full 2027 summer schedule is now posted"); or
- P marks the camp complete in the batch reply (e.g. "Y2 complete"). For every camp with dates, the review shows whether the saved page has any TBD / "more dates coming" wording, such as "TBD", "TBA", "coming soon", "more dates/sessions to be added" or "check back". The review shows either "None found" or the matching lines. P marks a camp complete only when none were found, and the batch refuses "complete" for a camp where some were found.

If the matched lines are clearly not about the 2027 summer schedule (e.g. 'check back for fall classes'), P can still mark the camp complete with 'Y2 complete (override)'. The batch records this as the notes line 'Full 2027 schedule posted (P override: matched lines not about summer)'.

Either way, it's recorded as the notes line `Full 2027 schedule posted`. The camp becomes `confirmed` only if its ages and hours are also filled.

**Leaving out sessions.** 'Complete' may be combined with 'without session N' only when the left-out sessions are not youth sessions under §1 (family, adult, staff, holdover). When a session is left out because the camp's page likely has a mistake (e.g. a day camp starting on a Saturday), don't mark the camp complete: it stays partial and keeps being checked until the page is fixed.

**Fill in as found.** Every verified field goes into the next review batch right away, even if other fields are still missing. The status stays `partial` until every required field is filled, then becomes `confirmed`.

**"Not published by camp."** A required field counts as filled when either:
- the camp's own page says it isn't published or given (with a quote, under the proof rule); or
- dates are posted and the field hasn't been found in 3 checks in a row. This is only proposed, as yellow, and is recorded only when P approves it in a batch. It is never set automatically.

It's recorded as a line in `notes`: `Not published by camp: ages` (or `hours`). This stops a camp from being checked forever.

**When to check:**
- `not_posted`, `partial` and `needs_review` camps stay on the normal schedule.
- For `partial` camps, the sweep looks only for the missing fields.
- `confirmed` (complete and approved) and `not_running` camps are no longer checked.
- **Optional May check** (a setting, on by default, that can be turned off): in early May, confirmed camps are re-checked by page fingerprint only, and the review lists just the ones whose page changed.

## 13. Finding a camp's new link (the finder)
When a camp's link may have moved, the weekly sweep searches for the camp's own new page. It never changes a listing by itself: **P approves every link change**, and approved changes ship as a reviewed listing batch with its undo file (section 0).

**When the finder runs** (any one of these):
- The camp's page returns 404 or 410, or can't be reached.
- It redirects to another domain, or to the site's home page.
- It's a parked or "domain for sale" page (e.g. hugedomains).
- It shows no camp content.
- From **February 1** (a setting), the camp still has no 2027 information.

**Where it looks, in order:**
1. The camp's own site: home page, `sitemap.xml`, and links named "camp" or "summer". At most 3 pages per camp per run.
2. A web search limited to the camp's own domain.
3. An open web search for the camp's name, city and "WA".

**Priority and limits.** Camps whose link is broken or moved (404/410, unreachable, redirect, parked page, no camp content) come first. Camps that only have no 2027 information yet get step 1 (their own site) only; they go on to steps 2 and 3 only after 4 weekly checks in a row with nothing new. At most 30 camps get web searches per run (a setting); the rest wait their turn, oldest first.

When several pages qualify for one camp, the finder offers the most specific camp page (a home page only when no camp page qualifies); between equally good pages, the one whose title or address says "summer".

Aggregators and listing sites (section 8) may point the finder to a page, but never count as proof. All page reads follow the same rules as the weekly check: honest identification, robots.txt, no logins, no forms, nothing that gets around a block.

**Proof that it's the same camp:**
- **Required:** the same name or organization, **and** a Washington location, each shown by a saved quote from the new page.
- **Stronger:** a matching phone number, a matching street address, or a link to the new page from the camp's old site.
- **Confidence:** *high* = required proof plus at least one stronger match; *medium* = required proof only; *low* = anything less, shown for information and never offered for approval.
- A page that shows a location outside Washington is rejected, even when the name matches (section 0).

**On the review page:**
- **Possible new link:** old link → new link, the proof quotes, and the confidence. P approves or rejects each one.
- **Still looking:** camps the finder couldn't place, with what it tried and the next try date. They're retried every 2 weeks.

**"May be closed."** Only suggested from March, and only after at least 3 tries in a row found nothing. It's a suggestion for P; nothing is removed, hidden or marked `not_running` without P's yes.

## 14. Weekly link check (`camps.website_url`)
Every camps.website_url is checked weekly. Each link is OK (loads, and it's the same camp: name or organization plus a Washington location), BROKEN (404/410, unreachable, parked or for-sale page) or SUSPICIOUS (loads but doesn't match the camp, or redirects to a home page or another domain). Broken and suspicious links go to the finder. The website never shows them; it shows 'Website link being updated' instead. Links added or changed by camp owners or through 'List Your Camp' must pass the same check before they're approved.

- A link the sweep may not read (robots.txt says no, the site blocks automated reading, or a CAPTCHA) is CHECK BY HAND: it's listed for P each week and stays shown on the website. If P confirms it's broken, it's treated as BROKEN.
- A link on an approved registration account (§8) is OK when the camp's name or organization is on the page.
- A link stored without https:// is OK if it works with https:// added (the website adds it); the audit reports it separately until the stored link is fixed.
- A redirect within the camp's own site (www or a subdomain of the same domain) is the same site, and OK if it's still this camp. A redirect to a home page, even on the same site, is SUSPICIOUS.

## Open questions for P
1. **Inclusive tag later:** camps focused on neurodiverse or disabled kids (currently typed "Neurodiversity") need a separate "Inclusive" tag, not a camp type. To design later.
2. **Optional:** if the "Dates at" line in `notes` gets messy, a dedicated column could replace it. That would be a small schema change, as its own batch with P's yes.
3. Camp 1611 (Challenge Island STEAM Camps – Edmonds) is on Edmonds Parks' registration account, but its provider tag says "City of Olympia". Is the tag wrong? (Not changed now; fixed later in a batch with P's yes.)
