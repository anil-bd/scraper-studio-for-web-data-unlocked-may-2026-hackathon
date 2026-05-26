# A working web scraper in 10 minutes. Without writing one.
### Describe what you want. Bright Data writes and runs the scraper for you.

**[→ For the Bright Data AI Agents + Web Data Hackathon at lablab.ai](https://lablab.ai/ai-hackathons/brightdata-ai-agents-web-data-hackathon)**

**▶ Watch first (2 min each):** [Scraper Studio demo](https://drive.google.com/file/d/1DBJUSPZ_uamz_nY37vNm8SqdEEiR9yPk/view?usp=sharing) · [Self-healing demo](https://drive.google.com/file/d/1gEhb1qzsVgDazlcA7K3ZwJzNjNdCdecB/view?usp=sharing)

---

## Contents

1. [How it works](#how-it-works)
2. [In 10 minutes you get](#in-10-minutes-you-get)
3. [Two ways to build](#two-ways-to-build)
4. [Hackathon track prompts (paste into your coding agent)](#hackathon-track-prompts)
5. [Links](#links)

---

## How it works

Three steps:

1. **You** paste a URL and one sentence of English.
2. **Bright Data's AI** writes the scraper and runs it on managed proxies, browsers, and anti-bot.
3. **You** get a Scraper API endpoint. Pull JSON / NDJSON / CSV from it, or have results delivered to a webhook, S3, GCS, Azure Blob, or Snowflake.

---

## In 10 minutes you get

| 🧠 A working scraper | 🩹 An auto-healing parser | 🛡️ Anti-bot infra, free | 📤 Pipeline-ready output |
|---|---|---|---|
| Plain English in, scraper API out | Site redesign? AI regenerates the parser in one click | Proxies, browsers, CAPTCHA, geo. Zero config. | JSON / NDJSON / CSV. Webhooks, cron, S3 / GCS / Snowflake |

---

## Two ways to build

### Path A · In the Control Panel (no install)

You'd rather click than type.

1. Open [brightdata.com/cp](https://brightdata.com/cp) (free trial, no card).
2. Sidebar: **Scrapers → Scraper Studio**.
3. Enter a URL, hit **Start scraping with AI**. The chat agent guides you: approve the output schema, generate the scraper.
4. Get a **Scraper API endpoint**. Call it from anywhere to pull JSON / NDJSON / CSV (or have results delivered to a webhook, S3, GCS, Azure, Snowflake).

### Path B · From your coding agent (Cursor, Claude Code, Codex, any terminal)

For when the agent IS the user.

```bash
# One-time setup
npm install -g @brightdata/cli
brightdata login                       # opens browser to grab your API key

# Build the scraper (AI generation, 5 to 10 min)
brightdata scraper create https://news.ycombinator.com \
    "Extract top stories: title, url, points, author, comment count"
# → Collector ID: c_mp3tuab31lswoxvpws   (your Scraper API handle, save it)

# Run it (returns JSON, seconds)
brightdata scraper run c_mp3tuab31lswoxvpws https://news.ycombinator.com --pretty
```

**Using Claude Code?** Drop in the [`scraper-studio` skill](https://github.com/brightdata/skills/tree/main/skills/scraper-studio). Then chat: *"build me a Hacker News top-stories scraper"*. The skill teaches Claude the commands.

**Using Cursor or Codex?** The CLI runs in any embedded terminal. Pin the Collector ID in `.cursor/rules` or `CODEX.md` so the agent re-uses it across sessions.

---

## Hackathon track prompts

Three paste-into-your-agent prompts, one per lablab track. Each uses Scraper Studio's strength: **one scraper template, many similar pages on one site**.

| Track | Pattern | Target site | What you ship |
|---|---|---|---|
| **GTM Intelligence** | Discovery + PDP | A competitor's `/customers` page | CSV of "who they're actually selling to" |
| **Finance & Market Intelligence** | Discovery + PDP | `ycombinator.com/companies?batch=W26` | Growth-leader CSV ranked by team size |
| **Security & Compliance** | Discovery + PDP | `fortiguard.com/psirt` (or any vendor PSIRT) | Patch-tickets matched against your fleet |

**Before you paste any of them:**

```bash
# One-time install + auth (browser opens for API key)
npm install -g @brightdata/cli
brightdata login

# Sanity check
bdata --version
bdata zones
```

---

### Track 1: GTM Intelligence · Competitor customer intel

**Use case:** Reverse-engineer who a competitor is selling to. Their `/customers` page is their ideal customer profile, leaked. Build a scraper for one competitor; iterate per competitor with a fresh scraper.

**Target site:** A competitor's case-study index. Public examples: [`linear.app/customers`](https://linear.app/customers), [`notion.com/customers`](https://notion.com/customers), [`stripe.com/customers`](https://stripe.com/customers). Same template across every case study on ONE competitor's site.

**Paste this into your coding agent:**

> Build me a competitor-customer intelligence pipeline using Bright Data Scraper Studio.
>
> Target: `https://linear.app/customers` (swap to your competitor of choice; pick ONE site, not multiple).
>
> 1. Verify `bdata --version` and `bdata zones`. Stop if either fails.
> 2. Build a Discovery scraper: `bdata scraper create https://linear.app/customers` with the description: *"For each customer card on this page, extract company_name, industry_tag, short_summary, case_study_url (the link to the detailed customer story). Return one array element per card."* Save the Collector ID as `DISCOVERY_ID`.
> 3. Build a PDP scraper: `bdata scraper create <pick any single case-study URL>` with the description: *"Extract from this customer case study page: company_name, industry, company_size_if_mentioned, use_case (one-sentence summary of what they use this product for), customer_quote (the main testimonial), results (key metrics quoted, like '40% faster' or '$2M saved'), customer_role_if_quoted, primary_product_used."* Save as `PDP_ID`.
> 4. Run Discovery: `bdata scraper run $DISCOVERY_ID https://linear.app/customers --json | jq -r '.[].case_study_url' > case-studies.txt`
> 5. Batch-run PDP: `bdata scraper run $PDP_ID --input-file case-studies.txt -o customers.json`
> 6. Write 30 lines of Python or Node that loads `customers.json`, groups by `industry`, counts customers per industry, prints the top 5 industries the competitor sells to.
> 7. Suggest how to wire this into a daily monitor (cron + Slack) that alerts when a new competitor case study lands.
>
> Constraints: real data only. Pick ONE competitor for v1; iterate per competitor with a fresh scraper. If the listing page has lazy-load / "view more" pagination, mention that in step 2 and the AI will handle the scroll behavior.

---

### Track 2: Finance & Market Intelligence · YC W26 growth velocity

**Use case:** Hiring + team-size + funding velocity across an entire YC batch as alt-data. Run weekly to surface which startups are scaling fastest, which are stalling. Same scraper works for any future batch (S26, W27, …).

**Target site:** [`ycombinator.com/companies?batch=W26`](https://www.ycombinator.com/companies?batch=W26) (batch listing, ~80 companies) + each company's profile page (`/companies/<slug>`). YC's directory is JS-rendered; Scraper Studio handles that via Bright Data's browser stack, no flag needed.

**Paste this into your coding agent:**

> Build me a YC W26 growth-velocity tracker using Bright Data Scraper Studio.
>
> Targets: `https://www.ycombinator.com/companies?batch=W26` (batch listing) and `https://www.ycombinator.com/companies/<slug>` (per-company profile, same template across all companies).
>
> 1. Verify `bdata --version` and `bdata zones`.
> 2. Discovery scraper on the batch page: `bdata scraper create https://www.ycombinator.com/companies?batch=W26` with description: *"For each company card on this page, extract company_name, one_line_tagline, vertical_tags (array of category badges), profile_url (the link to /companies/<slug>). Return one array element per card. The page is a single grid; no pagination."* Save as `DISCOVERY_ID`.
> 3. PDP scraper on one profile (seed with any YC company): `bdata scraper create https://www.ycombinator.com/companies/anthropic` with description: *"Extract from this YC company profile: company_name, yc_batch (e.g. S21, W26), one_line_tagline, long_description, vertical_tags (array), team_size_number (integer), status (Active / Acquired / Public / Inactive), founded_year, locations (array of city names), founders (array of objects with name and title), launched_products (array of names if shown), website_url."* Save as `PDP_ID`.
> 4. Run Discovery: `bdata scraper run $DISCOVERY_ID https://www.ycombinator.com/companies?batch=W26 --json | jq -r '.[].profile_url' > w26-companies.txt`
> 5. Batch-run PDP: `bdata scraper run $PDP_ID --input-file w26-companies.txt -o w26-profiles.json`. W26 has ~80 companies; the CLI silently auto-falls back to the batch endpoint when needed (that's expected).
> 6. Write 30 lines that loads `w26-profiles.json`, sorts by `team_size_number` descending, prints the top 10 W26 companies by team size, and saves the run as `w26-profiles-$(date +%Y%m%d).json`.
> 7. Suggest how to re-run weekly and diff against the previous week's snapshot to surface companies whose `team_size_number` grew > 25% week-over-week.
>
> Constraints: real data only. YC profile pages are public, no login required. If a specific field is not on the page (e.g. founder LinkedIn URLs), return `null` rather than inventing a value.

---

### Track 3: Security & Compliance · Fortinet PSIRT vulnerability monitor

**Use case:** Daily monitor of Fortinet PSIRT advisories. Match each new advisory's affected products against your fleet inventory. Auto-ticket the patches you actually need to apply. Same pattern works for any vendor PSIRT (Cisco, Microsoft MSRC, VMware, Palo Alto), one scraper per vendor.

**Target site:** [`fortiguard.com/psirt`](https://www.fortiguard.com/psirt). Public, structured table, ~298 advisories across 20 paginated pages. Same template per row. **No clean JSON feed exists**, which is exactly where Scraper Studio earns its place.

**Paste this into your coding agent:**

> Build me a Fortinet PSIRT vulnerability monitor using Bright Data Scraper Studio.
>
> Targets: `https://www.fortiguard.com/psirt` (advisory index, paginated via `?page=1..20`) + per-advisory detail pages.
>
> 1. Verify `bdata --version` and `bdata zones`.
> 2. Discovery scraper on the index: `bdata scraper create https://www.fortiguard.com/psirt` with description: *"For each advisory row in the main table on this page, extract advisory_id (FG-IR-XX-XXX format), cve_ids (array of CVE-YYYY-NNNNN strings; can be multiple), title, description_snippet, affected_products_summary (raw product+versions text), published_date_iso (parse 'May 12, 2026' into '2026-05-12'), severity (Critical / High / Medium / Low / Info), component (CLI / API / GUI / OTHERS), attack_type (Authenticated / Unauthenticated), discovered (Internal / External / Third-Party), advisory_url. Return one array element per row. The page has pagination; only scrape what's rendered on this single URL."* Save as `DISCOVERY_ID`.
> 3. (Optional, for depth) PDP scraper on a single advisory: `bdata scraper create https://www.fortiguard.com/psirt/FG-IR-26-131` (use any advisory ID from step 2) with description: *"Extract from this Fortinet PSIRT advisory: advisory_id, cve_ids (array), title, full_description, severity, cvss_score (if shown), affected_products (array of objects with product_name and affected_versions array), patch_status (e.g. 'Fixed in FortiOS 7.4.5'), published_date_iso, last_updated_iso, references (array of URLs)."* Save as `PDP_ID`.
> 4. Run Discovery across the first 5 pages: build `pages.txt` with `https://www.fortiguard.com/psirt?page=1` through `?page=5`, then `bdata scraper run $DISCOVERY_ID --input-file pages.txt -o fortinet-advisories.json`.
> 5. Write 30 lines that: (a) loads `fortinet-advisories.json`, (b) loads a `fleet.yml` listing your installed Fortinet products + versions (e.g. `FortiOS: [7.4.3, 7.6.1]`, `FortiAnalyzer: [7.6.4]`), (c) matches each advisory against your fleet using fuzzy product-name + version-range match on `affected_products_summary`, (d) prints CVE / severity / required-patch for matches, (e) exits non-zero if any match was published in the last 7 days (so a cron can page you).
> 6. Suggest how to extend with the PDP scraper (step 3) to enrich each match with the exact patched version + remediation steps from the advisory body.
>
> Constraints: security demo only; no live exploits, no PoC code, no exploit details beyond what Fortinet publicly publishes. Monitoring + fleet-matching metadata only.

---

## Links

- **Start free:** [brightdata.com/cp](https://brightdata.com/cp) · **Docs:** [docs.brightdata.com/datasets/scraper-studio/ai-agent](https://docs.brightdata.com/datasets/scraper-studio/ai-agent)
- **CLI source:** [github.com/brightdata/cli](https://github.com/brightdata/cli) · **Claude Code skill:** [github.com/brightdata/skills](https://github.com/brightdata/skills)
- **Sample apps:** [Node.js](https://github.com/brightdata/bright-data-scraper-studio-nodejs-project) · [Python](https://github.com/brightdata/bright-data-scraper-studio-python-project)
- **Hackathon page:** [lablab.ai/ai-hackathons/brightdata-ai-agents-web-data-hackathon](https://lablab.ai/ai-hackathons/brightdata-ai-agents-web-data-hackathon)

---

## License

[MIT](./LICENSE)
