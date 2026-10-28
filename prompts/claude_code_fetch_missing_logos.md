Build a "Fetch missing logos" action in the Settings workspace of this app (codex_market_screener).

## Goal
One button in Settings that:
1. takes every enabled instrument that has no logo from any source today (own icon, NVSTly, Logo.dev),
2. downloads that instrument's TradingView logo,
3. writes it as a PNG to `data/logo-library/own/<EXCHANGE>/<TICKER>.png` (the F210 own-icon rules), and
4. shows the result: how many were written, plus the list of tickers that have no TradingView logo.
New files show at the next page load (OwnIcons re-reads a file when its size or time changes), so no restart is needed.
I (the owner) authorize the provider calls to TradingView for this feature and the writes to my logo library's `own/` folder.

## Read first
- `AGENTS.md`, then the logo parts of `docs/current/ui_and_tables.md` (section 3, "Logo reads" and "Own icons (F210)"), and search `docs/current/ATLAS.md` and `docs/current/DECISIONS.md` for "logo".
- Owners: `backend/market_screener/{logo_library.py,logo_provider.py,logo_catalogue.py,logo_own.py,logo_config.py,logo_routes.py}`; `api.py` builds the library (`build_logo_library`, `install_logo_routes`). Settings UI: `frontend/src/SettingsWorkspace.tsx`.
- Reuse what exists: `logo_own.own_icon_name(instrument)` for the file name, `logo_library.validate_png` for the PNG rules, and `LogoLibrary.resolve(ids)` (in batches of at most 100) to find instruments with no logo. Don't build a second identity or logo truth.

## A proven method (a standalone version already worked for 350 of my 351 missing tickers)
Reference: https://github.com/Jory81/Logo_repo/blob/claude/inspiring-noether-j1nkee/tools/logos.py (`cmd_tradingview`, `tradingview_logoids`, `tradingview_search_logoid`, `fit_svg`, `render_svg`).
1. TradingView symbol: `EXCHANGE:SYMBOL`. If the Market Data symbol already contains a colon (for example `EURONEXT:ASM`, `XETR:DTE`, `NYSE:NEM`, `TSX:CNR`, `OMXSTO:EQT`), use it as is. Otherwise map the venue: NYQ->NYSE, NMS->NASDAQ, ASE->AMEX, JPX->TSE, HKG->HKEX, GER->XETR, PAR/AMS/BRU/LIS->EURONEXT, LSE->LSE, TOR->TSX, SHH->SSE, SHZ->SZSE, KSC->KRX, TAI->TWSE, EBS->SIX, MIL->MIL, MCE->BME, OSL->OSL, STO->OMXSTO, HEL->OMXHEX, CPH->OMXCOP, WSE->GPW, ASX->ASX, SES->SGX. Check this against the TradingView ids the app already keeps (the ticker files in `data/ticker files/*.txt`, `fundamentals/tradingview_io.py`) and prefer the app's own id where there is one.
2. Logo id: POST `https://scanner.tradingview.com/global/scan` with `{"symbols":{"tickers":[...up to 100]},"columns":["logoid","description"]}`; the response is `data[].s` and `data[].d[0]`. Fallback per ticker: GET `https://symbol-search.tradingview.com/symbol_search/v3/?text=<SYMBOL>&exchange=<EXCHANGE>&lang=en&domain=production&search_type=undefined` with Origin and Referer `https://www.tradingview.com`; take the hit whose `symbol` (with `<em>` tags removed) equals the symbol. Ignore logo ids starting with `country/`.
3. SVG: GET `https://s3-symbol-logo.tradingview.com/<logoid>--big.svg`.
4. Rasterize to 250x250 RGBA. Important lesson: many TradingView SVGs have width/height but **no viewBox**. Remove the root width/height, add `viewBox="0 0 w h"` from them, then set width/height to 250. Without this the logo is drawn small in the top-left corner and comes out as a sliced "wedge". Clip it to a circle (TradingView's look). My PC has Microsoft Edge; the reference used Playwright with `channel="msedge"`. Prefer the smallest dependency choice that fits this repo (for example headless Edge's `--screenshot` with a transparent background, or an optional extra), and say which you chose and why.
5. Before writing a file, check it with `validate_png` and that it covers the 250x250 canvas. A file that fails is reported, not written. Never overwrite an existing own icon unless I tick an "overwrite" option.

## Shape
- A backend owner module (for example `logo_tradingview_fill.py`) with a bounded job: one run at a time, progress and a result list (written / not found / failed, with reason codes, no raw exception text). Plus a route to start it and a route to read its state. Run it off the request thread, like the existing provider worker.
- The Settings UI: a button, a progress line, and the result table (ticker, company, outcome). Follow the UI rules in `docs/current/ui_and_tables.md` section 4.
- Tests offline with fixtures (no TradingView calls in tests): the venue mapping, the viewBox fix, own-file naming, skip-existing and overwrite, an invalid PNG refused, the route contract, and the UI button and result.

## Finish
1. Run the tests that cover it.
2. Then, with my permission given here, run it once for real against my library.
3. Report how many logos were written, which tickers were not found, and the exact files changed.
4. Update `docs/current/ui_and_tables.md` (Own icons) and the other places `AGENTS.md` asks for.
5. Commit on a new branch (not `main`), push, and tell me how to merge.

Notes from my side: my local folders contain more than the GitHub repo (data stays local), and the app currently works, so keep the change bounded.
