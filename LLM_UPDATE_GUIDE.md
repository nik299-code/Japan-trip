# Japan Itinerary HTML: LLM Update Guide

This project is a single-page itinerary app with a late-stage data override that makes places editable from a JSON file.

## Files to know
- `/Users/nikunj/Documents/Apps/raw-trail-sorter/Japan-html/index.html`
- `/Users/nikunj/Documents/Apps/raw-trail-sorter/Japan-html/data/places.json`

## How data loading works
- `index.html` still contains legacy hardcoded datasets/scripts from the original file.
- The **final script block at the end of `index.html`** is the active override for places.
- That script fetches `./data/places.json` and renders these sections:
  - Restaurants tab (`id="restaurants"`) -> `#rest-grid`
  - Coffee tab (`id="coffee"`) -> `#coffee-grid` (mapped to `cafes` key in JSON)
  - Shops tab (`id="shops"`) -> `#shops-grid`
  - Photos tab (`id="photos"`) -> `#photos-grid` (mapped to `photos` key in JSON)

## Source of truth for GitHub updates
- Use `data/places.json` as the canonical data source for restaurants/cafes/shops/photos.
- Commit changes to `data/places.json` on GitHub to make permanent updates.
- Do not edit embedded legacy constants unless intentionally cleaning technical debt.

## JSON schema
Top-level object:

```json
{
  "restaurants": [Place],
  "cafes": [Place],
  "shops": [Place],
  "photos": [Place]
}
```

`Place` fields used by renderer:
- `name` (string, required)
- `city` (string)
- `area` (string)
- `cat` (string)
- `rating` (string)
- `price` (string)
- `note` (string)
- `mapUrl` (string URL)
- `bestDay` (string)
- `bestTime` (string)
- `bookmarked` (boolean)

Extra fields are tolerated but ignored by current override renderer.

## In-page editing behavior
- Each directory tab has controls injected by JS:
  - `+ Add <Type> (Link/Name)`
  - `Download custom <type>.json`
- Add flow now uses one input:
  - Paste Google Maps URL OR type place name
  - App does best-effort extraction from URL and runs OpenStreetMap (Nominatim) lookup to auto-fill metadata
- Added entries are saved to browser `localStorage` key:
  - `japan-itinerary-custom-places-v1`
- These local entries are merged with `places.json` at runtime.
- LocalStorage edits are browser-local and **not** committed to GitHub.

## If asked to add new places permanently
1. Edit `/Users/nikunj/Documents/Apps/raw-trail-sorter/Japan-html/data/places.json`.
2. Add entries into the correct array (`restaurants`, `cafes`, `shops`, `photos`).
3. Keep JSON valid.
4. Optional: ask user to clear localStorage if they need a clean view.

## If asked to remove legacy duplication
- Safest path is staged cleanup:
  1. Keep HTML structure/CSS.
  2. Remove old large constants and old render overrides only after verifying final data-driven script still runs.
  3. Test tab switching (`restaurants`, `coffee`, `shops`, `photos`) and counts.

## Quick test checklist
- Open `index.html` in a static server (recommended; `fetch` may fail under `file://`).
- Confirm places load from `data/places.json`.
- Add one place in UI and verify it appears.
- Refresh and verify localStorage persistence.
- Download custom JSON button works.
- If auto-lookup fails (network/rate-limit), entry should still be added with minimal info.

## Known caveat
- The file has multiple historical `showTab` overrides. The final override script re-wraps `showTab` again and should remain the **last script** in `index.html`.
