# Antar's Bookshop — Autonomous AI System

> Multi-agent AI automation system built for a real bookshop in Saida, Lebanon. Covers intent detection, CRM logging, follow-up sequences, restock alerts, weekly reporting, sentiment tracking, supplier outreach, and autonomous sales re-engagement — zero human intervention required.

---

## Overview

This project is a fully autonomous, production-grade multi-agent AI system built for **Antar's Bookshop** in Saida, Lebanon. It was designed and orchestrated using **n8n**, **Groq API (LLaMA 3.3 70B)**, and **Google Sheets** as a live data layer — with zero budget and zero manual intervention once deployed.

The system handles the entire customer lifecycle — from the moment a customer sends an email, to inventory checking, response generation, follow-up, sentiment analysis, supplier outreach, and autonomous re-engagement of inactive customers.

---

## System Architecture

```
Customer Email
      │
      ▼
┌─────────────────────────────────────────────────┐
│           WORKFLOW 1: Customer Service Agent     │
│                                                  │
│  Gmail → Intent Detector (Groq) → Store Intent  │
│       → Switch →                                 │
│           ├── BOOK_REQUEST → Inventory Filter    │
│           │   → Aggregate → Set → Groq → Reply  │
│           ├── COMPLAINT → Groq → Reply           │
│           └── GENERAL_INQUIRY → Groq → Reply    │
│                    │                             │
│              Merge → CRM Log → Gmail Send        │
│         Also: Restock Alert on unavailable books │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│           WORKFLOW 2: Follow Up Agent            │
│  Schedule (Daily 9am) → CRM → Filter (3d old)   │
│  → Sort → Deduplicate → Inventory → Filter(Yes) │
│  → Aggregate → Set → Groq → Gmail → CRM Update  │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│           WORKFLOW 3: Weekly Report Agent        │
│  Schedule (Sunday 8am) → CRM → Filter (7d)      │
│  → Code (Compile Stats) → Groq → Gmail          │
│  → Reports Log (Google Sheets)                   │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│           WORKFLOW 4: Supplier Outreach Agent    │
│  Schedule (Monday 8am) → Restock Demand Log     │
│  → Code (Deduplicate + High Demand Filter)       │
│  → Set → Groq → Gmail (Supplier + CC Owner)     │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│        WORKFLOW 5: Sentiment Analysis Agent      │
│  Schedule (Daily 10am) → CRM → Filter (24h)     │
│  → Groq (Score each interaction 1-10)           │
│  → Code (Breakdown by intent) → Read History    │
│  → Trend Code → IF (below threshold?)           │
│      ├── Alert Email to Owner                   │
│      └── Log to Sentiment Sheet                 │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│        WORKFLOW 6: Autonomous Sales Agent        │
│  Schedule (Monday 9am) → CRM                    │
│  → Code (Identify 30d+ inactive customers)      │
│  → Set → Inventory → Aggregate → Set            │
│  → Groq (Personalized re-engagement email)      │
│  → Gmail Send                                   │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│        WORKFLOW 7: Orchestrator (Designed)       │
│  Webhook → AI Agent (LLaMA) →                   │
│      Routes to any of the 6 workflows above     │
│      based on natural language instruction       │
└─────────────────────────────────────────────────┘
```

---

## Agents

| Agent | Workflow | Trigger | Function |
|---|---|---|---|
| Intent Detector | Customer Service | Email received | Classifies email into BOOK_REQUEST, COMPLAINT, GENERAL_INQUIRY |
| Inventory Manager | Customer Service | After intent detection | Filters available books, aggregates for Groq |
| Response Generator | Customer Service | After inventory check | Generates personalized response per intent |
| Restock Alert | Customer Service | On unavailable book request | Logs demand, alerts owner, sends restock email |
| Follow Up Agent | Follow Up | Daily 9am | Emails customers 3+ days after BOOK_REQUEST |
| Weekly Report Agent | Weekly Report | Sunday 8am | Analyzes CRM, generates performance report |
| Supplier Outreach | Supplier Outreach | Monday 8am | Identifies high-demand unavailable books, emails supplier |
| Sentiment Analysis | Sentiment | Daily 10am | Scores interactions, tracks trends, alerts on decline |
| Autonomous Sales | Sales | Monday 9am | Re-engages inactive customers with personalized emails |

---

## Tech Stack

| Tool | Role |
|---|---|
| n8n (Cloud) | Workflow automation and agent orchestration |
| Groq API (LLaMA 3.3 70B) | AI language model for all generation and classification tasks |
| Google Sheets | Live inventory database and CRM |
| Gmail | Customer communication and owner alerts |

**Total cost: $0**

---

## Data Layer (Google Sheets)

| Sheet | Purpose |
|---|---|
| Inventory | Book titles, authors, availability status |
| Antar's Bookshop CRM | All customer interactions, intents, AI responses, follow-up status |
| Restock Demand Log | Unavailable books being requested with demand count |
| Sentiment Log | Daily sentiment scores, trends, alert history |
| Antar's Bookshop Reports | Weekly performance report archive |

---

## Key Features

- **Zero human intervention** — entire customer lifecycle automated
- **Intent-based routing** — three distinct pipelines for different customer needs
- **Inventory-aware responses** — AI only recommends available books
- **Deduplication** — no customer receives duplicate follow-ups
- **Sentiment trend tracking** — detects satisfaction decline before it becomes a problem
- **Supplier outreach** — demand-driven procurement emails generated automatically
- **Autonomous re-engagement** — inactive customers contacted with personalized emails
- **Self-logging CRM** — every interaction logged automatically
- **Orchestrator architecture** — single AI brain designed to coordinate all agents via natural language

---

## Workflow Files

| File | Description |
|---|---|
| `customer_service_agent.json` | Main customer-facing workflow with intent detection and routing |
| `follow_up_agent.json` | Daily follow-up workflow for book requests |
| `weekly_report_agent.json` | Weekly performance report generation |
| `supplier_outreach_agent.json` | Automated supplier procurement outreach |
| `sentiment_analysis_agent.json` | Daily sentiment scoring and trend analysis |
| `autonomous_sales_agent.json` | Weekly inactive customer re-engagement |
| `orchestrator.json` | Master AI orchestrator routing to all agents |

---

## Setup Instructions

### Prerequisites
- n8n Cloud account
- Groq API key (free at console.groq.com)
- Google account (for Sheets and Gmail)

### Installation

1. Clone this repository
2. Import each `.json` workflow file into your n8n instance
3. Configure credentials:
   - **Groq:** Add your API key under n8n credentials
   - **Google Sheets:** Connect your Google account
   - **Gmail:** Connect your Gmail account
4. Update Google Sheet IDs in each workflow to point to your own sheets
5. Set your owner email address in alert nodes
6. Activate each workflow

### Google Sheets Setup

Create five sheets with these exact column headers:

**Inventory:** Title, Author, Available

**CRM:** Timestamp, Customer Email, Intent, Customer Request, AI Response, Follow Up Sent

**Restock Demand Log:** Timestamp, Customer Email, Requested Book Topic, Times Requested

**Sentiment Log:** Timestamp, Average Sentiment, Total Analyzed, Alert Sent, Book Request Sentiment, Complaint Sentiment, General Inquiry Sentiment

**Reports:** Timestamp, Total Interactions, Book Requests, Complaints, General Inquiries, Top Request, Report

---

## Case Study

**Client:** Antar's Bookshop, Saida, Lebanon

**Problem:** A small independent bookshop receiving customer inquiries manually — slow response times, no CRM, no follow-up system, no demand visibility, no supplier communication automation.

**Solution:** A fully autonomous 9-agent AI system that handles every stage of the customer lifecycle without human intervention.

**Outcome:**
- Response time reduced from hours to seconds
- Zero manual effort required for customer communication
- Automatic demand tracking for inventory decisions
- Weekly performance visibility for the owner
- Proactive supplier outreach based on real demand data
- Autonomous re-engagement of inactive customers

**Tools:** n8n, Groq API, Google Sheets, Gmail

**Budget:** $0

---

## Author

**Housni Antar**
CS Student — American University of Beirut
AI Systems Design & Agent Orchestration

[LinkedIn](https://www.linkedin.com/in/housni-antar-803418294)

---

*Built in Saida, Lebanon — 2026*

---

## Setting up the Groq key

The workflows no longer contain an API key. Each Groq call reads it from an n8n environment variable named `GROQ_API_KEY`.

1. Create a key in the Groq console.
2. Set `GROQ_API_KEY` in the environment n8n runs in (for Docker, add it to the container's environment; for n8n Cloud, add it under Variables).
3. Import the workflow files and run them as before.

Never paste a key into a workflow file or commit it to this repository.
