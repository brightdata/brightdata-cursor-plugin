---
name: bd-scrape
description: "Fetch and extract content from any URL, bypassing bot protection, CAPTCHAs, and geo-blocks using Bright Data's Web Unlocker. Returns clean Markdown by default. For multiple URLs use scrape_batch. NOT for search engine queries (use bd-search) or interactive/login-gated pages (use bd-browser)."
compatibility: Requires Bright Data MCP server with API_TOKEN and WEB_UNLOCKER_ZONE configured.
allowed-tools: mcp(Bright Data:scrape_as_markdown), mcp(Bright Data:scrape_as_html), mcp(Bright Data:scrape_batch)
metadata:
  author: Bright Data
---

# URL Content Extraction

Extract content from: $ARGUMENTS

## Step 1: Choose the right tool

**Single URL — clean readable output (default):**
```
scrape_as_markdown({ url: "<target url>" })
```

**Single URL — raw HTML needed (DOM parsing, link extraction):**
```
scrape_as_html({ url: "<target url>" })
```

**Multiple URLs (up to 10) — batch processing:**
```
scrape_batch({
  urls: ["<url1>", "<url2>", "<url3>"]
})
```

Web Unlocker handles proxy rotation, CAPTCHA solving, and JavaScript rendering automatically. You do not need to configure any of this.

## Step 2: Return the content

Present extracted content as:

**[Page Title](URL)**

Then the content, following these rules:
- Keep content faithful to the source — do not paraphrase or editorialize
- Extract ALL numbered/bulleted list items exhaustively
- Strip only obvious noise: nav menus, cookie banners, footers, ads
- Preserve all facts, names, numbers, dates, and quotes
- Cite the source URL inline for any facts you reference: `[Page Title](url)`

For batch results, present each URL's content in sequence, labeled with its URL.

## Step 3: If follow-up is needed

If the page content is insufficient, consider:
- Using `bd-search` to find additional sources
- Using `bd-browser` if the page requires login or user interaction to reveal content

## If the MCP server is not connected

If the tool is unavailable, **stop immediately**. Do NOT fetch the URL yourself or use built-in fetch tools. Tell the user:

1. The Bright Data MCP server is not connected
2. Run `/bd-setup` to configure and verify the connection
3. Then retry their request