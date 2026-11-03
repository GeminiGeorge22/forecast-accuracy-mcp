# Forecast Accuracy & Track Record MCP: forecasts vs actuals, prediction markets, archived reports

**Check a forecast against what actually happened, as MCP tools**: institutional forecast track records, economic releases with forecast vs actual, earnings estimates vs results, prediction-market prices and leaderboards, plus archived web pages and news for what was said at the time. One hosted endpoint, one Actor per job.

Hosted remote MCP server (Streamable HTTP) on [Apify's MCP gateway](https://mcp.apify.com). **Client config plus a small open-source stdio proxy server. No Actor source.**

**Disclosure:** entries marked "(maintained by us)" are Actors I maintain on Apify Store (publisher `publicrecords`). Every other Actor is independent, made by its own publisher and picked on merit (Store usage, ratings and recent maintenance). This repo is not affiliated with those publishers or with the sites the Actors read.

## MCP endpoint

```
https://mcp.apify.com/?tools=publicrecords/calibration-ledger,pintostudio/economic-calendar-data-investing-com,scrapemint/stock-earnings-estimates,fatihtahta/kalshi-scraper,parsebird/polymarket-scraper,ryanclinton/wayback-machine-search,data_xplorer/google-news-scraper-fast
```

Auth: sign in with Apify (OAuth) when your client prompts, or send an Apify API token as a Bearer token. Runs are billed on your own Apify account under each Actor's own Store pricing; see each Store page.

## Tools

| Actor | Store page | MCP tool | What it returns |
|---|---|---|---|
| **Calibration Ledger** (maintained by us) | [publicrecords/calibration-ledger](https://apify.com/publicrecords/calibration-ledger) | `publicrecords--calibration-ledger` | Given a claim from an organization (IMF, OECD, Fed/FOMC, SPF, ECB SPF or a listed issuer), returns its forecast track record: error distribution, bias, band coverage and worst misses with source links, or an honest 'no coverage'. |
| **Economic Calendar (Investing.com)** | [pintostudio/economic-calendar-data-investing-com](https://apify.com/pintostudio/economic-calendar-data-investing-com) | `pintostudio--economic-calendar-data-investing-com` | Economic calendar events across 45+ countries with actual, forecast and previous values and importance. |
| **Stock Earnings Estimates & Results** | [scrapemint/stock-earnings-estimates](https://apify.com/scrapemint/stock-earnings-estimates) | `scrapemint--stock-earnings-estimates` | US stock EPS estimates vs reported results: beat or miss by quarter and how often a company beats. |
| **Kalshi Scraper** | [fatihtahta/kalshi-scraper](https://apify.com/fatihtahta/kalshi-scraper) | `fatihtahta--kalshi-scraper` | Kalshi prediction markets: tickers, categories, prices and volume. |
| **Polymarket Leaderboard Scraper** | [parsebird/polymarket-scraper](https://apify.com/parsebird/polymarket-scraper) | `parsebird--polymarket-scraper` | Polymarket leaderboard rankings with profit/loss, volume, positions and recent trades, by time range and category. |
| **Wayback Machine Search** | [ryanclinton/wayback-machine-search](https://apify.com/ryanclinton/wayback-machine-search) | `ryanclinton--wayback-machine-search` | Internet Archive snapshots of any site: timestamps, URLs and content hashes, to see what a page said and when it changed. |
| **Google News Scraper** | [data_xplorer/google-news-scraper-fast](https://apify.com/data_xplorer/google-news-scraper-fast) | `data_xplorer--google-news-scraper-fast` | Google News results by query: titles, sources, dates and links, to see what was reported at the time. |

The Apify gateway also exposes helper tools to read run results (`get-actor-run`, `get-dataset-items`). Want fewer tools? Drop any Actor from the `tools=` list.

## Who it's for

- Analysts and journalists: how accurate has this institution's forecast been before?
- Macro and markets: did the release beat or miss consensus?
- Researchers: what did prediction markets price before the outcome?
- Fact-checkers: what did a page or the news say on a given date?

## Example prompts

- The IMF projects `<country>` GDP growth of `<x>%` next year. How accurate have its past projections been?
- Show this week's economic releases for the US with forecast vs actual.
- How often has `<ticker>` beaten its EPS estimate over the last 8 quarters?
- What did `<url>` look like in the Wayback Machine around `<date>`?

## Cursor / Claude Code / any remote-MCP client

```json
{
  "mcpServers": {
    "forecast-accuracy": {
      "url": "https://mcp.apify.com/?tools=publicrecords/calibration-ledger,pintostudio/economic-calendar-data-investing-com,scrapemint/stock-earnings-estimates,fatihtahta/kalshi-scraper,parsebird/polymarket-scraper,ryanclinton/wayback-machine-search,data_xplorer/google-news-scraper-fast"
    }
  }
}
```

Claude Code:

```sh
claude mcp add --transport http forecast-accuracy "https://mcp.apify.com/?tools=publicrecords/calibration-ledger,pintostudio/economic-calendar-data-investing-com,scrapemint/stock-earnings-estimates,fatihtahta/kalshi-scraper,parsebird/polymarket-scraper,ryanclinton/wayback-machine-search,data_xplorer/google-news-scraper-fast"
```

## Claude Desktop (local bridge)

```json
{
  "mcpServers": {
    "forecast-accuracy": {
      "command": "npx",
      "args": ["-y", "@apify/actors-mcp-server", "--actors", "publicrecords/calibration-ledger,pintostudio/economic-calendar-data-investing-com,scrapemint/stock-earnings-estimates,fatihtahta/kalshi-scraper,parsebird/polymarket-scraper,ryanclinton/wayback-machine-search,data_xplorer/google-news-scraper-fast"],
      "env": { "APIFY_TOKEN": "YOUR_APIFY_TOKEN" }
    }
  }
}
```

## Local stdio server (open source)

This repo also ships a small MCP server in [`src/`](src/) for clients that only run local stdio servers. It answers `initialize` and `tools/list` locally from [`tools.json`](tools.json), which is generated from each Actor's public input schema and checked for drift in CI and weekly, so any client or directory can inspect the tools with no token. `tools/call` is forwarded to the hosted Apify MCP endpoint above with **your own** `APIFY_TOKEN`; without a token a call returns a clear error instead of running anything. The token is only sent in the `Authorization` header and is never logged.

```json
{
  "mcpServers": {
    "forecast-accuracy": {
      "command": "npx",
      "args": ["-y", "github:GeminiGeorge22/forecast-accuracy-mcp"],
      "env": { "APIFY_TOKEN": "YOUR_APIFY_TOKEN" }
    }
  }
}
```

Or from a clone:

```bash
git clone https://github.com/GeminiGeorge22/forecast-accuracy-mcp.git
cd forecast-accuracy-mcp && npm install
APIFY_TOKEN=YOUR_APIFY_TOKEN node src/server.js
```

Optional env: `APIFY_ACTORS` (comma-separated subset of the Actors above) and `APIFY_MCP_URL` (full endpoint override).

Tests run against an in-memory mock server, with no network and no token: `npm test`. Requires Node.js 18+.

### Python (uvx/pip)

A matching Python version of this server lives in [`python/`](python/), built on the official `mcp` Python SDK: same tools, same `APIFY_TOKEN`, same hosted endpoint. Requires Python 3.10+.

```json
{
  "mcpServers": {
    "forecast-accuracy": {
      "command": "uvx",
      "args": ["forecast-accuracy-mcp"],
      "env": { "APIFY_TOKEN": "YOUR_APIFY_TOKEN" }
    }
  }
}
```

Or with pip: `pip install forecast-accuracy-mcp`, then run `forecast-accuracy-mcp`. Python tests: `cd python && pip install -e ".[test]" && env -u APIFY_TOKEN python -m pytest`.

## See also

- [ecommerce-search-rankings-mcp](https://github.com/GeminiGeorge22/ecommerce-search-rankings-mcp): E-commerce search results and rankings
- [ecommerce-sales-mcp](https://github.com/GeminiGeorge22/ecommerce-sales-mcp): E-commerce sales signals
- [sports-betting-stats-mcp](https://github.com/GeminiGeorge22/sports-betting-stats-mcp): Sports betting odds and stats
- [sports-odds-mcp](https://github.com/GeminiGeorge22/sports-odds-mcp): Kambi sportsbook odds
- [etsy-sales-mcp](https://github.com/GeminiGeorge22/etsy-sales-mcp): Etsy search and shop sales
- [ats-jobs-mcp](https://github.com/GeminiGeorge22/ats-jobs-mcp): ATS job postings (Greenhouse, Lever, Ashby, SmartRecruiters)
- [gov-tenders-mcp](https://github.com/GeminiGeorge22/gov-tenders-mcp): Government tenders and grants

## Registry

Official MCP Registry name: `io.github.GeminiGeorge22/forecast-accuracy-mcp`

## License

MIT, for this documentation, config and the proxy server code in `src/`. Each Actor belongs to its publisher.
