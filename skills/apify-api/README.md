# Apify API Integration Skill

## Purpose

Provides expert guidance for integrating Apify actors (web scrapers, automation tools) via their REST API. Covers authentication, synchronous and asynchronous execution patterns, result retrieval, and n8n-specific integration patterns.

## File Structure

| File | Lines | Description |
|------|-------|-------------|
| `SKILL.md` | ~350 | Main guide — authentication, sync/async execution, n8n patterns, common mistakes |
| `README.md` | ~60 | This file — overview, triggers, file descriptions |

## Activation Triggers

This skill activates when the conversation involves:

- Apify actor execution or API calls
- Web scraping via Apify platform
- Facebook Pages Scraper (`4Hv5RhChiaDk6iwad`)
- `run-sync-get-dataset-items` endpoint
- Apify dataset retrieval
- HTTP Request nodes calling Apify APIs
- Async polling patterns for external actor services

## What You'll Learn

- How to authenticate with the Apify API (Bearer token)
- When to use synchronous vs. asynchronous execution
- How to configure n8n HTTP Request nodes for Apify
- Facebook Pages Scraper input/output schema
- Common integration mistakes and how to avoid them
- Polling patterns for long-running actors

## Key Topics

### Authentication
Bearer token via `Authorization` header. Never put tokens in URLs.

### Synchronous Execution
`POST /v2/acts/{actorId}/run-sync-get-dataset-items` — starts actor, waits for completion, returns results directly. Best for actors that finish in < 5 minutes.

### Asynchronous Execution
`POST /v2/acts/{actorId}/runs` — starts actor and returns immediately. Use `waitForFinish` param, polling, or webhooks to wait for completion.

### Facebook Pages Scraper
Official Apify actor for scraping public Facebook page data (email, phone, title, address, followers, etc.). Actor ID: `4Hv5RhChiaDk6iwad`.

### n8n Integration
Three patterns: sync HTTP Request (recommended), async + Code node polling, and async + Wait node (not recommended due to unreliable timing).

## Quick Reference

```
Sync run:  POST /v2/acts/{actId}/run-sync-get-dataset-items
Async run: POST /v2/acts/{actId}/runs
Poll:      GET  /v2/actor-runs/{runId}
Results:   GET  /v2/actor-runs/{runId}/dataset/items
```

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-17 | Initial creation — auth, sync/async execution, Facebook scraper, n8n patterns |
