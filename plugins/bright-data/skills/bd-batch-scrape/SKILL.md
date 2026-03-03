---
name: bd-batch-scrape
description: "Process up to 10 URLs or search queries in a single parallel call. Use when given a list of URLs to scrape or a list of queries to search simultaneously. Significantly faster than running them one by one."
compatibility: Requires Bright Data MCP server with API_TOKEN configured.
allowed-tools: mcp(Bright Data:scrape_batch), mcp(Bright Data:search_engine_batch)
metadata:
  author: Bright Data
---

# Batch Scrape / Batch Search

Process in parallel: $ARGUMENTS

## Choose the right batch tool

### Batch URL scraping (up to 10 URLs)

Use when the user provides a list of URLs to fetch:

```
scrape_batch({
  urls: [
    "https://example.com/page1",
    "https://example.com/page2",
    "https://example.com/page3"
  ]
})
```

Returns an array of `{ url, content }` pairs in Markdown format. Web Unlocker handles bot protection for each URL automatically.

### Batch search (up to 10 queries)

Use when the user provides a list of search queries:

```
search_engine_batch({
  queries: [
    { query: "topic one", engine: "google" },
    { query: "topic two", engine: "google" },
    { query: "topic three", engine: "bing" }
  ]
})
```

Returns JSON (Google) or Markdown (Bing/Yandex) for each query.

## Limits

- Maximum **10 URLs** per `scrape_batch` call
- Maximum **10 queries** per `search_engine_batch` call
- If the user provides more than 10, split into multiple batch calls and process sequentially

## Presenting results

For batch URL scrapes:
- Label each result clearly with its source URL
- Apply the same content fidelity rules as `bd-scrape` — preserve facts, strip only noise
- Include a Sources section listing all URLs processed

For batch searches:
- Synthesize findings across all queries when they relate to a common topic
- Cite every fact with the originating source URL
- Include a consolidated Sources section at the end

## If the MCP server is not connected

If batch tools are unavailable, **stop immediately**. Do NOT process URLs or queries yourself. Tell the user:

1. The Bright Data MCP server is not connected
2. Run `/bd-setup` to configure and verify the connection
3. Then retry the batch request