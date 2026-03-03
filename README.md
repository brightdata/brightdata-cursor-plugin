# Cursor plugin template

Build and publish Cursor Marketplace plugins from a single repo.

Three plugins are included:

- **starter-simple**: rules and skills only
- **starter-advanced**: rules, skills, agents, commands, hooks, MCP, and scripts
- **bright-data**: web search, scraping, browser automation, and structured data powered by Bright Data

## Bright Data plugin

The `plugins/bright-data/` plugin integrates the [Bright Data MCP server](https://github.com/brightdata/brightdata-mcp) into Cursor. It covers four products:

| Product | Skill | Command |
|---------|-------|---------|
| SERP API (Google, Bing, etc.) | `bd-search` | `/bd-search` |
| Web Unlocker | `bd-scrape`, `bd-batch-scrape` | `/bd-scrape` |
| Scraping Browser | `bd-browser` | `/bd-browser` |
| Web Scraper / Dataset API | `bd-structured-data` | `/bd-data` |
| Code generation (Python & Node.js) | `bd-code` | `/bd-code` |

**Skills** — what the AI uses to query live data or generate code:
- `bd-search` — live web search via SERP API
- `bd-scrape` — single URL extraction via Web Unlocker
- `bd-batch-scrape` — parallel multi-URL extraction
- `bd-browser` — interactive browser automation via Scraping Browser (CDP)
- `bd-structured-data` — structured platform data (Amazon, LinkedIn, Instagram, etc.)
- `bd-code` — generate complete Python or Node.js integration code for any Bright Data product

**Commands** — slash commands that invoke the skills:
- `/bd-setup` — configure and verify the MCP connection
- `/bd-search <query>` — search the web
- `/bd-scrape <url>` — extract a URL
- `/bd-browser <url>` — open and interact with a page
- `/bd-data <platform> <url>` — pull structured platform data
- `/bd-status` — check MCP connection status
- `/bd-code <product> <language> [use-case]` — generate integration code

### Quick setup

1. Get your API token from [brightdata.com/cp/setting/users](https://brightdata.com/cp/setting/users).
2. Add the MCP server to your Cursor `mcp.json`:

```json
{
  "mcpServers": {
    "Bright Data": {
      "command": "npx",
      "args": ["@brightdata/mcp"],
      "env": {
        "API_TOKEN": "YOUR_API_TOKEN"
      }
    }
  }
}
```

3. Run `/bd-setup` in Cursor to verify the connection.

