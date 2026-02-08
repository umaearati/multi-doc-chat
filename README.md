<!-- AI-Powered Document Portal (RAG-Based)
Problem:
Modern teams work with large documents such as PDFs, reports, and policies.
Finding specific information inside these documents is time-consuming and manual.
Traditional search cannot understand meaning or context.

Solution:
This project is an AI-powered Document Portal that allows users to:
Upload documents (PDFs, text files)
Ask natural language questions about the documents
Receive accurate answers based only on document content

The system uses a Retrieval-Augmented Generation (RAG) approach:
Documents are processed and converted into embeddings
Relevant document chunks are retrieved using semantic search
An LLM generates answers using retrieved context only

This reduces hallucinations and improves answer accuracy.

Tech Stack:
Backend: Python, FastAPI
AI / GenAI: OpenAI (LLM + embeddings)
RAG Framework: LangGraph
Vector Storage: FAISS (or equivalent vector store)
Document Processing: PDF/Text parsing & chunking
Infra & DevOps: Docker, GitHub Actions (CI/CD)
Logging & Safety: Structured logging, exception handling

Architecture

High-level flow:

1.User uploads a document
2.Document is validated, parsed, and chunked
3.Chunks are converted into embeddings and stored in a vector DB
4.User asks a question
5.Relevant document chunks are retrieved using similarity search
6.LLM generates an answer using retrieved context
7.Response is returned via API

Key design principles:
Modular services (API, retrieval, AI, storage)
Stateless APIs for scalability
Controlled prompts to reduce hallucinations -->

# Document Portal – GenAI / RAG / LLM Project

## Overview

This project is a **GenAI-powered Document Portal** built using **FastAPI**. It provides three core capabilities:

1. **Document Analysis** – Analyze a single PDF and extract structured metadata and summary using an LLM.
2. **Document Comparison** – Compare two PDF documents and generate page-wise differences using an LLM.
3. **Document Chat (RAG)** – Chat with uploaded documents using chunking, embeddings, FAISS vector store, and retrieval-augmented generation.

The project is designed to demonstrate **real GenAI workflows**, not just prompt calls.

---

## Key Features

- FastAPI-based backend
- Local file-based processing (no cloud dependency)
- Retrieval-Augmented Generation (RAG)
- Structured LLM outputs using Pydantic parsers
- Support for PDF, DOCX, and TXT documents (chat indexing)
- Modular and interview-friendly code structure

---

## High-Level Architecture

```
Client / UI
     |
     v
FastAPI Application (main.py)
     |
     |-- /analyze   -> DocumentAnalyzer (LLM)
     |-- /compare   -> DocumentComparatorLLM (LLM)
     |-- /chat
          |-- /index -> Load docs -> Chunk -> Embed -> FAISS
          |-- /query -> Retrieve -> LLM Answer (RAG)
```

---

## Core Components

### 1. API Layer (Routes)

- `routes/analyze.py` – Single PDF analysis
- `routes/compare.py` – Two-PDF comparison
- `routes/chat.py` – Document indexing and RAG-based querying

Routes are **thin** and delegate all logic to service classes.

---

### 2. Service Layer

- `DocumentAnalyzer`
  - Extracts text from PDF
  - Sends content to LLM with a structured prompt
  - Parses output into a Pydantic schema

- `DocumentComparatorLLM`
  - Combines two documents
  - Uses LLM to generate page-wise differences
  - Returns structured comparison results

- `ConversationalRAG`
  - Implements LCEL-based RAG chain
  - Rewrites question using prompt
  - Retrieves relevant chunks from FAISS
  - Generates grounded answers using LLM

---

### 3. Document Processing

- Format-specific loaders:
  - PDF → PyPDFLoader
  - DOCX → Docx2txtLoader
  - TXT → TextLoader

- Chunking:
  - RecursiveCharacterTextSplitter

Chunking improves retrieval accuracy and reduces LLM context usage.

---

### 4. Vector Store

- FAISS (local disk-based)
- Stores embeddings generated using Google Generative AI embeddings
- Index saved and loaded from disk

This design favors **simplicity and transparency** over distributed scalability.

---

### 5. LLM Integration

LLMs are used in three places:

1. Document analysis
2. Document comparison
3. RAG-based document chat

Provider selection is controlled via configuration and environment variables.

---

## Configuration

- `config/config.yaml` – Model and application configuration
- `.env` – API keys and runtime secrets (not committed)

---

## Logging and Error Handling

- Centralized logger and custom exception utilities exist
- Logging is applied selectively to avoid noise
- FastAPI default exception handling is used where appropriate

---

## Deployment

### AWS / ECS Deployment Architecture (Interview-Focused)

The application is deployed on **AWS ECS** using a containerized setup.

High-level flow:

```
Developer Push
   |
   v
GitHub Actions (CI/CD)
   |
   |-- Build Docker Image
   |-- Authenticate with AWS
   |-- Push Image to ECR
   |-- Update ECS Service
   v
AWS ECS (Running FastAPI + GenAI App)
```

Key points you should say in interviews:

- The app runs as a **Docker container** on ECS
- ECS pulls the latest image from **Amazon ECR**
- Environment variables are injected at runtime
- LLM calls and vector search happen inside the running container

This is a **real cloud deployment**, not local-only.

---

### CI vs CD (As Implemented in This Project)

#### CI – Continuous Integration (`ci.yaml`)

What CI does in this project:

- Triggers on **push / pull request**
- Checks out the code
- Sets up Python
- Installs dependencies
- Runs basic sanity checks

What CI does NOT do:

- No deployment
- No AWS interaction
- No Docker push

CI exists to **validate code correctness**.

---

#### CD – Continuous Deployment (`aws.yaml`)

What CD does in this project:

- Builds Docker image
- Logs into AWS (via GitHub secrets)
- Pushes image to **Amazon ECR**
- Updates **AWS ECS service**

This results in:

- New container version running on ECS
- Zero manual deployment steps

CD is **environment-specific** and tied to your AWS account.

---

## Limitations (Intentional)

- No authentication or authorization
- No persistent chat history
- No distributed vector store
- No LLM evaluation framework
- No production observability stack

These are intentionally left out to keep the focus on **core GenAI workflows**.

---

## Future Improvements (Not Implemented)

- Auth and user isolation
- Persistent chat memory
- Vector DB replacement for scale
- RAG evaluation (RAGAS / metrics)
- Tracing, monitoring, and rate limiting

---

## Interview Note

This project is designed to be **honest, minimal, and real**.

If a feature is not listed above, it is **not present in the project and should not be claimed in interviews**.

---

## How to Run (Local)

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

Or using Docker:

```bash
docker build -t document-portal .
docker run -p 8000:8000 document-portal
```
