
# LoanDocQA+ — Intelligent Loan Document Query Assistant

LoanDocQA+ is a complete **LLM-powered financial document assistant** that can:

* Extract structured text from PDFs or scanned images (using OCR)
* Build semantic embeddings with **Chroma + HuggingFace**
* Answer finance-related questions with **Ollama local LLMs (Phi-3, Mistral, etc.)**
* Perform real-time math reasoning and document-grounded analysis

---

## 📁 Project Structure

```
DL_project/
├── data/
│   └── loan_docs/                # Raw loan documents (PDFs, images)
├── out/
│   └── clean_texts/              # Extracted OCR text output
├── LLMquery/
│   ├── api_server.py             # FastAPI + Ollama backend
│   ├── prompts/                  # Custom prompt modules
│   └── vectorstores/             # Vector index builder (Chroma)
├── extraction_pipeline/
│   ├── main_extractor.py         # Runs OCR + preprocessing
│   └── utils.py
├── requirements.txt
└── README.md
```

---

## ⚙️ 1️⃣ Installation & Environment Setup

### 🧩 Clone the Repository

```bash
git clone https://github.com/nkousik18/doc_project.git
cd doc_project
```

### 🐍 Create a Virtual Environment

```bash
python3 -m venv .venv
source .venv/bin/activate     # On Mac/Linux
# OR
.venv\Scripts\activate        # On Windows
```

### 📦 Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 2️⃣ Ollama Setup (Local LLM)

LoanDocQA+ runs locally using Ollama — a lightweight local inference engine for open LLMs.

### 🔹 Install Ollama

Visit [https://ollama.com/download](https://ollama.com/download)
and install it for your operating system.

### 🔹 Pull a Model (Phi-3, Mistral, or WizardMath)

Run in your terminal:

```bash
ollama pull phi3
# OR
ollama pull mistral
# OR
ollama pull wizard-math
# OR
ollama pull phi3
```

Confirm models are downloaded:

```bash
ollama list
```

You should see your chosen model listed.

---

##  3️⃣ Extract Text from Loan Documents

To extract text from any **PDF or scanned image**, run the following:

```bash
python -m extraction_pipeline.main_extractor 
```

The extracted text will be saved automatically in:

```
data/clean_texts/
```

✅ This pipeline uses **PaddleOCR + PyMuPDF** for document layout-aware extraction.

---

## 🧱 4️⃣ Build the Vector Index (for semantic search)

After extraction, build embeddings for all cleaned `.txt` files:

```bash
python -m LLMquery.vectorstores.build_index \
  --input_dir data/clean_texts \
  --persist_dir LLMquery/vectorstores/loan_doc_index
```

This will:

* Load `sentence-transformers/all-MiniLM-L6-v2`
* Generate embeddings for each text file
* Store them in a persistent **Chroma vectorstore**.

You’ll see output like:

```
✅ Loaded 6 documents
🧠 Building vector embeddings...
💾 Saved to LLMquery/vectorstores/loan_doc_index
```

---

## 🚀 5️⃣ Run the FastAPI Server

Once the index is ready, start your API server:

```bash
uvicorn LLMquery.api_server:app --reload
```

You should see:

```
✅ LoanDocQA+ API (Local + SymPy Evaluator) is running.
📂 Loaded Chroma vectorstore and embeddings.
🧠 LLM model: phi3 loaded successfully.
```

By default, it runs on:
👉 **[http://127.0.0.1:8000](http://127.0.0.1:8000)**

But open the interface in :
```
LLMquery/static/chat.html
```

This is a chatbot interface 
---

##  6️⃣ Query the LLM API

Start quering the bot and verify the time in python terminal and the answers with source in the UI

---

## 🧮 7️⃣ Supported Query Modes

Your LLM automatically selects the right **prompt type**:

| Query Type       | Example                                                        | Mode               |
| ---------------- | -------------------------------------------------------------- | ------------------ |
| **Summary**      | “Summarize the Axis Gold Loan form”                            | summary_prompt     |
| **Math/Finance** | “Calculate simple interest for a $1000 loan at 10% for 1 year” | finance_prompt     |
| **Translation**  | “What is ‘interest’ in Spanish?”                               | translation_prompt |
| **Explanation**  | “Explain what a revolving loan is”                             | explanation_prompt |
| **Retrieval**    | “What documents mention repayment options?”                    | retrieval_prompt   |

---

## 🧰 8️⃣ Developer Notes

* All extracted `.txt` files go to → `data/clean_texts/`
* All vector embeddings persist in → `LLMquery/vectorstores/loan_doc_index/`
* Chat memory is in-memory (not persistent across sessions)
* For re-indexing, delete `loan_doc_index/` and rerun the build command

---

## 🧪 9️⃣ Example Workflow

```bash
# Step 1: Extract text
python -m extraction_pipeline.main_extractor 

# Step 2: Build embeddings
python -m LLMquery.vectorstores.build_index --input_dir data/clean_texts --persist_dir LLMquery/vectorstores/loan_doc_index

# Step 3: Start API
uvicorn LLMquery.api_server:app --reload

# Step 4: Ask a question

```

---

## 🧱 10️⃣ Requirements

See [`requirements.txt`](requirements.txt) for all dependencies.

To install:

```bash
pip install -r requirements.txt
```

---

## 🤖 Models Supported

| Model           | Type                             | Command                   |
| --------------- | -------------------------------- | ------------------------- |
| **phi3**        | General reasoning                | `ollama pull phi3`        |
| **mistral**     | Language reasoning               | `ollama pull mistral`     |
| **wizard-math** | Numerical/mathematical reasoning | `ollama pull wizard-math` |

---

## 🧠 Author & Maintainer

**Project:** LoanDocQA+
**Developer:** [Kousik Nandury](https://github.com/nkousik18)



