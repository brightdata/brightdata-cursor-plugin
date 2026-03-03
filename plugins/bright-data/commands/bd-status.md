---
name: bd-status
description: Check Bright Data MCP connection status and view tool usage for this session
---

# Bright Data Status

## Step 1: Get session stats

Call the `session_stats` MCP tool:

```
session_stats()
```

## Step 2: Report to the user

Present the results clearly:

- **Connection:** MCP server connected / not connected
- **Tool usage this session:** List each tool and how many times it has been called
- **Free tier tools available:** `search_engine`, `scrape_as_markdown`, `scrape_as_html`, `scrape_batch`, `search_engine_batch`, `extract`
- **Pro tools status:** Report whether `scraping_browser_*` and `web_data_*` tools are accessible

## If session_stats fails

The MCP server is not connected. Tell the user to run `/bd-setup` to configure credentials and reconnect.