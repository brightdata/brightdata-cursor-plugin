---
name: bd-structured-data
description: "Pull pre-structured, cleaned data from 100+ platforms including Amazon, LinkedIn, Instagram, TikTok, YouTube, Reddit, Zillow, Crunchbase, and more. Returns clean JSON — no parsing required. Requires PRO_MODE=true or GROUPS config. Always prefer this over bd-browser for supported platforms — it is faster and cheaper."
compatibility: Requires Bright Data MCP server with API_TOKEN configured and PRO_MODE=true or relevant GROUPS enabled.
allowed-tools: mcp(Bright Data:web_data_amazon_product), mcp(Bright Data:web_data_amazon_product_search), mcp(Bright Data:web_data_amazon_product_reviews), mcp(Bright Data:web_data_walmart_product), mcp(Bright Data:web_data_walmart_seller), mcp(Bright Data:web_data_ebay_product), mcp(Bright Data:web_data_homedepot_products), mcp(Bright Data:web_data_zara_products), mcp(Bright Data:web_data_etsy_products), mcp(Bright Data:web_data_bestbuy_products), mcp(Bright Data:web_data_linkedin_person_profile), mcp(Bright Data:web_data_linkedin_company_profile), mcp(Bright Data:web_data_linkedin_job_listings), mcp(Bright Data:web_data_linkedin_posts), mcp(Bright Data:web_data_linkedin_people_search), mcp(Bright Data:web_data_instagram_profiles), mcp(Bright Data:web_data_instagram_posts), mcp(Bright Data:web_data_instagram_reels), mcp(Bright Data:web_data_instagram_comments), mcp(Bright Data:web_data_facebook_posts), mcp(Bright Data:web_data_facebook_marketplace_listings), mcp(Bright Data:web_data_facebook_company_reviews), mcp(Bright Data:web_data_facebook_events), mcp(Bright Data:web_data_tiktok_profiles), mcp(Bright Data:web_data_tiktok_posts), mcp(Bright Data:web_data_tiktok_shop), mcp(Bright Data:web_data_tiktok_comments), mcp(Bright Data:web_data_x_posts), mcp(Bright Data:web_data_youtube_profiles), mcp(Bright Data:web_data_youtube_videos), mcp(Bright Data:web_data_youtube_comments), mcp(Bright Data:web_data_reddit_posts), mcp(Bright Data:web_data_google_maps_reviews), mcp(Bright Data:web_data_google_shopping), mcp(Bright Data:web_data_google_play_store), mcp(Bright Data:web_data_apple_app_store), mcp(Bright Data:web_data_zillow_properties_listing), mcp(Bright Data:web_data_booking_hotel_listings), mcp(Bright Data:web_data_crunchbase_company), mcp(Bright Data:web_data_zoominfo_company_profile), mcp(Bright Data:web_data_reuter_news), mcp(Bright Data:web_data_yahoo_finance_business), mcp(Bright Data:web_data_github_repository_file)
metadata:
  author: Bright Data
---

# Structured Data Extraction

Retrieve structured data for: $ARGUMENTS

## Step 1: Identify the right tool

Match the user's request to the platform and tool using this table:

### E-Commerce
| Platform | Use case | Tool |
|----------|----------|------|
| Amazon | Product details (use /dp/ URL) | `web_data_amazon_product` |
| Amazon | Search results by keyword | `web_data_amazon_product_search` |
| Amazon | Product reviews | `web_data_amazon_product_reviews` |
| Walmart | Product details (use /ip/ URL) | `web_data_walmart_product` |
| Walmart | Seller profile | `web_data_walmart_seller` |
| eBay | Product listing | `web_data_ebay_product` |
| Best Buy | Electronics catalog | `web_data_bestbuy_products` |
| Home Depot | Home improvement products | `web_data_homedepot_products` |
| Etsy | Marketplace listings | `web_data_etsy_products` |
| Zara | Fashion inventory | `web_data_zara_products` |
| TikTok Shop | E-commerce products | `web_data_tiktok_shop` |

### Professional & Business
| Platform | Use case | Tool |
|----------|----------|------|
| LinkedIn | Person profile | `web_data_linkedin_person_profile` |
| LinkedIn | Company page | `web_data_linkedin_company_profile` |
| LinkedIn | Job listings | `web_data_linkedin_job_listings` |
| LinkedIn | Posts & discussions | `web_data_linkedin_posts` |
| LinkedIn | People search | `web_data_linkedin_people_search` |
| Crunchbase | Startup / company info | `web_data_crunchbase_company` |
| ZoomInfo | Business intelligence | `web_data_zoominfo_company_profile` |
| Yahoo Finance | Company financials | `web_data_yahoo_finance_business` |

### Social Media & Content
| Platform | Use case | Tool |
|----------|----------|------|
| Instagram | Account profile | `web_data_instagram_profiles` |
| Instagram | Posts | `web_data_instagram_posts` |
| Instagram | Reels | `web_data_instagram_reels` |
| Instagram | Comments | `web_data_instagram_comments` |
| TikTok | Creator profile | `web_data_tiktok_profiles` |
| TikTok | Post metadata | `web_data_tiktok_posts` |
| TikTok | Comments | `web_data_tiktok_comments` |
| Facebook | Posts | `web_data_facebook_posts` |
| Facebook | Marketplace listings | `web_data_facebook_marketplace_listings` |
| Facebook | Company reviews | `web_data_facebook_company_reviews` |
| Facebook | Events | `web_data_facebook_events` |
| X / Twitter | Posts | `web_data_x_posts` |
| YouTube | Channel profile | `web_data_youtube_profiles` |
| YouTube | Video metadata | `web_data_youtube_videos` |
| YouTube | Comments (default: 10) | `web_data_youtube_comments` |
| Reddit | Posts | `web_data_reddit_posts` |

### Search, Maps & Services
| Platform | Use case | Tool |
|----------|----------|------|
| Google Shopping | Product search | `web_data_google_shopping` |
| Google Maps | Location reviews | `web_data_google_maps_reviews` |
| Google Play | Android app details | `web_data_google_play_store` |
| Apple App Store | iOS app details | `web_data_apple_app_store` |
| Zillow | Real estate listings | `web_data_zillow_properties_listing` |
| Booking.com | Hotel listings | `web_data_booking_hotel_listings` |
| Reuters | News articles | `web_data_reuter_news` |
| GitHub | Repository file content | `web_data_github_repository_file` |

## Step 2: Call the tool

Pass the platform URL or search keyword as required by the specific tool. Example:

```
web_data_amazon_product({ url: "https://www.amazon.com/dp/B0XXXXXX" })
web_data_linkedin_person_profile({ url: "https://www.linkedin.com/in/username" })
web_data_amazon_product_search({ keyword: "wireless headphones" })
```

## Step 3: Present the results

Return the structured JSON data to the user. Summarize key fields clearly and offer to go deeper on specific attributes if needed.

## If PRO tools are unavailable

If `web_data_*` tools are not available, the account may need PRO_MODE enabled. Tell the user:

1. Structured data tools require PRO_MODE or GROUPS configuration
2. Add `"PRO_MODE": "true"` to the `env` section in `mcp.json`, or set `"GROUPS": "ecommerce,social,business"` for selective access
3. Run `/bd-setup` to verify the updated configuration