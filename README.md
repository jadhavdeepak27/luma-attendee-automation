# Luma Attendee Automation POC

This repository contains a proof-of-concept Python workflow for scraping Luma event attendees, enriching profiles, pushing to CRM, and triggering personalized outreach.

## Workflow
1. Scrape attendee list (Python + Selenium)
2. Enrich with LinkedIn, Twitter, and email (Clearbit / Apollo / PhantomBuster)
3. Push into CRM (HubSpot / Salesforce)
4. Trigger personalized outreach (SendGrid / LinkedIn DM)
5. Optional feedback loop to optimize engagement

## Tech Stack
- Scraping: Python + Selenium / Puppeteer
- Enrichment: Clearbit, Apollo, PhantomBuster, Twitter API
- CRM: HubSpot / Salesforce
- Outreach: SendGrid, Zapier, Make
- Storage: JSON/CSV or SQL/Airtable

## Usage
- Configure API keys in `config.yaml`
- Run `python luma_attendee_poc.py` to test POC
