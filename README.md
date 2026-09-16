# ForexFactory Calendar Scraper

A small Python scraper that downloads the [Forex Factory](https://www.forexfactory.com/calendar) economic calendar — **full history from 2007-01-01 to today** — and publishes it as a CSV on a GitHub Release, **refreshed every day at 00:10 UTC via GitHub Actions**.

## Features

- Scrapes any year, year range (`2007-2026`), or specific month/week URL
- Extracts: `date`, `time` (GMT), `currency`, `impact`, `event`, `actual`, `forecast`, `previous`
- Detects event impact (High / Medium / Low / Holiday) from icon colors or CSS classes
- Handles the year rollover (Dec → Jan) in weekly views
- Retries failed requests (3 attempts with backoff) so a flaky network can't kill a long run
- Exports `forexfactory_calendar.csv` with `utf-8-sig` encoding so Excel opens it correctly
- Polite crawling: 2-second delay between requests to avoid IP bans

## Download the data

The CSV is **not** committed to the repo. It lives as an asset on the rolling release tagged `calendar-data`, overwritten in place every night:

```
https://github.com/<OWNER>/<REPO>/releases/download/calendar-data/forexfactory_calendar.csv
```

This URL is stable — scripts can always pull the latest full dataset from it.

Columns:

| date          | time  | currency | impact | event | actual | forecast | previous |
| ------------- | ----- | -------- | ------ | ----- | ------ | -------- | -------- |
| Thu Jan 2 2026 | 13:30 | USD     | High   | ...   | ...    | ...      | ...      |

All times are **GMT** (controlled by the `fftimezone` cookie, 24-hour format).

## Usage (local)

```bash
pip install -r requirements.txt

# Default: all 12 months of the current year
python forexfactory_scraper.py

# A specific year
python forexfactory_scraper.py 2025

# Full history 2007 → today (this is what the daily Action runs)
python forexfactory_scraper.py 2007-2026

# Specific month/week URLs
python forexfactory_scraper.py "https://www.forexfactory.com/calendar?month=jan.2025"
```

The full-history run fetches ~240 monthly pages; with the 2-second politeness delay it takes roughly 10–15 minutes.

## GitHub Actions automation

The workflow in `.github/workflows/daily-update.yml`:

1. Runs **every day at 00:10 UTC** (`cron: "10 0 * * *"`), or manually via *Actions → Daily calendar update → Run workflow*.
2. Installs Python + dependencies.
3. Runs the scraper for `2007` → current year.
4. Creates the release `calendar-data` on first run, then uploads `forexfactory_calendar.csv` with `--clobber` (in-place overwrite, stable download URL).
5. Makes a tiny `LAST_RUN.txt` commit so GitHub's 60-day inactivity rule never pauses the schedule.

Notes:

- Scheduled runs can be delayed a few minutes by GitHub under heavy load — this is normal.
- The scheduled trigger only runs from the **default branch**.
- The workflow uses the built-in `GITHUB_TOKEN`; no extra secrets are needed.

## راهنمای فارسی

این پروژه تقویم اقتصادی Forex Factory را **از ۲۰۰۷-۰۱-۰۱ تا امروز** اسکرپ می‌کند و هر روز ساعت **00:10 UTC** (۳:۴۰ بامداد به وقت ایران) به‌صورت خودکار به‌روز می‌شود.

- **خروجی کجاست؟** فایل `forexfactory_calendar.csv` به‌جای کامیت شدن، روی یک Release ثابت با تگ `calendar-data` قرار می‌گیرد و هر شب همان فایل جایگزین می‌شود؛ پس لینک دانلودش همیشه ثابت می‌ماند:
  `https://github.com/<OWNER>/<REPO>/releases/download/calendar-data/forexfactory_calendar.csv`
- **اجرای محلی:** `pip install -r requirements.txt` و سپس `python forexfactory_scraper.py 2007-2026` — بازه سال به‌صورت `YYYY-YYYY` پشتیبانی می‌شود.
- **مدت اجرای کامل تاریخچه:** حدود ۱۰ تا ۱۵ دقیقه (به‌خاطر وقفه‌ی ۲ ثانیه‌ای بین درخواست‌ها برای جلوگیری از بن شدن IP).
- **خطاهای موقتی:** هر درخواست تا ۳ بار با وقفه تلاش مجدد می‌کند.
- ساعت‌ها به GMT و فرمت ۲۴ ساعته است و فایل با انکودینگ `utf-8-sig` ذخیره می‌شود تا در اکسل درست باز شود.

## License

MIT — see [LICENSE](LICENSE).
