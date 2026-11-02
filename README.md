# layer5.io on Hugo — proof of concept

A working skeleton for **Phase 0** of the Gatsby → Hugo migration plan, plus the tooling for Phases 1, 4 and 5 so the team can start in parallel on day one.

```
make serve         # full site, live reload, no "lite" mode
make test          # 15 tests for the migration tooling (Node ≥ 18, nothing to install)
make bench         # 1,210 synthetic pages / 17 collections → wall time + peak RAM
make index         # build + Pagefind search index
make verify-urls   # fail if any live Gatsby URL would 404
```
Needs **Hugo Extended** (CI pins 0.145.0). Node is used only for the tooling in `scripts/` and for Pagefind. The site itself builds without it.

## Verification status (read this first)

| Piece | Status |
|---|---|
| `scripts/migrate-mdx.mjs` (MDX → shortcodes, front matter, reports) | **Run and tested**: 12 tests, idempotent, fence/inline-code safe |
| `scripts/verify-urls.mjs` (sitemap parity + redirects) | **Run and tested**: 3 tests |
| `scripts/generate-synthetic-content.mjs` | **Run**: produces 1,210 pages in 17 collections |
| Browser JS (`main.js`, `topology.js`, `calendar.js`) | Syntax-checked only |
| Hugo templates, shortcodes, `hugo.yaml`, `bench.sh`, CI | **Not built.** Hugo wasn't available where this was written. Templates were statically linted (balanced blocks, every referenced partial/shortcode exists) |

So: expect to spend the first ~15 minutes running `make serve` and fixing a typo or two in a template. **No performance number in this repo has been measured.** The dashboard's “Measured” column is empty on purpose until `make bench` has run on CI hardware. Quote measured numbers in the meeting, not the plan's targets.

## What's in the box

| Plan reference | Where | Notes |
|---|---|---|
| §3 config, permalinks, taxonomies | `hugo.yaml` | `buildFuture: true` (events are future-dated; Hugo hides them otherwise), `unsafe` Goldmark for inline HTML |
| §3 layouts / partials | `layouts/` | home, blog list+single (related posts, JSON-LD), integrations (filter), members, handbook (sidebar + TOC), migration dashboard |
| WS1 shortcodes | `layouts/shortcodes/` | `button`, `cta-fullwidth`, `terminal` (copy), `accordion`, `card`, `topology`, `calendar` |
| WS1 converter | `scripts/migrate-mdx.mjs` | Dry-run report lists **unknown components**, which is your real work queue |
| WS2 theme | `partials/head.html` | Inline script sets theme before first paint: no flash, no hydration |
| WS2 search | `partials/search-dialog.html`, `main.js` | Pagefind modal, `/` or ⌘K; degrades with a clear message if no index |
| WS2 Cytoscape / FullCalendar | `static/js/*.js` | Pinned versions, loaded only when the hook exists *and* is on screen |
| WS2 forms | `partials/newsletter.html` | HTML5 validation + `fetch`; demo mode until an endpoint is set in `hugo.yaml` |
| WS3 images | `partials/responsive-image.html` | `<picture>`, WebP, srcset, no upscaling, width/height set; skips SVG/GIF |
| WS4 redirects | `aliases:` + `layouts/index.redirects.txt` | Emits a Netlify/Cloudflare-style `_redirects` from every alias |
| WS4 link safety | `verify-urls.mjs`, CI `lychee` step | |
| §5 Phase 0 CI | `.github/workflows/hugo.yml` | Build, Pagefind, link check, benchmark, 10-minute tripwire |
| §6 contribution model | `docs/` | Cheatsheet, 10 first issues with acceptance criteria, 8-minute demo script |
| §6.3 burndown | `/migration/` | Reads `data/migration.json`; the converter's `--progress` flag updates it |

Sample content is placeholder (titles start with “Sample:”). Member roles come from the plan; bios are intentionally empty.

## Design notes

- **Zero third-party requests on a normal page**: system fonts, no icon fonts, no analytics. CDN scripts load only for Cytoscape/FullCalendar pages.
- CSS variables (`--accent`, `--bg`, …) are the seam for **Sistent tokens**: swap values, not selectors (see feedback note on Sistent in `docs/PLAN_FEEDBACK.md`).
- Accessibility floor: skip link, visible focus, `prefers-reduced-motion`, `aria-pressed` on filters/theme toggle, `<noscript>` calendar fallback.

## Suggested next 48 hours
1. `make serve`; fix whatever Hugo says. 2. `make bench` on the CI runner; paste real numbers into `data/migration.json`.
3. Check out `layer5io/layer5` beside this repo → `make migrate-dry SRC=../layer5/src/collections` → turn each *unknown component* into an issue.
4. Download the live sitemap → `make verify-urls`. 5. Open the epic and file `docs/GOOD_FIRST_ISSUES.md`.
