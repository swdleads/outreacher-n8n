---
name: dataforseo-api
description: Use when integrating with DataForSEO APIs — backlinks analysis, traffic estimation, keyword research, on-page parsing, and n8n integration patterns
---

# DataForSEO API Integration Skill

Reference for integrating DataForSEO's SEO data APIs into n8n workflows. Covers authentication, the universal response structure, bulk vs. single-domain endpoints, n8n-specific integration patterns, and common mistakes.

---

## Quick Start — Essential Rules

1. **Always use Basic auth** — `Authorization: Basic base64(login:password)` (never pass credentials as query params)
2. **All endpoints are POST** with JSON body — even "read" operations like fetching backlinks
3. **Response data lives at `tasks[0].result[0].items[]`** — always navigate the nested structure
4. **Bulk endpoints accept up to 1,000 targets** per call — batch domains, don't send one at a time
5. **Set HTTP timeout to 120000ms+** for bulk calls (default 30s is too low)
6. **Domain targets must omit protocol and www** — use `example.com`, not `https://www.example.com`
7. **Rate limit:** 2,000 calls/min, max 30 simultaneous requests

---

## Authentication

### Basic Auth (Only Method)

All DataForSEO API v3 endpoints use HTTP Basic Authentication.

```
Authorization: Basic <base64(login:password)>
```

Get credentials at: https://app.dataforseo.com/api-access

### n8n HTTP Request Node

Use **Generic Credential Type > HTTP Header Auth**:
- Header Name: `Authorization`
- Header Value: `Basic <pre-encoded-base64-string>`

### n8n Code Node

```javascript
const auth = Buffer.from('login:password').toString('base64');
const headers = { 'Authorization': `Basic ${auth}`, 'Content-Type': 'application/json' };
```

Or with the universal HTTP helper pattern (recommended):
```javascript
const response = await _httpRequest({
  method: 'POST',
  url: 'https://api.dataforseo.com/v3/backlinks/bulk_ranks/live',
  headers: {
    'Authorization': `Basic ${Buffer.from('login:password').toString('base64')}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify([{ targets: ['example.com', 'test.com'] }])
});
```

---

## Universal Response Structure

Every DataForSEO API v3 endpoint returns the same top-level structure:

```json
{
  "version": "0.1.20241220",
  "status_code": 20000,
  "status_message": "Ok.",
  "time": "0.1500 sec.",
  "cost": 0.0101,
  "tasks_count": 1,
  "tasks_error": 0,
  "tasks": [
    {
      "id": "02181234-5678-0001-0000-abc123def456",
      "status_code": 20000,
      "status_message": "Ok.",
      "time": "0.1200 sec.",
      "cost": 0.0101,
      "result_count": 1,
      "path": ["v3", "backlinks", "bulk_ranks", "live"],
      "data": { "api": "backlinks", "function": "bulk_ranks", "targets": ["example.com"] },
      "result": [
        {
          "total_count": 1,
          "items_count": 1,
          "items": [
            { "target": "example.com", "rank": 842 }
          ]
        }
      ]
    }
  ]
}
```

### Key Rules

| Rule | Detail |
|------|--------|
| Success check | `tasks[0].status_code === 20000` |
| Data access | `tasks[0].result[0].items` |
| No data found | `items` is `null` (not empty array `[]`) |
| Error codes | 40000+ = client error, 50000+ = server error |
| Cost tracking | `cost` field on both top-level and per-task |

### Navigating the Response in Code

```javascript
const body = response; // from HTTP request
const task = body.tasks && body.tasks[0];
if (!task || task.status_code !== 20000) {
  throw new Error(`DataForSEO error: ${task?.status_message || 'No task returned'}`);
}
const items = task.result && task.result[0] && task.result[0].items;
if (!items) {
  // Domain not in database — handle gracefully
  return [{ json: { target: domain, error: 'No data available' } }];
}
```

---

## Endpoint Reference

Base URL: `https://api.dataforseo.com/v3`

All request bodies are wrapped in an array: `[{ ...params }]`

### Bulk Endpoints (up to 1,000 targets per call)

These endpoints are the most cost-efficient way to get data for many domains at once.

#### Backlinks — Bulk Backlinks

```
POST /backlinks/bulk_backlinks/live
Body: [{ "targets": ["domain1.com", "domain2.com", ...] }]
```

**Response items:** `{ target, backlinks }`

#### Backlinks — Bulk Ranks

```
POST /backlinks/bulk_ranks/live
Body: [{ "targets": ["domain1.com", ...], "rank_scale": "one_thousand" }]
```

**Response items:** `{ target, rank }`

| `rank_scale` | Range | Default |
|--------------|-------|---------|
| `one_thousand` | 0–1000 | Yes |
| `one_hundred` | 0–100 | No |

**Note:** DataForSEO rank != Ahrefs DR. No clean conversion formula exists. Mid-range (DFSEO 350-550) has +/-10 DR variance. Extremes correlate well (DFSEO 800+ ~ DR 95+). See `dataforseo-vs-ahrefs-dr.md` for full analysis.

#### Backlinks — Bulk Referring Domains

```
POST /backlinks/bulk_referring_domains/live
Body: [{ "targets": ["domain1.com", ...] }]
```

**Response items:** `{ target, referring_domains }`

#### Backlinks — Bulk Spam Score

```
POST /backlinks/bulk_spam_score/live
Body: [{ "targets": ["domain1.com", ...] }]
```

**Response items:** `{ target, spam_score }` (0–100 scale)

#### Labs — Bulk Traffic Estimation

```
POST /dataforseo_labs/google/bulk_traffic_estimation/live
Body: [{
  "targets": ["domain1.com", ...],
  "location_code": 2840,
  "language_code": "en"
}]
```

**Response items:**
```json
{
  "target": "domain1.com",
  "metrics": {
    "organic": { "etv": 12345.6, "count": 890 },
    "paid": { "etv": 100.0, "count": 5 },
    "featured_snippet": { "etv": 50.0, "count": 3 },
    "local_pack": { "etv": 20.0, "count": 2 }
  }
}
```

**Common location codes:**

| Country | Code |
|---------|------|
| United States | 2840 |
| United Kingdom | 2826 |
| Canada | 2124 |
| Australia | 2036 |
| Germany | 2276 |
| France | 2250 |
| India | 2356 |
| Brazil | 2076 |
| Mexico | 2484 |
| Philippines | 2608 |

### Single-Domain Endpoints

These endpoints take one domain/URL at a time and return detailed data.

#### Labs — Ranked Keywords

```
POST /dataforseo_labs/google/ranked_keywords/live
Body: [{
  "target": "domain.com",
  "location_code": 2840,
  "language_code": "en",
  "limit": 100,
  "order_by": ["keyword_data.keyword_info.search_volume,desc"],
  "filters": [["keyword_data.keyword_info.search_volume", ">", 0]]
}]
```

**Response items:** Keyword objects with `keyword_data.keyword`, `keyword_data.keyword_info.search_volume`, `ranked_serp_element.serp_item.rank_absolute`, `ranked_serp_element.serp_item.etv`

#### Labs — Relevant Pages

```
POST /dataforseo_labs/google/relevant_pages/live
Body: [{
  "target": "domain.com",
  "location_code": 2840,
  "language_code": "en",
  "limit": 20
}]
```

**Response items:** Page objects with `page_address`, `metrics.organic.etv`, `metrics.organic.count`

#### Labs — Domain Rank Overview

```
POST /dataforseo_labs/google/domain_rank_overview/live
Body: [{
  "target": "domain.com",
  "location_code": 2840,
  "language_code": "en"
}]
```

**Response items:** Contains `organic` and `paid` metric objects with `etv`, `count`, `estimated_paid_traffic_cost`, position distribution (`pos_1` through `pos_91_100`), and change indicators (`is_new`, `is_up`, `is_down`, `is_lost`).

#### Labs — Historical Rank Overview

```
POST /dataforseo_labs/google/historical_rank_overview/live
Body: [{
  "target": "domain.com",
  "location_code": 2840,
  "language_code": "en"
}]
```

**Response items:** Monthly snapshots with same metrics as domain rank overview, including `date` field. Useful for detecting traffic trends (growth/decline over 6-12 months).

#### OnPage — Content Parsing

```
POST /on_page/content_parsing/live
Body: [{
  "url": "https://www.example.com/page",
  "enable_javascript": true
}]
```

**Response items:** `{ page_content: { header, footer, main_topic, secondary_topic, contacts, ... } }`

Useful for extracting page text when you need to analyze content without a full browser scrape.

#### Backlinks — Anchors

```
POST /backlinks/anchors/live
Body: [{
  "target": "domain.com",
  "limit": 100,
  "order_by": ["backlinks,desc"]
}]
```

**Response items:** `{ anchor, backlinks, referring_domains, referring_main_domains }`

> **Comprehensive reference:** For all DataForSEO API categories beyond these core endpoints — including keyword research, competitive analysis, SERP, AI optimization, trends, domain analytics, Lighthouse, content analysis, and business data — see `ENDPOINTS.md`.

---

## n8n Integration Patterns

### Pattern 1: Bulk Call via HTTP Request Node

Best for: Simple bulk data fetches where you need one API call for many domains.

**Node Configuration:**
```
Type: HTTP Request (v4.2)
Method: POST
URL: https://api.dataforseo.com/v3/backlinks/bulk_ranks/live
Authentication: Generic Credential > HTTP Header Auth
Body Type: JSON
Body: [{ "targets": {{ $json.domains }} }]
Options: timeout=120000
retryOnFail: true
maxTries: 3
waitBetweenTries: 5000
```

**Downstream Code node to parse:**
```javascript
const body = $input.first().json;
const items = body.tasks?.[0]?.result?.[0]?.items || [];
return items.map(item => ({ json: item }));
```

### Pattern 2: Multi-Location Bulk Calls in Code Node

Best for: Traffic estimation across multiple countries (e.g., calculating English traffic %).

Uses the **universal HTTP helper pattern** (see Key n8n Patterns in MEMORY.md).

```javascript
// Universal HTTP helper detection block goes here (see MEMORY.md)

const auth = Buffer.from('login:password').toString('base64');
const targets = $input.all().map(i => i.json.domain);

const locations = [
  { code: 2840, name: 'US' },
  { code: 2826, name: 'UK' },
  { code: 2124, name: 'CA' },
  { code: 2036, name: 'AU' },
  { code: 2276, name: 'DE' },
  { code: 2250, name: 'FR' },
  { code: 2356, name: 'IN' },
  { code: 2076, name: 'BR' },
  { code: 2484, name: 'MX' },
  { code: 2608, name: 'PH' }
];

const results = {};
const errors = [];

for (const loc of locations) {
  try {
    const resp = await _httpRequest({
      method: 'POST',
      url: 'https://api.dataforseo.com/v3/dataforseo_labs/google/bulk_traffic_estimation/live',
      headers: {
        'Authorization': `Basic ${auth}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify([{
        targets,
        location_code: loc.code,
        language_code: 'en'
      }])
    });

    const items = resp.tasks?.[0]?.result?.[0]?.items || [];
    for (const item of items) {
      if (!results[item.target]) results[item.target] = {};
      results[item.target][loc.name] = item.metrics?.organic?.etv || 0;
    }
  } catch (err) {
    errors.push(`${loc.name}: ${err.message}`);
  }
}

// Calculate English traffic %
const englishCodes = ['US', 'UK', 'CA', 'AU'];
const output = Object.entries(results).map(([domain, locs]) => {
  const englishEtv = englishCodes.reduce((sum, c) => sum + (locs[c] || 0), 0);
  const totalEtv = Object.values(locs).reduce((sum, v) => sum + v, 0);
  const englishPct = totalEtv > 0 ? Math.round((englishEtv / totalEtv) * 100) : 0;
  return {
    json: {
      domain,
      trafficByLocation: locs,
      englishTrafficPct: englishPct,
      totalEstTraffic: totalEtv,
      _apiErrors: errors.length > 0 ? errors : undefined
    }
  };
});

return output;
```

### Pattern 3: Per-Item Single Call in Code Node

Best for: Detailed per-domain data (ranked keywords, relevant pages, historical data).

```javascript
// Universal HTTP helper detection block goes here (see MEMORY.md)

const auth = Buffer.from('login:password').toString('base64');
const items = $input.all();
const output = [];
const errors = [];

for (const item of items) {
  const domain = item.json.domain;
  try {
    const resp = await _httpRequest({
      method: 'POST',
      url: 'https://api.dataforseo.com/v3/dataforseo_labs/google/ranked_keywords/live',
      headers: {
        'Authorization': `Basic ${auth}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify([{
        target: domain,
        location_code: 2840,
        language_code: 'en',
        limit: 100,
        order_by: ['keyword_data.keyword_info.search_volume,desc']
      }])
    });

    const task = resp.tasks?.[0];
    if (task?.status_code !== 20000) {
      errors.push(`${domain}: ${task?.status_message}`);
      output.push({ json: { domain, keywords: [], _error: task?.status_message } });
      continue;
    }
    const keywords = task.result?.[0]?.items || [];
    output.push({ json: { ...item.json, keywords, keywordCount: keywords.length } });
  } catch (err) {
    errors.push(`${domain}: ${err.message}`);
    output.push({ json: { domain, keywords: [], _error: err.message } });
  }
}

if (errors.length > 0) {
  output[0].json._apiErrors = errors;
}

return output;
```

---

## Common Mistakes

### 1. Wrong Response Navigation

```
WRONG:  const items = response.items
WRONG:  const items = response.result[0].items
RIGHT:  const items = response.tasks[0].result[0].items
```

### 2. Not Handling null Items

```
WRONG:  const items = resp.tasks[0].result[0].items
        items.forEach(...)  // TypeError if items is null

RIGHT:  const items = resp.tasks?.[0]?.result?.[0]?.items || []
```

### 3. Including Protocol in Domain Targets

```
WRONG:  targets: ["https://www.example.com"]
        Result: no data found or wrong target match

RIGHT:  targets: ["example.com"]
        For page-level: targets: ["https://example.com/specific-page"]
```

### 4. Not Wrapping Body in Array

```
WRONG:  body: { "targets": ["example.com"] }
        Result: 40000 error — "post data is not valid"

RIGHT:  body: [{ "targets": ["example.com"] }]
```

### 5. Using GET Instead of POST

```
WRONG:  method: 'GET'
        All DataForSEO v3 endpoints require POST

RIGHT:  method: 'POST'
```

### 6. HTTP Timeout Too Low

```
WRONG:  HTTP Request node with default 30s timeout + 500 domains
        Result: timeout error after 30s

RIGHT:  Set timeout to 120000ms (2 min) via node options
```

### 7. Not Batching Domains

```
WRONG:  Loop over 200 domains, call bulk_ranks 200 times with 1 domain each
        Result: 200 API calls, slow, expensive

RIGHT:  Single call with targets: [all 200 domains]
        Result: 1 API call, fast, cheap
```

### 8. Forgetting Base64 Encoding

```
WRONG:  Authorization: Basic login:password
        Result: 401 Unauthorized

RIGHT:  Authorization: Basic bG9naW46cGFzc3dvcmQ=
```

---

## Status Codes

| Code | Meaning |
|------|---------|
| 20000 | Success |
| 20100 | Task created (async) |
| 40000 | Bad request (invalid params) |
| 40001 | Insufficient credits |
| 40100 | Unauthorized (bad credentials) |
| 40200 | Payment required |
| 40400 | Not found |
| 40501 | Timeout |
| 50000 | Internal server error |

---

## Rate Limits and Costs

| Limit | Value |
|-------|-------|
| API calls per minute | 2,000 |
| Simultaneous requests | 30 |
| Bulk targets per call | 1,000 |
| Ranked keywords limit | 1,000 per call |

### Cost-Efficient Patterns

- **Bulk endpoints** cost ~$0.01 per 1,000 domains — always prefer bulk over single calls
- **Traffic estimation** across 10 locations for 100 domains = ~10 API calls = ~$0.10
- **Ranked keywords** per domain costs more — limit to essential domains
- **On-page content parsing** charges per page — use sparingly as a fallback

---

## API Reference Quick Lookup

| Action | Endpoint |
|--------|----------|
| Bulk backlink counts | `/backlinks/bulk_backlinks/live` |
| Bulk domain rank | `/backlinks/bulk_ranks/live` |
| Bulk referring domains | `/backlinks/bulk_referring_domains/live` |
| Bulk spam score | `/backlinks/bulk_spam_score/live` |
| Bulk traffic estimation | `/dataforseo_labs/google/bulk_traffic_estimation/live` |
| Ranked keywords (per domain) | `/dataforseo_labs/google/ranked_keywords/live` |
| Relevant pages (per domain) | `/dataforseo_labs/google/relevant_pages/live` |
| Domain rank overview | `/dataforseo_labs/google/domain_rank_overview/live` |
| Historical rank overview | `/dataforseo_labs/google/historical_rank_overview/live` |
| Content parsing (per URL) | `/on_page/content_parsing/live` |
| Anchor text analysis | `/backlinks/anchors/live` |
| Backlink details | `/backlinks/backlinks/live` |
| Domain competitors | `/dataforseo_labs/google/competitors_domain/live` |
| Keyword ideas | `/dataforseo_labs/google/keyword_ideas/live` |
| Keyword suggestions | `/dataforseo_labs/google/keyword_suggestions/live` |
| Search intent | `/dataforseo_labs/search_intent/live` |
| SERP overview | `/serp/google/organic/live/advanced` |

Base URL: `https://api.dataforseo.com/v3`

---

## Related Resources

- [DataForSEO API v3 Documentation](https://docs.dataforseo.com/v3/)
- [DataForSEO Pricing](https://dataforseo.com/pricing)
- [DataForSEO n8n Community Node](https://github.com/dataforseo/n8n-nodes-dataforseo)
- [DataForSEO MCP Server](https://github.com/dataforseo/mcp-server-typescript)
- [DataForSEO n8n Integration Guide](https://dataforseo.com/help-center/connecting-dataforseo-mcp-server-to-your-n8n-workflows)
