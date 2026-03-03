---
name: bd-data
description: "Pull structured data from 100+ platforms: Amazon, LinkedIn, Instagram, TikTok, YouTube, Reddit, Zillow, and more. Usage: /bd-data <platform> <url-or-keyword>"
---

# Structured Data

## Request: $ARGUMENTS

Use the **bd-structured-data** skill to retrieve structured data for this request. Follow the skill instructions exactly.

The skill will match your request to the correct `web_data_*` tool based on the platform and use case, then return clean JSON results.

Examples of valid requests:
- `/bd-data amazon https://www.amazon.com/dp/B0XXXXXX`
- `/bd-data amazon search wireless headphones`
- `/bd-data linkedin https://www.linkedin.com/in/username`
- `/bd-data instagram profiles https://www.instagram.com/username`
- `/bd-data google maps reviews https://maps.google.com/...`