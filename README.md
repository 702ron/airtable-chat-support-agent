# AI Support Agent — Airtable Chat

![n8n](https://img.shields.io/badge/n8n-workflow-orange) ![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4-blue) ![Airtable](https://img.shields.io/badge/Airtable-database-green) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-memory-purple) ![Open WebUI](https://img.shields.io/badge/Open%20WebUI-frontend-teal)

A production-style AI support agent that turns natural-language questions into structured Airtable lookups. Supports multi-table queries (Inventory, Sales Transactions, Returns), barcode image extraction, and session-aware chat via PostgreSQL memory.

## What I Built

- **AI Agent Logic**: Designed the system prompt and tool-selection strategy for querying multiple Airtable tables
- **n8n Workflow**: Built the complete orchestration pipeline from webhook to response
- **Open WebUI Integration**: Created a custom pipe function to connect Open WebUI to the n8n backend
- **Barcode Processing**: Implemented image-to-lot-number extraction using GPT-4 Vision
- **Session Management**: Configured PostgreSQL-backed chat memory for conversation continuity

## Demo

- **Video (60–90s):** _[Add your Loom/YouTube link here]_
- **Screenshots:** see [`/screenshots`](./screenshots)

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Open WebUI    │────▶│   n8n Workflow   │────▶│    Airtable     │
│  (Chat Frontend)│     │   (AI Agent)     │     │  (Data Source)  │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                       │                        │
        │                       ▼                        │
        │               ┌──────────────┐                 │
        │               │  PostgreSQL  │                 │
        │               │   (Memory)   │                 │
        │               └──────────────┘                 │
        │                       │
        ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                        OpenAI GPT-4                             │
│  • Query interpretation  • Tool selection  • Barcode reading    │
└─────────────────────────────────────────────────────────────────┘
```

### Flow

1. **Webhook (POST)** receives `{ sessionId, chatInput }` (optionally an image)
2. **Image path (optional)**: extract base64 → GPT-4 Vision barcode read → rewrite query
3. **AI Agent** interprets the question and chooses the correct Airtable tool + filters
4. **Airtable queries** run across Inventory / Transactions / Returns
5. **Formatter** returns a clean `output` field
6. **Response** sent back to the UI

## Frontend Options

This project supports two frontend approaches:

### Option 1: Open WebUI (Recommended)

[Open WebUI](https://github.com/open-webui/open-webui) is an open-source ChatGPT-style interface. This project includes a **custom pipe function** that routes messages to the n8n workflow.

**Setup:**
1. Install Open WebUI (Docker or local)
2. Go to Admin Panel → Functions → Add Function
3. Paste the contents of [`n8n_pipe_openWebUI.py`](./n8n_pipe_openWebUI.py)
4. Configure the Valves:
   - `n8n_url`: Your webhook URL
   - `n8n_bearer_token`: Your auth token
5. Enable the pipe for your workspace

### Option 2: Standalone API

The n8n webhook works as a standalone REST API. Use the included demo UI or integrate with any frontend.

**Files:**
- [`demo-ui/index.html`](./demo-ui/index.html) — Simple static chat interface
- [`postman_collection.json`](./postman_collection.json) — Ready-to-import API examples

## Repository Structure

```
airtable-chat-support-agent/
├── README.md
├── n8n-workflow-export.json      # n8n workflow (credentials redacted)
├── n8n_pipe_openWebUI.py         # Custom pipe for Open WebUI integration
├── postman_collection.json       # API testing collection
├── demo-ui/
│   └── index.html                # Optional static chat UI
├── data/                         # Mock datasets (safe to publish)
│   ├── inventory.csv
│   ├── transactions.csv
│   └── returns.csv
└── screenshots/
    ├── chat_ui.png
    ├── n8n_workflow.png
    ├── airtable_inventory.png
    ├── execution_view.png
    └── result_output.png
```

## Tech Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| Orchestration | n8n | Workflow automation, webhook handling |
| LLM | OpenAI GPT-4 | Query interpretation, tool selection |
| Vision | GPT-4 Vision | Barcode image recognition |
| Database | Airtable | Business data (Inventory, Sales, Returns) |
| Memory | PostgreSQL (Supabase) | Session-based chat history |
| Frontend | Open WebUI / Static HTML | User interface |

## Example Request

```bash
curl -X POST "https://your-n8n-instance/webhook/your-webhook-id" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "demo-123",
    "chatInput": "What are the sales totals for October by employee?"
  }'
```

## Local Setup

1. **Import workflow**: Load `n8n-workflow-export.json` into your n8n instance
2. **Configure credentials**: Set up Airtable, OpenAI, and PostgreSQL credentials
3. **Import mock data**: Use `/data/*.csv` files to populate a test Airtable base
4. **Set webhook auth**: Configure the bearer token in the webhook node
5. **Test**: Use Postman collection or demo UI

## Security Notes

- Never commit live API keys, bearer tokens, or real PII
- This repo uses mock data only
- Review `n8n-workflow-export.json` for any credentials before committing

## License

MIT
