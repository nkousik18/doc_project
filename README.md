Overview
Mini Document AI is a local, privacy-preserving pipeline for understanding and analyzing financial documents such as bank forms, gold loan applications, and loan agreements.
It combines OCR, text cleaning, financial-term reasoning, and context-aware glossary tagging to prepare documents for local LLM inference, without relying on cloud APIs or external services.
 Project Goals
Extract and clean text from scanned financial PDFs or images.
Identify and normalize domain-specific financial terminology.
Tag key-value entities such as loan type, applicant name, PAN number, etc.
Build a glossary-aware knowledge layer using the FinRAD dataset.
Enable offline semantic understanding, translation, and summarization through a local LLM (e.g., Llama 3, Mistral, or Phi-3).
 Folder Structure
DL_project/
│
├── data/
│   ├── financial_terms.csv                 # Glossary of finance terms + definitions
│   ├── Finance_terms_definitions_labels.csv # FinRAD dataset
│   ├── federal-loan-programs.pdf           # Sample input PDF
│   ├── federal-loan-programs_final.txt     # Extracted text output
│   └── federal-loan-programs_glossary.json # Context-aware glossary tagging result
│
├── modules/
│   ├── ingestion.py                        # PDF-to-image conversion and preprocessing
│   ├── ocr_engine.py                       # Tesseract & EasyOCR extraction
│   ├── layout_analysis.py                  # Column / layout splitting
│   ├── postprocess.py                      # Text normalization and spell correction
│   ├── glossary_utils.py                   # Financial glossary loader + context reasoning
│   └── extractor.py                        # Unified pipeline controller
│
├── build_glossary_from_finrad.py           # Script to build glossary from FinRAD dataset
├── main.py                                 # Entry point for full pipeline
├── requirements.txt
└── README.md

 Installation & Setup
1️⃣ Clone Repository
git clone <your_repo_url>
cd DL_project

2️⃣ Create Virtual Environment
python3 -m venv .venv
source .venv/bin/activate

3️⃣ Install Dependencies
pip install -r requirements.txt

4️⃣ Install OCR Engines
brew install tesseract   # macOS
# or
sudo apt install tesseract-ocr  # Linux

 Key Components
🔹 1. Text Extraction (OCR)
Preprocesses PDF pages using OpenCV (grayscale, denoising, thresholding).
Runs Tesseract for fast OCR.
Falls back to EasyOCR for complex layouts or low-confidence pages.
Saves both raw and cleaned text outputs.
Output Example:
data/federal-loan-programs_final.txt

🔹 2. Financial Glossary (FinRAD Integration)
Uses the FinRAD dataset to build a large financial vocabulary (~13K entries).
Generates a filtered glossary (financial_terms.csv) with term + definition columns.
Extends coverage by merging with manually curated glossary entries.
Command to build glossary:
python build_glossary_from_finrad.py

Output:
data/financial_terms.csv


🔹 3. Context-Aware Glossary Tagging
Implemented in glossary_utils.py:
Detects financial terms within extracted text.
Filters out false positives (e.g., “cover”, “option”, “life” used in non-financial sense).
Uses context window analysis to confirm financial relevance (neighboring terms like “loan”, “bank”, “fund”, etc.).
Produces a glossary JSON with only meaningful terms and definitions.
Output Example:
{
  "loan": "A sum of money borrowed for repayment with interest.",
  "interest rate": "The percentage charged for borrowing money.",
  "collateral": "Asset pledged as security for repayment of a loan."
}

🔹 4. Postprocessing and Cleanup
Corrects OCR errors (\/We → I/We, confide → bona fide).
Normalizes punctuation, whitespace, and casing.
Optionally performs spell correction and regex-based fixes.
🔹 5. Local LLM-Ready Output
After OCR + tagging, the pipeline produces structured outputs ready for a local language model (Llama 3, Mistral, Phi-3, etc.) for:
Summarization
Q&A (“What documents are required for this loan?”)
Translation
Compliance or classification
 Running the Full Pipeline
python main.py --file data/federal-loan-programs.pdf

Outputs:
File
Purpose
federal-loan-programs_final.txt
Cleaned extracted text
federal-loan-programs_glossary.json
Tagged glossary definitions


Features Summary
Module
Purpose
OCR Engine
Extracts text from PDFs/images using Tesseract and EasyOCR
Preprocessing
Denoising, thresholding, and adaptive binarization for cleaner OCR
Glossary Builder
Builds and filters financial glossary from FinRAD dataset
Glossary Utils
Context-aware tagging to avoid false positives
Postprocessing
Fixes OCR artifacts and spacing errors
Main Pipeline
Orchestrates all steps for one-command processing


 Privacy and Offline Design
Runs entirely locally, no API calls or internet access required.
Keeps sensitive financial information (PAN, Aadhaar, bank details) secure.
Ideal for on-premise deployments or financial document compliance tools.
🧠 Next Steps
Planned future enhancements:
🧾 Layout-Aware Parsing — integrate PaddleOCR or LayoutLMv3 for better key-value extraction.
🧠 Local LLM Integration — use Llama-3-Instruct or Phi-3-mini for Q&A and summarization.
🏦 Domain Fine-Tuning — train local embeddings for finance-specific semantic similarity.
🧩 Visualization Layer — build a small dashboard to review extracted terms and glossary matches interactively.
