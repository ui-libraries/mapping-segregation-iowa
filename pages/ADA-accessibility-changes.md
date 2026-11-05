# Accessibility Changes: Mapping Segregation in Iowa (county map pages)

**Files updated:** `index-bhc.html`, `index-jc.html`, `index-lc.html`, `index-pc.html`, `index-dbq.html`, `index-sc.html`
**Target standard:** WCAG 2.1 Level AA (the benchmark commonly used for ADA Title II/III compliance)

> **Important caveat.** I can't certify legal ADA compliance. These changes address the WCAG 2.1 AA failures I found by reading the code. I validated the HTML structure and JavaScript syntax and tested the new data-table logic against sample records. I could **not** load the real GeoJSON data or test in a browser or screen reader, so please do a manual pass (see "Recommended testing" at the end).

---

## 1. Summary of what was wrong and what changed

| # | Problem found | WCAG criterion | Fix |
|---|---|---|---|
| 1 | No viewport meta tag (breaks mobile zoom/reflow) | 1.4.4, 1.4.10 | Added `<meta name="viewport" content="width=device-width, initial-scale=1">` (zoom not disabled) |
| 2 | `index-dbq.html` and `index-sc.html` had the title "…Polk County" | 2.4.2 Page Titled | Titles now match each page's county |
| 3 | No skip link; no `<main>` or `<nav>` landmarks | 2.4.1, 1.3.1 | Added "Skip to map" link, `<main id="main-content">`, `<nav aria-label="Main">` |
| 4 | Menu labels were `<h2>` headings; popups used `<h3>`, slider used `<h5>`; headings chosen for styling, not structure | 1.3.1, 2.4.6 | Page now has one `<h1>` (the county title). Menu labels are buttons/links, popup text uses styled `<div>`s, the data panel uses `<h2>` |
| 5 | Dropdown menus were `<a href>` links with `preventDefault`, no state, no Escape, and DOM order was the **reverse** of the visual order (floats) | 2.1.1, 2.4.3, 4.1.2 | Rebuilt as proper disclosure menus: `<button aria-expanded aria-controls>`, a real `<ul>` of links, Escape closes and returns focus, closes when focus or click leaves. DOM order now matches visual order. Current page marked `aria-current="page"` |
| 6 | Hover effects used inline `onmouseover` JavaScript; no keyboard focus styling anywhere | 2.4.7, 1.4.1 | Removed inline handlers. Added CSS `:hover` (underline plus color) and strong `:focus-visible` outlines on links, buttons, slider, map, popups |
| 7 | Header text contrast was 3.9:1 (white-ish on `#ac6853`); skyblue hover was 2.5:1 | 1.4.3 | Gradient start darkened to `#8a4b38` (6.1:1); hover is white plus underline |
| 8 | Map, slider and legend had no accessible names | 1.1.1, 4.1.2 | Map is `role="region"` with `aria-label` and keyboard help via `aria-describedby`. Slider has a label, description and a group label |
| 9 | Slider's "Year" readout was only visual | 4.1.3 Status Messages | Added a polite live region announcing "Showing N covenant records filed in or before YEAR." (debounced) |
| 10 | **All record details were available only on mouse hover**; map polygons can't be focused by keyboard | 1.1.1, 2.1.1, 1.4.13 | Added a **"Data table view"** toggle (see section 2). Popups still open on hover but persist and close with Escape or the close button |
| 11 | Color alone distinguished covenant types (red/orange/blue). Red vs. orange is indistinguishable for many color-blind users | 1.4.1, 1.4.11 | Switched to a color-blind-safe palette (below), added borders to legend swatches, and text equivalents via the table |
| 12 | Legend text `#555` on 70%-transparent white over a map | 1.4.3 | Legend backgrounds raised to 94% opacity, text `#1a1a1a`, restructured as a labelled group with a real list; swatches hidden from screen readers |
| 13 | Popup link text "Click here to see the covenant." opened a new tab with no warning; global `a {color:#f7f4ea}` rule | 2.4.4, 3.2.5 | Link text is now "View scanned covenant", with hidden text "(opens in a new tab)", plus `rel="noopener noreferrer"`. Link color set explicitly (8.4:1) |
| 14 | Fixed 60px header, `position:fixed` slider at 30% width, pixel font sizes (12-14px) | 1.4.4, 1.4.10 | Header now wraps (flexbox), slider scales to the screen, fonts use `rem`, text bumped to at least 14px |
| 15 | Navigation and button targets were small | 2.5.8 (WCAG 2.2) | Menu buttons at least 44px tall, menu links padded, range input at least 24px |
| 16 | Layer-control checkbox groups and popup close button unnamed (Leaflet 1.6) | 4.1.2 | Added `role="group"`/`aria-labelledby` to layer groups and `role="button"`/`aria-label="Close popup"` to the popup close button at runtime |
| 17 | No message if JavaScript or data fails | 3.3.1 | Added `<noscript>` message and a visible `role="alert"` error if a data file fails to load |
| 18 | Animations ignore user preference | 2.3.3 (AAA, cheap to add) | Map zoom/fade animation disabled when `prefers-reduced-motion` is set |
| 19 | Google Fonts requested only light (300) weights | 1.4.3 / legibility | Now loads regular and bold weights so body text and `<b>` render properly instead of faux-bold light |

## 2. New: Data table view (text alternative to the map)

A **"Data table view"** button (`aria-pressed`) sits above the map. It opens a panel over the map containing a real `<table>` with a caption, `<th scope="col">` headers, and one row per covenant record filed in or before the slider year. The slider stays usable below the panel, so the table updates as the year changes. Escape closes the panel and returns focus to the button; the map is made `inert` while the panel is open so focus can't get lost behind it.

Columns by file:

| File | Table columns |
|---|---|
| `bhc` | Type, Lot (legal description), Recorder's number, Date filed, Scanned document link |
| `jc` | Type, Current address, Subdivision, Year filed, Scanned document link. **Plus** a second table of African-American and Latin-American households |
| `lc` | Type, Subdivision, Year filed |
| `pc` | Type, Subdivision, Date filed, plus a one-line text summary of the dot-density layer |
| `dbq` | Type, Subdivision, Date filed, plus dot-layer summary |
| `sc` | Type, Subdivision, Year filed, Book/page, plus dot-layer summary |

All values are HTML-escaped; empty values read "Not available".

## 3. Things that now behave differently (please review)

1. **Color palette.** Petition: red → `#D55E00`; Subdivision: orange → `#009E73`; Parcel: blue → `#0072B2` (Okabe-Ito color-blind-safe set). In `jc`, Latin-American households: brown `#8B4513` → `#CC79A7`. To revert, edit the colors in `getColorRestrictive` / `getColorPoc`.
2. **`index-jc.html` household popups** used to close on mouse-out and had no close button. They now stay open until dismissed (Escape, close button, or map click), which is needed for WCAG 1.4.13 (hoverable, persistent).
3. **`index-bhc.html` bug fix.** The code built `'../' + rel_path` and then compared it to `'absent'`, which could never match, so every record got a link, including those with no document. It now checks `rel_path` itself. In `jc`, a link is only shown when `weblink` exists.
4. **Layout.** The header is no longer fixed-height/fixed-position. The page is now a flex column (header, toolbar, map). Visually it is close to the original, with a slim toolbar row added above the map.
5. **Popup/tooltip markup.** `<h3>` became `<div class="popup-text">`; same look, no false headings.
6. Dot-density and census code, GeoJSON file names and field names, map bounds, and slider ranges are **unchanged**.

## 4. Not fixed / needs your attention

- **Map polygons are still not keyboard-focusable.** Leaflet 1.6.0 vector layers can't take focus. The data table is the accessible equivalent. Upgrading Leaflet (1.9.x) would add built-in improvements (zoom/popup button labels), but I didn't change library versions without being able to test against your data.
- **Town-name tooltips** (hover only) have no keyboard equivalent. They label boundaries only and aren't in the table.
- **Dot-density layer** (`pc`, `dbq`, `sc`): dots are placed randomly on every update and can't be read individually; the panel gives a total per census year instead.
- **Possible data problem (not changed):** `index-dbq.html` and `index-sc.html` load `DSM_Census_1920/1930/1940.geojson` (Des Moines enumeration districts) and carry a "NEED TO CHECK" comment. The new dot summary reports whatever that layer holds, so verify those files are right for Dubuque and Scott counties.
- **Commented-out code** in `pc`/`dbq`/`sc` (old link code) and `jc` (county search block) still contains old patterns such as "Click here". It is inactive; delete it or fix it if you ever re-enable it.
- **Third-party basemap and Leaflet attribution** colors are outside this page's control.
- **Other pages** (`introduction.html`, `deedcovs.html`, `demo-*.html`, `seg-*.html`, `restr-*.html`, etc.) were not provided and should get the same review. Linked scanned-covenant PDFs also need to be accessible (tagged or with a text version); that's a separate remediation.
- The `fullscreenControl` option is set, but no fullscreen plugin is loaded, so it does nothing. Left as is.

## 5. Recommended testing

1. Keyboard only: Tab through the skip link, menus (Enter/Space opens, Escape closes), map (arrow keys, +/−), layer checkboxes, slider (arrow keys), Data table view toggle.
2. Screen reader (NVDA/JAWS with Firefox or Chrome, VoiceOver with Safari): confirm the slider announces "Year", the live status updates, and the table reads correctly.
3. Zoom to 200% and 400% (and a 320px-wide window) and check nothing is cut off.
4. Run axe DevTools or WAVE on each page with real data loaded.
5. Check the new colors with a color-blindness simulator.
