# 🏢 Enterprise HR Copilot: Agentic RAG System

[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green.svg)](https://fastapi.tiangolo.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-Agentic-orange.svg)](https://langchain-ai.github.io/langgraph/)
[![Pinecone](https://img.shields.io/badge/Pinecone-VectorDB-blue.svg)](https://www.pinecone.io/)

## 🌟 Executive Summary
The **Enterprise HR Copilot** is a production-ready AI system designed to eliminate the operational bottlenecks of HR policy management. By implementing an **Agentic Retrieval-Augmented Generation (RAG)** workflow, it transforms static, fragmented HR documentation into a reliable, self-correcting digital assistant.

This project demonstrates the role of a **Forward Deployed Engineer (FDE)**: identifying a high-impact business problem (HR burnout and compliance risk) and architecting a full-stack AI solution that bridges the gap between raw data and executive-level reliability.

---

## 🎯 Business Impact
### The Challenge
In large-scale enterprises, HR knowledge is trapped in monolithic PDFs. This results in:
- **Operational Inefficiency**: HR teams spend 40% of their time answering repetitive policy questions.
- **Employee Friction**: High latency in getting accurate answers regarding leave, payroll, and benefits.
- **Compliance Risk**: Inconsistent interpretations of policy across different HR representatives.

### The Solution
A **Self-Correcting Copilot** that ensures:
- **Zero-Hallucination Goal**: Answers are strictly grounded in official documentation.
- **Autonomous Verification**: The system grades its own retrieved evidence and iterates until a high-confidence answer is found.
- **Seamless Fallbacks**: Intelligent routing between private company knowledge and verified external HR standards.

---

## 🛠 Technical Architecture
This system is built as a complete engineering pipeline, combining data engineering, AI orchestration, and scalable backend design.

### Core Innovation: The Agentic Loop
Unlike standard RAG (Retrieve $\rightarrow$ Generate), this system utilizes a **LangGraph state machine** to implement a cognitive loop:
1. **Intent Routing**: Distinguishes between casual interaction and policy inquiries.
2. **Evidence Grading**: An LLM-based "grader" evaluates if the retrieved context is sufficient.
3. **Iterative Refinement**: If evidence is weak, the agent autonomously rewrites the query to improve retrieval.
4. **Multi-Source Routing**: Intelligently switches between the Private Vector Store and Web Search (Tavily).

### The Stack
- **Orchestration**: `LangGraph` (State-machine based AI agents)
- **Inference**: `Groq` (Llama 3 for ultra-low latency reasoning)
- **Vector Store**: `Pinecone Serverless` (High-scale similarity search)
- **Embeddings**: `Ollama` (Local embeddings for enhanced data privacy)
- **API Layer**: `FastAPI` (Asynchronous Python framework)
- **Infrastructure**: `Docker` for consistent deployment across environments

For a deep dive into the architectural decisions, trade-offs, and data flow, see the [**DESIGN.md**](./DESIGN.md) file.

---

## 📂 Project Structure
```text
Enterprise-HR-RAG/
├── app/
│   ├── api/            # Asynchronous FastAPI routes
│   ├── core/           # Enterprise config & logging
│   ├── rag/            # Agentic Logic (LangGraph, Pinecone, State)
│   └── services/       # Business logic (Ingestion, Auditing)
├── data/               # Sample Knowledge Base & Audit DB
├── static/             # Frontend assets (CSS/JS)
├── templates/          # HTML templates
├── Dockerfile          # Production containerization
└── ingest_sample_kb.py # Vector DB population utility
```

---

## ⚙️ Setup & Installation

### Prerequisites
- Python 3.10+
- Ollama (for local embeddings)
- API Keys: **Groq**, **Pinecone**, **Tavily**

### Quick Start
1. **Clone & Install**
   ```bash
   git clone https://github.com/your-username/enterprise-hr-rag.git
   cd enterprise-hr-rag
   pip install -r requirements.txt
   ```

2. **Configure Environment**
   Create a `.env` file:
   ```env
   GROQ_API_KEY=your_groq_key
   PINECONE_API_KEY=your_pinecone_key
   TAVILY_API_KEY=your_tavily_key
   OLLAMA_BASE_URL=http://localhost:11434
   ```

3. **Initialize Knowledge Base**
   ```bash
   python ingest_sample_kb.py
   ```

4. **Launch Application**
   ```bash
   python run.py
   ```

---

## 🔌 API Reference
| Method | Endpoint | Description | Access |
| :--- | :--- | :--- | :--- |
| `GET` | `/` | HR Copilot User Interface | Public |
| `POST` | `/api/chat` | Submit question to Agentic Workflow | Public |
| `POST` | `/api/ingest` | Securely index new HR documents | Admin (`X-Admin-Key`) |
| `GET` | `/api/health` | Service health check | Public |

---

## 📜 License
This project is licensed under the terms specified in the `LICENSE` file.
