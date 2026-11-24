# LeadFinder (SIGNAL)

A single-page browser tool that finds local businesses with no website listed on Google, scores them as sales leads, and tracks cold-call outreach against them. It is built for freelance web developers and small agencies prospecting for clients; the in-app name is "SIGNAL — Lead Finder for Web Dev Outreach".

**Live demo:** https://dbrokenai.github.io/LeadFinder/ (requires your own Google Places API key; see [Running it](#running-it))

## What it does

- **Finds businesses without a website.** Searches the Google Places API (New) around one or more locations and keeps only the places that have no website on their Google listing and are not permanently closed.
- **Bulk location search.** Accepts several cities or ZIP codes, one per line, with a selectable radius from 1.5 km to 50 km.
- **Targeted or broad sweep.** Enter a business type (for example "salons") to run one query, or leave it blank to sweep a built-in list of 44 small-business categories (restaurants, plumbers, dentists, locksmiths, and so on) for each location.
- **Lead scoring.** Each lead gets a 0-100 score and a hot / warm / cool / cold tier, shown as a badge on its card.
- **Outreach tracking.** Mark each lead as Called, Interested, or Pass, and keep free-text notes per lead. Tabs filter by status with live counts; interested leads are pinned to the top and passed leads to the bottom.
- **Sorting.** By score, newest, rating, review count, or name.
- **Call script templating.** Write one call script with `{{name}}`, `{{category}}`, `{{address}}`, `{{phone}}`, `{{rating}}`, and `{{city}}` placeholders; clicking "Use Script" on a lead fills it in for that business and makes it copyable to the clipboard.
- **CSV export.** Exports the current filter view (name, phone, address, category, rating, review count, score, status, notes, search location, Google Maps URL) as a dated CSV file.
- **Persistence.** Leads, notes, statuses, the script template, and the sort preference are saved in the browser and survive reloads.

## How it is built

**Stack:** one `index.html` file containing hand-written HTML, CSS, and vanilla JavaScript. No framework, no build step, no dependencies, and no backend. The only external resources are Google Fonts and the Google Places API.

**Data source:** the Google Places API (New) Text Search endpoint (`places:searchText`), called directly from the browser with `fetch`. The user supplies their own API key.

Engineering decisions visible in the code:

- **Two-step search.** Each location string is first resolved to coordinates with a one-result Text Search, then each category query runs with a circular `locationBias` around that point. This avoids needing a separate geocoding API.
- **Field masks.** Every request sends an `X-Goog-FieldMask` header naming only the fields the app uses (ID, name, address, phone, website, rating, review count, type, opening hours, photos, Maps link, business status), rather than requesting full place objects.
- **Deduplication by place ID.** A set of known place IDs is seeded from saved leads, so overlapping categories, overlapping locations, and repeat searches do not create duplicate leads.
- **Transparent scoring heuristic.** The score is a small, readable function: 30 points for having a phone number (the tool is for cold calling), up to 25 for rating (linear from 3.0 to 5.0), up to 25 for review count on a log scale, and 10 each for having a photo and listed opening hours. Leads saved before scoring existed are backfilled on load.
- **Partial-failure handling.** A failed geocode or a failed category query is logged and skipped, so one bad location or request does not abort a bulk run. Progress (location N of M, query N of M) is shown while the search runs.
- **Output safety.** All API-sourced text is HTML-escaped before being rendered into lead cards, and CSV fields are quoted with embedded quotes doubled.
- **Client-side state.** State lives in a few `localStorage` keys and the UI is re-rendered from that state by a single `render()` function. The CSV download is generated in the browser with a `Blob`.
- **Responsive layout.** A two-column layout (sticky script panel plus lead grid) that collapses at 1024px and 768px breakpoints.

## Running it

1. Open the [live demo](https://dbrokenai.github.io/LeadFinder/), or clone the repo and open `index.html` in a browser. There is nothing to install or build.
2. Create a Google Cloud project, enable **Places API (New)**, create an API key, and enable billing on the project. The app includes a short setup guide in its key dialog.
3. Paste the **Google Places API key** into the dialog that appears on first load (or via the settings button). The key is stored in your browser's `localStorage` and is sent only to Google.
4. Enter one or more locations, optionally a business type, pick a radius, and run the search.

Searches are billed to your own Google Cloud account. A blank business type runs 44 queries per location, so broad sweeps over many locations use correspondingly more API calls.

## Status and limitations

- A working personal tool, not a hosted product: no accounts, no server, and no sync between browsers or devices. Clearing browser data deletes saved leads, so export to CSV to keep them.
- Each query returns at most 20 places and results are not paginated, so dense areas are sampled rather than exhaustively covered. Narrower radii and more specific business types give better coverage.
- The radius is applied as a location bias, not a hard restriction, so some results can fall outside it.
- "No website" means no website on the Google listing. A business may still have a site that is not linked there.
- Queries run sequentially, so a broad sweep across several locations takes a while.
- The API key is used client-side and is embedded in the photo URLs saved with each lead, so restrict the key (HTTP referrer and API restrictions) in Google Cloud Console.
- No automated tests.
