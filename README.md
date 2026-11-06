# OnlineJobs.ph Scraper (Python): Philippine VA jobs and wages to JSON and CSV

[![Run on Apify](https://img.shields.io/badge/Run%20on-Apify-00A67E?logo=apify&logoColor=white)](https://apify.com/data_daemon/onlinejobs-ph-scraper)
![Output](https://img.shields.io/badge/output-JSON%20%7C%20CSV%20%7C%20Excel-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

Scrape **OnlineJobs.ph jobs** into JSON or CSV from Python: title, employer, type of work (full time, part time, gig), **wage as typed and parsed into min, max, currency and period**, hours per week, skills and the full description. Search by keyword, skill and work type, or follow the newest jobs on the whole board.

[OnlineJobs.ph](https://www.onlinejobs.ph) is the largest job board for hiring Filipino virtual assistants and remote workers. This repository is a small, tested Python client for the hosted
**[OnlineJobs.ph Scraper | Philippine Remote & VA Jobs with Wages](https://apify.com/data_daemon/onlinejobs-ph-scraper)** on Apify: no browser, proxy, login or scraper maintenance on your side. Price:
**$0.80 per 1,000 jobs, details included, plus $0.001 per run**; Apify's free plan ($5 of credit every month) covers about 6,000 jobs.

## Quick start

```bash
pip install apify-client
export APIFY_TOKEN=your_token      # Apify Console > Settings > Integrations
python scrape.py --keyword "virtual assistant" --max 50
```

`scrape.py` starts the Actor, waits for it and writes `onlinejobs-jobs.json` (every field) and `onlinejobs-jobs.csv`
(the main columns). For any other option, put the Actor input in a file and pass `--input`:

```bash
python scrape.py --input examples/input.json
```

```json
{
  "keywords": [
    "virtual assistant"
  ],
  "maxResults": 5
}
```

## Sample output

A real run of [`examples/input.json`](examples/input.json) on 5 November 2026
([full JSON](examples/sample-output.json), [CSV](examples/sample-output.csv); descriptions shortened):

| title | workType | salaryText | salaryMin | salaryMax | salaryCurrency | salaryPeriod |
|---|---|---|---|---|---|---|
| Lead Generation & Research Virtual Assistant — Travel Indust | Part Time | $4 per hour | 4 | 4 | USD | hour |
| Outbound Sales & Appointment Setting Virtual Assistant | Full Time | Php: 200.00 - 220.00 / Hour | 200 | 220 | PHP | hour |
| Lead Reactivation Specialist | Any | $7hr +bonuses | 7 | 7 | USD | hour |
| Virtual Operations Assistant – GoHighLevel, Project Manageme | Part Time | N/A |  |  |  |  |
| WhatsApp Campaign Specialist Needed: One-Time Project (10,00 | Gig | +1 |  |  |  |  |

## Output fields

| Field | Description |
|---|---|
| `id` | Job id: the number at the end of the job URL (stable). |
| `url` | Public job page on onlinejobs.ph. |
| `title` | Job title as posted. |
| `companyName` | Employer's display name when the job card shows a logo (about 1 in 7 jobs); null otherwise. |
| `companyLogo` | Employer logo URL when shown; null otherwise. |
| `workType` | Type of work: Full Time, Part Time, Gig or Any. |
| `postedAt` | Posting time, ISO 8601 UTC. |
| `postedAtPhilippines` | Posting time in Philippine time (UTC+8), as the site shows it. |
| `dateUpdated` | Date the employer last updated the post (job page), YYYY-MM-DD. |
| `salaryText` | Wage exactly as the employer typed it, e.g. "$4-$8 per hour". |
| `salaryMin` | Lower end of the wage, read from salaryText. |
| `salaryMax` | Upper end of the wage; null when open-ended ("$6+"). |
| `salaryCurrency` | USD, PHP, AUD, CAD, GBP or EUR. |
| `salaryPeriod` | hour, day, week, month, year or task. |
| `salaryInferred` | True when currency or period was not stated and was inferred from the size of the number. |
| `hoursPerWeek` | Hours per week from the job page (number or "TBD"). |
| `skills` | Skill tags of the job. |
| `skillCategories` | OnlineJobs.ph category slugs of the skill tags, e.g. "cms--wordpress". |
| `summary` | Description excerpt from the search page. |
| `description` | Full job description from the job page. |
| `employerId` | OnlineJobs.ph employer account id (groups posts by the same employer; no name or contact). |
| `detailsAvailable` | False when the job page could not be read; listing fields are still filled. |
| `search` | The keyword search that found the job; null for the whole-board feed. |
| `scrapedAt` | When the row was collected, ISO 8601 UTC. |

<!-- agent-recipes:start -->
## Use with AI agents

Load [OnlineJobs.ph jobs](https://apify.com/data_daemon/onlinejobs-ph-scraper) as **cited LangChain documents**, or run a daily only-new feed
with bounded pagination and a USD charge cap. Both [runnable recipes](examples/agents/README.md) include offline fixtures,
failure/resume tests and pinned dependencies; no LLM key is needed.
[India Jobs MCP](https://apify.com/data_daemon/india-jobs-mcp) is the ten-Indian-board alternative for agents
(WTTJ, Gupy and OnlineJobs.ph use their own filtered Apify MCP endpoints).
<!-- agent-recipes:end -->

## Daily feed of only new jobs

```bash
python scrape.py --keyword "virtual assistant" --only-new onlinejobs-daily --max 0
```

With `--only-new <name>` the Actor remembers which jobs it has delivered under that name and returns only new ones;
skipped jobs are not charged. Run it from cron, a GitHub Actions schedule or an Apify schedule:

```yaml
# .github/workflows/onlinejobs-daily.yml
on:
  schedule: [{cron: "30 2 * * *"}]
  workflow_dispatch:
jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install apify-client && python scrape.py --keyword "virtual assistant" --only-new onlinejobs-daily --max 0
        env: {APIFY_TOKEN: "${{ secrets.APIFY_TOKEN }}"}
      - uses: actions/upload-artifact@v4
        with: {name: onlinejobs-jobs, path: "onlinejobs-jobs.*"}
```

## Other ways to call it

**curl**, one request that runs the Actor and returns the rows:

```bash
curl -X POST "https://api.apify.com/v2/acts/data_daemon~onlinejobs-ph-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"keywords": ["virtual assistant"], "maxResults": 5}'
```

**Apify CLI:** `apify call data_daemon/onlinejobs-ph-scraper --input-file examples/input.json`

**No code:** open the [Actor page](https://apify.com/data_daemon/onlinejobs-ph-scraper), fill the form and export to CSV/Excel, or connect it to Google Sheets,
Zapier, Make or n8n. AI agents can call it through the Apify MCP server (`https://mcp.apify.com`).

## FAQ

### Does OnlineJobs.ph have an API?

No public jobs API. This repo calls a hosted Apify Actor that reads the public job listings and job pages.

### How are wages parsed?

Employers type wages freely (`$5/hr`, `PHP 25,000 - 30,000 / month`, `negotiable`). The Actor keeps the original text in `salaryText` and adds `salaryMin`, `salaryMax`, `salaryCurrency` and `salaryPeriod`; `salaryInferred` marks guesses.

### Can I follow every new job each day?

Run with no keyword, `--max 0` and `--only-new my-feed` once a day: each search returns up to 285 of the newest jobs, and only the ones not delivered before are kept and charged.

### Is it legal to scrape OnlineJobs.ph?

The Actor reads only public job listings, without logging in, at a polite request rate, and returns job and company
data, not personal profiles. Check the rules that apply to your use of the data in your jurisdiction.

## Related

- [Career Site Jobs API – Workday, Greenhouse, Lever, Ashby](https://apify.com/data_daemon/career-site-jobs-scraper)
- [Welcome to the Jungle Jobs Scraper | WTTJ API & New Job Feed](https://apify.com/data_daemon/welcome-to-the-jungle-jobs-scraper)
- [Gupy Jobs Scraper | Brazil Jobs with Descriptions (Gupy Portal)](https://apify.com/data_daemon/gupy-jobs-scraper)
- [India Job Alerts](https://github.com/datadmon/india-job-alerts): new jobs from seven Indian boards every morning via GitHub Actions
- [All Daemon Data Actors](https://apify.com/data_daemon)

Unofficial project, not affiliated with OnlineJobs.ph. Issues and questions: open an issue here or on the
[Actor's Issues tab](https://apify.com/data_daemon/onlinejobs-ph-scraper/issues).
