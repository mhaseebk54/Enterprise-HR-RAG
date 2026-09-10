# 🏗 System Design: Enterprise HR Copilot

## 1. Overview
The **Enterprise HR Copilot** is an Agentic Retrieval-Augmented Generation (RAG) system designed to provide high-precision, grounded answers to employee queries regarding company HR policies. Unlike traditional RAG pipelines that follow a linear "Retrieve $\rightarrow$ Generate" path, this system implements a **self-correcting agentic loop** using a state-machine architecture to minimize hallucinations and maximize reliability.

## 2. Engineering Goals
- **High Precision**: Ensure answers are strictly grounded in official documentation.
- **Reliability**: Implement self-grading mechanisms to detect and correct poor retrieval.
- **Fallback Capability**: Seamlessly transition from private knowledge bases to verified external sources when internal data is insufficient.
- **Transparency**: Provide a clear execution trace and citations for every response.
- **Enterprise Readiness**: Modular design with admin-controlled ingestion and secure API access.

## 3. Architecture

### 3.1 High-Level Components
The system is decomposed into four primary layers:
1. **API Layer (FastAPI)**: Handles asynchronous request/response cycles and manages administrative tasks like document ingestion.
2. **Orchestration Layer (LangGraph)**: A state-machine that manages the agent's cognitive flow, including routing, grading, and query rewriting.
3. **Data Layer (Pinecone & Ollama)**: A serverless vector database for high-performance similarity search, powered by locally hosted embeddings.
4. **LLM Layer (Groq)**: High-throughput inference for reasoning, routing, and response generation.

### 3.2 Agentic Workflow (The State Machine)
The core of the system is a **LangGraph** state machine. This allows the agent to "think" and "verify" before responding.

#### Workflow Stages:
1. **Intelligent Routing**: The `route_question` node analyzes the intent. Casual chat is routed to a `direct_answer` node, while policy queries are sent to the `retrieve_kb` node.
2. **KB Retrieval & Grading**:
   - The agent retrieves relevant chunks from **Pinecone**.
   - A **Grader** node evaluates the retrieved documents. If the evidence is `good`, it proceeds to generation. If `weak`, it triggers a fallback.
3. **Web Fallback & Grading**:
   - If internal KB is insufficient, the agent uses **Tavily Search** to find general HR best practices or public information.
   - The web evidence is also graded for relevance.
4. **Query Rewriting (Self-Correction)**:
   - If both KB and Web searches fail to provide a "good" grade, the agent invokes a **Query Rewriter**.
   - The rewriter optimizes the query for better retrieval and restarts the KB retrieval loop (up to a configured `max_retries`).
5. **Grounded Generation**:
   - The final generation node is strictly prompted to use *only* the provided context, explicitly mentioning the source (Private KB vs. Web).

### 3.3 Data Pipeline
#### Ingestion Flow:
`Document (.pdf, .docx, .md, .txt)` $\rightarrow$ `Text Extraction` $\rightarrow$ `Recursive Character Splitting` $\rightarrow$ `Ollama Embeddings` $\rightarrow$ `Pinecone Index`.

- **Chunking Strategy**: Uses `RecursiveCharacterTextSplitter` with a chunk size of 900 and overlap of 120. This ensures that semantic context is preserved across chunk boundaries.
- **Indexing**: Utilizes Pinecone Serverless with cosine similarity for efficient, scalable vector search.

## 4. Technical Trade-offs & Decisions

| Decision | Chosen Approach | Alternative | Justification |
| :--- | :--- | :--- | :--- |
| **Orchestration** | LangGraph | Linear Chain | Linear chains cannot handle loops (like query rewriting) or complex conditional routing. |
| **Embeddings** | Ollama (Local) | OpenAI (Cloud) | Local embeddings ensure that the actual data vectors remain within the organization's infrastructure, reducing data privacy risks. |
| **Vector DB** | Pinecone Serverless | FAISS / Chroma | Serverless Pinecone provides better scalability and managed infrastructure compared to local vector stores. |
| **LLM Inference** | Groq | GPT-4 | Groq's LPU architecture provides near-instant inference, which is critical for a multi-step agentic loop that may call the LLM 3-5 times per query. |

## 5. Security & Governance
- **Admin Authentication**: The `/ingest` endpoint is protected by an `X-Admin-Key` header, preventing unauthorized modification of the knowledge base.
- **Audit Logging**: Every query, the source used, and the agent's execution trace are logged to an audit database (`audit.db`) for compliance and performance monitoring.
- **Strict Prompting**: System prompts are engineered to prevent the LLM from inventing policies, forcing a "I don't know" response when evidence is missing.
