# BillFlow

A contract intelligence tool for B2B billing agreements. Uses RAG to query contracts and has some experimental features around revenue optimization.

## What it does

I built this to solve a real problem: when you have 20+ billing contracts with different terms, SLAs, and pricing models, it becomes impossible to keep track of what's actually in them. This tool lets you:

1. **Ask questions about your contracts** - "What's the SLA for Sky Digital?" or "Which clients are on revenue share?"
2. **See risk across your portfolio** - Which contracts have compliance gaps? Who's about to renew?
3. **Find revenue issues** - Are you charging below market rate? Missing volume discounts?
4. **Compare contracts** - Side-by-side diff of any two agreements

## Getting started

You'll need Azure OpenAI credentials (or modify the code for regular OpenAI).

```bash
# Setup
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Create .env with your Azure keys
cp .env.example .env

# Generate sample contracts (or add your own to /data)
python generate_contracts.py

# Build the vector store
python ingestion.py

# Run the API
uvicorn backend_api:app --reload --port 8001
```

For the frontend:
```bash
cd frontend
npm install
npm run dev
```

Then open http://localhost:5173

## Configuration

Create a `.env` file:

```
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_KEY=your-key-here
AZURE_OPENAI_DEPLOYMENT=gpt-4
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=text-embedding-3-small
AZURE_OPENAI_API_VERSION=2024-02-15-preview
```

## Project structure

```
├── backend_api.py          # FastAPI server
├── rag_chat.py             # RAG chain for Q&A
├── contract_intelligence.py # Risk scoring, churn prediction, etc.
├── revenue_intelligence.py  # Leakage detection, opportunity finding
├── ingestion.py            # PDF/TXT → ChromaDB
├── config.py               # Settings
├── data/                   # Contract files (TXT or PDF)
└── frontend/               # React app
```

## The four views

**Dashboard** - Portfolio overview with key metrics

**Chat** - Ask anything about your contracts. Uses RAG with MMR retrieval to pull relevant chunks from multiple documents.

**Intelligence** - Risk scores, churn predictions, what-if scenarios, contract comparison, and a contract generator (describe what you want in plain English).

**Revenue** - This is the experimental part. Tries to find:
- Revenue leakage (pricing below market, uncollected fees, etc.)
- Opportunities (tier upgrades, upsells, term extensions)
- Signals (renewal risk, engagement drops)
- Prioritized actions with AI-generated outreach scripts

## API endpoints

The main ones:

| Endpoint | What it does |
|----------|--------------|
| `POST /chat` | Ask a question about contracts |
| `GET /contracts` | List all contracts |
| `GET /metrics` | Portfolio summary |
| `GET /intelligence/risk` | Risk analysis |
| `GET /intelligence/churn` | Churn predictions |
| `POST /intelligence/simulate` | What-if scenarios |
| `GET /revenue/command-center` | Full revenue intelligence data |
| `POST /revenue/generate-outreach` | AI writes an email/call script |

## How the RAG works

1. Contracts (PDF or TXT) get chunked into ~1500 char pieces
2. Each chunk is embedded using Azure's text-embedding-3-small
3. Stored in ChromaDB locally
4. When you ask a question, we retrieve top-60 chunks using MMR (for diversity)
5. LLM generates answer with sources

The high k=60 is intentional - comparison questions like "which client has the best SLA" need to see multiple contracts.

## Known limitations

- Revenue intelligence uses simulated data for some signals (payment history, engagement metrics) since we don't have real integrations yet
- Contract generation is basic - it parses your description and fills a template
- No auth - this is a demo/prototype

## Tech

- FastAPI for the backend
- React + Tailwind for frontend
- LangChain for RAG
- ChromaDB for vectors
- Azure OpenAI (GPT-4 + embeddings)

## Ideas for later

- Salesforce/HubSpot sync
- Slack bot for quick queries
- Email integration to send outreach directly
- Real payment/usage data ingestion
- Mobile app for alerts

---

Built as a proof of concept for AI-powered contract intelligence. The revenue optimization stuff is experimental but shows what's possible when you combine RAG with domain-specific analysis.
