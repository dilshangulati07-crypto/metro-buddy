# Metro Buddy — project rules

Bangkok rail route finder. Read and follow these rules on every change.

## Stack
- Plain HTML, CSS and JavaScript (ES modules). No frameworks, no build step, no backend, no npm runtime dependencies.
- Static site hosted on Netlify. Everything must work when the folder is served as-is.
- Node is only used for local tests (`node tests/run.mjs`), never needed by the site.

## Design
- Clean, modern, premium. Not cartoonish, not "AI-looking".
- Colours (tokens on `:root` in `css/styles.css`):
  - Hero and top bar: deep navy `#0F1B33` (`--hero`) with white text.
  - Main accent for buttons and highlights: BTS green `#7AB800` (`--accent`). Text on it is navy `#0F1B33` (white fails contrast). Green text on light backgrounds uses `--accent-text` `#3F6600`.
  - Page below the hero: warm white `#F7F6F2` (`--page`), cards white, text navy.
  - Keep WCAG AA contrast (4.5:1 for text).
- Official line colours are only for line pills, badges and route lines (tokens in `css/styles.css` and `data/transit.json` `lines[].color`):
  BTS Sukhumvit light green, BTS Silom dark green, BTS Gold, MRT Blue, MRT Purple, MRT Yellow, MRT Pink, Airport Rail Link, SRT Red.
  Line pills are rounded, show `lines[].label_en/label_th` ("BTS Sukhumvit"), and use `lines[].text` for the text colour: white when it reaches 4.5:1, otherwise navy (computed in `tools/build-data.mjs`).
- Hero layout: left column has "Metro Buddy" in large bold Bricolage Grotesque, a one-line tagline and the search card under it. Right column has the photo `images/hero-train-*.{webp,jpg}`, large, rounded, fading into the navy. On phones the photo sits on top (shorter, cropped) with the text and card below. If the photo is missing the hero still works (the media block hides itself).
- Hero photo: drop the original at `images/hero-train.jpg` and run `python tools/optimize-hero.py`. It writes 1280 and 720 wide WebP + JPG, 4:3, each under about 250 KB. `index.html` sets width/height so nothing jumps.
- Radius scale `--r-sm/--r/--r-lg/--r-pill`; shadows only on the main cards (planner, summary, journey, fare).
- Fonts: Bricolage Grotesque (headings), Manrope (body), from Google Fonts.
- No emoji anywhere (UI, data, docs). No purple gradients. No drawn cartoon illustrations.
- Mobile first: base styles target ~380px, enhance with `min-width` media queries. No horizontal scroll at 380px.
- Thai/English switch for every user-facing string (`js/i18n.js`). Station names come from the data in both languages.

## Data (`data/transit.json`)
- Every station on every operating line: id, name_en, name_th, line, order, lat, lng, interchanges (same-station and walking transfers).
- Fares: each operator in `fares.systems` with a model, `sources` (`{label, url}`), `effective` date, and `status` per passenger type (`official` or `estimate`). Official station-to-station tables live in `tools/sources/fares/*.json` (with source links and dates) and are merged by `tools/build-data.mjs`. Never invent precision.
- Passenger types: adult, student, senior. Show all three on every route. An estimate is always shown as "about X THB" with "Check at the ticket machine", never as an exact fare.
- `data/fare-checks.json`: fares checked in person at a machine (`from`, `to`, `operator`, `adult`, `checked_on`). A check overrides the table for that trip and shows "Checked at station". Tests fail if the site shows a different fare from a check.
- Platforms: `tools/sources/platforms.json` (`stations[id].platforms = [{no, towards, via?, next}]` plus `sources`), merged into `stations[].platforms` / `platform_sources`. Only platforms confirmed by a source and validated against our network (the next station must match). Where they are not confirmed, show the direction only ("Towards Kheha"). Never guess a platform number.
- Average minutes between stations per line, transfer minutes per interchange.

## Code layout
- `js/route.js` — Dijkstra route finder, pure functions, step-by-step comments. No DOM access.
- `js/fares.js` — fare calculation, pure. Each operator charges separately. Returns all three passenger types with a status each.
- `js/platforms.js` — platform and direction for a ride ("Platform 2 · towards Kheha"), pure.
- `js/app.js` — UI wiring. `js/map.js` — schematic SVG. `js/i18n.js` — strings.
- Keep logic pure and testable; DOM code stays in UI files.

## Behaviour
- Shareable links via hash: `#from=<id>&to=<id>`.
- Recent and favourite routes in `localStorage` (always wrapped in try/catch).
- Kafé Bangkok cafés fetched from `https://kafebangkok.netlify.app/data/cafes.json`; if the fetch fails, hide the café sections silently.

## Testing
- `node tests/run.mjs` runs route, fare, fare-check and platform tests. Run after any change to data, fares or routing.
- Rebuild data with `node tools/build-data.mjs` after editing anything in `tools/`.
- Check layouts at 380px and 1280px.
- Deploy bundle: `metro-buddy-deploy.zip` on the Desktop with site files at the top level (index.html, 404.html, css, js, data, img, images, favicon.svg, netlify.toml, _redirects). No CLAUDE.md, README, tests, or tools inside.

## Git and GitHub
- Repo: https://github.com/dilshangulati07-crypto/metro-buddy (public, branch `main`). Live site: https://metro-buddy.netlify.app
- Always commit and push as **dilshangulati07-crypto**: git identity `Dilshan Gulati <278357136+dilshangulati07-crypto@users.noreply.github.com>` (set in this repo's local git config), and push with the `dilshangulati07-crypto` GitHub account (`gh auth switch -u dilshangulati07-crypto` if another account is active).
- After every task, commit the changes with a clear message and push to `origin main`. Never commit secrets, `.env` files or deploy zips (see `.gitignore`).
