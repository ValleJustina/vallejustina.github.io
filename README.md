# Valle Justina Mountain Resort, website handover

## What is in this folder

```
index.html                                  the whole site
img/                                        all photography, logo, icons, GCash QR
Valle Justina Master Calendar Template.csv  sample rows for the master sheet
README.md                                   this file
```

No build step, no framework, no dependencies. Drag the whole folder onto
Netlify Drop, Cloudflare Pages, or a GitHub Pages repo and it is live.

## Deploying

1. Drag this folder to https://app.netlify.com/drop (free, HTTPS included)
2. Point the domain at it when one is registered
3. Replace `https://vallejustina.example/` in three places in `index.html`
   (the canonical link, the `og:image` URL, and the schema `url`) with the
   real domain. Facebook link previews will not render until that is done.

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

## If the calendar does not load

The page reads availability from the Apps Script web app:
`https://script.google.com/macros/s/AKfycbxvITWOYCqUiNQ7DZtlzakOVIe-s9D7IGvTLZBLkAfbjEg75RZ4BZq2zNd4bIJ3qXwY/exec`

I could not test that endpoint from my side, so check it once on the live
site. Open the page, scroll to Availability, and read the small status line:

- **"Synced with the resort calendar"** in green means it is working
- **"Could not reach the calendar"** in red usually means a CORS block

If it is blocked, the fix takes two minutes. In the Sheet go to
`File > Share > Publish to web`, choose the sheet, choose **CSV**, publish,
copy the link, and paste it into `CONFIG.fallbackCsvUrl`. The page tries the
Apps Script first and falls back to the CSV automatically, so you can leave
both in place.

The loader accepts JSON or CSV and either a `Room` or a `Unit` column, so
whatever shape the Apps Script returns should work without changes.

**The calendar fails closed on purpose.** If it cannot read the data, every
night shows as unavailable and the guest is told to message instead. Showing
someone a free night that is actually taken is worse than showing no calendar.

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
