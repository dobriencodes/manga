# Manga Collection

A self-contained, static viewer for my manga collection. The presentation
(`index.html`) and the data (`data.json`) are kept separate so the data can be
edited, diffed, and automated against without touching the rendering code.

## Files

| File | What it is |
| --- | --- |
| `index.html` | The viewer — all styling, layout, sorting, filtering, and stats logic. Fetches `data.json` on load. |
| `data.json` | The collection itself — one JSON object per series. **This is the file you edit.** |

## Viewing it

Because `index.html` loads the data with `fetch()`, it must be served over
**http**, not opened as a `file://` path (browsers block local file fetches).

Locally:

```bash
cd manga-collection
python3 -m http.server 8000
# then open http://localhost:8000
```

## Hosting it on GitHub Pages

1. Push this folder to a GitHub repo.
2. Repo → **Settings → Pages**.
3. Source: **Deploy from a branch**, branch **main**, folder **/ (root)**.
4. Save. The site goes live at `https://<username>.github.io/<repo>/` within a minute.

Every push redeploys automatically. On a free account the repo (and the published
site) is public; that's fine for a collection catalog.

## Editing the collection

Edit `data.json`, then:

```bash
git add data.json
git commit -m "One Piece vol 112; Akane-Banashi vol 17"
git push
```

Each change shows up as a clean, field-level diff, so the commit history doubles
as a changelog of the collection.

## Record schema

Each entry in `data.json` is an object. Only `title` is strictly required; the
viewer handles missing optional fields gracefully.

| Field | Type | Notes |
| --- | --- | --- |
| `title` | string | Series title. |
| `owned` | string | Human-readable owned volumes, e.g. `"1–16"`, `"1–9 (complete)"`, `"Omnibus 1–4 (JP vols 1–8)"`. |
| `ownerStatus` | enum | `"Complete set"` · `"Partial run"` · `"Single volume"` · `"Keepsake"` · `"Not owned"`. |
| `jpStatus` | enum | `"Complete"` · `"Ongoing"` · `"Hiatus"`. |
| `enStatus` | enum | `"Collected"` · `"Caught Up"` · `"Behind"` · `"Catching Up"`. |
| `enVols` | string | Count of English volumes available — used as the progress-bar denominator. |
| `jpVols` | string? | JP volume count, e.g. `"21+"`, `"8"`. |
| `nextVol` | string? | `"YYYY-MM-DD — Vol. N"`, or `"TBD — Vol. N"`, or `""`. |
| `notes` | string | Free text. |
| `donate` | boolean | `true` shows the 🎁 marker and donate-row styling. |
| `gap` | string \| null | Missing-volume count. `null` suppresses the gap (keepsakes, one-shots). |
| `verified` | string | `"YYYY-MM-DD"` — last time the row was checked. |
| `progOverride` | string? | Manual numerator for the progress bar when `owned` can't be auto-parsed (omnibus / non-sequential runs). |

### Conventions

- **Keepsakes** (`ownerStatus: "Keepsake"`) are 1-of-1, gap-suppressed (`gap: null`),
  and badged 🔖 — variant/exclusive volumes, not part of regular progress tracking.
- **progOverride** is used where the owned string isn't a simple `1–N` run.
  Example: One Piece owns 1–23, Box Set 2 (24–46), and 101–111, so `progOverride: "57"`.
- **Dates** are parsed as local time (`new Date(y, mo-1, d)`) to avoid the
  off-by-one timezone shift that `new Date("YYYY-MM-DD")` causes in browsers.

## Note on in-page toggles

The Donate and Verified buttons update the view in the current session only —
they do not write back to `data.json`. Persist a change by editing `data.json`
and committing.
