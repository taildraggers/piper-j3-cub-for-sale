# Piper J-3 Cub For Sale

Daily aggregator of Piper J-3 Cub aircraft classified listings from
[Barnstormers.com](https://www.barnstormers.com), published as a static page
(`docs/index.html`) meant to be embedded via `<iframe>` on taildraggers.com.

## How it works

- `scraper/barnstormers.py` runs a Barnstormers advanced search for ads whose
  headline contains **"Piper J3"**, and walks every page of results: Barnstormers
  shows 25 ads per page with a `&page=N` pager, so the scraper requests page 1, 2, 3,
  ... until a page turns up no new ads. It then visits each listing's detail page to
  pull out the price, location, and posted date (falling back to regex heuristics over
  the visible text since the site doesn't expose structured data). The title is
  derived from the listing URL's own SEO slug, since every detail page shares one
  generic `<title>`/`<h1>`.
- Only whole-aircraft-for-sale listings are published. Each ad's title must name a
  J-3 (`J-3`, `J3`, `J3C-65`, ...) or its military L-4, and no other Cub model
  (PA-11, PA-18 Super Cub, J-4, J-5, ...); see `_J3_RE`/`_L4_RE`/`_OTHER_MODEL_RE` in
  `scraper/barnstormers.py`. Titles that read as parts, accessories, services, or
  raffles are dropped (phrases like "Clipped Wing" or "on floats" are allowed - see
  `ALLOWED_PHRASES` in `scraper/common.py`), as are ads priced under $1,000
  (`MIN_AIRCRAFT_PRICE`). Every surviving listing's title is rewritten to a
  canonical **`YEAR Piper J-3 Cub`** (or `YEAR Piper L-4 Grasshopper`) form, leaving
  the year off when the ad doesn't state one.
- `main.py` runs the scraper, de-duplicates results, and renders them into
  `docs/index.html` titled **"Other Piper J-3 Cub Ads on the Web"**, with one row per
  listing: Title (linked to the original ad), Price, Location, Date Posted, and Site
  Posted On. Links use `rel="noopener noreferrer"` and the page sets a
  `no-referrer` meta policy, so Barnstormers never sees that the click came from
  taildraggers.com.
- `.github/workflows/daily-scrape.yml` runs the whole thing once a day (13:00 UTC),
  commits the regenerated `docs/index.html` if it changed, and can also be triggered
  manually from the Actions tab (`workflow_dispatch`).

## One-time setup: enable GitHub Pages

This repo publishes `docs/index.html` as a plain static file — GitHub Pages just needs
to be pointed at it once:

1. Go to **Settings → Pages** in this repository.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Branch: `main`, folder: `/docs`. Save.
4. GitHub will publish the page at `https://taildraggers.github.io/piper-j3-cub-for-sale/`
   (may take a minute or two the first time).

## Embedding on taildraggers.com

```html
<iframe
  src="https://taildraggers.github.io/piper-j3-cub-for-sale/"
  title="Other Piper J-3 Cub Ads on the Web"
  style="width: 100%; height: 800px; border: 0;"
  loading="lazy">
</iframe>
```

## Running locally

```bash
pip install -r requirements.txt
python main.py
```

This writes/overwrites `docs/index.html`.

## Notes

- If Barnstormers changes its markup or is briefly unreachable, the run logs will
  show a `[warn]`/`[error]` line pointing at what broke rather than failing silently.
- The scraper identifies itself with a browser-like `User-Agent` and adds a short
  delay between requests to be polite to the site.
