# Cold-Chain Logistics FDE Project

A production-style AI-powered cold-chain operations assistant that allows dispatch teams to query fleet telemetry, inspect live corridor conditions, and retrieve compliance guidance from SOP documents using a LangGraph-powered reasoning flow.

This repository demonstrates an end-to-end architecture for operational decision support in cold-chain logistics, combining:

- Microsoft SQL Server telemetry access
- Streamlit-based user interface
- LangGraph orchestration for tool-using agent workflows
- Pinecone vector retrieval for SOP documents
- Weather/corridor lookup via Open-Meteo API
- Audit logging and secure access patterns for enterprise-style governance

## Project Goal

The goal of the project is to enable business stakeholders to "chat with their data" and get operationally useful answers about:

- active shipments and temperature anomalies
- weather or congestion risks in specific corridor locations
- cold-chain compliance requirements from SOP documents
- recommended escalations and response actions

The system is designed for scenarios like:

- fresh produce or perishable cargo moving through high-risk transit corridors
- cold-chain drift events requiring fast intervention
- operational questions from dispatch or logistics teams
- policy-grounded decision recommendations tied to internal SOPs

## Architecture Overview

```text
User Query
   ↓
Streamlit UI (src/ui.py)
   ↓
LangGraph Agent (src/orchestrator.py)
   ├── query_telemetry_db() → SQL telemetry view
   ├── fetch_corridor_conditions() → Open-Meteo API
   └── search_compliance_sop() → Pinecone vector retrieval
   ↓
Actionable response with SOP-grounded recommendations
```

## Key Capabilities

- Query active fleet telemetry from a secure SQL view
- Check local weather and corridor conditions for a route or location
- Search SOP documents using semantic vector retrieval
- Execute tool-based reasoning in a structured LangGraph workflow
- Show the user the raw tool inputs/outputs in the Streamlit UI
- Log execution traces in a SQL audit table
- Run in local development mode with fallback LLMs or cloud LLMs

## Repository Structure

```text
Cold-Chain-Logistics-FDE-Project/
├── .github/
│   └── workflows/
├── .streamlit/
│   └── config.toml
├── Misc/
│   └── Materials/
│       ├── FDE-YT-Project-Business-Presentation.pdf
│       └── Technical Design Document (TDD)_ Cold-Chain Logistics AI-Assistant.pdf
├── data/
│   ├── cache/
│   │   └── ingestion_hash_cache.json
│   ├── policy/
│   │   └── Cold_Chain_Incident_SOP_v2.md
│   ├── raw/
│   │   └── dynamic_supply_chain_logistics_dataset.csv
│   └── source/
│       └── data.txt
├── docs/
│   └── instrutions.md
├── scripts/
│   ├── ingest_legacy_data.py
│   ├── ingest_sop_pinecone.py
│   └── setup_security_and_view.sql
├── src/
│   ├── agent_tools.py
│   ├── orchestrator.py
│   ├── ui.py
│   └── prompts/
│       └── system_prompt.txt
├── .gitignore
├── notes.pdf
├── requirements.txt
├── README.md
└── .env.example (if added later)
```

## Technology Stack

- Python 3.12+
- Streamlit
- LangChain
- LangGraph
- SQLAlchemy + pyodbc
- Microsoft SQL Server
- Pinecone vector database
- Open-Meteo API
- Hugging Face embeddings
- OpenAI / DeepSeek model integration
- Docker

## Workflow

The project follows a multi-phase workflow:

1. Ingest raw fleet telemetry data into SQL Server
2. Configure secure database views and restricted access
3. Load SOP documents into Pinecone for semantic search
4. Run the LangGraph orchestrator with LLM + tool chain
5. Ask operational questions in the dispatch UI
6. Review tool traces and audit logs for accountability

## File-by-File Overview

### `src/orchestrator.py`

This is the core reasoning engine. It:

- loads environment variables from `.env`
- initializes the LLM (OpenAI, DeepSeek, or Ollama fallback)
- defines the tool-enabled LangGraph agent
- connects the tool node to actions like SQL telemetry lookup and SOP search
- exposes a chat-based operational loop for testing or interactive use

### `src/agent_tools.py`

Contains the actual tool implementations used by the agent:

- `query_telemetry_db(sql_query: str)`
  - Executes a SELECT query against the FDE telemetry view
  - Enforces a safe restricted access pattern

- `fetch_corridor_conditions(latitude: float, longitude: float)`
  - Calls the Open-Meteo API
  - Returns alternate weather/corridor risk context

- `search_compliance_sop(query: str)`
  - Uses Pinecone retrieval to find relevant SOP clauses and operational rules

### `src/ui.py`

This is the Streamlit application that powers the dispatch dashboard.

It provides:

- a chat interface for operator queries
- a view of generated tool calls and tool outputs
- an enterprise-style security and audit panel
- an admin login flow for audit log inspection
- session-based tracking per user interaction

### `src/prompts/system_prompt.txt`

This file defines the agent’s decision-making policy and business output format. It instructs the model to:

- always check telemetry first
- then evaluate live corridor conditions
- then retrieve relevant SOP guidance
- return concise action-oriented results in a standardized format

### `scripts/ingest_legacy_data.py`

Loads the warehouse / legacy dataset into the SQL Server environment.

### `scripts/ingest_sop_pinecone.py`

Indexes SOP documents in Pinecone so the agent can retrieve them semantically during runtime.

### `scripts/setup_security_and_view.sql`

Creates restricted views and database roles to limit the agent’s access to safe operational data.

### `data/`

Contains:

- raw fleet/cold-chain data
- policy documents
- metadata cache
- source text files used in ingestion pipelines

### `docs/instrutions.md`

Project setup guide with step-by-step instructions covering:

- SQL Server container setup
- database connection instructions
- data ingestion steps
- SOP ingestion flow
- security configuration
- running the orchestrator and UI
-deployment guidance for remote hosting

## Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/iatharva6119/Cold-Chain-Logistics-FDE-Project.git
cd Cold-Chain-Logistics-FDE-Project
```

### 2. Create a virtual environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the project root.

Example:

```env
Agent_llm=DEEPSEEK
Embeddings_model=LOCAL
Local_Embedding_Model=BAAI/bge-m3

SQL_SERVER_HOST=localhost
SQL_SERVER_PORT=1433
SQL_AGENT_USER=USR_FDE_RO
SQL_AGENT_PASSWORD=AgentPassword2026!
SQL_ADMIN_USER=sa
SQL_ADMIN_PASSWORD=FdeEnterprisePass123!

PINECONE_API_KEY=your_pinecone_api_key
DEEPSEEK_API_KEY=your_deepseek_api_key
```

### 5. Start the SQL Server container

```bash
docker run -e "ACCEPT_EULA=Y" \
  -e "MSSQL_SA_PASSWORD=FdeEnterprisePass123!" \
  -p 1433:1433 \
  --name legacy-mssql \
  -d mcr.microsoft.com/mssql/server:2022-latest
```

### 6. Ingest the fleet dataset

```bash
python scripts/ingest_legacy_data.py
```

### 7. Ingest SOP documents into Pinecone

```bash
python scripts/ingest_sop_pinecone.py
```

### 8. Apply security model and data access views

Open the SQL script and run it against the SQL Server instance:

```sql
scripts/setup_security_and_view.sql
```

### 9. Start the application

```bash
streamlit run src/ui.py
```

## Example Operational Questions

The app is intended for use cases like:

```text
Find any active shipments near Los Angeles (Latitude ~33.8, Longitude ~-118.1). Check the local weather there, and tell me if the current cargo temperature violates the SOP for fresh perishables.
```

```text
I'm a new dispatcher on the night shift. Can you quickly explain the difference between a Tier 1 and Tier 2 escalation?
```

These prompts validate whether the system can:

- access real telemetry data
- reason over location conditions
- retrieve SOP guidance
- present a concise operational response with downtime or escalation impacts

## Security and Audit Model

A major part of the project is the controlled access model.

It enforces:

- restricted read-only SQL access for the operational agent
- rule-based protection against unsafe SQL actions
- a separate admin UI for reviewing execution logs
- audit table logging of tool usage and session actions

This simulates a more enterprise-safe pattern for AI-assisted operations.

## Operational Benefits

This system provides a useful framework for:

- cold-chain anomaly investigation
- route disruption triage
- SOP-grounded dispatch support
- explainable AI for operations teams
- early governance of AI actions in a regulated logistics environment

## Limitations and Production Considerations

This repository is a functional proof-of-concept / advanced prototype. Before production deployment, consider:

- secret management with `.env` replacement or cloud secret stores
- robust RBAC for SQL and agent access
- monitoring and observability for runtime failures
- validation against real SOP sources and legal/regulatory requirements
- more rigorous testing for edge-case routing and tool output handling
- scaling and performance optimizations for production workloads

## License

No explicit license file is included in the repository, so please check the repository owner or project policy before commercial reuse or redistribution.

## Credits and Context

This project appears to be built as a demonstration or portfolio project for enterprise AI in logistics, combining operational data, policy automation, and multi-tool reasoning.

## Related Files

- `docs/instrutions.md` — full environment setup and deployment notes
- `src/prompts/system_prompt.txt` — prompt contract for business-facing reasoning
- `requirements.txt` — dependencies for the project
- `scripts/setup_security_and_view.sql` — database security logic
- `data/policy/Cold_Chain_Incident_SOP_v2.md` — incident SOP-source content

## Future Improvements

Potential extensions include:

- adding a custom domain-specific dashboard for dispatch operations
- using a more advanced RAG pipeline with chunking and metadata filtering
- integrating message queues for incident event processing
- adding role-based user access and operator escalation workflows
- connecting with live route optimization or ETL systems

## Conclusion

The Cold-Chain Logistics FDE Project is a practical demonstration of building an AI-assisted logistics operations system that can reason over telemetry, check external conditions, and ground responses in SOP documentation. It is especially relevant for use cases where cold-chain compliance, logistics resilience, and operational decision support must be balanced in real time.

