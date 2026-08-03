# Valle Justina Mountain Resort, website handover

## What is in this folder

```
index.html                                  the main site
rules.html                                   Resort Rules & Agreement page
shuttle.html                                 Shuttle Service booking page
img/                                        all photography, logo, icons, GCash QR
Code.gs                                     the Google Apps Script backend
Valle Justina Master Calendar Template.csv  sample rows for the master sheet
README.md                                   this file
```

`rules.html` and `shuttle.html` are separate pages, not sections of
`index.html`, so they can be linked to directly (e.g. shared on Facebook) and
so `index.html` never has to grow any further. Each one is fully
self-contained — its own styles and scripts live inside the file, the same
way `index.html` works — so there is no separate `assets` folder to remember
to bring along. Drag all three `.html` files up next to `img/` and they work
on their own. They are already linked from the header nav and footer of
`index.html`.

`Code.gs` does not get uploaded to the website. It goes in the Google Sheet.

No build step, no framework, no dependencies. Drag the whole folder onto
Netlify Drop, Cloudflare Pages, or a GitHub Pages repo and it is live.

## Deploying

1. Drag this folder to https://app.netlify.com/drop (free, HTTPS included)
2. Point the domain at it when one is registered
3. The live URL is already set to `https://vallejustina.com/` in the canonical
   link, the `og:image` tag, and the schema. If the domain ever changes, those
   are the three places to update.

## Editing content

Everything the resort will want to change lives in one place: the `CONFIG`
object near the bottom of `index.html`, right after the comment banner that
says `CONFIG`. Rates, room names, phone numbers, the Messenger link, the
early-bird rule, and the availability source are all there. Nothing below
that block needs touching to change a price.

The calendar rolls forward twelve months from whatever today is, so it never
needs a yearly edit.

## The master calendar

Sheet: https://docs.google.com/spreadsheets/d/1Z0sT4xBUGa6RPW5x-mQ4yrZG8Y5Hzcv9pf440U4FZJs

Three columns, exactly these headers:

| Date | Room | Status |
|---|---|---|
| 2026-08-15 | Standard 1 | booked |
| 2026-08-15 | Standard 2 | hold |
| 2026-08-16 | Balcony | booked |
| 2026-08-22 | Camping | booked |

- **Date** must be `YYYY-MM-DD`
- **Room** must match a `sheet` value in `CONFIG.units`: Standard 1,
  Standard 2, Standard 3, Balcony, Camping. Note the balcony row is keyed
  `Balcony` in the sheet even though the page displays it as Balcony Room
- **Status** is `booked` or `hold`. Anything else, including a blank, counts
  as open, so you only ever add rows for nights that are NOT available
- Capitalisation and stray spaces do not matter. Malformed rows are skipped
  rather than breaking the calendar

Only add rows for unavailable nights. An empty sheet means everything is open.

## Shuttle Service

`shuttle.html` works exactly like Day Pass and Camping: the guest pays first,
types their GCash or BDO reference, and the page issues an instant ticket,
then logs it to a new **Shuttle** tab in the master sheet and emails the
resort, same as the others.

The fare is flat &#8369;1,200 round trip **per vehicle**, up to 8 people, not
per person. There is no fixed timetable built in on purpose — the guest picks
a preferred date and pickup time, and the resort confirms the exact time with
them afterwards (by Messenger or call), the same way you would coordinate a
shared van in practice. If this ever needs to become a real fixed-schedule
system with seat tracking, that is a bigger job than today's version and
worth a separate conversation.

Change the fare, the passenger cap, or the payment details at the top of the
form in `shuttle.html` if any of those change.

## Resort Rules & Agreement

`rules.html` is a plain content page: house rules, safety notes, and the
guest agreement, covering rooms, camping, and day pass visitors alike.
Several details are marked with a dashed **to be confirmed** tag, the same
convention used elsewhere on the site, for policies that were not settled
yet when this was written — quiet hours, pet policy, and the cancellation
and refund policy. Search `rules.html` for `todo` to find and fill in each
one once the resort has decided.

## The Apps Script backend

`Code.gs` does two jobs for the site:

- **GET** hands the website the availability calendar as JSON
- **POST** receives a reservation request, writes it to a **Bookings** tab in
  the master sheet, and emails the resort

### Installing it

1. Open the master sheet, **Extensions > Apps Script**
2. Delete whatever is in the editor and paste the whole of `Code.gs`
3. Save. In the function dropdown pick **`setup`** and click **Run**
4. Google asks you to authorise. Pick your account, then **Advanced >
   Go to ... (unsafe) > Allow**. That warning is normal for your own scripts
5. **Deploy > New deployment > gear icon > Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**  ← this is the one that matters
6. Deploy, copy the `/exec` URL, and make sure it matches `CONFIG.dataUrl`
   in `index.html`

After any edit to `Code.gs` you must **Deploy > Manage deployments > edit
(pencil) > Version: New version > Deploy**. Saving alone does not update the
live URL, and this catches almost everyone at least once.

After running `setup`, close and reopen the spreadsheet tab (or reload the
page) so the new **Valle Justina** menu appears in the Sheet's own menu bar,
next to Help. It is created by `onOpen`, which only runs when the sheet is
opened, not when the script is saved.

### Multiple rooms in one request

When a guest selects more than one room for the same dates, the site sends
every room in that request together, and `Code.gs` writes a Hold row for
**every night, in every one of those rooms** — not just the first one. If you
ever see a multi-room request only holding one room in the calendar, it is
almost always the 10-minute auto-refresh on the page catching you before the
new rows have loaded, not a room actually missing its Hold. Refresh the page,
or check the `Sheet1` tab directly, before assuming a room was skipped.

### Confirming a booking once the deposit lands

The Sheet now has its own menu: open the master sheet and look for
**Valle Justina** in the menu bar, next to Help. Choose
**Confirm booking by reference…**, type the reference number from the
reservation email (e.g. `VJ-20260803-AB12`), and it flips every Hold row that
request created — across every room and every night — to **Booked**, all at
once. That is the live calendar's "confirmed" state, and the only thing that
makes a room unavailable to future guests.

This only applies to room reservations. Day Pass, Camping, and Shuttle
tickets are already pay-first, so there is nothing to confirm about them —
they are simply a record once the reference is entered.

You can still edit the Status column in `Sheet1` by hand instead, at any
time; the menu tool is just a shortcut for when a booking covers several
rooms or several nights and editing every row by hand would be tedious. If a
reference is not found, or is from a booking made before this tool existed
and has no saved room/date detail, the tool tells you so and you edit the
Status cells directly, exactly as before.

**This tool must be run inside the Sheet, not from the website.** The
website's script runs `Anyone` access, so nothing that marks a booking as
paid and confirmed can safely live there — it has to stay in a place only
you and your staff can open.

### Emails

Reservation requests go to `vallejustina25@gmail.com`, copied to
`apayaresorts@gmail.com` and `vallejustina@gmail.com`, so all three see every
request. Change `NOTIFY_TO` and `NOTIFY_CC` at the top of `Code.gs` to adjust.
`NOTIFY_CC` takes a comma separated list with no spaces. If the guest gave an email, replying to the notification
replies straight to them.

Gmail allows about 100 of these a day on a free account, which is far more
than this site will ever produce.

### If the calendar still does not load

Open the page, scroll to Availability, and read the small status line:

- **"Synced with the resort calendar"** in green means it is working
- **"Could not reach the calendar"** in red means the request is being refused

Nine times out of ten this is **"Who has access"** set to anything other than
**Anyone**, or a deployment that was never given a new version after an edit.
Check both first.

If it still refuses, use the backup route: in the Sheet go to
`File > Share > Publish to web`, choose the sheet, choose **CSV**, publish,
copy the link, and paste it into `CONFIG.fallbackCsvUrl` in `index.html`. The
page tries the Apps Script first and falls back to the CSV on its own, so you
can safely leave both in place.

The loader accepts JSON or CSV and either a `Room` or a `Unit` column, so the
exact response shape does not matter.

**The calendar fails closed on purpose.** If it cannot read the data, nothing
is bookable. Rather than paint a grid of grey "taken" cells, which would read
as "this resort is fully booked forever", the section hides the grid and shows
a short "ask us about your dates" card with a Messenger button instead.

Reservation requests are still captured while the calendar is down, because
the form posts to the script independently of the calendar read.

## Rates on the page

| | |
|---|---|
| Standard Room (three of them) | P3,000 per night, good for 4 to 5 |
| Balcony Room (one only) | P3,500 per night, good for 2 to 3 |
| Both include | pool for 4 guests, breakfast for 2 |
| Booked a month ahead | 10% off, applied automatically |
| Day pass entrance | P50 per person |
| Daytime swim, non-guest | P250 |
| Night swim, non-guest | P300 |
| Staying guests | swimming free |
| Camping | per pitch, bring your own tent, rate to confirm |
| Exclusive use | P50,000 |

## Still to fill in

Each of these appears on the page as a visible dashed "to be confirmed" tag,
so nothing looks broken while you wait for the client:

1. Room names, if they are called something other than Standard 1 to 3
2. Camping rate per pitch
3. Day pass opening hours
4. Extra guest charges
5. Travel times from Masbate City and the airport, and the road condition

Items 1 to 4 are one-line edits in `CONFIG` and the matching section of the
page. Item 5 is in the Getting Here section.

## Location

The map is embedded directly using the Plus Code `8JGP+7M Mobo, Masbate`,
so nobody has to click through to see where the resort is. There is still an
"Open in Google Maps" button underneath for people who want directions on
their phone, and a copy button for the Plus Code itself.

## Known limitation

Every photograph came from the company profile PDF and had already been
compressed for print layout. The largest usable width is 800px and several
are 480px. They look good at the sizes used here, which is why the hero is a
split panel rather than full-bleed, but original camera files would let the
whole page go up a level. The logo is likewise small, so a vector version is
worth asking for before anything gets printed.

## What was tested

Chromium, at 320, 360, 390, 414, 560, 768, 1024, and 1440px wide.

- Weekday alignment verified against real dates, Monday-start
- Availability parsing, including messy status values, ISO datetimes, and junk rows
- Date-range selection, blocked-range rejection, nightly totals
- Early-bird maths: 3 nights in the Balcony room 40 days out gives 9,450 from 10,500
- Fail-closed behaviour with the data source unreachable
- Form validation and the generated enquiry message
- Keyboard: arrow-key navigation across the calendar, tab controls on the
  time-of-day section, one tab stop per calendar rather than 175
- Screen reader text on every cell, `aria-disabled` on taken nights,
  `scope` on every row and column header
- No horizontal overflow and no touch target under 44px at any width
- Contrast ratios calculated, not eyeballed
