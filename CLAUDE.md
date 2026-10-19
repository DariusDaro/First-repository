# CLAUDE.md

This repo is a single static page, `index.html`: a "Stock outlook" page with a dropdown that shows one stock at a time (bottom line, key stats, bull points, risks, analyst targets, dates to watch, live headlines, sources).

There is no build step, package manager, framework or test suite. Open `index.html` in a browser to check changes. All CSS and JS are inline in that one file; keep it that way.

## Layout of `index.html`

The script is split into numbered sections, each marked with a `/* ===== */` banner:

1. **DATA**: the `STOCKS` object (one entry per ticker) and the `ORDER` array that sets dropdown order. Almost every content update happens here.
2. **HELPERS**: `money()`, `upsideCell()`, `theme()`.
3. **SECTION BUILDERS**: functions that return HTML strings for each card.
4. **LIVE WIDGETS**: TradingView embeds (quote, chart, news timeline), loaded from `s3.tradingview.com` with no account.
5. **RENDER + START-UP**: `render(sym)` and `init()`. The URL hash (e.g. `#ORCL`) selects the stock.

## Stock entry shape

Every entry in `STOCKS` uses the same fields:

- `name`, `tv` (TradingView symbol, e.g. `"NASDAQ:MSFT"`), `price` (last close, a number), `priceDate` (e.g. `"Fri 16 Oct"`)
- `bottom`: one paragraph summary
- `stats`: four `{ label, value, note }` tiles (last close, average target, next earnings, one more key figure)
- `positivesTitle`, `positives`, `risks`: arrays of plain sentences
- `targets`: `{ firm, rating, target }` per analyst firm; `target` is in USD or `null` when unavailable
- `summary`: average, highest and lowest target rows, same shape with `label` instead of `firm`
- `dates`, `sources` (`[title, url]` pairs), `caveat` (where sources disagree)

Upside percentages in the targets table are computed from `price`, so update `price` whenever targets change.

## Adding or updating a stock

- Add the entry to `STOCKS` and its ticker to `ORDER`; a ticker missing from `ORDER` will not appear in the dropdown.
- Update the snapshot line under the `<h1>` (date written and the close it uses) whenever the data is refreshed.
- Every figure should trace to a link in that stock's `sources`. When sources disagree, say so in `caveat` rather than picking silently, and mark unverified claims as such.
- Write in plain, short sentences. Dates are written like "16 Oct"; money as `$529.05`, `$41B`, `$80.05M`.

## Content rules

- The page is a research summary, not investment advice. Keep the existing footer disclaimer and keep the tone neutral: report what analysts and sources say, with both positives and risks.
- Charts, quotes and headlines are live; the written summaries are a dated snapshot. Don't describe snapshot data as current.

## Style

- Colours are CSS variables on `:root`, redefined under `prefers-color-scheme: dark`. Use the variables (`--up`, `--down`, `--muted`, etc.) rather than hard-coded colours.
- The layout must work at phone width (the `.wrap` container is max 900px with 16px side padding).
- Match the existing JS style: `const`, plain `function` declarations, string concatenation for HTML.
