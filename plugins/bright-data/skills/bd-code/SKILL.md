---
name: bd-code
description: "Use when the user wants to WRITE CODE that integrates Bright Data APIs — Python or Node.js scripts, scrapers, data pipelines, or full applications using Web Unlocker, SERP API, Scraping Browser, or Web Scraper API / Dataset API. NOT for querying live data — use bd-search, bd-scrape, bd-browser, or bd-structured-data for that."
compatibility: No runtime dependencies required — generates code for the user to run.
allowed-tools: mcp(Bright Data:session_stats)
metadata:
  author: Bright Data
---

# Bright Data Code Generation

Generate integration code for: $ARGUMENTS

## When this skill fires

Trigger this skill when the user says things like:
- "write a scraper", "build a script", "integrate Bright Data", "code example"
- "how do I use [product] in Python / Node.js"
- "help me build", "create a project", "scaffold"
- References to `requests`, `playwright`, `puppeteer`, `selenium`, `brightdata-sdk`

Do NOT use this skill when the user simply asks to search or scrape live data — use `bd-search`, `bd-scrape`, `bd-browser`, or `bd-structured-data` instead.

---

## Step 1: Identify what the user wants to build

```
What does the user want to build?
│
├── Fetch a URL / bypass bot protection → Section 1 (Web Unlocker)
├── Search engine results at scale     → Section 2 (SERP API)
├── Interact with a page (click/type)  → Section 3 (Scraping Browser)
├── Pull structured platform data      → Section 4 (Web Scraper API / Dataset API)
└── Full project from scratch          → Section 8 (Scaffold) + relevant sections above
```

Read the user's request, select the matching section(s), and emit complete, working code.

---

## Section 1 — Web Unlocker API

### 1A. Python — `requests` (simplest)

```python
import requests

API_TOKEN = "YOUR_API_TOKEN"   # from brightdata.com/cp/setting/users
ZONE_NAME = "web_unlocker1"    # your Web Unlocker zone name

def unlock(url: str, country: str = None, data_format: str = "markdown") -> str:
    payload = {
        "zone": ZONE_NAME,
        "url": url,
        "format": "raw",
        "data_format": data_format,   # "html" | "markdown" | "screenshot"
    }
    if country:
        payload["country"] = country  # ISO 3166-1 e.g. "us", "gb", "de"

    response = requests.post(
        "https://api.brightdata.com/request",
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {API_TOKEN}",
        },
        json=payload,
        timeout=30,
    )
    response.raise_for_status()
    return response.json()["body"]

if __name__ == "__main__":
    content = unlock("https://example.com", country="us")
    print(content)
```

### 1B. Python — Official SDK (`brightdata-sdk`)

```bash
pip install brightdata-sdk
```

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        result = await client.scrape_url("https://example.com")
        print(result.data)

asyncio.run(main())
```

**Batch (concurrent):**
```python
import asyncio
from brightdata import BrightDataClient

async def main():
    urls = [
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3",
    ]
    async with BrightDataClient() as client:
        tasks = [client.scrape_url(url) for url in urls]
        results = await asyncio.gather(*tasks)
        for r in results:
            print(r.data)

asyncio.run(main())
```

**Async mode (non-blocking, ~2 min wait):**
```python
async with BrightDataClient() as client:
    result = await client.scrape_url(
        url="https://example.com",
        mode="async",
        poll_interval=5,
        poll_timeout=180,
    )
    print(result.data)
```

### 1C. Node.js — native `fetch`

```javascript
const API_TOKEN = process.env.BRIGHT_DATA_API_TOKEN;
const ZONE_NAME = "web_unlocker1";

async function unlock(url, { country, dataFormat = "markdown" } = {}) {
  const payload = { zone: ZONE_NAME, url, format: "raw", data_format: dataFormat };
  if (country) payload.country = country;

  const res = await fetch("https://api.brightdata.com/request", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${API_TOKEN}`,
    },
    body: JSON.stringify(payload),
  });

  if (!res.ok) {
    const text = await res.text();
    throw new Error(`Bright Data error ${res.status}: ${text}`);
  }

  const data = await res.json();
  return data.body;
}

unlock("https://example.com", { country: "us" })
  .then(console.log)
  .catch(console.error);
```

### 1D. Via Proxy (alternative — no REST API call needed)

```python
import requests
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

ACCOUNT_ID  = "YOUR_ACCOUNT_ID"   # brd-customer-XXXXXXX
ZONE_NAME   = "web_unlocker1"
ZONE_PASS   = "YOUR_ZONE_PASSWORD"

proxy_url = f"http://brd-customer-{ACCOUNT_ID}-zone-{ZONE_NAME}:{ZONE_PASS}@brd.superproxy.io:33335"
proxies = {"http": proxy_url, "https": proxy_url}

response = requests.get("https://example.com", proxies=proxies, verify=False)
print(response.text)
```

### 1E. Response schema

```json
{
  "status_code": 200,
  "headers": { "content-type": "text/html; charset=utf-8", "..." : "..." },
  "body": "<html>...</html>"
}
```

**Common error codes:**
| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad request (malformed payload, missing zone/url) |
| 401 | Authentication failed — check API token |
| 403 | Zone not authorized |
| 429 | Rate limit exceeded |
| 500 | Target site unreachable after retries |

---

## Section 2 — SERP API

### 2A. Python — Direct API

```python
import requests

API_TOKEN = "YOUR_API_TOKEN"
ZONE_NAME = "serp_zone1"   # your SERP API zone name

def search_google(query: str, country: str = "us", lang: str = "en",
                  num: int = 10, parsed: bool = True) -> dict:
    """
    Returns parsed JSON results when parsed=True,
    or raw HTML string when parsed=False.
    """
    params = f"q={requests.utils.quote(query)}&gl={country}&hl={lang}&num={num}"
    if parsed:
        params += "&brd_json=1"

    response = requests.post(
        "https://api.brightdata.com/request",
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {API_TOKEN}",
        },
        json={
            "zone": ZONE_NAME,
            "url": f"https://www.google.com/search?{params}",
            "format": "raw",
        },
        timeout=30,
    )
    response.raise_for_status()
    return response.json()

if __name__ == "__main__":
    results = search_google("bright data mcp", country="us")
    for r in results.get("organic", []):
        print(r["position"], r["title"], r["url"])
```

### 2B. Python — Official SDK

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        result = await client.search.google(
            query="bright data web scraping",
            num_results=10,
        )
        for item in result.data:
            print(item["title"], item["link"])

asyncio.run(main())
```

### 2C. Node.js — Direct API

```javascript
const API_TOKEN = process.env.BRIGHT_DATA_API_TOKEN;
const ZONE_NAME  = "serp_zone1";

async function searchGoogle(query, { country = "us", lang = "en", num = 10, parsed = true } = {}) {
  const params = new URLSearchParams({ q: query, gl: country, hl: lang, num });
  if (parsed) params.set("brd_json", "1");

  const res = await fetch("https://api.brightdata.com/request", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${API_TOKEN}`,
    },
    body: JSON.stringify({
      zone: ZONE_NAME,
      url: `https://www.google.com/search?${params}`,
      format: "raw",
    }),
  });
  if (!res.ok) throw new Error(`SERP API error ${res.status}: ${await res.text()}`);
  return res.json();
}

searchGoogle("mcp servers 2025", { country: "us" })
  .then((data) => (data.organic || []).forEach((r) => console.log(r.position, r.title, r.url)))
  .catch(console.error);
```

### 2D. Via Proxy (alternative)

```python
import requests, urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxy_url = "http://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:33335"
proxies = {"http": proxy_url, "https": proxy_url}

response = requests.get(
    "https://www.google.com/search?q=bright+data&brd_json=1",
    proxies=proxies, verify=False,
)
print(response.json())
```

### 2E. All supported search engines + URL formats

| Engine | URL Format |
|--------|-----------|
| Google | `https://www.google.com/search?q={query}&brd_json=1` |
| Bing | `https://www.bing.com/search?q={query}` |
| Yandex | `https://yandex.com/search/?text={query}` |
| DuckDuckGo | `https://duckduckgo.com/?q={query}` |
| Yahoo | `https://search.yahoo.com/search?p={query}` |
| Baidu | `https://www.baidu.com/s?wd={query}` |
| Naver | `https://search.naver.com/search.naver?query={query}` |

### 2F. All `brd_*` URL parameters

| Parameter | Values | Description |
|-----------|--------|-------------|
| `brd_json=1` | `1`, `html` | `1` → parsed JSON; `html` → JSON + raw HTML nested |
| `gl` | ISO country code | Geo-target results (e.g. `gl=us`, `gl=de`) |
| `hl` | Language code | UI language (e.g. `hl=en`, `hl=ja`) |
| `num` | integer | Results per page (default 10, max 100) |
| `start` | integer | Pagination offset (0, 10, 20 …) |
| `tbm` | `shop`, `nws`, `isch`, `vid` | Search type (Shopping, News, Images, Video) |
| `brd_mobile` | `1`, `ios`, `android` | Device emulation |
| `brd_browser` | `chrome`, `safari`, `firefox` | Browser fingerprint |

### 2G. Parsed JSON response schema

```json
{
  "engine": "google",
  "query": "bright data mcp",
  "organic": [
    {
      "position": 1,
      "title": "Bright Data MCP Server",
      "url": "https://github.com/brightdata/brightdata-mcp",
      "description": "..."
    }
  ],
  "ads": [],
  "shopping": [],
  "related_searches": []
}
```

---

## Section 3 — Scraping Browser

### 3A. Prerequisites

```bash
# Puppeteer (Node.js)
npm install puppeteer-core

# Playwright (Node.js)
npm install playwright-core

# Playwright (Python)
pip install playwright
playwright install chromium

# Selenium (Node.js)
npm install selenium-webdriver

# Selenium (Python)
pip install selenium
```

### 3B. Credential format

```
brd-customer-<ACCOUNT_ID>-zone-<ZONE_NAME>:<ZONE_PASSWORD>
```

Get credentials from: `brightdata.com/cp/zones` → select your Browser zone → Overview tab.

**Country targeting** — append `-country-XX` to the username:
```
brd-customer-C1234567-zone-my_browser-country-us:mypassword
```

### 3C. Puppeteer (Node.js) — canonical example

```javascript
const puppeteer = require("puppeteer-core");

const SBR_WS = "wss://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9222";

async function main() {
  const browser = await puppeteer.connect({ browserWSEndpoint: SBR_WS });
  try {
    const page = await browser.newPage();
    await page.goto("https://example.com", { waitUntil: "domcontentloaded" });
    const html = await page.content();
    console.log(html);
  } finally {
    await browser.close();
  }
}

main().catch((err) => console.error(err.stack || err));
```

### 3D. Playwright — Node.js

```javascript
const { chromium } = require("playwright-core");

const SBR_WS = "wss://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9222";

async function main() {
  const browser = await chromium.connectOverCDP(SBR_WS);
  try {
    const context = browser.contexts()[0];
    const page = context.pages()[0];
    await page.goto("https://example.com", { waitUntil: "domcontentloaded" });
    console.log(await page.content());
  } finally {
    await browser.close();
  }
}

main().catch(console.error);
```

### 3E. Playwright — Python

```python
import asyncio
from playwright.async_api import async_playwright

SBR_WS = "wss://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9222"

async def main():
    async with async_playwright() as pw:
        browser = await pw.chromium.connect_over_cdp(SBR_WS)
        try:
            page = await browser.new_page()
            await page.goto("https://example.com", wait_until="domcontentloaded")
            print(await page.content())
        finally:
            await browser.close()

asyncio.run(main())
```

### 3F. Selenium — Node.js

```javascript
const { Builder, Browser } = require("selenium-webdriver");

const SBR_WD = "https://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9515";

async function main() {
  const driver = await new Builder()
    .forBrowser(Browser.CHROME)
    .usingServer(SBR_WD)
    .build();
  try {
    await driver.get("https://example.com");
    console.log(await driver.getPageSource());
  } finally {
    await driver.quit();
  }
}

main().catch(console.error);
```

### 3G. Selenium — Python

```python
from selenium import webdriver
from selenium.webdriver.chromium.remote_connection import ChromiumRemoteConnection

SBR_WD = "https://brd-customer-ACCOUNT_ID-zone-ZONE_NAME:ZONE_PASS@brd.superproxy.io:9515"

def main():
    conn = ChromiumRemoteConnection(SBR_WD, "goog", "chrome")
    driver = webdriver.Remote(conn, options=webdriver.ChromeOptions())
    try:
        driver.get("https://example.com")
        print(driver.page_source)
    finally:
        driver.quit()

main()
```

### 3H. Session limits & best practices

| Limit | Value |
|-------|-------|
| Idle timeout | 5 minutes |
| Max session duration | 30 minutes |
| SSL verification | Not required for CDP WSS connections |

**Best practices:**
- Always use `try/finally` to close the browser — avoids idle timeout charges
- Use `waitUntil: "domcontentloaded"` instead of `"networkidle"` — faster and sufficient for most pages
- For Playwright, `browser.contexts()[0]` accesses the existing context (don't create a new one)
- Country targeting is done in the username string, not as a proxy header

---

## Section 4 — Web Scraper API / Dataset API

### 4A. Official Python SDK (recommended for platform data)

```bash
pip install brightdata-sdk
```

```python
import asyncio
from brightdata import BrightDataClient

async def main():
    async with BrightDataClient() as client:
        # Amazon product
        result = await client.scrape.amazon.products(
            url="https://www.amazon.com/dp/B0CRMZHDG8"
        )
        print(result.data)

        # Amazon reviews
        result = await client.scrape.amazon.reviews(
            url="https://www.amazon.com/dp/B0CRMZHDG8"
        )

        # LinkedIn
        result = await client.scrape.linkedin.profiles(
            url="https://www.linkedin.com/in/username"
        )

        # Instagram
        result = await client.scrape.instagram.profiles(
            url="https://www.instagram.com/username"
        )

asyncio.run(main())
```

**Platform method map:**

| Platform | SDK call |
|----------|----------|
| Amazon products | `client.scrape.amazon.products(url=...)` |
| Amazon reviews | `client.scrape.amazon.reviews(url=...)` |
| Amazon sellers | `client.scrape.amazon.sellers(url=...)` |
| LinkedIn profiles | `client.scrape.linkedin.profiles(url=...)` |
| LinkedIn companies | `client.scrape.linkedin.companies(url=...)` |
| LinkedIn jobs | `client.scrape.linkedin.jobs(url=...)` |
| LinkedIn posts | `client.scrape.linkedin.posts(url=...)` |
| Instagram profiles | `client.scrape.instagram.profiles(url=...)` |
| Instagram posts | `client.scrape.instagram.posts(url=...)` |
| Instagram reels | `client.scrape.instagram.reels(url=...)` |
| Facebook posts | `client.scrape.facebook.posts(url=...)` |

### 4B. Raw Dataset API — async polling pattern

Use this when the Python SDK doesn't cover your platform or you need Node.js.

**Endpoints:**
```
POST https://api.brightdata.com/datasets/v3/trigger?dataset_id=<ID>&include_errors=true
GET  https://api.brightdata.com/datasets/v3/progress/<snapshot_id>
GET  https://api.brightdata.com/datasets/v3/snapshot/<snapshot_id>?format=json
```

**Python:**
```python
import requests, time

API_TOKEN   = "YOUR_API_TOKEN"
DATASET_ID  = "gd_YOUR_DATASET_ID"   # get from brightdata.com/cp/datasets

HEADERS = {
    "Authorization": f"Bearer {API_TOKEN}",
    "Content-Type": "application/json",
}

def trigger(urls: list[str]) -> str:
    body = [{"url": u} for u in urls]
    res = requests.post(
        f"https://api.brightdata.com/datasets/v3/trigger?dataset_id={DATASET_ID}&include_errors=true",
        headers=HEADERS, json=body, timeout=30,
    )
    res.raise_for_status()
    return res.json()["snapshot_id"]

def poll(snapshot_id: str, timeout: int = 600, interval: int = 5) -> None:
    url = f"https://api.brightdata.com/datasets/v3/progress/{snapshot_id}"
    deadline = time.time() + timeout
    while time.time() < deadline:
        res = requests.get(url, headers=HEADERS, timeout=10)
        res.raise_for_status()
        status = res.json().get("status")
        if status == "ready":
            return
        if status == "failed":
            raise RuntimeError(f"Collection failed: {snapshot_id}")
        time.sleep(interval)
    raise TimeoutError(f"Timed out after {timeout}s")

def download(snapshot_id: str) -> list:
    res = requests.get(
        f"https://api.brightdata.com/datasets/v3/snapshot/{snapshot_id}?format=json",
        headers=HEADERS, timeout=60,
    )
    res.raise_for_status()
    return res.json()

def collect(urls: list[str]) -> list:
    snap = trigger(urls)
    print(f"Triggered snapshot: {snap}")
    poll(snap)
    results = download(snap)
    print(f"Got {len(results)} records")
    return results

if __name__ == "__main__":
    data = collect(["https://www.amazon.com/dp/B0XXXXXX"])
    print(data[0])
```

**Node.js:**
```javascript
const API_TOKEN  = process.env.BRIGHT_DATA_API_TOKEN;
const DATASET_ID = "gd_YOUR_DATASET_ID";

const HEADERS = {
  Authorization: `Bearer ${API_TOKEN}`,
  "Content-Type": "application/json",
};

async function trigger(urls) {
  const res = await fetch(
    `https://api.brightdata.com/datasets/v3/trigger?dataset_id=${DATASET_ID}&include_errors=true`,
    { method: "POST", headers: HEADERS, body: JSON.stringify(urls.map((url) => ({ url }))) }
  );
  if (!res.ok) throw new Error(`Trigger failed: ${res.status}`);
  return (await res.json()).snapshot_id;
}

async function poll(snapshotId, timeoutMs = 600_000, intervalMs = 5_000) {
  const deadline = Date.now() + timeoutMs;
  while (Date.now() < deadline) {
    const res = await fetch(
      `https://api.brightdata.com/datasets/v3/progress/${snapshotId}`,
      { headers: HEADERS }
    );
    if (!res.ok) throw new Error(`Poll failed: ${res.status}`);
    const { status } = await res.json();
    if (status === "ready") return;
    if (status === "failed") throw new Error(`Collection failed: ${snapshotId}`);
    await new Promise((r) => setTimeout(r, intervalMs));
  }
  throw new Error(`Timed out waiting for ${snapshotId}`);
}

async function download(snapshotId) {
  const res = await fetch(
    `https://api.brightdata.com/datasets/v3/snapshot/${snapshotId}?format=json`,
    { headers: HEADERS }
  );
  if (!res.ok) throw new Error(`Download failed: ${res.status}`);
  return res.json();
}

async function collect(urls) {
  const snap = await trigger(urls);
  console.log(`Triggered snapshot: ${snap}`);
  await poll(snap);
  const results = await download(snap);
  console.log(`Got ${results.length} records`);
  return results;
}

collect(["https://www.amazon.com/dp/B0XXXXXX"])
  .then((data) => console.log(data[0]))
  .catch(console.error);
```

### 4C. SDK Datasets API (pre-built datasets, not URL-based)

```python
import asyncio
from brightdata import BrightDataClient
from brightdata.datasets import export

async def main():
    async with BrightDataClient() as client:
        # Sample 5 records from a pre-built dataset
        snapshot_id = await client.datasets.imdb_movies.sample(records_limit=5)
        data = await client.datasets.imdb_movies.download(snapshot_id)
        export(data, "results.json")

asyncio.run(main())
```

---

## Section 5 — Authentication Reference

### When to use which credential type

| Credential type | Format | Used for |
|-----------------|--------|----------|
| API Token (Bearer) | `Bearer eyJ...` | REST API (`/request`, Dataset API, MCP `API_TOKEN` env var) |
| Zone credentials | `brd-customer-ID-zone-NAME:PASS` | Direct proxy connections (Puppeteer, Playwright, Selenium, proxy routing) |

### Environment variable best practice

**Python `.env` / `os.environ`:**
```python
import os
from dotenv import load_dotenv  # pip install python-dotenv

load_dotenv()

API_TOKEN   = os.getenv("BRIGHT_DATA_API_TOKEN")
ZONE_NAME   = os.getenv("BRIGHT_DATA_UNLOCKER_ZONE", "web_unlocker1")
ACCOUNT_ID  = os.getenv("BRIGHT_DATA_ACCOUNT_ID")
ZONE_PASS   = os.getenv("BRIGHT_DATA_ZONE_PASS")
```

**Node.js:**
```javascript
import "dotenv/config";  // npm install dotenv

const API_TOKEN  = process.env.BRIGHT_DATA_API_TOKEN;
const ZONE_NAME  = process.env.BRIGHT_DATA_UNLOCKER_ZONE ?? "web_unlocker1";
const ACCOUNT_ID = process.env.BRIGHT_DATA_ACCOUNT_ID;
const ZONE_PASS  = process.env.BRIGHT_DATA_ZONE_PASS;
```

**`.env` file:**
```
BRIGHT_DATA_API_TOKEN=your_api_token_here
BRIGHT_DATA_UNLOCKER_ZONE=web_unlocker1
BRIGHT_DATA_SERP_ZONE=serp_zone1
BRIGHT_DATA_BROWSER_ZONE=mcp_browser
BRIGHT_DATA_ACCOUNT_ID=brd-customer-XXXXXXX
BRIGHT_DATA_ZONE_PASS=your_zone_password
```

---

## Section 6 — MCP Integration in Code

For developers building AI agents or tools that themselves use Bright Data via MCP:

### Cursor/Claude Desktop config (`mcp.json`)

**Hosted (simplest):**
```json
{
  "mcpServers": {
    "Bright Data": {
      "url": "https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN"
    }
  }
}
```

**Local with all options:**
```json
{
  "mcpServers": {
    "Bright Data": {
      "command": "npx",
      "args": ["@brightdata/mcp"],
      "env": {
        "API_TOKEN": "YOUR_API_TOKEN",
        "WEB_UNLOCKER_ZONE": "mcp_unlocker",
        "BROWSER_ZONE": "mcp_browser",
        "PRO_MODE": "true",
        "POLLING_TIMEOUT": "600",
        "RATE_LIMIT": "100/1h"
      }
    }
  }
}
```

**Selective groups (cheaper than full PRO_MODE):**
```json
"GROUPS": "browser,ecommerce,social"
```

Valid group values: `browser`, `ecommerce`, `social`, `business`, `finance`, `research`, `travel`, `app_stores`

---

## Section 7 — Common Patterns & Pitfalls

### Pitfall 1: Wrong env var name in mcp.json

The MCP server uses `API_TOKEN` (not `BRIGHT_DATA_API_TOKEN`). The longer name is a local `.env` convention.

CORRECT in mcp.json:
```json
{ "API_TOKEN": "..." }
```

WRONG in mcp.json:
```json
{ "BRIGHT_DATA_API_TOKEN": "..." }
```

### Pitfall 2: Using `requests` with SERP proxy — SSL warnings

Always suppress or handle SSL warnings when using proxy mode:
```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
response = requests.get(url, proxies=proxies, verify=False)
```

### Pitfall 3: Playwright context vs new context

When connecting to Scraping Browser via CDP, the browser already has a context. Use it — don't create a new one:
```python
# CORRECT
context = browser.contexts[0]
page = context.pages[0]

# WRONG (creates extra context, wastes session time)
context = await browser.new_context()
```

### Pitfall 4: Dataset API poll timeout

Default poll in the MCP is 600s. If writing raw polling code, match this or make it configurable:
```python
POLLING_TIMEOUT = int(os.getenv("POLLING_TIMEOUT", "600"))
poll(snapshot_id, timeout=POLLING_TIMEOUT)
```

### Pitfall 5: `brd_json=1` vs `brd_json=html`

- `brd_json=1` → Parsed JSON only (organic results, ads, etc.) — use this for most cases
- `brd_json=html` → Same JSON + raw HTML nested inside — much larger, use only when you need HTML too

### Pitfall 6: Scraping Browser — session idle timeout

If your code does `page.goto()` then waits a long time before the next action, the session may expire. Always keep interactions flowing within 5 minutes. For long waits, use `page.waitForSelector()` rather than `setTimeout`.

---

## Section 8 — Minimal Starter Projects

### Project A — Simple URL unlocker (Python)

```
my-scraper/
├── .env
├── requirements.txt   # requests python-dotenv
└── scraper.py         # uses Section 1A pattern
```

### Project B — SERP monitor (Node.js)

```
serp-monitor/
├── .env
├── package.json       # { "type": "module" }
└── index.js           # uses Section 2C pattern, saves to JSON
```

### Project C — Puppeteer scraper (Node.js)

```
browser-scraper/
├── .env
├── package.json
├── node_modules/      # puppeteer-core
└── index.js           # uses Section 3C pattern
```

### Project D — Full pipeline (Python, SDK)

```
data-pipeline/
├── .env
├── requirements.txt   # brightdata-sdk python-dotenv
└── pipeline.py        # SDK pattern from Section 4A
```

---

## Output instructions

1. Identify the product (Web Unlocker / SERP / Scraping Browser / Dataset API) from the user's request
2. Identify the language (Python or Node.js) — if not specified, ask or default to Python
3. Emit the complete, working code block from the matching section above — do not truncate
4. Add the relevant `.env` entries from Section 5 below the code
5. Add any applicable pitfalls from Section 7
6. If the user asked for a full project, emit the scaffold from Section 8 and fill in the actual code

Always emit complete code — never summarize or omit lines.
