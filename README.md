# Shifteryadak Price-Sync Set with PHP Scraper Port

The same client fix set as `csv-price-sync-client-fix` - three WooCommerce price-sync plugins and two
Python scrapers - plus the new work in this copy: a standalone PHP port of both scrapers. The port
moves CSV generation out of a Windows Playwright script and into code that can run on the WordPress
host itself, which is what the plugins downstream consume.

**Suggested repo name:** `csv-price-sync-php-port`
**Stack:** PHP 8 with cURL and DOM parsing (WordPress/WooCommerce plugin side), Python 3 with Playwright, requests, BeautifulSoup (legacy scraper side)
**Status:** experimental
**Last modified:** 2026-09-03

## What it does

- `grimoire-scrapers.php` - dependency-free function library, explicitly documented as the "PHP port of
  `price.py` + `other_sites_scraper.py`". Exposes `grscr_fetch_url` (cURL with UA, retries, exponential
  backoff, TLS verification on), `grscr_clean_price`, `grscr_rial_to_toman`, `grscr_hash_id`,
  `grscr_build_features`, `grscr_extract_woo_price`, `grscr_save_csv` (utf-8-sig), and the five site
  scrapers: `grscr_scrape_cookma` (AJAX API crawl, paged, progress callback), `grscr_scrape_mrkasket`,
  `grscr_scrape_starcycle` (static WooCommerce), `grscr_scrape_kolahkasket`, `grscr_scrape_gazzero`
  (HTML card parsers). Designed to run under wp-cron or plain CLI.
- `grscr-run.php` - CLI test runner: `php grscr-run.php [cookma|others|all] [--full]`. Default is a
  smoke sample (2 cookma pages ≈ 100 items); writes `cookma_<date>.csv` / `other_sites_<date>.csv` to
  `C:/Users/Administrator/Downloads`, the same formats the plugins read.
- The bundled plugins (`cookma-woo.php` v4.0, `check-new-product.php`, `others.php`) are byte-identical
  to the copies in `Documents\Projects\client-fix`; they are the consumer side, matched by
  `شناسه (Hash ID)` for cookma and by `لینک محصول` for the other shops.
- `final-files/Final Files/{price.py, other_sites_scraper.py}` - the May revision of the Python
  originals being ported.

## Layout

```
grimoire-scrapers.php                 the port (library)
grscr-run.php                         CLI smoke/full test runner
check-new-product/check-new-product/check-new-product.php
cookma-woo/cookma-woo/cookma-woo.php
others/others/others.php
final-files/Final Files/price.py
final-files/Final Files/other_sites_scraper.py
```

## Running it

```bash
php grscr-run.php                      # smoke: 2 cookma pages + one pass per other site
php grscr-run.php cookma --full        # full cookma crawl
COOKMA_PHPSESSID=... php grscr-run.php all --full
```

CSV column contracts are in the header comment of `grimoire-scrapers.php`. The Python path still runs
with `python "final-files/Final Files/price.py"` if a browser-driven crawl is needed.

## Notes

- Only `grimoire-scrapers.php` and `grscr-run.php` are new relative to the client-fix drop; the plugin
  and Python files are the same bytes, so this folder and `client-fix` are duplicates apart from those
  two files and the renamed `final-files` dir.
- `grscr-run.php:22` falls back to a hardcoded cookma `PHPSESSID` literal when `COOKMA_PHPSESSID` is
  unset; `final-files/Final Files/price.py:12` does the same. Both are live session credentials and
  must be removed before any publish.
- CSV output is hardcoded to the current user's `Downloads` directory - a test harness value, not a
  deployment target.
- The port is unproven against the Python originals at scale; `shiftery-plugins` holds the dated CSV
  exports and production sync logs that would serve as the comparison corpus.
