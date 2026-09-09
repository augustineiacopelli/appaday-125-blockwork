# App 125 &mdash; Blockwork

**An offline shift coverage scheduler.** Build a roster, set each person's recurring weekly hours by location, then watch a rolling two-week grid resolve those hours against approved time off, campus closures, and one-date overrides, with coverage gaps drawn underneath each week.

**Live:** https://augustineiacopelli.github.io/appaday-125-blockwork/
**Portfolio:** https://augustineiacopelli.github.io/appaday/

Part of [AppADay](https://augustineiacopelli.github.io/appaday/), one complete web app shipped every day.

---

## What it does

A published schedule is never just the sum of everyone's regular hours. Somebody is on vacation Tuesday, the campus is closed Monday, a student's evening shift gets cut on a reduced-hours day, and one person swapped to a split shift for a single Wednesday. Blockwork holds all four of those layers and resolves them into one grid, then shows you where the resulting coverage falls below what the location actually needs.

Everything runs in the browser with no account, no server, and no network. Open the file from a thumb drive and it works.

## The resolution engine

The order matters more than any single rule, because it is what makes overrides win and closures compose correctly. Every date is painted into fifteen-minute blocks per location, then seven stages run in sequence.

1. **Regular hours** are painted into blocks. Each start is floored down to the nearest quarter hour so a 9:20 shift stays contiguous, while the true start and end are stored in a lookup so the grid still displays 9:20 rather than 9:15.
2. **A snapshot** of that pure result is kept for deviation detection.
3. **Closures** apply. A Holiday wipes every location. Reduced Hours trims Student shifts to the 8:00 to 5:00 window and then drops any student left with two hours or less. A Site Closure moves that location's staff to the primary location and records the redirect.
4. **Approved time off** is removed from every block it overlaps, including a stray-tail rule: a single orphaned block sitting within fifteen minutes of the leave end, with nothing scheduled before the leave began, gets wiped too.
5. **Overrides** apply last and beat everything. On the destination location the person is scrubbed from the whole day and repainted only inside the override window; on every other location only the blocks inside that window are scrubbed.
6. **A full-day-off map** is built from dates where approved leave covered the entire normal shift, so the grid can distinguish OFF from simply not scheduled.
7. **Coverage** is computed by walking each location's configured window in fifteen-minute steps on weekdays, counting staff per block, and flagging anything below the minimum.

Any cell whose resolved value differs from the step-two snapshot picks up a gold accent border, so what changed is visible at a glance.

## Screens

**Schedule** holds the two-week grids, one per location, with week navigation and a coverage strip under each week. Every cell is tappable and opens the override editor.

**Roster** manages staff and their recurring weekly hours, with multiple rows per weekday allowed for split shifts and multi-location days.

**Time Off** takes requests and holds them in a queue. Approving one recomputes the grid immediately.

**Calendar** holds closures, typed as Holiday, Reduced Hours, or Site Closure.

**Data** renames the four location slots, sets each one's staffing window and minimum, and handles JSON export, import, and reset to sample data.

## Coverage strips

Under each week, an inline SVG draws one bar per fifteen-minute block across the configured window, with height proportional to staff on the clock, blocks below the minimum drawn in red, and a dashed rule at the required level. No canvas and no charting library, so the visualization works offline like everything else.

## Printing

A print stylesheet drops the navigation and controls, switches to landscape, and prints one location per page with the two-week grid intact. No popup window and no blob URL, so no popup blocker gets in the way.

## Data

All state lives in `localStorage` under the single key `appaday-125-blockwork`, wrapped in `try/catch` throughout. Sample data loads on first run and is fully editable afterward. Export writes a JSON snapshot; import replaces everything after a confirmation.

An optional `.ics` export produces all-day events for approved leave and timed events for resolved shifts, in `America/Chicago` with an embedded `VTIMEZONE` block.

## Built with

One file. Vanilla HTML, CSS, and JavaScript. No frameworks, no build step, no dependencies beyond Google Fonts. Hand-drawn inline SVG for the coverage visualization.

## Known future additions

**Effective-dated regular hours.** Today a change to someone's recurring schedule applies to every week, past and future. Real schedules change mid-semester, and the honest fix is to give each regular-hours row a start and end date and resolve against the date being rendered. It was deliberately left out to keep this a one-day build.

Also out of scope by design: shift trades, approver hierarchies, proxy approvers, email of any kind, audit logging, reports, a four-week view, and multi-user anything.

One implementation note beyond the original spec: closures carry a location field, used only by Site Closure, since that type has no other way to name the site that closed.
