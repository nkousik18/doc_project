

### *Automated Extraction, Indexing, and Q&A System using Airflow, LangChain, and FastAPI*

---

### 🏗️ Overview

**LoanDocQA+** is a fully automated pipeline for processing and understanding loan-related PDF documents.
It performs:

* **Text extraction** (via OCR fallback for scanned PDFs)
* **Vector embedding & indexing** (for semantic search)
* **LLM-powered Q&A** (retrieval-augmented reasoning via local models)
* **End-to-end orchestration** using **Apache Airflow**

---

## 🧩 System Architecture

```plaintext
📂 Data Flow:
Upload PDF → Extract Text (OCR/Text) → Build/Update Vector Index → LLM Prompt → Answer

🔁 DAG Workflow:
extract_text  ➜  update_vector_index  ➜  generate_llm_prompt
```

| Component                              | Role                                                          |
| -------------------------------------- | ------------------------------------------------------------- |
| **FastAPI Server**                     | Provides `/upload_file` & `/query_stream` endpoints for users |
| **Airflow DAGs**                       | Automates the document extraction and indexing pipeline       |
| **Chroma Vectorstore**                 | Stores document embeddings for retrieval                      |
| **LangChain + HuggingFace Embeddings** | Converts text to semantic vectors                             |
| **Ollama LLM (Phi-3 / Llama3)**        | Generates factual, context-aware responses                    |
| **Docker Compose Stack**               | Runs Airflow (Scheduler, Webserver, Redis, Postgres)          |

---

## 🧱 Project Structure

```plaintext
DL_project/
├── dags/
│   └── loan_doc_pipeline_dag.py          # Airflow DAG orchestrating the full pipeline
│
├── extraction_pipeline/                  # Text & OCR extraction modules
│   ├── main_extractor.py
│   ├── ocr_utils.py
│   └── utils.py
│
├── LLMquery/
│   ├── api_server.py                     # FastAPI service (Upload + Query)
│   ├── build_index.py                    # Rebuilds / updates Chroma vector index
│   ├── prompts/                          # Prompt templates & logic
│   │   ├── prompt_router.py
│   │   ├── finance_prompt.py
│   │   ├── retrieval_prompt.py
│   │   ├── summary_prompt.py
│   │   ├── explanation_prompt.py
│   │   └── translation_prompt.py
│   └── vectorstores/
│       └── loan_doc_index/               # Persistent Chroma index
│
├── data/
│   ├── loan_docs/                        # Uploaded PDFs
│   └── clean_texts/                      # Extracted text files (.txt)
│
├── docker-compose.yaml                   # Airflow multi-service environment
├── Dockerfile.airflow                    # Custom Airflow image with OCR & LangChain deps
├── requirements.txt                      # Python dependencies
└── README.md
```

---

## ⚙️ Setup & Installation

### 1️⃣ Clone the repository

```bash
git clone https://github.com/<your-username>/DL_project.git
cd DL_project
```

---

### 2️⃣ Create a virtual environment (for local FastAPI)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

---

### 3️⃣ Airflow Docker Setup

#### 🧩 Docker Compose Stack

The `docker-compose.yaml` provisions:

* **Airflow Scheduler**
* **Airflow Webserver**
* **Redis**
* **Postgres**
* **Preinstalled OCR / LangChain libraries**

Run the stack:

```bash
docker-compose up -d --build
```

> 🧠 The Airflow UI will be available at [http://localhost:8090](http://localhost:8090)

#### Check service status:

```bash
docker ps
```

#### Access the Airflow CLI inside the container:

```bash
docker exec -it dl_project-airflow-scheduler-1 bash
```

---

## 🪶 Airflow DAG: `loan_doc_pipeline_dag`

### DAG Flow:

```plaintext
extract_text → update_vector_index → generate_llm_prompt
```

### Manual Trigger (via CLI)

```bash
docker exec -it dl_project-airflow-scheduler-1 airflow dags trigger loan_doc_pipeline_dag --run-id manual_test
```

### View DAG runs and task states

```bash
docker exec -it dl_project-airflow-scheduler-1 airflow dags list-runs -d loan_doc_pipeline_dag
docker exec -it dl_project-airflow-scheduler-1 airflow tasks states-for-dag-run loan_doc_pipeline_dag manual_test
```

---

## 🧠 Updated `build_index.py`

Key features:

* Auto-creates persistent Chroma DB under `LLMquery/vectorstores/loan_doc_index`
* Rebuilds or appends new `.txt` documents dynamically
* Compatible with new LangChain versions
* Includes improved exception handling

```python
# ✅ New safe version
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_chroma import Chroma
```

---

## 🧩 Example `docker-compose.yaml`

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:13
    environment:
      - POSTGRES_USER=airflow
      - POSTGRES_PASSWORD=airflow
      - POSTGRES_DB=airflow
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U airflow"]
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:latest

  airflow-scheduler:
    build:
      context: .
      dockerfile: Dockerfile.airflow
    restart: always
    environment:
      - AIRFLOW__CORE__EXECUTOR=CeleryExecutor
    volumes:
      - ./:/opt/airflow
    depends_on:
      - postgres
      - redis

  airflow-webserver:
    build:
      context: .
      dockerfile: Dockerfile.airflow
    ports:
      - "8090:8080"
    restart: always
    depends_on:
      - airflow-scheduler

volumes:
  postgres_data:
```

---

## 🚀 Running the FastAPI Server

Once your Airflow DAGs are running smoothly, start the unified API server:

```bash
uvicorn LLMquery.api_server:app --reload
```

### Endpoints:

| Route           | Method | Description                          |
| --------------- | ------ | ------------------------------------ |
| `/upload_file`  | `POST` | Upload new PDF → Extract → Index     |
| `/query_stream` | `POST` | Streamed Q&A over uploaded documents |

Example:

```bash
curl -X POST "http://127.0.0.1:8000/upload_file" -F "file=@data/loan_docs/sample.pdf"
```

---

## 🧩 Logs & Monitoring

| Type             | Path                                    |
| ---------------- | --------------------------------------- |
| Airflow DAG logs | `/opt/airflow/logs` inside container    |
| API logs         | `logs/query_logs.csv`                   |
| Extracted text   | `data/clean_texts/`                     |
| Vector index     | `LLMquery/vectorstores/loan_doc_index/` |

---

## 🧠 Common Issues

| Error                                   | Cause                                  | Fix                                                        |
| --------------------------------------- | -------------------------------------- | ---------------------------------------------------------- |
| `attempt to write a readonly database`  | Chroma index lock or permissions issue | `rm -rf LLMquery/vectorstores/loan_doc_index` then rebuild |
| `ModuleNotFoundError: langchain.schema` | Outdated import                        | Use `from langchain_core.documents import Document`        |
| Airflow “invalid user”                  | Permissions in Dockerfile              | Add `USER airflow` before installing pip packages          |
| Slow OCR                                | Running PaddleOCR CPU models           | Optional: switch to GPU runtime                            |


Would you like me to include a **“Quick Start (1 command)”** section — e.g.,
`bash start_pipeline.sh` that wraps Airflow + FastAPI setup into one script?
