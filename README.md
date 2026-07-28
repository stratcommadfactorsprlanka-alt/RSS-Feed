# Sri Lanka News/Business RSS Aggregator

Pulls headlines from ~46 news and business sites and merges them into one
combined `feed.xml`, refreshed automatically once a day via GitHub Actions.

## How it works

For each source in `sources.py`, the script:
1. **Tries to auto-discover a real RSS/Atom feed** — checks the page's
   `<link rel="alternate">` tags, then tries common paths like `/feed/`,
   `/rss/`, `/rss.xml`, etc.
2. If a real feed is found, pulls items from it directly (title, link,
   summary, published date — all accurate).
3. **If no feed exists**, falls back to lightweight scraping: it grabs
   headline-like links from `<h1>`–`<h4>` tags and `<article>` blocks on
   the homepage/category page. These items won't have real publish dates
   (the scrape time is used instead) and quality varies by site layout.
4. All items from all sources are merged, sorted by date, and written out
   as one standard RSS 2.0 file (`feed.xml`) that any reader (Feedly,
   Inoreader, NetNewsWire, etc.) can subscribe to.

## Files

- `aggregate.py` — the main script
- `sources.py` — the list of sources (edit this to add/remove sites)
- `requirements.txt` — Python dependencies
- `.github/workflows/aggregate.yml` — runs the script daily and commits
  the updated `feed.xml` back to the repo

## Setup (GitHub Actions — recommended, fully automated)

1. Create a new GitHub repo and push this folder's contents to it.
2. In the repo settings, under **Actions → General → Workflow permissions**,
   make sure "Read and write permissions" is enabled (so the workflow can
   commit the updated feed.xml).
3. That's it — it'll run daily at 03:00 UTC, or you can trigger it manually
   from the **Actions** tab ("Update aggregated RSS feed" → Run workflow).
4. Once it's run at least once, your feed will be available at:
   `https://raw.githubusercontent.com/<you>/<repo>/main/feed.xml`
   — subscribe to that URL in any RSS reader.

## Running locally

```bash
pip install -r requirements.txt
python aggregate.py --out feed.xml --per-source 8 --workers 10
```

## Tuning

- `--per-source N` — max items pulled per source (default 8)
- `--workers N` — how many sources to process in parallel (default 10)
- Edit `sources.py` to add/remove/rename sources, or hardcode a feed URL
  in `KNOWN_FEEDS` if you've already confirmed one (skips discovery,
  faster & more reliable than auto-detecting every run).

## Important notes

- **This wasn't live-tested against all 46 sites** — auto-discovery and
  the scraping fallback use general-purpose techniques, but some sites
  with unusual layouts, JS-rendered content, or anti-bot protection may
  return zero items or messy titles. Run it once, check the console
  output (it logs which sources found a real feed vs. fell back to
  scraping vs. failed), and refine `sources.py` / add entries to
  `KNOWN_FEEDS` for any that need it.
- Sites that render content via JavaScript (rather than plain HTML) won't
  work with the scraping fallback as-is — those would need a headless
  browser (e.g. Playwright), which isn't included here to keep things
  lightweight.
- Respect each site's `robots.txt` and terms of use — this pulls
  headlines/links/summaries for personal aggregation, not full article
  content.
