<p align="center"><a href="https://apify.com/DevilScrapes/bamboohr-jobs-scraper"><img src="assets/banner.svg" alt="BambooHR Jobs Scraper — JavaScript example" width="100%"></a></p>

# BambooHR Jobs Scraper for Node.js: careers-page jobs for widgets, boards and APIs

<p align="center">
<a href="https://apify.com/DevilScrapes/bamboohr-jobs-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/bamboohr-jobs-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/bamboohr-jobs-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

Use this Node.js example to put BambooHR job listings inside your own software. The **[BambooHR Jobs Scraper](https://apify.com/DevilScrapes/bamboohr-jobs-scraper)** on Apify reads the public careers data of any employer whose openings live at `<company>.bamboohr.com/careers`, and this ES module runs it through the official `apify-client` package. You get back an array of job objects, each with a numeric id, a public job URL, department, employment type and location fields, plus the full description as HTML and as plain text when you enable it. That is enough to render a careers widget, sync a job board or index roles for search, from a backend, a worker or a serverless function.

> **Actor:** [BambooHR Jobs Scraper](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [bamboohr-jobs-scraper-python](https://github.com/devilscrapes/bamboohr-jobs-scraper-python)

## 🔥 What you get

- **Two depths of data**: the careers list supplies title, department, employment status, city, state and the remote flag; switch on `fetchFullDescription` and each posting also gains its status, country, posting date and full body.
- **Descriptions in two shapes**: `description_html` keeps the employer's markup for display, while `description_text` carries the same body with tags removed and whitespace collapsed, ready for search or keyword counts.
- **The cap also limits detail requests**: `maxJobsPerCompany` trims each company's list before descriptions are fetched, so a small cap keeps both the row count and the run time down.
- **Remote is three-valued**: `is_remote` is `true`, `false` or `null`, and `null` means the employer never said, which is not the same as on-site.
- **Input checked before any request**: bare subdomains and `*.bamboohr.com` URLs are accepted; a custom-domain URL or a malformed entry stops the run with an error that names the entry.
- **Unclaimed subdomains are not errors**: a subdomain that BambooHR redirects away from, or a company with no openings, simply yields zero rows.
- **Partial failures degrade, they do not drop rows**: temporary errors are retried with backoff, and if one posting's detail request still fails, the row is kept with its list fields and the detail columns left `null`.

## 🎯 Use cases

### Embed a client's openings on another site

Agencies that build websites for small companies can fetch the client's BambooHR roles on a schedule and render them in the site's own design, using `description_html` for the body and `job_url` for the apply button.

### Aggregate many small employers into one board

A regional or industry job board can store BambooHR subdomains in its database, run them as one batch and upsert rows on `company` plus `job_id`. Rows missing from the newest run can be marked closed.

### Index roles for full-text search

`description_text` arrives free of markup, so it can go into Elasticsearch, Meilisearch or a Postgres `tsvector` column without an HTML cleaning step in your pipeline.

### Ping a channel when a target account hires

Sales and partnership teams can run a short account list nightly from a cron container and post new job ids to a webhook, department attached, so the right owner sees the signal.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/bamboohr-jobs-scraper-javascript.git
cd bamboohr-jobs-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [BambooHR Jobs Scraper](https://apify.com/DevilScrapes/bamboohr-jobs-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Below is the minimal version of what `index.js` does. Await `call()` to get the finished run, then read its rows with `listItems()` on the default dataset.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/bamboohr-jobs-scraper').call({"companies": ["adobe", "nycballet"], "maxJobsPerCompany": 5, "fetchFullDescription": true, "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

As shipped, `input.json` asks for five postings each from the `adobe` and `nycballet` subdomains with descriptions on, ten rows at most. In an app you would build the object from your own data: subdomains from a table of employers, `maxJobsPerCompany` sized to what you display, and `fetchFullDescription` off for a listing view that shows only titles.

```json
{
  "companies": [
    "adobe",
    "nycballet"
  ],
  "maxJobsPerCompany": 5,
  "fetchFullDescription": true,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `companies` | array | yes | `["adobe", "nycballet"]` | — |
| `maxJobsPerCompany` | integer | no | `100` | — |
| `fetchFullDescription` | boolean | no | `true` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

Every item is a flat object. `job_id` is a number rather than a string and repeats across companies, so build composite keys such as `adobe:15`. `is_remote` can be `null`; render that as unknown, not as an office job.

| Field | Type | Always present |
|---|---|---|
| `company` | string | yes |
| `job_id` | integer | yes |
| `job_title` | string | yes |
| `job_url` | string | yes |
| `department` | string or null | no |
| `employment_type` | string or null | no |
| `job_status` | string or null | no |
| `location_city` | string or null | no |
| `location_state` | string or null | no |
| `location_country` | string or null | no |
| `is_remote` | boolean or null | no |
| `date_posted` | string or null | no |
| `description_html` | string or null | no |
| `description_text` | string or null | no |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.0015 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

This script converts the saved results into a compact `jobs-feed.json` that a front end can fetch: entries grouped by department, each with a 200-character teaser cut from `description_text`. It needs only Node's built-in modules.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const rows = JSON.parse(await readFile('results.json', 'utf8'));
const remoteLabel = { true: 'Remote', false: 'On-site' };

const feed = {};
for (const row of rows) {
    const place = [row.location_city, row.location_state, row.location_country].filter(Boolean).join(', ');
    const text = row.description_text ?? '';
    const department = row.department ?? 'Other';
    (feed[department] ??= []).push({
        id: `${row.company}:${row.job_id}`,
        title: row.job_title,
        type: row.employment_type,
        place: place || 'Location not listed',
        remote: remoteLabel[row.is_remote] ?? 'Not stated',
        posted: row.date_posted,
        teaser: text.length > 200 ? `${text.slice(0, 197)}...` : text,
        url: row.job_url,
    });
}

await writeFile('jobs-feed.json', JSON.stringify(feed, null, 2));
console.log(Object.entries(feed).map(([dept, jobs]) => `${dept}: ${jobs.length}`).join('\n'));
```

## ❓ FAQ

### Should I enable `fetchFullDescription` in production?

Only where you display or search the body. It adds one request per posting, which lengthens the run, while the per-row price stays the same. A title-only listing view can leave it off.

### How do I know whether some companies failed?

Read the run's status message. It reports how many companies returned data, in the form `Done — 12 job(s) across 2/3 companies.` The run still finishes as succeeded when individual companies fail, so check that line in your monitoring.

### Will this fit inside a Lambda or Vercel function?

It will when the platform's time limit is longer than the Actor run. For long lists, start the Actor with `start()`, return at once, and let an Apify webhook call your endpoint when the dataset is ready.

### Does it work for companies on a custom careers domain?

Only through their `bamboohr.com` subdomain. A custom-domain URL makes input validation fail before any request goes out, which keeps a typo from turning into an empty run that looks successful.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [BambooHR Jobs Scraper on Apify Store](https://apify.com/DevilScrapes/bamboohr-jobs-scraper)
- [Python version of this example](https://github.com/devilscrapes/bamboohr-jobs-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Workable Jobs Scraper](https://apify.com/DevilScrapes/workable-jobs-scraper) — the equivalent board export for employers that recruit through Workable.
- [Personio Jobs Scraper](https://apify.com/DevilScrapes/personio-jobs-scraper) — Personio career pages, read from Personio's public XML feed.
- [Teamtailor Jobs Scraper](https://apify.com/DevilScrapes/teamtailor-jobs-scraper) — career sites built on Teamtailor, with publish dates and descriptions.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/bamboohr-jobs-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with BambooHR.
