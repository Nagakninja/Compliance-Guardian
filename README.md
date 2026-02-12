# Compliance Guardian AI - Compliance QA Pipeline

## 📋 Project Overview

**Compliance Guardian** is an intelligent AI-powered compliance audit system designed to analyze YouTube videos and detect brand compliance violations in real-time. Using a sophisticated multi-stage workflow, it extracts video content (transcripts, OCR text) and performs Retrieval-Augmented Generation (RAG) against a knowledge base of brand compliance rules to identify violations.

### Key Features
- 🎥 **Automated YouTube Video Processing** - Downloads and processes YouTube videos
- 🔍 **Intelligent Content Extraction** - Extracts transcripts and on-screen text (OCR) using Azure Video Indexer
- 🧠 **AI-Powered Compliance Auditing** - Uses Azure OpenAI and RAG to detect regulatory violations
- 📊 **Structured Reporting** - Generates detailed compliance audit reports with severity levels
- ☁️ **Cloud-Native Architecture** - Built on Azure services with OpenTelemetry observability
- ⚡ **RESTful API** - FastAPI-based REST endpoints for integration

---

## 🏗️ Project Structure

```
ComplianceQAPipeline/
├── main.py                           # Entry point for CLI simulation/testing
├── pyproject.toml                    # Project metadata and dependencies
├── README.md                         # This file
├── .env                              # Environment variables (not in git)
├── .gitignore                        # Git exclusion rules
├── .python-version                   # Python version specification
│
├── backend/                          # Backend application code
│   ├── Dockerfile                    # Docker containerization
│   ├── data/                         # Data storage directory
│   │
│   ├── scripts/
│   │   ├── index_documents.py        # Script to index compliance rules into Azure Search
│   │   └── explanation.txt           # Documentation for scripts
│   │
│   ├── src/
│   │   ├── api/                      # REST API layer
│   │   │   ├── __init__.py
│   │   │   ├── server.py             # FastAPI application with endpoints
│   │   │   └── telemetry.py          # Azure Monitor OpenTelemetry setup
│   │   │
│   │   ├── graph/                    # LangGraph workflow orchestration
│   │   │   ├── __init__.py
│   │   │   ├── workflow.py           # Graph definition (DAG structure)
│   │   │   ├── state.py              # Data schema and state management
│   │   │   └── nodes.py              # Workflow nodes (Indexer & Auditor)
│   │   │
│   │   └── services/                 # Business logic services
│   │       ├── __init__.py
│   │       └── video_indexer.py      # Azure Video Indexer integration
│   │
│   └── tests/                        # Test suite (future expansion)
│

```

---

## 📁 File Descriptions

### Root Level Files

| File | Purpose |
|------|---------|
| **main.py** | Entry point for CLI-based testing. Demonstrates the full workflow: creates a unique session ID, calls the compliance graph, and displays results. Used for local development and testing. |
| **pyproject.toml** | Project configuration with metadata, dependencies, and Python version requirements (3.12+). Defines all external packages needed. |
| **.env** | Environment variables for Azure services (API keys, endpoints, credentials). Not committed to git for security. |
| **.gitignore** | Specifies files to exclude from version control. |
| **.python-version** | Python interpreter version specification. |

### Backend API Layer (`backend/src/api/`)

| File | Purpose |
|------|---------|
| **server.py** | FastAPI REST API server that exposes endpoints for video compliance auditing. Handles HTTP requests, initializes telemetry, and invokes the compliance workflow graph. |
| **telemetry.py** | Configures Azure Monitor OpenTelemetry for observability. Automatically tracks HTTP requests, errors, dependencies, and performance metrics sent to Azure Monitor. |

### Workflow Graph (`backend/src/graph/`)

| File | Purpose |
|------|---------|
| **workflow.py** | Defines the LangGraph DAG (Directed Acyclic Graph) that orchestrates the compliance audit process. Connects nodes in sequence: START → Indexer → Auditor → END. |
| **state.py** | Defines the data schema (`VideoAuditState`) for all data flowing through the workflow. Includes state classes like `ComplianceIssue` for structured violation reporting. |
| **nodes.py** | Implements the two worker nodes:<br/>- **index_video_node**: Downloads YouTube videos, uploads to Azure Video Indexer, waits for processing, and extracts transcripts/OCR text.<br/>- **audit_content_node**: Performs RAG queries against the knowledge base and uses Azure OpenAI to detect compliance violations. |

### Services (`backend/src/services/`)

| File | Purpose |
|------|---------|
| **video_indexer.py** | Encapsulates Azure Video Indexer integration. Handles YouTube video downloads (via yt-dlp), Azure Video Indexer authentication, video uploads, processing status polling, and content extraction (transcripts, OCR, metadata). |

### Scripts (`backend/scripts/`)

| File | Purpose |
|------|---------|
| **index_documents.py** | One-time setup script to index compliance rules into Azure Cognitive Search. Reads regulatory rules and creates embeddings for RAG retrieval. |
| **explanation.txt** | Documentation explaining how to use the scripts. |

### Other

| File | Purpose |
|------|---------|
| **tests/** | Test suite directory (currently empty, reserved for future test cases). |

---

## 🔄 Workflow Architecture

### LangGraph Workflow: Three-Stage Process

```
INPUT (Video URL)
    ↓
[Node: Indexer] ← Extracts transcript & OCR from YouTube video
    ├─ Download YouTube video
    ├─ Upload to Azure Video Indexer
    ├─ Wait for processing completion
    └─ Extract: transcript, OCR text, metadata
    ↓
[Node: Auditor] ← Analyzes content against compliance rules
    ├─ Retrieve compliance rules from Azure Cognitive Search (RAG)
    ├─ Query Azure OpenAI with LLM
    └─ Identify violations and generate report
    ↓
OUTPUT (Compliance Report)
    └─ Status: PASS/FAIL
    └─ Violations: List of issues with severity levels
    └─ Report: Markdown summary
```

### Data Flow Through the State

The `VideoAuditState` carries all data through the workflow:

1. **Input**: `video_url`, `video_id`
2. **After Indexer Node**: `transcript`, `ocr_text`, `video_metadata`
3. **After Auditor Node**: `compliance_results`, `final_status`, `final_report`
4. **Throughout**: `errors` (append-only list for system errors)

---

## 🚀 Setup Instructions

### Prerequisites

- **Python 3.12+** (specified in `.python-version`)
- **Azure Account** with:
  - Azure Video Indexer resource
  - Azure OpenAI with Chat and Embedding deployments
  - Azure Cognitive Search instance
  - Application Insights for telemetry (optional)
- **Git** for version control
- **FFmpeg** (required by yt-dlp for video downloading)
  - macOS: `brew install ffmpeg`
  - Ubuntu: `sudo apt-get install ffmpeg`
  - Windows: Download from [ffmpeg.org](https://ffmpeg.org/download.html)

### Step 1: Clone and Navigate to Project

```bash
git clone <repository-url>
cd ComplianceQAPipeline
```

### Step 2: Set Up Python Environment

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# macOS/Linux:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# Upgrade pip
pip install --upgrade pip
```

### Step 3: Install Dependencies

```bash
# Install all project dependencies from pyproject.toml
pip install -e .
```

### Step 4: Configure Environment Variables

Create/update the `.env` file with your Azure credentials:

```bash
# Azure Storage
AZURE_STORAGE_CONNECTION_STRING="your-connection-string"

# Azure OpenAI
AZURE_OPENAI_API_KEY="your-api-key"
AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
AZURE_OPENAI_API_VERSION="2024-02-15"  # Update to current version
AZURE_OPENAI_CHAT_DEPLOYMENT="your-chat-deployment-name"
AZURE_OPENAI_EMBEDDING_DEPLOYMENT="text-embedding-3-small"

# Azure Cognitive Search
AZURE_SEARCH_ENDPOINT="https://your-search.search.windows.net"
AZURE_SEARCH_API_KEY="your-search-key"
AZURE_SEARCH_INDEX_NAME="compliance-rules"  # Index name for RAG

# Azure Video Indexer (Identity-based authentication)
AZURE_VI_NAME="your-vi-resource-name"
AZURE_VI_LOCATION="your-region"  # e.g., eastus
AZURE_VI_ACCOUNT_ID="your-account-id"
AZURE_SUBSCRIPTION_ID="your-subscription-id"
AZURE_RESOURCE_GROUP="your-resource-group"

# Azure Monitor (Observability)
APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey=...;IngestionEndpoint=..."

# LangSmith (Optional - for tracing)
LANGCHAIN_TRACING_V2=true
LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
LANGCHAIN_API_KEY="your-langsmith-key"
LANGCHAIN_PROJECT="compliance-guardian"
```

### Step 5: Initialize the Knowledge Base (One-Time Setup)

Index your compliance rules into Azure Cognitive Search:

```bash
python backend/scripts/index_documents.py
```

This script reads compliance rules and creates embeddings for RAG retrieval. Only needs to be run once when setting up.

### Step 6: Verify Installation

```bash
# Test imports
python -c "from backend.src.graph.workflow import app; print('✓ Workflow imports OK')"
python -c "from backend.src.api.server import app as api_app; print('✓ API imports OK')"
```

---

## ▶️ Running Instructions

### Option 1: CLI Simulation (Local Testing)

Run the complete workflow with a test YouTube video:

```bash
python main.py
```

**Output:**
- Session ID
- Input payload summary
- Video download and processing status
- Compliance audit results
- Final report with violations (if any)

### Option 2: Start FastAPI Server (Development)

Launch the REST API server:

```bash
# Start the server on http://localhost:8000
uvicorn backend.src.api.server:app --reload --host 0.0.0.0 --port 8000
```

**Available Endpoints:**
- `POST /audit` - Submit a video for compliance audit
- `GET /health` - Health check endpoint
- `GET /docs` - Interactive API documentation (Swagger UI)

**Example API Request:**
```bash
curl -X POST "http://localhost:8000/audit" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://youtu.be/dT7S75eYhcQ",
    "video_id": "my_test_video"
  }'
```

### Option 3: Docker Deployment

Build and run the backend in a Docker container:

```bash
# Build Docker image
docker build -f backend/Dockerfile -t compliance-guardian:latest .

# Run container with environment variables
docker run -p 8000:8000 \
  --env-file .env \
  -e APPLICATIONINSIGHTS_CONNECTION_STRING="..." \
  compliance-guardian:latest
```

### Option 4: Azure Functions (Serverless)

Deploy to Azure Functions for serverless execution:

```bash
# Install Azure Functions CLI (if not already installed)
# macOS: brew tap azure/azure && brew install azure-functions
# Or: npm install -g azure-functions-core-tools@4

# Test locally
func start

# Deploy to Azure
func azure functionapp publish <your-function-app-name>
```

---

## 🔧 Configuration Guide

### Environment Variables Reference

| Variable | Purpose | Example |
|----------|---------|---------|
| `AZURE_OPENAI_CHAT_DEPLOYMENT` | Name of your Chat model deployment | `gpt-4` |
| `AZURE_SEARCH_INDEX_NAME` | Name of the compliance rules index | `compliance-rules` |
| `AZURE_VI_LOCATION` | Region where Video Indexer is hosted | `eastus` |
| `LANGCHAIN_TRACING_V2` | Enable LangSmith debugging | `true` |

### Azure Service Configuration Tips

1. **Azure Video Indexer**: Use managed identity authentication (recommended over keys)
2. **Azure Cognitive Search**: Use API keys or managed identity
3. **Azure OpenAI**: Ensure both Chat and Embedding deployments are created
4. **Application Insights**: Optional but recommended for production monitoring

---

## 📊 Understanding the Compliance Report

The final compliance report includes:

```json
{
  "video_id": "vid_abc12345",
  "final_status": "FAIL",
  "compliance_results": [
    {
      "category": "FTC_DISCLOSURE",
      "severity": "CRITICAL",
      "description": "Missing required disclosure at 0:30",
      "timestamp": "0:30"
    }
  ],
  "final_report": "## Compliance Audit Results\n\n1 **CRITICAL** issue found..."
}
```

**Status Levels:**
- **PASS**: No violations detected
- **FAIL**: One or more violations found

**Severity Levels:**
- **CRITICAL**: Regulatory requirement violation
- **WARNING**: Best practice or recommended guideline violation


---

## 📦 Key Dependencies

| Package | Purpose |
|---------|---------|
| **FastAPI** | Web framework for REST API |
| **LangGraph** | Workflow orchestration and DAGs |
| **LangChain** | LLM abstractions and RAG |
| **Azure OpenAI** | Chat and embedding models |
| **Azure Search Documents** | Vector search for RAG retrieval |
| **Azure Video Indexer** | Video content extraction |
| **yt-dlp** | YouTube video downloading |
| **Pydantic** | Data validation |
| **SQLAlchemy** | Database ORM (optional, for persistence) |
| **Redis** | Caching (optional) |
| **Azure Monitor OpenTelemetry** | Observability and monitoring |

---

## 🐛 Troubleshooting

### Issue: "ModuleNotFoundError: No module named 'backend'"

**Solution:**
```bash
# Ensure you're in the project root directory
cd ComplianceQAPipeline
pip install -e .
```

### Issue: Azure Video Indexer authentication fails

**Check:**
1. Correct `AZURE_VI_ACCOUNT_ID` and `AZURE_VI_NAME`
2. Service principal has Video Indexer Contributor role
3. Subscription ID and resource group are correct

### Issue: Azure Search returns no results

**Check:**
1. Index exists and is named correctly in `.env` (`AZURE_SEARCH_INDEX_NAME`)
2. Run `index_documents.py` to populate the index
3. Verify search endpoint and API key

### Issue: OpenAI API returns rate limit error

**Solution:**
1. Check quota in Azure Portal
2. Implement exponential backoff in code
3. Consider higher tier or different region

### Issue: YouTube download fails

**Check:**
1. FFmpeg is installed: `ffmpeg -version`
2. URL is a valid YouTube link
3. Video is publicly accessible

---

## 📚 Additional Resources

- [Azure Video Indexer Documentation](https://learn.microsoft.com/en-us/azure/azure-video-indexer/)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Azure Cognitive Search Documentation](https://learn.microsoft.com/en-us/azure/search/)
- [OpenTelemetry Documentation](https://opentelemetry.io/)

---

## 📝 Development Workflow

### Add New Compliance Rules

1. Add rules to your compliance documents
2. Run `python backend/scripts/index_documents.py` to re-index
3. The RAG system will automatically consider new rules

### Extend the Workflow

To add new workflow nodes:

1. Create node function in `backend/src/graph/nodes.py`
2. Update `VideoAuditState` if new data fields are needed
3. Register node in `backend/src/graph/workflow.py`
4. Connect edges in the workflow

### Add API Endpoints

In `backend/src/api/server.py`:

```python
@app.post("/new-endpoint")
async def new_endpoint(request: YourRequestModel):
    result = compliance_graph.invoke(request.dict())
    return result
```

---

## 📄 License

This project is licensed under the MIT License.

