# DataForSEO API — Comprehensive Endpoint Reference

All endpoints use `POST` with JSON body wrapped in an array: `[{ ...params }]`

Base URL: `https://api.dataforseo.com/v3`

Auth: `Authorization: Basic base64(login:password)`

Response: `tasks[0].result[0].items[]` (see SKILL.md for full response structure)

---

## Backlinks

### Bulk Endpoints (up to 1,000 targets)

Core bulk endpoints are documented in SKILL.md. Additional bulk endpoints below.

#### Bulk New/Lost Backlinks

```
POST /backlinks/bulk_new_lost_backlinks/live
Body: [{ "targets": ["domain1.com", ...], "date_from": "2025-01-01" }]
```

Track new and lost backlinks over a date range. Default `date_from`: today minus 1 month. Minimum: today minus 1 year.

**Response items:** `{ target, new_backlinks, lost_backlinks }`

#### Bulk New/Lost Referring Domains

```
POST /backlinks/bulk_new_lost_referring_domains/live
Body: [{ "targets": ["domain1.com", ...], "date_from": "2025-01-01" }]
```

**Response items:** `{ target, new_referring_domains, lost_referring_domains }`

#### Bulk Pages Summary

```
POST /backlinks/bulk_pages_summary/live
Body: [{ "targets": ["https://domain1.com/page1", ...] }]
```

Page-level backlink summary. Targets must be full URLs for page-level data.

**Response items:** `{ target, backlinks, referring_domains }`

### Single-Target Endpoints

#### Backlinks Summary

```
POST /backlinks/summary/live
Body: [{ "target": "domain.com" }]
```

Comprehensive backlink profile for one target. Most detailed single-target endpoint.

**Response items:** `{ target, backlinks, referring_domains, referring_domains_nofollow, referring_main_domains, referring_ips, referring_subnets, broken_backlinks, referring_pages, external_links_count, internal_links_count }`

#### Backlinks (detailed list)

```
POST /backlinks/backlinks/live
Body: [{
  "target": "domain.com",
  "limit": 100,
  "mode": "as_is",
  "order_by": ["rank,desc"],
  "filters": [["dofollow", "=", true]]
}]
```

| Parameter | Values |
|-----------|--------|
| `mode` | `as_is` (all), `one_per_domain`, `one_per_anchor` |
| `limit` | max 1,000 |
| `filters` | up to 8 conditions |

**Response items:** `{ url_from, url_to, anchor, domain_from, domain_from_rank, page_from_rank, dofollow, first_seen, last_seen, is_new, is_lost }`

#### Referring Domains (detailed list)

```
POST /backlinks/referring_domains/live
Body: [{
  "target": "domain.com",
  "limit": 100,
  "order_by": ["backlinks,desc"]
}]
```

**Response items:** `{ domain, backlinks, first_seen, lost_date, rank, backlinks_spam_score }`

#### Referring Networks

```
POST /backlinks/referring_networks/live
Body: [{ "target": "domain.com", "limit": 100 }]
```

**Response items:** `{ network_address, backlinks, referring_domains, first_seen }`

#### Anchors

```
POST /backlinks/anchors/live
Body: [{
  "target": "domain.com",
  "limit": 100,
  "order_by": ["backlinks,desc"]
}]
```

**Response items:** `{ anchor, backlinks, referring_domains, referring_main_domains, first_seen, last_seen }`

#### Competitors

```
POST /backlinks/competitors/live
Body: [{
  "target": "domain.com",
  "limit": 100,
  "order_by": ["intersecting_domains,desc"]
}]
```

Finds domains competing for the same backlinks.

**Response items:** `{ target, backlinks, referring_domains, intersecting_domains }`

#### Domain Intersection

```
POST /backlinks/domain_intersection/live
Body: [{
  "targets": { "1": "domain1.com", "2": "domain2.com" },
  "limit": 100
}]
```

Finds referring domains shared between targets (up to 20 targets). Use numbered keys.

**Response items:** referring domains with per-target backlink details

#### Page Intersection

```
POST /backlinks/page_intersection/live
Body: [{
  "targets": { "1": "https://domain1.com/page", "2": "https://domain2.com/page" },
  "limit": 100
}]
```

Same as domain intersection but at page level.

#### Domain Pages

```
POST /backlinks/domain_pages/live
Body: [{ "target": "domain.com", "limit": 100, "order_by": ["backlinks,desc"] }]
```

**Response items:** `{ page, backlinks, referring_domains, first_seen, last_seen }`

#### Domain Pages Summary

```
POST /backlinks/domain_pages_summary/live
Body: [{ "target": "domain.com", "limit": 100 }]
```

#### Timeseries Summary

```
POST /backlinks/timeseries_summary/live
Body: [{
  "target": "domain.com",
  "date_from": "2025-01-01",
  "date_to": "2026-01-01"
}]
```

Monthly backlink metrics over time.

**Response items:** `{ date, backlinks, referring_domains, referring_main_domains, new_backlinks, lost_backlinks }`

#### Timeseries New/Lost Summary

```
POST /backlinks/timeseries_new_lost_summary/live
Body: [{ "target": "domain.com", "date_from": "2025-01-01" }]
```

**Response items:** monthly new/lost backlink counts

---

## Keyword Research

### Google Ads Search Volume

```
POST /keywords_data/google_ads/search_volume/live
Body: [{
  "keywords": ["seo tools", "backlink checker"],
  "location_name": "United States",
  "language_code": "en"
}]
```

Up to 700 keywords per call. Returns Google Ads metrics.

**Response items:** `{ keyword, search_volume, competition, competition_level, cpc, monthly_searches[{year, month, search_volume}] }`

**Notes:**
- `competition` is 0-1 float (paid search competition)
- `competition_level` is "LOW", "MEDIUM", or "HIGH"
- `monthly_searches` provides last 12 months of data
- Location can be country, region, or city level

### Keyword Overview

```
POST /dataforseo_labs/google/keyword_overview/live
Body: [{
  "keywords": ["seo tools", "backlink checker"],
  "location_code": 2840,
  "language_code": "en"
}]
```

Up to 700 keywords. Uses DataForSEO's own database (not Google Ads).

**Response items:** `{ keyword, keyword_info { search_volume, cpc, competition_level, monthly_searches, categories }, keyword_properties { keyword_difficulty, se_type }, impressions_info, serp_info, avg_backlinks_info }`

### Keyword Ideas

```
POST /dataforseo_labs/google/keyword_ideas/live
Body: [{
  "keywords": ["seo"],
  "location_code": 2840,
  "language_code": "en",
  "limit": 100,
  "filters": [["keyword_info.search_volume", ">", 100]],
  "order_by": ["keyword_info.search_volume,desc"]
}]
```

Generates keyword ideas from seed keywords using DataForSEO's SERP database.

**Response items:** keyword objects with `keyword_data.keyword`, `keyword_data.keyword_info`, `keyword_data.keyword_properties`

### Keyword Suggestions

```
POST /dataforseo_labs/google/keyword_suggestions/live
Body: [{
  "keyword": "seo tools",
  "location_code": 2840,
  "language_code": "en",
  "limit": 100
}]
```

Autocomplete-based keyword suggestions. Single seed keyword.

### Related Keywords

```
POST /dataforseo_labs/google/related_keywords/live
Body: [{
  "keyword": "seo tools",
  "location_code": 2840,
  "language_code": "en",
  "depth": 2,
  "limit": 100
}]
```

Keywords from "searches related to" SERP element. `depth` 1-4 (higher = more results, up to 4,680 keywords at depth 4).

### Bulk Keyword Difficulty

```
POST /dataforseo_labs/bulk_keyword_difficulty/live
Body: [{
  "keywords": ["seo tools", "backlink checker", ...],
  "location_code": 2840,
  "language_code": "en"
}]
```

Up to 1,000 keywords. Returns difficulty score on logarithmic 0-100 scale.

**Response items:** `{ keyword, keyword_difficulty }`

### Historical Keyword Data

```
POST /dataforseo_labs/google/historical_keyword_data/live
Body: [{
  "keywords": ["seo tools"],
  "location_code": 2840,
  "language_code": "en"
}]
```

Up to 700 keywords. Historical search volume data since August 2021.

**Response items:** `{ keyword, keyword_info { search_volume, cpc, competition, monthly_searches[{year, month, search_volume}] } }`

### Search Intent

```
POST /dataforseo_labs/google/search_intent/live
Body: [{
  "keywords": ["buy seo tools", "what is seo", "ahrefs login"],
  "language_code": "en"
}]
```

Classifies keywords by search intent.

**Response items:** `{ keyword, keyword_intent { label, probability } }`

| Intent | Description |
|--------|-------------|
| `informational` | User wants to learn something |
| `navigational` | User wants a specific website |
| `commercial` | User researching before purchase |
| `transactional` | User ready to buy/act |

---

## Competitive Analysis

### Domain Competitors

```
POST /dataforseo_labs/google/competitors_domain/live
Body: [{
  "target": "domain.com",
  "location_code": 2840,
  "language_code": "en",
  "limit": 50
}]
```

Finds domains competing for the same organic keywords.

**Response items:** `{ domain, avg_position, sum_position, intersections, full_domain_metrics { organic { etv, count, pos_1, ... } } }`

### Domain Intersection

```
POST /dataforseo_labs/google/domain_intersection/live
Body: [{
  "targets": { "1": "domain1.com", "2": "domain2.com", "3": "domain3.com" },
  "target_type": "organic",
  "location_code": 2840,
  "language_code": "en",
  "limit": 100
}]
```

Finds keywords shared between domains (up to 20 targets). Use numbered keys for targets.

**Response items:** keyword data with per-target ranking info

### Page Intersection

```
POST /dataforseo_labs/google/page_intersection/live
Body: [{
  "pages": { "1": "https://domain1.com/page1", "2": "https://domain2.com/page2" },
  "location_code": 2840,
  "language_code": "en",
  "limit": 100
}]
```

Keyword overlap at the page level.

### SERP Competitors

```
POST /dataforseo_labs/google/serp_competitors/live
Body: [{
  "keywords": ["seo tools", "backlink checker"],
  "location_code": 2840,
  "language_code": "en",
  "limit": 50
}]
```

Up to 200 keywords. Finds domains ranking for the specified keyword set.

**Response items:** `{ domain, avg_position, median_position, rating, etv, relevant_serp_items, keywords_count }`

### Subdomains

```
POST /dataforseo_labs/google/subdomains/live
Body: [{
  "target": "domain.com",
  "location_code": 2840,
  "language_code": "en",
  "limit": 50
}]
```

**Response items:** subdomain metrics with organic/paid ETVs

### Keywords for Site

```
POST /dataforseo_labs/google/keywords_for_site/live
Body: [{
  "target": "domain.com",
  "location_code": 2840,
  "language_code": "en",
  "limit": 100
}]
```

Returns keywords the site is relevant for (broader than ranked_keywords — includes keywords the site *could* rank for).

### Historical SERP

```
POST /dataforseo_labs/google/historical_serp/live
Body: [{
  "keyword": "seo tools",
  "location_code": 2840,
  "language_code": "en"
}]
```

Historical SERP snapshots for a keyword. Shows how rankings changed over time.

---

## SERP

### Google Organic (Live)

```
POST /serp/google/organic/live/advanced
Body: [{
  "keyword": "seo tools",
  "location_name": "United States",
  "language_code": "en",
  "device": "desktop",
  "depth": 100
}]
```

Real-time Google SERP results. `depth` up to 700. `device`: `desktop` or `mobile`. Location can be country, region, or city.

**Response items:** SERP elements with `type` (organic, paid, featured_snippet, local_pack, people_also_ask, etc.), `rank_group`, `rank_absolute`, `domain`, `url`, `title`, `description`

### YouTube Organic

```
POST /serp/youtube/organic/live/advanced
Body: [{
  "keyword": "seo tutorial",
  "location_name": "United States",
  "language_code": "en"
}]
```

YouTube search results. `block_depth` up to 700.

### YouTube Video Info

```
POST /serp/youtube/video_info/live/advanced
Body: [{ "video_id": "dQw4w9WgXcQ" }]
```

Video metadata: title, description, views, likes, channel info.

### YouTube Video Comments

```
POST /serp/youtube/video_comments/live/advanced
Body: [{ "video_id": "dQw4w9WgXcQ", "depth": 100 }]
```

### YouTube Video Subtitles

```
POST /serp/youtube/video_subtitles/live/advanced
Body: [{ "video_id": "dQw4w9WgXcQ", "subtitles_language": "en" }]
```

Returns transcript/subtitle text. Useful for content analysis.

---

## AI Optimization

### AI Search Volume

```
POST /ai_optimization/keyword_data/search_volume/live
Body: [{
  "keywords": ["best seo tools", "how to build backlinks"],
  "location_name": "United States",
  "language_code": "en"
}]
```

Up to 1,000 keywords. Estimated usage of keywords in AI LLM conversations.

### LLM Response

```
POST /ai_optimization/llm_response/live
Body: [{
  "llm_type": "chat_gpt",
  "model_name": "gpt-4o",
  "user_prompt": "What are the best SEO tools?",
  "web_search": true
}]
```

Get responses from AI models. `llm_type`: `claude`, `gemini`, `chat_gpt`, `perplexity`. Use `ai_optimization/llm_models` to list available models per provider.

### LLM Models

```
POST /ai_optimization/llm_models/live
Body: [{ "llm_type": "chat_gpt" }]
```

Lists available model names for a provider.

### LLM Mentions — Search

```
POST /ai_optimization/llm_mentions/search/live
Body: [{
  "target": [
    { "domain": "example.com", "search_scope": ["answer"] },
    { "keyword": "seo tools", "match_type": "word_match", "search_scope": ["question"] }
  ],
  "platform": "chat_gpt",
  "location_name": "United States",
  "language_code": "en",
  "limit": 100
}]
```

Search for mentions of domains or keywords in LLM responses. `platform`: `chat_gpt` or `google`.

**Response items:** pages with mention frequency, context, and aggregated metrics

### LLM Mentions — Aggregated Metrics

```
POST /ai_optimization/llm_mentions/agg_metrics/live
Body: [{ "target": [{"domain": "example.com"}], "platform": "chat_gpt" }]
```

### LLM Mentions — Top Domains

```
POST /ai_optimization/llm_mentions/top_domains/live
Body: [{ "target": [{"keyword": "seo tools"}], "platform": "chat_gpt", "limit": 50 }]
```

Most-mentioned domains for a keyword in LLM responses.

### LLM Mentions — Top Pages

```
POST /ai_optimization/llm_mentions/top_pages/live
Body: [{ "target": [{"domain": "example.com"}], "platform": "chat_gpt", "limit": 50 }]
```

---

## Trends

### DataForSEO Trends — Explore

```
POST /keywords_data/dataforseo_trends/explore/live
Body: [{
  "keywords": ["seo", "content marketing"],
  "type": "web",
  "location_name": "United States",
  "time_range": "past_12_months"
}]
```

Up to 5 keywords. `type`: `web`, `news`, `ecommerce`. Alternative to Google Trends with more flexibility.

| Time Range | Value |
|------------|-------|
| Past 4 hours | `past_4_hours` |
| Past day | `past_day` |
| Past 7 days | `past_7_days` |
| Past 30 days | `past_30_days` |
| Past 90 days | `past_90_days` |
| Past 12 months | `past_12_months` |
| Past 5 years | `past_5_years` |

Or use `date_from`/`date_to` for custom ranges (overrides `time_range`).

### DataForSEO Trends — Demography

```
POST /keywords_data/dataforseo_trends/demography/live
Body: [{
  "keywords": ["seo tools"],
  "location_name": "United States",
  "time_range": "past_12_months"
}]
```

Demographic breakdown (age, gender) of keyword popularity.

### DataForSEO Trends — Subregion Interests

```
POST /keywords_data/dataforseo_trends/subregion_interests/live
Body: [{
  "keywords": ["seo tools"],
  "location_name": "United States",
  "time_range": "past_12_months"
}]
```

Location-specific keyword popularity within a country.

### Google Trends — Explore

```
POST /keywords_data/google_trends/explore/live
Body: [{
  "keywords": ["seo", "content marketing"],
  "location_code": 2840,
  "time_range": "past_12_months"
}]
```

### Google Trends — Categories

```
POST /keywords_data/google_trends/categories/live
Body: [{}]
```

Returns the full Google Trends category taxonomy.

---

## Domain Analytics

### WHOIS Overview

```
POST /domain_analytics/whois/overview/live
Body: [{
  "limit": 10,
  "filters": [["domain", "like", "%example%"]],
  "order_by": ["metrics.organic.etv,desc"]
}]
```

WHOIS data enriched with backlink stats and traffic info.

**Response items:** `{ domain, create_date, update_date, expiry_date, registrar, registrant, metrics { organic { etv, count }, paid { etv, count } } }`

### Technology Detection

```
POST /domain_analytics/technologies/domain_technologies/live
Body: [{ "target": "domain.com" }]
```

Detects the tech stack: CMS, analytics, frameworks, CDNs, advertising platforms, etc.

**Response items:** `{ technology_paths, groups[{ group_id, group_name, categories[{ category_id, category_name, technologies[{ name, version }] }] }] }`

---

## OnPage

### Content Parsing

Documented in SKILL.md. Summary:

```
POST /on_page/content_parsing/live
Body: [{ "url": "https://example.com/page", "enable_javascript": true }]
```

### Instant Pages

```
POST /on_page/instant_pages/live
Body: [{
  "url": "https://example.com",
  "enable_javascript": true,
  "enable_browser_rendering": true
}]
```

Quick page-level SEO analysis: meta tags, headings, images, links, page speed.

### Lighthouse

```
POST /on_page/lighthouse/live
Body: [{
  "url": "https://example.com",
  "for_mobile": false,
  "categories": ["performance", "seo", "accessibility", "best_practices"]
}]
```

Full Google Lighthouse audit.

| Category | Scores |
|----------|--------|
| `performance` | FCP, LCP, CLS, TBT, Speed Index |
| `seo` | Meta tags, crawlability, structured data |
| `accessibility` | ARIA, contrast, alt text |
| `best_practices` | HTTPS, console errors, image formats |

**Response items:** `{ categories { performance { score }, seo { score }, ... }, audits { ... } }`

---

## Content Analysis

### Content Search

```
POST /content_analysis/search/live
Body: [{
  "keyword": "seo tools review",
  "search_mode": "as_is",
  "limit": 50,
  "filters": [["content_info.sentiment_connotations.positive", ">", 0.5]]
}]
```

Search for web content by keyword. Returns sentiment analysis, social metrics.

**Response items:** `{ url, domain, title, content_info { sentiment { name, score }, connotation { name, score }, text_category, date_published, content_quality_score }, social_metrics { facebook_likes, reddit_subreddit } }`

### Content Summary

```
POST /content_analysis/summary/live
Body: [{
  "keyword": "seo tools",
  "limit": 10
}]
```

Aggregated content analysis: sentiment distribution, top domains, publication frequency.

### Phrase Trends

```
POST /content_analysis/phrase_trends/live
Body: [{
  "keyword": "seo tools",
  "date_from": "2025-01-01",
  "date_to": "2026-01-01"
}]
```

Tracks how often a phrase appears in content over time.

---

## Business Data

### Business Listings Search

```
POST /business_data/business_listings/search/live
Body: [{
  "categories": ["Restaurant"],
  "location_name": "San Francisco,California,United States",
  "limit": 50,
  "filters": [["rating.value", ">", 4]],
  "order_by": ["rating.value,desc"]
}]
```

Search Google, Trustpilot, Tripadvisor business listings.

**Response items:** `{ title, domain, url, address, phone, rating { value, votes_count, rating_max }, category, is_claimed }`

---

## Filter Syntax (Universal)

All filterable endpoints use the same syntax:

```json
// Simple filter
[["field_name", "operator", value]]

// Combined with AND
[["field1", ">", 100], "and", ["field2", "=", "value"]]

// Combined with OR
[["field1", ">", 100], "or", ["field2", "<", 50]]

// Nested
[["field1", ">", 100], "and", [["field2", "like", "%seo%"], "or", ["field3", "=", true]]]
```

| Operator | Description |
|----------|-------------|
| `=` | Equals |
| `<>` | Not equals |
| `<`, `<=`, `>`, `>=` | Comparison |
| `in` | In array |
| `not_in` | Not in array |
| `like` | String match (use `%` wildcard) |
| `not_like` | String not match |
| `ilike` | Case-insensitive like |
| `regex` | Regular expression match |
| `match` | Full-text match |

Maximum 8 filter conditions per request.

---

## Sorting Syntax (Universal)

```json
"order_by": ["field_name,asc"]
"order_by": ["field1,desc", "field2,asc"]
```

Maximum 3 sorting rules per request.

---

## Location Codes (Common)

| Country | Code | Country | Code |
|---------|------|---------|------|
| United States | 2840 | Germany | 2276 |
| United Kingdom | 2826 | France | 2250 |
| Canada | 2124 | Spain | 2724 |
| Australia | 2036 | Italy | 2380 |
| India | 2356 | Netherlands | 2528 |
| Brazil | 2076 | Japan | 2392 |
| Mexico | 2484 | South Korea | 2410 |
| Philippines | 2608 | Indonesia | 2360 |

Use `location_name` (string) or `location_code` (integer) — not both. Country-level for Labs endpoints; city/region allowed for SERP and Keywords Data.
