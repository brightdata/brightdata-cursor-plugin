---
name: bd-setup
description: Set up the Bright Data plugin — configure API credentials and verify MCP connection
---

# Bright Data Plugin Setup

## Step 1: Check MCP server connection

Try calling the `session_stats` MCP tool to verify the server is reachable:

```
session_stats()
```

If this succeeds, the MCP server is connected. Skip to **Step 3: Verify zones**.

## Step 2: Configure credentials

If `session_stats` fails, the MCP server is not connected. Guide the user through configuration:

### Option A — Hosted server (recommended, zero Node.js install)

Add this to Cursor's MCP settings (`~/.cursor/mcp.json` or via Cursor Settings → MCP):

```json
{
  "mcpServers": {
    "Bright Data": {
      "url": "https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN_HERE"
    }
  }
}
```

Replace `YOUR_API_TOKEN_HERE` with the API token from:
**https://brightdata.com/cp/setting/users**

### Option B — Local server (full control over zones and Pro mode)

Requires Node.js 18+. Add to Cursor's MCP settings:

```json
{
  "mcpServers": {
    "Bright Data": {
      "command": "npx",
      "args": ["@brightdata/mcp"],
      "env": {
        "API_TOKEN": "YOUR_API_TOKEN_HERE",
        "WEB_UNLOCKER_ZONE": "mcp_unlocker",
        "BROWSER_ZONE": "mcp_browser"
      }
    }
  }
}
```

**To enable Pro tools** (structured data, browser automation), add:
```json
"PRO_MODE": "true"
```

Or enable specific groups only:
```json
"GROUPS": "browser,ecommerce,social"
```

After saving, reload Cursor or restart the MCP server.

## Step 3: Verify zones

Once connected, check what's available by calling `session_stats` again and reporting:

- MCP server version and connection status
- Tool usage counts for this session
- Whether Pro tools (`web_data_*`, `scraping_browser_*`) are accessible

Test core tools:
1. Call `search_engine` with a simple query to verify the SERP / Web Unlocker zone
2. If the user has Pro mode: confirm `scraping_browser_navigate` is available

## Step 4: Report status

Tell the user:
- Which tools are available (free tier vs Pro)
- Their zone names (Unlocker zone, Browser zone)
- Any missing configuration and exactly what to add

If everything works, confirm: **Bright Data MCP is connected and ready.**