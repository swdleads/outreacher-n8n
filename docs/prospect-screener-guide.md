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
   - **Remove Sites in Vetting Tracker** — Select "Yes" to reject prospects that are already in our master vetting tracker (default). Select "No" to allow previously vetted sites through. Use "No" when re-screening a list that may include previously vetted sites you want to reconsider.
   - **Exclude Sites Linking to Shady Niches** — Select "Yes" to scan prospects for outbound links to gambling, adult, or other shady sites and reject them (default). Select "No" to skip this scan. Disabling also saves ~1 Firecrawl API call per prospect. Only disable if you've already pre-vetted the list or the niche is borderline (e.g., financial services that might trigger false positives on "betting").
   - **Custom Instructions** *(optional)* — Add specific guidance for the AI relevance rater. For example: "The client only wants links from B2B sites in the food industry" or "Prioritize sites with a strong editorial focus on sustainability." The AI will factor these instructions into every relevance rating.

3. Hit submit and wait. You'll get a Slack notification in the #3-prospect-inspector channel when it starts and another when it finishes, along with the Google sheet output link.

4. Open the Google Sheet link from the Slack message to see your results.

---

## What Happens Behind the Scenes

The workflow runs prospects through three stages of checks, from cheapest to most expensive. This way, obviously bad prospects get eliminated early without wasting time or money on deeper analysis.

### Stage 1 — Quick Checks (Free, Instant)

These are fast, automatic filters that don't cost anything:

- **Self-match** — Removes the client's own domain if it was accidentally included
- **TLD filter** — Removes domains with non English country-specific extensions that aren't relevant (like `.ru`, `.cn`, `.com.mx`)
- **Vetting Tracker** — Removes domains we've already vetted in past campaigns (checked against our master vetting sheet). *Can be disabled via the "Remove Sites in Vetting Tracker" form option.*
- **Link Tracking** — Removes domains that already have an active link placement with the client
- **Persistent Blocklist** — Removes domains that were flagged as gray niche sites in previous runs. The workflow automatically remembers bad sites so they never waste API calls again.
- **Custom Blocklist** — Removes any domains you specifically listed as blocked

### Stage 2 — Deeper Check (1 API Call)

- **Already Linking** — Uses DataForSEO to check if the prospect already has a backlink pointing to the client's site. If they do, there's no need to reach out — they're already linking.

### Stage 3 — Content Analysis (Per Prospect, in Batches)

This is where the real work happens. Prospects that survived Stages 1 and 2 get analyzed in batches of 10:

- **Gray Niche Detection** — The workflow uses a 3-layer detection system to scan each prospect for links to gambling, adult, or other shady websites. First, it checks the domain name itself for suspicious patterns (casino, betting, gambling, poker, slots, adult, etc.). Next, it uses Firecrawl to search the site's pages for gray-niche content and scans up to 2 suspicious inner pages for outbound links to shady domains. Finally, an AI confirms whether the site genuinely promotes gray-niche content or if it was a false positive (e.g., a news article *about* gambling doesn't count — only actual promotion of or outbound *links* to those sites). Any site rejected here is automatically added to the persistent blocklist so it gets filtered for free in Stage 1 on future runs. *Can be disabled via the "Exclude Sites Linking to Shady Niches" form option.*

- **Website Description** — AI reads the prospect's homepage and writes a short summary of what the site is about.

- **Relevance Rating** — AI analyzes the prospect against ALL of the client's target pages in a single evaluation. The AI considers all of the prospect's topics (not just its primary industry) and compares them against full summaries of each target page. It returns the prospect's main topics, the best-matching target URL, a score from 0 to 10, and a justification. If you provided Custom Instructions, the AI applies those criteria when scoring.
  - **0** = Completely unrelated (a cooking blog for a car parts client)
  - **3-4** = Loose connection (a general lifestyle site for a fashion client)
  - **5-6** = Decent overlap (a men's style blog for a clothing brand)
  - **7-10** = Strong match (a fabric care blog for a corduroy company)

---

## What You Get Back

The workflow creates a Google Sheet with four tabs:

### Tab 1: Passed Prospects

A clean, simple view of websites that made it through all three stages. Rows are sorted by relevance rating, highest first.

| Column | What It Means |
|--------|--------------|
| Prospect Website | The domain that passed |
| Relevance Rating | Score from 0-10 |

### Tab 2: All Passed Prospects Data

The full detail view for every passed prospect. Same rows as Tab 1 but with all the analysis data included.

| Column | What It Means |
|--------|--------------|
| Prospect Website | The domain that passed |
| Relevance Rating | Score from 0-10 |
| Target Pages Analyzed | How many of the client's target pages were evaluated |
| Target Page Analysis Method | How we got the target page info (Website Content, SEO Keywords, or URL Only) |
| Target Page Summaries | The AI-generated summaries of each target page's content |
| Prospect Website Description | AI-written summary of the prospect site |
| Raw Website Content | The scraped text used to analyze the prospect |
| Prospect Description Analysis Method | How we got the prospect info (Website Content, SEO Keywords, or Domain Name Only) |
| Relevance Rating Reasoning | Why the AI gave that relevance score |

### Tab 3: Reject Sites

Every prospect that got removed, along with the reason. Examples:
- "TLD blocked: .ru"
- "Found in Vetting Tracker"
- "Already linking to client"
- "Gray Niche (outbound links to gambling/adult content)"
- "In Persistent Blocklist (known gray niche)"

### Tab 4: Summary

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
| 50 | ~$0.15 | ~$0.01 | ~$0.10 | ~$0.25 |
| 500 | ~$1.50 | ~$0.05 | ~$0.10 | ~$1.65 |
| 3,000 | ~$7.00 | ~$0.30 | ~$0.10 | ~$7.40 |

These estimates assume roughly 25% of prospects pass Stages 1 and 2 (where the expensive per-prospect API calls happen). If most of your prospects get filtered early, the cost goes down. In worst-case scenarios where most prospects survive to Stage 3, costs for a 3,000-prospect run could reach $12–15.

---

## Tips

- **Paste more, not less.** It's cheap to filter out bad prospects, so don't pre-filter your list manually. Let the workflow do it.
- **Use the Link Tracking Sheet field** if the client has one. It prevents you from reaching out to sites that already have a placement.
- **Check the Reject Sites tab** after a run. If you see prospects getting rejected for the wrong reasons, let us know so we can tune the filters.
- **Relevance scores are guidelines, not gospel.** A score of 3 doesn't mean "never contact." It means the AI thinks the connection is loose. Use your judgment.
- **Use Custom Instructions** when the client has specific criteria beyond topical relevance. For example: "Only B2B sites" or "Must have a blog section." The AI will factor these into every relevance score.

---

## Slack Notifications

You'll receive two Slack messages per run:

1. **Start notification** — Confirms the run kicked off, shows the client name, number of prospects, and a link to the Google Sheet. If any options are changed from their defaults (Remove Sites in Vetting Tracker, Exclude Sites Linking to Shady Niches, or Custom Instructions), they'll be shown here.
2. **Completion notification** — Shows how many prospects passed vs. got rejected, with a breakdown by rejection reason. Any non-default options are listed at the bottom under "Options."

---

## Questions?

If something looks wrong or a run fails reach out to Scott.
