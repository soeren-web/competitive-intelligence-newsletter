# Automated Competitive Research Pipeline (n8n)

This project is an automated **competitive research → synthesis → email delivery** pipeline built with **n8n**, **Supabase**, and **LLMs**.

It continuously researches a defined set of competitors, extracts recent public signals, synthesizes them into short, source-backed HTML briefs, and sends them as an email update.

The complete n8n workflow definition is included in this repository. :contentReference[oaicite:0]{index=0}

---

## What It Does

- Loads competitors from a database  
- Runs structured, web-backed competitive research  
- Generates concise, evidence-bound HTML snippets  
- Stores results for traceability and reuse  
- Merges snippets into a single email brief  
- Sends the brief via transactional email  

---

## Design Principles

- **Evidence-only** (no uncited claims)  
- **Freshness-first** (last 7–30 days)  
- **Material changes only**  
- **Email-native output**  
- **Automation-first, minimal ops**  

---

## Stack

- **n8n** – workflow orchestration  
- **Supabase** – storage and merge logic  
- **OpenAI Responses API** – research & synthesis  
- **Transactional email provider** – delivery  

---

## Status

Working MVP.  
Opinionated, schema-coupled, meant to be adapted—not installed as-is.
