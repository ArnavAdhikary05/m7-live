# M7 Live

A single-file, dark-themed stock monitoring dashboard for the **Magnificent Seven** — AAPL, MSFT, GOOGL, AMZN, NVDA, META and TSLA. It pulls live data from **Finnhub** over REST and renders everything client-side, with no build step, no bundler and no dependencies to install.

Created by **Arnab Adhikary** — [github.com/ArnavAdhikary05](https://github.com/ArnavAdhikary05)

---

## Features

| Feature | Detail |
| --- | --- |
| **Company identity** | Name, ticker symbol, exchange and logo per card |
| **Live price + trend arrow** | Green ▲ when the price is above today's open, red ▼ when below, yellow double-headed arrow when flat |
| **Trend-matched glow** | Each card casts a coloured glow — green, red or yellow — that matches its price arrow |
| **Opening price** | Today's open, previous close and intraday day range |
| **News feed** | The 5 most recent headlines per stock, with thumbnails, source, relative timestamp and outbound links |
| **Analyst consensus** | Buy / Hold / Sell breakdown from the latest Finnhub reporting period, stacked vertically as animated bars |
| **Auto-refresh** | Quotes re-fetch every 60 seconds; polling pauses while the browser tab is hidden |
| **Responsive** | 3–4 columns on desktop, 2 on tablet, 1 on phone — verified down to 360px wide |
| **Accessible** | Honours `prefers-reduced-motion` and uses semantic landmark elements |

---

## Requirements

- A **Finnhub API key** — free tier is fine, sign up at <https://finnhub.io/register>
- **Python 3** (or any static file server) to serve the page locally
- A modern browser

> **Why a server instead of opening the file directly?**
> The app reads its API key from `.env` at runtime. A page opened via `file://` cannot fetch that file — the browser blocks it. Serving over `http://localhost` takes one command and avoids the problem entirely.

---

## Setup

1. **Clone or download** this folder.

2. **Create the `.env` file** in the project root and add your key:

   ```dotenv
   FINNHUB_API_KEY=your_finnhub_api_key_here
   ```

   The key is never written into `index.html`, so it stays out of version control. `.env` is already listed in `.gitignore` for that reason.

3. **Serve the folder**, bound to loopback only:

   ```bash
   python -m http.server 5500 --bind 127.0.0.1
   ```

4. **Open** <http://localhost:5500/index.html>

The dashboard loads immediately: cards render as skeletons, then fill in as data arrives (roughly 1–3 seconds for all seven stocks).

---

## Configuration

All tunables live in the `Configuration` block near the top of the script in `index.html`:

| Constant | Default | Purpose |
| --- | --- | --- |
| `API_BASE` | `https://finnhub.io/api/v1` | Finnhub REST base URL |
| `CONCURRENCY` | `4` | Maximum in-flight requests during the initial load |
| `AUTO_REFRESH_MS` | `60000` | Quote refresh interval in milliseconds |
| `NEWS_DAYS` | `30` | How far back to look for headlines |
| `NEWS_COUNT` | `5` | Headlines rendered per stock |
| `STOCKS` | The Magnificent 7 | Swap or extend the watchlist by editing this array |

To track different tickers, edit the `STOCKS` array — each entry needs a `symbol`, `name` and `exchange`.

### Supplying the key another way

`resolveApiKey()` checks `window.__ENV` **before** fetching `.env`. If you would rather inject the key without a `.env` file, define it before the main script runs:

```html
<script>window.__ENV = { FINNHUB_API_KEY: 'your_key_here' };</script>
```

---

## Project structure

```
stock-screening/
├── index.html     # The entire application — markup, styles and logic
├── .env           # FINNHUB_API_KEY (git-ignored, never committed)
├── .env.example   # Template showing the required variable names
├── .gitignore     # Excludes .env from version control
└── README.md
```

---

## How it works

Four Finnhub REST endpoints are called per stock, 28 requests in total on first load, dispatched through a small concurrency-limited queue with exponential backoff on `HTTP 429`:

| Endpoint | Used for |
| --- | --- |
| `/quote` | Current price, open, previous close, day high/low |
| `/stock/profile2` | Company name, exchange, logo |
| `/stock/recommendation` | Analyst buy / hold / sell counts by period |
| `/company-news` | Recent headlines within the look-back window |

**Trend logic.** The arrow compares the current price (`c`) against today's open (`o`), not the previous close — so the arrow always agrees with the "day's move" it reports. If a flat value comes back for the open, the app falls back to the previous close as the baseline. Cards carry a matching `trend-up`, `trend-down` or `trend-flat` class that drives both the arrow colour and the card's border and box-shadow glow.

**Analyst consensus.** The latest reporting period is selected, then `strongBuy + buy` is grouped as **Buy** and `sell + strongSell` as **Sell**, with **Hold** unchanged. Percentages are computed from the total analyst count for that period.

**Refresh strategy.** Only `/quote` is re-polled on the 60-second timer — 7 calls per cycle, comfortably inside the free tier's 60-calls-per-minute ceiling. Profile, news and consensus data load once per page visit.

---

## Security notes

Because the browser reads `.env` over HTTP, that file is served to any client that can reach the server — which is why the setup command binds to `127.0.0.1`. Do not serve this folder on a public interface or host it on the open web while `.env` sits next to `index.html`.

`.env` is listed in `.gitignore` and should never be committed. This arrangement keeps the key out of source control, but it is not a secret-hiding mechanism against anyone with access to your machine or the served directory.

---

## Free tier limitations

- US equities quotes are **delayed roughly 15 minutes** on the free plan; real-time data requires a paid Finnhub plan.
- News thumbnails are served from third-party publisher CDNs and may occasionally fail to load — the dashboard degrades gracefully by hiding the thumbnail and letting the headline span the full row.
- Intraday behaviour outside market hours reflects the most recent session.

---

## Troubleshooting

| Symptom | Cause and fix |
| --- | --- |
| Red banner: *"No Finnhub API key found"* | `.env` is missing or has no `FINNHUB_API_KEY` line. Create it and reload. |
| Banner appears even though `.env` exists | The page was opened via `file://`. Serve it over `http://localhost` instead. |
| *"Finnhub unreachable"* status | Network issue, an inactive API key, or the free-tier rate limit was hit. Wait a moment and press **Refresh**. |
| Some cards show `—` for price | That symbol failed to return a quote. Press **Refresh** to retry just the quotes. |
| Logos show as letter tiles | The logo URL from Finnhub did not load; the ticker letter is used as a fallback. |

---

## Disclaimer

This project is for educational and informational purposes only. It is not investment advice. Market data is provided by Finnhub and may be delayed or inaccurate.

---

## Author

**Arnab Adhikary**

- GitHub: [github.com/ArnavAdhikary05](https://github.com/ArnavAdhikary05)

---

## License

MIT — do with it as you like.
