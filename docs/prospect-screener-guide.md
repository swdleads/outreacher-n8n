# Prospect Screener — Team Guide

## What Does This Workflow Do?

When we're doing link building for a client, we need to find websites that are a good fit to link back to them. But before we reach out to anyone, we need to check that each prospect website is actually worth contacting.

That's what this workflow does. You give it a list of prospect websites and the client's target pages, and it automatically checks every single prospect for you. It throws out the bad ones and scores the good ones based on how relevant they are to the client.

Think of it like a bouncer at a club. The bouncer checks IDs, dress codes, and guest lists before letting anyone in. This workflow does the same thing — but for websites.

---

## How to Use It

1. Go to the form: **https://outreacher.app.n8n.cloud/form/prospect-screener**

2. Fill in these fields:
   - **Client Website** — The client's domain (e.g. `cordsclub.com`)
   - **Target Pages** — The specific pages on the client's site you want to build links to. One URL per line.
   - **Prospect Websites** — The list of websites you want to check. One domain per line. You can paste up to 3,000 at once.
   - **Client Link Tracking Sheet** *(optional)* — If the client has a Google Sheet tracking existing link placements, paste the URL here. The workflow will skip any prospects already in that sheet.
   - **Custom Blocklist** *(optional)* — Any domains you want to automatically reject. One per line.

3. Hit submit and wait. You'll get a Slack notification in the #3-prospect-inspector channel when it starts and another when it finishes, along with the Google sheet output link.

4. Open the Google Sheet link from the Slack message to see your results.

---

## What Happens Behind the Scenes

The workflow runs prospects through three stages of checks, from cheapest to most expensive. This way, obviously bad prospects get eliminated early without wasting time or money on deeper analysis.

### Stage 1 — Quick Checks (Free, Instant)

These are fast, automatic filters that don't cost anything:

- **Self-match** — Removes the client's own domain if it was accidentally included
- **TLD filter** — Removes domains with non English country-specific extensions that aren't relevant (like `.ru`, `.cn`, `.com.mx`)
- **Vetting Tracker** — Removes domains we've already vetted in past campaigns (checked against our master vetting sheet)
- **Link Tracking** — Removes domains that already have an active link placement with the client
- **Custom Blocklist** — Removes any domains you specifically listed as blocked

### Stage 2 — Deeper Check (1 API Call)

- **Already Linking** — Uses DataForSEO to check if the prospect already has a backlink pointing to the client's site. If they do, there's no need to reach out — they're already linking.

### Stage 3 — Content Analysis (Per Prospect, in Batches)

This is where the real work happens. Prospects that survived Stages 1 and 2 get analyzed in batches of 10:

- **Gray Niche Detection** — The workflow scans each prospect's site for outbound links to gambling, adult, or other shady websites. If a site links out to those kinds of places, it gets rejected. (Note: a news article *about* gambling doesn't count — only actual outbound *links* to those sites.)

- **Website Description** — AI reads the prospect's homepage and writes a short summary of what the site is about.

- **Relevance Rating** — AI compares the prospect's content to each of the client's target pages and picks the best match. It gives a score from 0 to 10:
  - **0** = Completely unrelated (a cooking blog for a car parts client)
  - **3-4** = Loose connection (a general lifestyle site for a fashion client)
  - **5-6** = Decent overlap (a men's style blog for a clothing brand)
  - **7-10** = Strong match (a fabric care blog for a corduroy company)

---

## What You Get Back

The workflow creates a Google Sheet with three tabs:

### Tab 1: Passed Prospects

These are the websites that made it through all three stages. Each row includes:

| Column | What It Means |
|--------|--------------|
| Prospect Website | The domain that passed |
| Relevance Rating | Score from 0-10 |
| Target Page | Which client page is the best match |
| Target Topics | What that target page is about |
| Target Source | How we got the target page info (Firecrawl, SEO keywords, or URL only) |
| Prospect Description | AI-written summary of the prospect site |
| Justification | Why the AI gave that relevance score |
| Prospect Description Source | How we got the prospect info (Firecrawl content or domain name guess) |

Rows are sorted by relevance rating, highest first.

### Tab 2: Reject Sites

Every prospect that got removed, along with the reason. Examples:
- "TLD blocked: .ru"
- "Found in Vetting Tracker"
- "Already linking to client"
- "Gray Niche (outbound links to gambling/adult content)"

### Tab 3: Summary

A stage-by-stage breakdown showing how many prospects were removed at each step and how many survived. Useful for understanding why a large batch got narrowed down.

---

## How Long Does It Take?

| Prospects Submitted | Approximate Time |
|--------------------|-----------------|
| 10-50 | 1-2 minutes |
| 100-200 | 5-10 minutes |
| 500 | 15-20 minutes |
| 1,000 | 20-30 minutes |
| 3,000 | 30-45 minutes |

Most of the time is spent in Stage 3 (fetching and analyzing each prospect website). Stages 1 and 2 usually finish in under 30 seconds regardless of how many prospects you submit.

---

## Cost Per Run

The workflow uses three paid services. Here's a rough cost breakdown:

| Prospects Submitted | Firecrawl | OpenAI | DataForSEO | Total |
|--------------------|-----------|--------|------------|-------|
| 50 | ~$0.10 | ~$0.01 | ~$0.10 | ~$0.21 |
| 500 | ~$1.00 | ~$0.05 | ~$0.10 | ~$1.15 |
| 3,000 | ~$4.00 | ~$0.30 | ~$0.10 | ~$4.40 |

Stage 1 and 2 filtering happens before any expensive API calls, so if most of your prospects get filtered early, the cost goes down.

---

## Tips

- **Paste more, not less.** It's cheap to filter out bad prospects, so don't pre-filter your list manually. Let the workflow do it.
- **Use the Link Tracking Sheet field** if the client has one. It prevents you from reaching out to sites that already have a placement.
- **Check the Reject Sites tab** after a run. If you see prospects getting rejected for the wrong reasons, let us know so we can tune the filters.
- **Relevance scores are guidelines, not gospel.** A score of 3 doesn't mean "never contact." It means the AI thinks the connection is loose. Use your judgment.

---

## Slack Notifications

You'll receive two Slack messages per run:

1. **Start notification** — Confirms the run kicked off, shows the client name, number of prospects, and a link to the Google Sheet.
2. **Completion notification** — Shows how many prospects passed vs. got rejected, with a breakdown by rejection reason.

---

## Questions?

If something looks wrong or a run fails reach out to Scott.
