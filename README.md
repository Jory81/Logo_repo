# Logo_repo

Logos for the instruments in `instrument_catalogue.xlsx` that the nvstly icon set
(github.com/nvstly/icons) does not cover. Files follow the rules in `README.txt`
(`own/<EXCHANGE>/<TICKER>.png`, 250x250 RGBA PNG).

| Path | What |
|---|---|
| `own/` | the logos, ready to copy into `data\logo-library\own` |
| `own/sources.csv` | where each logo came from (`tradingview:`, `nvstly:`, `simple-icons:`, `placeholder`) |
| `reports/logo_status.md` / `.csv` | every instrument: already covered by nvstly, real logo added, placeholder, or missing |
| `tools/logo_sources.csv` | the 351 instruments needing a logo: company name, TradingView symbol, US-listing alias, simple-icons name |
| `tools/logos.py` | builds and updates `own/` and the report |

Source priority: TradingView first, then the nvstly logo of the company's US listing, then a
simple-icons glyph, then a lettered placeholder.

## Replace the placeholders with TradingView logos (on your PC)

Once, in PowerShell:

```
py -m pip install pillow playwright openpyxl
```

Then, from this repo's folder:

```
py tools\logos.py tradingview
```

It looks up each ticker's TradingView logo, downloads it, renders it with Microsoft Edge to a
250x250 PNG (round, like TradingView shows it; add `--square` for the square version), keeps the
previous file for any ticker TradingView has no logo for, and rewrites the report.

Copy the result into the app (the app picks it up at the next page load):

```
robocopy own "C:\Users\joryp\Documents\Codex\codex_market_screener\data\logo-library\own" /E /XF sources.csv
```

Other commands: `py tools\logos.py placeholders` (fill anything still empty), `py tools\logos.py report`.
New tickers: add a row to `tools/logo_sources.csv` and run `tradingview` again.
