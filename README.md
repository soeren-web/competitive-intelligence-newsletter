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
