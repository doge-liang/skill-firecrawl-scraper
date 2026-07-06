# Firecrawl Scraper Skill

A web scraping skill that wraps the [Firecrawl](https://firecrawl.dev) API for deep content extraction, format conversion, and page interaction from within an agent context.

## Overview

This skill lets an agent scrape web pages, extract structured data, capture screenshots, parse PDFs, and crawl entire websites through the Firecrawl v2 API. It is built around a small Node.js CLI wrapper (`firecrawl-api.cjs`) that the agent invokes with a command and a JSON payload, plus a two-phase skill design that keeps API responses out of the main conversation context.

Triggers include: `firecrawl`, `scrape`, `extract content`, `screenshot`, `parse pdf`, `crawl website`, 抓取网页, 提取内容, 网页截图.

## Features

- **Scrape** a single page to `markdown`, `html`, `rawHtml`, `links`, `images`, `summary`, structured `json`, or `screenshot`.
- **Structured extraction** via a JSON format with a custom prompt and JSON schema.
- **Page interaction / browser automation** through actions: `wait`, `click`, `write`, `press`, `scroll`, `screenshot`, `scrape`, `executeJavascript`.
- **PDF parsing** using the `pdf` parser.
- **Crawl** an entire site with path filters (`includePaths` / `excludePaths`), depth control (`maxDiscoveryDepth`), and page limits, with optional subdomain/external-link following.
- **Map** a site to quickly retrieve a list of its URLs.
- **Batch scrape** multiple URLs in one call.
- **Crawl status** polling by job ID, with an optional `--wait` flag that blocks until the job reaches a terminal state.
- **Content control** options such as `onlyMainContent`, `includeTags`, `excludeTags`, `waitFor`, `timeout`, and `maxAge` caching.

## Architecture

The skill uses a two-phase design:

1. **Main skill (`SKILL.md`)** — interprets the user's intent, selects the right Firecrawl endpoint, and assembles the JSON payload.
2. **Fetcher sub-skill (`firecrawl-fetcher.md`)** — runs in a forked context and is responsible only for executing the HTTP call, so large API responses do not consume the main conversation's token budget.

Both phases call the same helper script, `firecrawl-api.cjs`.

## Installation

1. Place this skill in your agent's skills directory, for example `.claude/skills/firecrawl-scraper/`.
2. Ensure Node.js is available (the wrapper uses only Node's built-in modules — no external dependencies).
3. Configure your Firecrawl API key using one of the following (priority: environment variable over `.env`):
   - Environment variable: `FIRECRAWL_API_KEY`
   - A `.env` file next to `firecrawl-api.cjs` containing `FIRECRAWL_API_KEY=...`

## Usage

The wrapper is a CLI. It accepts a command plus a JSON payload passed as an argument, via `--data` / `--file`, or piped through stdin.

```
node firecrawl-api.cjs <scrape|crawl|map|batch-scrape|crawl-status> [--wait]
```

Scrape a single page:

```bash
cat <<'JSON' | node firecrawl-api.cjs scrape
{
  "url": "https://example.com",
  "formats": ["markdown", "links"],
  "onlyMainContent": true,
  "excludeTags": ["nav", "footer"]
}
JSON
```

Crawl a site and wait for completion:

```bash
cat <<'JSON' | node firecrawl-api.cjs crawl --wait
{
  "url": "https://docs.example.com",
  "formats": ["markdown"],
  "includePaths": ["^/docs/.*"],
  "limit": 100
}
JSON
```

Check the status of a crawl job:

```bash
node firecrawl-api.cjs crawl-status <crawl-id> [--wait]
```

Responses are returned as pretty-printed JSON. Scrape-style calls return a `success` flag and a `data` object; `crawl` returns a job ID that you pass to `crawl-status` (or resolve inline with `--wait`).

## Documentation

See [`SKILL.md`](./SKILL.md) for the full endpoint reference, all payload examples (scrape, actions, PDF, structured JSON, crawl, map, batch scrape, crawl status), the complete list of formats and options, and API-key configuration details. The internal fetcher is documented in [`firecrawl-fetcher.md`](./firecrawl-fetcher.md).

## License

Released under the [MIT License](./LICENSE).
