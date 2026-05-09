# AI Document Analysis Agent (Vendor Risk Agent)

An AI-powered document analysis agent for vendor onboarding and third-party risk management. Built with n8n, Claude API, and Google Sheets.

---

## What it does

This agent automates the first-pass review of vendor onboarding document packets. It reads every document submitted by a vendor, classifies each document type, extracts structured data, flags risks and anomalies and produces an actionable output in Google Sheets without any manual review.

A compliance team submits a vendor packet through a form. The agent handles everything from that point of classification, extraction, risk flagging, cross-document conflict detection and escalation alerts.

---

## Key capabilities

- Classifies 6 document types: vendor registration, compliance certificate, contract, financial statement, data privacy agreement, insurance certificate
- Extracts structured fields specific to each document type
- Flags critical risks: expiring certificates, missing contract clauses, financial anomalies, regulatory gaps, data residency conflicts
- Detects conflicts across documents in the same packet
- Routes critical flags to Gmail alerts automatically
- Writes structured output to Google Sheets for compliance team review
- Handles PDF, CSV, XLSX, DOCX, PNG, and JPEG file formats

---

## Architecture

![Workflow Architecture](docs/architecture_diagram.png)

The workflow has six phases:

1. **Ingestion** - Form trigger captures vendor packet with metadata
2. **Preprocessing** - Files split by format and routed to correct extractor
3. **Intelligence** - Claude API classifies, extracts, and flags each document
4. **Flag Engine** - Deterministic rules layer normalises and scores all flags
5. **Output Routing** - Critical flags trigger Gmail alerts, all rows written to Sheets
6. **Cross-Document Validation** - Full packet compared for conflicts across documents

---

## Tech stack

| Tool | Purpose |
|---|---|
| n8n (self-hosted) | Workflow orchestration |
| Claude Sonnet (claude-sonnet-4-20250514) | Document intelligence |
| Google Sheets | Structured output layer |
| Gmail | Critical flag alerts |

---

## Sample output

![Google Sheets Output](docs/sample_output.png)

The agent produces two output sheets:

**Document Analysis** - one row per document with document type, extracted fields, flags, severity, urgency score, and recommended action.

**Packet Summary** - one row per vendor packet with cross-document conflict detection results, missing document types, and overall packet recommended action.

---

## Sample documents

The `sample_documents` folder contains a realistic 7-document vendor packet for a fictional IT services vendor - Nexaflow Technologies Pvt. Ltd. Each document is designed to test a different agent capability:

| Document | Type | Seeded flag |
|---|---|---|
| DOC01 | Vendor Registration | None — clean baseline |
| DOC02 | ISO 27001 Certificate | Expiry in 14 days |
| DOC03 | Master Services Agreement | Missing liability cap and indemnification |
| DOC04 | Financial Statement | 46% revenue drop, qualified audit, going concern |
| DOC05 | Data Privacy Agreement | Sub-processors unlisted, data residency conflict |
| DOC06 | Vendor Registration | Entity name conflict with DOC01 |
| DOC07 | Insurance Certificate | Image file, low confidence fields |

---

## Setup instructions

### Prerequisites

- n8n instance (self-hosted or cloud)
- Anthropic API key
- Google Cloud project with Sheets API and Gmail API enabled
- Google Service Account with Editor access to your spreadsheet

### Environment variables

Copy `.env.example` and fill in your values:

ANTHROPIC_API_KEY=your_key_here
GOOGLE_SERVICE_ACCOUNT_EMAIL=your_service_account@project.iam.gserviceaccount.com
N8N_WEBHOOK_URL=https://your-n8n-domain.com

### Import the workflow

1. Open your n8n instance
2. Go to Workflows and click Import
3. Upload `workflow/vendor_risk_agent.json`
4. Add your credentials for Claude HTTP Request, Google Sheets, and Gmail nodes
5. Activate the workflow

### Google Sheets setup

Create a Google Spreadsheet with two sheets:
- Sheet 1 named `Document Analysis`
- Sheet 2 named `Packet Summary`

---

## Prompts

The Claude prompts are in the `prompts` folder:

- `system_prompt.md` - Main extraction prompt covering all 6 document types, 
  extraction schemas, and flagging rules
- `cross_doc_validation_prompt.md` - Cross-document conflict detection prompt

---

## Known limitations

- Image files are flagged for manual review - OCR integration planned for v2
- Fixed flag thresholds - configurable per organisation in v2
- Single document type per file - multi-type document splitting in v2
- Feedback loop is structural but not yet automated - v2 adds prompt refinement from reviewer corrections

---

## Built by

Shijin - Product Manager with background in EdTech and AI products.
Built as part of a technical assignment exploring agentic AI workflows for third-party risk management.

---
