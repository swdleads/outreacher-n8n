# DataForSEO API Integration Skill

## Purpose

Provides expert guidance for integrating DataForSEO's SEO data APIs into n8n workflows. Covers authentication, response parsing, bulk vs. single-domain endpoints, n8n HTTP Request and Code node patterns, and common mistakes. Includes comprehensive endpoint reference for all DataForSEO API categories.

## File Structure

| File | Lines | Description |
|------|-------|-------------|
| `SKILL.md` | ~420 | Main guide — auth, response structure, core endpoints, n8n patterns, common mistakes |
| `ENDPOINTS.md` | ~500 | Comprehensive endpoint reference — all API categories with params and response fields |
| `README.md` | ~80 | This file — overview, triggers, file descriptions |

## Activation Triggers

This skill activates when the conversation involves:

- DataForSEO API calls or configuration
- Backlinks analysis (bulk ranks, spam score, referring domains, competitors, intersection)
- Traffic estimation across locations
- Ranked keywords, keyword research, keyword difficulty
- Competitive analysis (domain competitors, keyword overlap, SERP competitors)
- On-page content parsing, Lighthouse audits
- SERP data (Google, YouTube)
- AI optimization (LLM mentions, AI search volume)
- Google Trends or DataForSEO Trends
- Domain analytics (WHOIS, technology detection)
- Content analysis and sentiment
- SEO data fetching in n8n workflows
- Domain authority / rank comparisons

## What You'll Learn

- How to authenticate with DataForSEO API (Basic auth, Base64 encoding)
- The universal response structure (`tasks[0].result[0].items`)
- When to use bulk endpoints vs. single-domain endpoints
- How to configure n8n HTTP Request nodes for DataForSEO
- Multi-location traffic estimation pattern
- Per-item Code node API call pattern
- Common integration mistakes and how to avoid them
- All available endpoint categories, parameters, and response fields

## Key Topics

### Authentication
Basic auth with Base64-encoded `login:password`. Use HTTP Header Auth credential in n8n.

### Response Structure
All endpoints return `{ tasks: [{ result: [{ items: [...] }] }] }`. Status 20000 = success. Items can be `null` if domain not in database.

### Bulk Endpoints
`bulk_backlinks`, `bulk_ranks`, `bulk_referring_domains`, `bulk_spam_score`, `bulk_traffic_estimation` — up to 1,000 targets per call.

### Single-Domain Endpoints
`ranked_keywords`, `relevant_pages`, `domain_rank_overview`, `historical_rank_overview`, `content_parsing` — one domain/URL at a time.

### n8n Integration
Three patterns: HTTP Request node for simple bulk calls, Code node for multi-location calls, Code node for per-item iteration.

### Comprehensive API Coverage (ENDPOINTS.md)
All DataForSEO categories: Backlinks, Keyword Research, Competitive Analysis, SERP, AI Optimization, Trends, Domain Analytics, OnPage/Lighthouse, Content Analysis, Business Data.

## Quick Reference

```
Bulk ranks:          POST /v3/backlinks/bulk_ranks/live
Bulk spam:           POST /v3/backlinks/bulk_spam_score/live
Bulk traffic:        POST /v3/dataforseo_labs/google/bulk_traffic_estimation/live
Ranked keywords:     POST /v3/dataforseo_labs/google/ranked_keywords/live
Keyword difficulty:  POST /v3/dataforseo_labs/bulk_keyword_difficulty/live
Domain competitors:  POST /v3/dataforseo_labs/google/competitors_domain/live
SERP (Google):       POST /v3/serp/google/organic/live/advanced
Content parsing:     POST /v3/on_page/content_parsing/live
Lighthouse:          POST /v3/on_page/lighthouse/live
LLM mentions:        POST /v3/ai_optimization/llm_mentions/search/live
```

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-18 | Initial creation — auth, response structure, 11 endpoints, 3 n8n patterns, common mistakes |
| 1.1 | 2026-02-18 | Added ENDPOINTS.md — comprehensive reference for all API categories (60+ endpoints) |
