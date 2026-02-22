---
name: apify-api
description: Use when integrating with Apify actors via API — running scrapers, retrieving datasets, authentication, and n8n integration patterns
---

# Apify API Integration Skill

Reference for integrating Apify actors (web scrapers, automation tools) via their REST API. Covers authentication, running actors synchronously and asynchronously, retrieving results, and n8n-specific integration patterns.

---

## Quick Start — Essential Rules

1. **Always use Bearer token auth** via `Authorization: Bearer <token>` header (never expose token in URLs)
2. **Prefer synchronous execution** for actors that finish in < 5 minutes — one request, direct results
3. **Never use fixed Wait nodes** for async runs — actor completion times vary; use polling or sync endpoints
4. **Set HTTP client timeout** to at least 120s when using sync endpoints (actors can take 10-60s typically)
5. **Handle empty results gracefully** — set `alwaysOutputData: true` in n8n to prevent downstream nodes from being skipped

---

## Authentication

### Bearer Token (Recommended)

```
Authorization: Bearer <YOUR_API_TOKEN>
```

Find your token at: Apify Console > Settings > Integrations

### Query Parameter (Less Secure)

```
?token=<YOUR_API_TOKEN>
```

Avoid this method — tokens in URLs get logged in server/proxy logs.

### n8n Configuration

Use **Generic Credential Type > HTTP Header Auth** in HTTP Request nodes:
- Header Name: `Authorization`
- Header Value: `Bearer <YOUR_API_TOKEN>`

---

## Running Actors

### Synchronous Execution (Recommended for < 5 min actors)

**Endpoint:** `POST https://api.apify.com/v2/acts/{actorId}/run-sync-get-dataset-items`

Starts the actor, waits for it to finish, and returns dataset items directly in the response body.

| Feature | Detail |
|---------|--------|
| Max timeout | 300 seconds (API-enforced hard limit) |
| Timeout response | HTTP 408 |
| Failure response | HTTP 400 with error type and message |
| Success response | HTTP 201 with dataset items as JSON array |
| Default format | JSON |

**Key Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `timeout` | number | Execution timeout in seconds (default: actor's config) |
| `memory` | number | RAM in MB (128 min, power of 2) |
| `format` | string | Output format: `json`, `jsonl`, `csv`, `html`, `xlsx`, `xml`, `rss` |
| `limit` | number | Max dataset items to return |
| `offset` | number | Items to skip from start |
| `fields` | string | Comma-separated fields to include |
| `clean` | boolean | Exclude empty/hidden fields |
| `maxItems` | number | Charge limit for pay-per-result actors |

**Example — Run and get results in one call:**

```
POST https://api.apify.com/v2/acts/4Hv5RhChiaDk6iwad/run-sync-get-dataset-items?format=json&limit=1
Authorization: Bearer <token>
Content-Type: application/json

{
  "startUrls": [
    { "url": "https://www.facebook.com/ExamplePage", "method": "GET" }
  ]
}
```

**Response (HTTP 201):**
```json
[
  {
    "facebookUrl": "https://www.facebook.com/ExamplePage",
    "email": "contact@example.com",
    "phone": "(555) 123-4567",
    "title": "Example Company",
    ...
  }
]
```

Pagination headers included: `X-Apify-Pagination-Offset`, `X-Apify-Pagination-Limit`, `X-Apify-Pagination-Count`, `X-Apify-Pagination-Total`

---

### Asynchronous Execution

**Endpoint:** `POST https://api.apify.com/v2/acts/{actorId}/runs`

Starts the actor and returns immediately with run metadata (including the run ID).

**Key Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `timeout` | number | Execution timeout in seconds |
| `memory` | number | RAM in MB |
| `build` | string | Build version/tag (default: `latest`) |
| `waitForFinish` | number | Hold connection 0-60 seconds before responding |
| `maxItems` | number | Charge limit for pay-per-result |

**Example — Start a run:**

```
POST https://api.apify.com/v2/acts/4Hv5RhChiaDk6iwad/runs
Authorization: Bearer <token>
Content-Type: application/json

{
  "startUrls": [
    { "url": "https://www.facebook.com/ExamplePage", "method": "GET" }
  ]
}
```

**Response (HTTP 201):**
```json
{
  "data": {
    "id": "PWVviPQf6ofIFYslb",
    "actId": "4Hv5RhChiaDk6iwad",
    "status": "READY",
    "defaultDatasetId": "ZBuMxO7t1HhS01uid",
    ...
  }
}
```

### Waiting for Async Completion

Three strategies, in order of preference:

**1. `waitForFinish` parameter (simplest, 0-60s max)**

```
POST /v2/acts/{actId}/runs?waitForFinish=60
```

If the actor finishes within 60s, response includes `status: "SUCCEEDED"`. Otherwise, returns `status: "RUNNING"` and you must poll.

**2. Polling (reliable, any duration)**

```
GET https://api.apify.com/v2/actor-runs/{runId}
```

Check `data.status` field:
- `READY` — queued, not started
- `RUNNING` — in progress
- `SUCCEEDED` — completed successfully
- `FAILED` — error occurred
- `ABORTED` — manually stopped
- `TIMED-OUT` — exceeded timeout

Poll every 5-10 seconds until terminal status.

**3. Webhooks (event-driven)**

Configure a webhook URL in the actor settings or via API. Apify POSTs a JSON payload with run info when the actor finishes. Must respond with HTTP 200 to prevent retries.

---

## Retrieving Results

### Dataset Items

**Endpoint:** `GET https://api.apify.com/v2/datasets/{datasetId}/items`

Or via run: `GET https://api.apify.com/v2/actor-runs/{runId}/dataset/items`

| Parameter | Description |
|-----------|-------------|
| `format` | `json` (default), `csv`, `jsonl`, `xlsx`, `xml`, `rss` |
| `limit` | Max items (max 250,000 per request) |
| `offset` | Skip N items |
| `fields` | Comma-separated field names |
| `clean` | Remove empty fields |

### Key-Value Store

For single outputs or files:

```
GET https://api.apify.com/v2/key-value-stores/{storeId}/records/{key}
```

Convention: single JSON output uses key `OUTPUT`.

---

## Rate Limits

| Scope | Limit |
|-------|-------|
| Global (per user/IP) | 250,000 requests/minute |
| Standard endpoints | 60 requests/second |
| Key-value CRUD | 200 requests/second |
| High-priority | 400 requests/second |

Exceeded limits return HTTP 429. Implement exponential backoff for retries.

---

## Facebook Pages Scraper

### Actor Details

| Property | Value |
|----------|-------|
| Actor ID | `4Hv5RhChiaDk6iwad` |
| Publisher | Apify (official) |
| Pricing | Pay-per-dataset-item ($0.012/page) |
| Typical runtime | 10-30 seconds per page |

### Input Schema

```json
{
  "startUrls": [
    {
      "url": "https://www.facebook.com/PageName",
      "method": "GET"
    }
  ]
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `facebookUrl` | string | Original Facebook page URL |
| `email` | string/null | Public email address listed on the page |
| `phone` | string/null | Public phone number |
| `title` | string | Page display title |
| `pageName` | string | Facebook page username/slug |
| `pageId` | string | Facebook numeric page ID |
| `pageUrl` | string | Canonical page URL |
| `category` | string | Primary category (e.g., "Advertising Agency") |
| `categories` | array | All categories (e.g., `["Page", "Board Game"]`) |
| `intro` | string | Page about/intro text |
| `info` | array | Summary info lines |
| `website` | string/null | External website URL |
| `websites` | array | All linked websites |
| `address` | string/null | Physical address |
| `addressUrl` | string/null | Map link for the address |
| `likes` | number | Page like count |
| `followers` | number | Follower count |
| `followings` | number | Following count |
| `messenger` | string/null | Messenger link |
| `rating` | string/null | Rating summary (e.g., "100% recommend (11 Reviews)") |
| `ratingOverall` | number/null | Numeric rating (0-100) |
| `ratingCount` | number/null | Number of reviews |
| `priceRange` | string/null | Price range indicator (e.g., "$$") |
| `profilePictureUrl` | string | Profile picture URL |
| `coverPhotoUrl` | string | Cover photo URL |
| `profilePhoto` | string | Photo page link |
| `instagram` | array | Instagram accounts `[{username, url}]` |
| `youtube` | array | YouTube channels `[{username, url}]` |
| `alternativeSocialMedia` | string/null | Other social media link |
| `creation_date` | string | Page creation date (e.g., "May 20, 2014") |
| `ad_status` | string | Ad running status |
| `facebookId` | string | Facebook numeric ID |
| `pageAdLibrary` | object | Ad library info |

---

## n8n Integration Patterns

### Pattern 1: Synchronous Run (Recommended)

Single HTTP Request node — simplest and most reliable.

**Node Configuration:**
```
Type: HTTP Request (v4.2)
Method: POST
URL: https://api.apify.com/v2/acts/{actorId}/run-sync-get-dataset-items
Authentication: Generic Credential → HTTP Header Auth
Query Params: format=json, limit=1
Body Type: JSON
Body: { "startUrls": [{ "url": "={{ $json.targetUrl }}", "method": "GET" }] }
Options: timeout=120000
retryOnFail: true
maxTries: 3
waitBetweenTries: 5000
onError: continueRegularOutput
alwaysOutputData: true
```

**Output:** Each dataset item becomes one n8n item. Access fields via `$json.email`, `$json.phone`, etc.

### Pattern 2: Async Run + Polling via Code Node

For actors that may take > 5 minutes, use a Code node to poll.

```javascript
// Async run + polling pattern
const actorId = '4Hv5RhChiaDk6iwad';
const apiToken = '<token>'; // Or use credential reference
const inputUrl = $json.targetUrl;

// Step 1: Start the run
const run = await $helpers.httpRequest({
  method: 'POST',
  url: `https://api.apify.com/v2/acts/${actorId}/runs`,
  headers: {
    'Authorization': `Bearer ${apiToken}`,
    'Content-Type': 'application/json'
  },
  body: {
    startUrls: [{ url: inputUrl, method: 'GET' }]
  }
});

const runId = run.data.id;
const MAX_POLLS = 24; // 24 x 5s = 2 min
const POLL_INTERVAL = 5000;

// Step 2: Poll until complete
let status = 'RUNNING';
for (let i = 0; i < MAX_POLLS; i++) {
  await new Promise(r => setTimeout(r, POLL_INTERVAL));
  const info = await $helpers.httpRequest({
    method: 'GET',
    url: `https://api.apify.com/v2/actor-runs/${runId}`,
    headers: { 'Authorization': `Bearer ${apiToken}` }
  });
  status = info.data?.status;
  if (['SUCCEEDED', 'FAILED', 'ABORTED', 'TIMED-OUT'].includes(status)) break;
}

if (status !== 'SUCCEEDED') {
  return [{ json: { error: `Actor ended with status: ${status}` } }];
}

// Step 3: Fetch results
const items = await $helpers.httpRequest({
  method: 'GET',
  url: `https://api.apify.com/v2/actor-runs/${runId}/dataset/items`,
  qs: { format: 'json', limit: '10' },
  headers: { 'Authorization': `Bearer ${apiToken}` }
});

return Array.isArray(items)
  ? items.map(item => ({ json: item }))
  : [{ json: { error: 'No items returned' } }];
```

### Pattern 3: Async Run + Wait Node (NOT RECOMMENDED)

```
HTTP Request (start run) → Wait (Ns) → HTTP Request (fetch results)
```

**Why this pattern fails:**
- Actor completion times are unpredictable (10s to 60s+)
- If the wait is too short: dataset is empty, results silently lost
- If the wait is too long: unnecessary delay for every execution
- No error visibility when timing fails — workflow reports success
- The Wait node passes through input data, masking the fact that no new data was fetched

---

## Common Mistakes

### 1. Fixed Wait Timing

```
❌ WRONG: Run Actor → Wait 15s → Fetch Results
   Actor takes 20s → empty dataset → email lost silently

✅ CORRECT: Use run-sync-get-dataset-items endpoint (1 node, waits automatically)
✅ CORRECT: Use polling loop in Code node (checks status before fetching)
```

### 2. HTTP Timeout Too Low

```
❌ WRONG: n8n HTTP Request with default 30s timeout calling sync endpoint
   Actor takes 45s → n8n times out → error

✅ CORRECT: Set HTTP Request timeout to 120000ms (2 min) via node options
```

### 3. Not Handling Empty Results

```
❌ WRONG: Fetch results → next node (no alwaysOutputData)
   Empty results → downstream nodes never execute → merge hangs or skips

✅ CORRECT: Set alwaysOutputData: true on the fetch node
✅ CORRECT: Code node returns [{json: {item: ""}}] as fallback
```

### 4. Token in URL

```
❌ WRONG: https://api.apify.com/v2/acts/xxx/runs?token=abc123
   Token logged in server access logs, browser history, proxy logs

✅ CORRECT: Authorization: Bearer abc123 (in header)
```

### 5. Not Setting Retry on External API Calls

```
❌ WRONG: Single attempt, no retry
   Transient network error → permanent failure

✅ CORRECT: retryOnFail: true, maxTries: 3-5, waitBetweenTries: 5000
```

---

## API Reference Quick Lookup

| Action | Method | Endpoint |
|--------|--------|----------|
| Run actor (async) | POST | `/v2/acts/{actorId}/runs` |
| Run actor (sync, get items) | POST | `/v2/acts/{actorId}/run-sync-get-dataset-items` |
| Run actor (sync, get run info) | POST | `/v2/acts/{actorId}/run-sync` |
| Get run status | GET | `/v2/actor-runs/{runId}` |
| Get dataset items | GET | `/v2/datasets/{datasetId}/items` |
| Get run dataset items | GET | `/v2/actor-runs/{runId}/dataset/items` |
| Get key-value record | GET | `/v2/key-value-stores/{storeId}/records/{key}` |
| List actor runs | GET | `/v2/acts/{actorId}/runs` |
| Abort run | POST | `/v2/actor-runs/{runId}/abort` |

Base URL: `https://api.apify.com`

---

## Related Resources

- [Apify API v2 Documentation](https://docs.apify.com/api/v2)
- [Run Actor and Retrieve Data Tutorial](https://docs.apify.com/platform/tutorials/run-actor-and-retrieve-data-via-api)
- [Sync Run Endpoint (POST)](https://docs.apify.com/api/v2/act-run-sync-get-dataset-items-post)
- [Facebook Pages Scraper](https://apify.com/apify/facebook-pages-scraper)
