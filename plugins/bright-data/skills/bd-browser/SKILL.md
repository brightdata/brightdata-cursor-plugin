---
name: bd-browser
description: "Real browser automation for JS-heavy, interactive, or login-gated pages. Use when the task requires clicking, typing, scrolling, filling forms, or taking screenshots. Supports a Puppeteer/Playwright-style flow via MCP tools. Requires Browser zone (PRO tier). Use bd-scrape for static pages that do not need interaction."
compatibility: Requires Bright Data MCP server with API_TOKEN and BROWSER_ZONE configured (PRO tier).
allowed-tools: mcp(Bright Data:scraping_browser_navigate), mcp(Bright Data:scraping_browser_snapshot), mcp(Bright Data:scraping_browser_click_ref), mcp(Bright Data:scraping_browser_type_ref), mcp(Bright Data:scraping_browser_get_text), mcp(Bright Data:scraping_browser_get_html), mcp(Bright Data:scraping_browser_screenshot), mcp(Bright Data:scraping_browser_scroll), mcp(Bright Data:scraping_browser_scroll_to_ref), mcp(Bright Data:scraping_browser_wait_for_ref), mcp(Bright Data:scraping_browser_go_back), mcp(Bright Data:scraping_browser_go_forward), mcp(Bright Data:scraping_browser_network_requests)
metadata:
  author: Bright Data
---

# Browser Automation

Automate browser interaction for: $ARGUMENTS

## Session limits

- **Idle timeout:** 5 minutes — keep interactions flowing to avoid session expiry
- **Max duration:** 30 minutes per session
- Sessions automatically terminate when idle or at max duration

## Step 1: Navigate to the page

```
scraping_browser_navigate({ url: "<target url>" })
```

## Step 2: Inspect the page structure

Take an ARIA snapshot to identify interactive elements and their refs:

```
scraping_browser_snapshot()
```

The snapshot lists all buttons, inputs, links, and other elements with their `ref` identifiers. Use these refs for all subsequent interactions.

## Step 3: Interact as needed

**Click a button or link:**
```
scraping_browser_click_ref({ ref: "<element ref>" })
```

**Type into a field (optionally submit with Enter):**
```
scraping_browser_type_ref({ ref: "<input ref>", text: "<value>", pressEnter: true })
```

**Scroll to bottom of page:**
```
scraping_browser_scroll()
```

**Scroll a specific element into view:**
```
scraping_browser_scroll_to_ref({ ref: "<element ref>" })
```

**Wait for an element to appear:**
```
scraping_browser_wait_for_ref({ ref: "<element ref>", timeout: 5000 })
```

**Navigate history:**
```
scraping_browser_go_back()
scraping_browser_go_forward()
```

## Step 4: Extract content

**Get all visible text:**
```
scraping_browser_get_text()
```

**Get page HTML (avoid full_page unless necessary — it is large):**
```
scraping_browser_get_html({ full_page: false })
```

**Take a screenshot for visual verification:**
```
scraping_browser_screenshot()
```

**Inspect network requests made by the page:**
```
scraping_browser_network_requests()
```

## Recommended flow

```
navigate → snapshot → [interact: click / type / scroll] → snapshot again → extract
```

Re-snapshot after each interaction to see updated page state before proceeding.

## If PRO tools are not available

If `scraping_browser_navigate` is unavailable, the Browser zone may not be configured or the account may be on the free tier. Tell the user:

1. Browser automation requires a Bright Data Browser zone (Pro tier)
2. Add `BROWSER_ZONE` to `mcp.json` env and ensure `PRO_MODE=true` or `GROUPS=browser`
3. Run `/bd-setup` to verify the connection