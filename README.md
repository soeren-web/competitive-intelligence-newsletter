Here’s a stronger, “drop-in” `README.md` you can use for this repo (it keeps your intent, but adds setup, architecture, and practical details). It’s written to match what’s currently in the repository: an n8n workflow + Supabase-backed storage + OpenAI Responses API + transactional email. ([GitHub][1])

````md
# Competitive Intelligence Newsletter (n8n + Supabase + LLM)

An automated **competitive intelligence → synthesis → email delivery** pipeline built with **n8n**, **Supabase**, and the **OpenAI Responses API**.

It continuously researches a defined set of competitors, extracts recent public signals, synthesizes them into **short, source-backed HTML briefs**, and sends them as an email update.

> Status: **Working MVP** — opinionated and schema-coupled, intended to be adapted rather than installed as-is.

---

## What it does

For each company in your competitor list, the workflow:

- Loads competitors from Supabase
- Runs structured, web-backed research (freshness-first)
- Generates concise **HTML snippets** (email-native, scannable)
- Stores per-company snippets for traceability and reuse
- Merges snippets into a single customer-specific brief
- Sends the brief via a transactional email provider

Key principles:

- **Evidence-only:** every bullet is backed by at least one URL
- **Freshness-first:** focuses on the last ~14 days (optionally up to 30)
- **Material changes only:** avoids fluff and undated hype
- **Email-native:** outputs minimal inline-safe HTML

---

## Repository contents

- `n8n workflow.json` — the full n8n workflow export (importable into n8n)
- `Supabase data schema.json` — schema / supporting logic (see notes below)
- `README.md`

> Note: The workflow JSON contains some hard-coded dates/values you’ll want to parameterize before production use. :contentReference[oaicite:1]{index=1}

---

## High-level architecture

```text
Supabase (company list)
        |
        v
n8n: "Get all companies"
        |
        v
OpenAI Responses API (+ web search tool)
        |
        v
Supabase: store per-company snippet (brief_company_snippet)
        |
        v
Supabase RPC: merge snippets per customer/date
        |
        v
Supabase: store full brief (brief_full_customer)
        |
        v
Transactional email (e.g., Brevo/Sendinblue node)
````

---

## Data model (minimum expected)

The workflow references these entities (names are taken directly from the n8n nodes):

### Tables

* `company`

  * `id` (UUID / int)
  * `name` (text)

* `brief_company_snippet`

  * `company_id` (FK → company.id)
  * `date` (date)
  * `content` (text; HTML)

* `brief_full_customer`

  * `customer_id` (UUID / int)
  * `date` (date)
  * `content` (text; HTML)

### RPC functions (Supabase)

The workflow calls RPC endpoints that typically map to Postgres functions:

* `get_distinct_customer_ids`
* `get_merged_snippets_for_customer_and_date(p_customer_id, p_day)`
* `get_customer_briefs_on_date(p_day)`

Your implementation can vary, but `get_customer_briefs_on_date` must return at least:

* `customer_email`
* `customer_brief` (merged HTML)

> Tip: Keep the merge logic in SQL so n8n stays thin and you can reuse outputs elsewhere.

---

## Prerequisites

* An n8n instance (self-hosted or n8n cloud)
* A Supabase project (Postgres + REST/RPC enabled)
* OpenAI API access (Responses API)
* A transactional email provider supported by n8n (the included workflow uses a Brevo/Sendinblue node)

---

## Setup

### 1) Supabase: create schema + RPCs

1. Create the tables listed in **Data model** (or adapt to your needs).
2. Implement the RPC functions used by the workflow.
3. Make sure your n8n Supabase credential has permissions to:

   * read from `company`
   * insert into `brief_company_snippet` and `brief_full_customer`
   * call the RPC functions

### 2) n8n: import the workflow

1. In n8n, go to **Workflows → Import from File**
2. Import `n8n workflow.json` from this repo. ([GitHub][1])
3. Replace credentials on these nodes:

   * **Supabase** nodes / HTTP calls to Supabase RPC
   * **OpenAI** (Responses API)
   * **Transactional email** provider

### 3) Configure OpenAI prompting + research constraints

The workflow prompt is designed for:

* short, high-signal bullets
* strict URL evidence per bullet
* a “Sources” section with titles + dates
* English-language output

Customize:

* target audience (e.g., PMs, founders, analysts)
* geography (e.g., DACH/EU)
* cadence (weekly/bi-weekly)
* source scope (press, changelogs, GitHub, socials, etc.)

### 4) Fix hard-coded dates and make it production-ready

In the provided export, some nodes use a fixed date (example: `2025-09-05`). You should:

* replace it with “today” (or the scheduled run date)
* store the run date consistently across snippet + merged brief
* optionally store `previous_brief` and feed it back into the prompt to reduce repetition

---

## Running

### Manual run (MVP)

* Use the **Manual Trigger** node to test end-to-end.
* Verify in Supabase that:

  * per-company snippets were created
  * merged customer briefs were created
* Confirm email delivery for a test customer.

### Scheduled run (recommended)

* Add a **Cron** trigger node in n8n (weekly or bi-weekly).
* Consider rate limits and cost:

  * number of companies × number of customers × research depth
  * add throttling, retries, and caching if needed

---

## Output format (HTML)

The LLM returns **HTML only** so you can paste it directly into an email template. Typical sections:

* Company name header
* 1-sentence verdict / takeaway
* 3–6 bullets with URLs as evidence
* Optional “ongoing developments”
* A “Sources” list with titles + dates

---

## Customization ideas

* **Segment customers** (different competitor sets per customer)
* **Add scoring** (e.g., “impact: low/med/high” per bullet)
* **Track deltas** (diff against last brief to avoid repetition)
* **Add Slack/Teams delivery** in addition to email
* **Add enrichment** (funding DBs, hiring velocity, GitHub activity)

---

## Security + compliance notes

* Don’t email secrets, internal docs, or non-public data.
* Treat competitor names + customer emails as sensitive (access control in Supabase).
* Log sources and store generated outputs for auditability.
* If you run this commercially, validate Terms of Service for each data source you crawl/summarize.
