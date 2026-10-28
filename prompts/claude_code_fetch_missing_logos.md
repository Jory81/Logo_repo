Add one button to the app's Settings/config tab: "Fetch missing logos". No more, no less.

## What the button does
1. Find every enabled instrument in the app that shows no logo today, from any source: own icon, assets, NVSTly or Logo.dev.
2. Download that instrument's TradingView logo.
3. Save it as `C:\Users\joryp\Documents\Codex\codex_market_screener\data\logo-library\own\<EXCHANGE>\<TICKER>.png`, following the rules in that folder's `README.txt` (it is `logo_own.README` in the code).
4. Never overwrite a file that already exists in `own\`.
5. Show one short result line, for example "12 added, 1 not found: LSE:SDR".

New files appear at the next page load; no restart.
I authorize the TradingView downloads and the writes to that `own\` folder for this button.

## Look at my local files first
My local `data\` folder holds more than GitHub does: `local`, `logo-library` (with `assets`, `catalogue\nvstly`, `own`), `supplements`, `ticker files`, `watchlists`, `instrument_catalogue.xlsx` and so on. Inspect these on disk before deciding how "no logo" is determined. Only read them; change nothing in `data\` except adding new PNGs to `logo-library\own\`.

## Code to reuse (read `AGENTS.md` first; keep the change small)
- `backend/market_screener/logo_own.py`: `own_icon_name(instrument)` gives the file name, `<venue>/<symbol with ':' as '_'>.png`.
- `backend/market_screener/logo_library.py`: `validate_png` for the PNG rules, and `LogoLibrary.resolve(ids)` (at most 100 ids per call) for which instruments have no logo.
- `api.py` builds the library (`build_logo_library`, `install_logo_routes` in `logo_routes.py`).
- `frontend/src/SettingsWorkspace.tsx` is where the button goes.

## How to fetch (this method already worked for 350 of my tickers)
Reference script: https://github.com/Jory81/Logo_repo/blob/claude/inspiring-noether-j1nkee/tools/logos.py
1. **TradingView symbol** = `EXCHANGE:SYMBOL`.
   - If the app's symbol already has a colon (`EURONEXT:ASM`, `XETR:DTE`, `NYSE:NEM`), use it as is.
   - Otherwise map the venue: NYQ->NYSE, NMS->NASDAQ, ASE->AMEX, JPX->TSE, HKG->HKEX, GER->XETR, PAR/AMS/BRU/LIS->EURONEXT, LSE->LSE, TOR->TSX, SHH->SSE, SHZ->SZSE, KSC->KRX, TAI->TWSE, EBS->SIX, MIL->MIL, MCE->BME, OSL->OSL, STO->OMXSTO, HEL->OMXHEX, CPH->OMXCOP, WSE->GPW, ASX->ASX, SES->SGX.
   - Where the app or `instrument_catalogue.xlsx` (column "TradingView") already holds the TradingView id, use that.
2. **Logo id**: POST `https://scanner.tradingview.com/global/scan` with body `{"symbols":{"tickers":[...]},"columns":["logoid","description"]}`; the answer is `data[].s` -> `data[].d[0]`.
   - Fallback per ticker: GET `https://symbol-search.tradingview.com/symbol_search/v3/?text=<SYMBOL>&exchange=<EXCHANGE>&lang=en&domain=production&search_type=undefined` with Origin and Referer `https://www.tradingview.com`. Take the hit whose symbol, with `<em>` tags removed, matches.
3. **SVG**: `https://s3-symbol-logo.tradingview.com/<logoid>--big.svg`.
4. **Render** at 250x250 RGBA, clipped to a circle.
   - Many of these SVGs have width/height but **no viewBox**. Remove the width/height, add `viewBox="0 0 w h"` from them, then set the size to 250. Otherwise the logo comes out as a sliced corner.
   - My PC has Microsoft Edge. Use the lightest way that fits the repo (for example headless Edge's `--screenshot` with a transparent background) and say what you chose.
5. **Check**: write the file only if it passes `validate_png`; otherwise report the ticker as failed.

## Finish
1. Add a few offline tests: the symbol mapping, the viewBox fix, skipping existing files, the route, and the button.
2. Press the button once for real, and tell me how many logos were added and which tickers were not found.
3. Commit on a new branch (not `main`), push, and tell me how to merge.
