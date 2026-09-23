# AI-Powered Sales Lead Qualification & Personalized Outreach

An end-to-end n8n workflow that automates sales lead research and outreach email generation using web scraping, LLMs, and Retrieval-Augmented Generation (RAG).

## Business Problem

Sales reps spend 15–20 minutes per lead manually researching a company and writing a personalized cold email before they can even qualify it. This slows down outreach, wastes selling time on unqualified leads, and leads to inconsistent email quality across a team.

## What This Workflow Does

Given a list of leads, the workflow automatically:

1. **Reads** leads from a Google Sheet (company name, website, industry)
2. **Scrapes** each company's live website for real content
3. **Summarizes** the company and identifies its likely business pain point using an LLM (Groq — Llama 3.1)
4. **Retrieves** the most relevant product case study from a small RAG knowledge base, based on that pain point (embeddings via Google Gemini `text-embedding-004`, stored in an in-memory vector store)
5. **Generates** a short, personalized cold outreach email grounded in the retrieved case study
6. **Writes** the score, reasoning, and email draft back into the Google Sheet — ready to send

The result: research-and-draft time drops from ~15–20 minutes to under a minute per lead.

## Architecture

```
Google Sheets (leads) 
    → HTTP Request (scrape website, with error-handling fallback)
    → HTML extraction + text trimming
    → Merge (reunite success/fallback branches)
    → Batched loop (rate-limit safe)
        → LLM: company summary + pain point
        → RAG retrieval: best-matching case study from vector store
        → LLM: personalized email generation
        → Google Sheets: write results back
```

## Data Sources

- **Leads dataset**: [Y Combinator Companies dataset](https://www.kaggle.com/) (Kaggle), filtered to 24 real companies with live websites across 9 industries.
- **RAG knowledge base**: 6 short, self-authored product case studies for a fictional workflow-automation product ("Flowbase"), covering distinct business pain points (operations scaling, data silos, developer productivity, compliance, customer retention, supply chain).

## Tools Used

- **n8n** — workflow orchestration
- **Google Sheets** — lead input and output tracker
- **Groq (Llama 3.1)** — chat/completion LLM
- **Google Gemini (`text-embedding-004`)** — embeddings for RAG
- **n8n Simple Vector Store** — in-memory semantic search

## Repository Contents

| File | Description |
|---|---|
| `workflow.json` | Exported n8n workflow — import this to run the pipeline |
| `leads_dataset.csv` | Cleaned leads dataset (24 companies, sourced from Kaggle YC dataset) |
| `case_studies.md` | The 6 RAG knowledge base documents used for retrieval |
| `presentation.pptx` | Project presentation slides |

## How to Reproduce

1. Import `workflow.json` into n8n (Cloud or self-hosted)
2. Connect your own Google Sheets credential and point it to a sheet with columns: `Company Name, Website, Industry, YC Description, Score, Reasoning, Email Draft, Status`
3. Upload `leads_dataset.csv` into that sheet
4. Add your own Groq and Google Gemini API keys as credentials in n8n
5. Run the workflow — results populate directly in the sheet

## Limitations & Future Improvements

- Some older company websites (2015–2016 startups) are unreachable; the workflow falls back to a backup description field in these cases
- The RAG knowledge base is a small demo set (6 case studies) rather than a full sales-enablement library
- Could be extended to pull leads from a live CRM instead of a static sheet, and send hot-lead alerts via Slack

## Author

Built as part of an Agentic AI coursework project.
