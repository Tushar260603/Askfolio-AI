<div align="center">

# 📄 AskDoc

### *Talk to your documents. Get answers, not search results.*

An enterprise-grade **Retrieval-Augmented Generation (RAG)** platform that lets you upload any document and have an intelligent AI conversation with it — powered by **Google Gemini**, **Pinecone**, and **LangGraph**.

[![Backend](https://img.shields.io/badge/Backend-FastAPI-009688?style=for-the-badge&logo=fastapi)](./askdoc/server)
[![Frontend](https://img.shields.io/badge/Frontend-Next.js%2016-black?style=for-the-badge&logo=next.js)](./askdoc/client)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)

</div>

---

## 🌍 The Real-World Problem AskDoc Solves

Every day, professionals waste hours **searching through massive documents** — legal contracts, academic syllabi, research papers, financial reports, policy handbooks, product manuals — looking for specific answers buried inside hundreds of pages.

> **"Which semester has Data Structures?"** — searching through a 200-page university curriculum PDF manually.  
> **"What are the termination clauses?"** — skimming through a 60-page legal contract.  
> **"What were the Q3 revenue numbers?"** — scanning a sprawling financial report.

Traditional search returns *pages*. People need *answers*.

**AskDoc** bridges this gap. You upload your document — PDF, DOCX, PPTX, XLSX, CSV, or plain text — and ask questions in plain English. The AI reads, understands, retrieves the most relevant passages, and responds with **precise, cited, hallucination-resistant answers** grounded strictly in your document.

No more Ctrl+F. No more page flipping. Just ask.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🧠 **Multi-format Ingestion** | Upload PDF, DOCX, PPTX, XLSX, CSV, and TXT files |
| 💬 **Streaming Chat** | Real-time Server-Sent Events (SSE) token streaming |
| 🔍 **Hybrid Search** | Combines vector similarity search (Pinecone) + BM25 keyword search |
| 📊 **Intelligent Reranking** | FlashRank (local) or Reciprocal Rank Fusion (serverless) |
| 🧩 **LangGraph Pipeline** | Multi-step agentic RAG with intent classification, query rewriting, retrieval grading, and clarification |
| 📎 **Inline Citations** | Every answer includes `[1]`, `[2]` citations linked to source passages |
| 🔐 **BYOK (Bring Your Own Key)** | Users can use their own Gemini API key — no vendor lock-in |
| 📤 **Chat Export** | Export full conversation with sources as a Markdown report |
| 📈 **RAG Evaluation** | Gemini-as-judge scoring for relevance, accuracy, and completeness |
| 🌙 **Dark / Light Mode** | Full theme support across the UI |
| 🔒 **Auth via Supabase** | Optional user authentication with persistent chat history |
| ⚡ **Query Caching** | LRU cache with TTL for instant repeat answers |
| 🚀 **Serverless-Ready** | Deployed on Vercel with Vercel Postgres + Pinecone |

---

## 🏗️ Architecture Overview

```
User Question
      │
      ▼
┌─────────────────────────────────────────────────────────┐
│                  LangGraph StateGraph                    │
│                                                         │
│  classify_intent                                        │
│       │                                                 │
│  ┌────┴──────────────────┐                              │
│  │ direct_answer         │ rephrase_query               │
│  │ (greetings/general)   │      │                       │
│  └───────────────────────┘      ▼                       │
│                            retrieve                     │
│                          (Pinecone VSS                  │
│                         + BM25 keyword)                 │
│                                 │                       │
│                              rerank                     │
│                         (FlashRank / RRF)               │
│                                 │                       │
│                          grade_retrieval                │
│                       ┌─────────┴──────────┐            │
│                  rewrite_query        check_ambiguity   │
│                       │            ┌───────┴────────┐   │
│                    retrieve    clarify?         generate │
│                                   │             answer  │
│                            clarifying_question    │     │
│                                                   ▼     │
│                                            streamed SSE │
└─────────────────────────────────────────────────────────┘
```

**Tech Stack:**

| Layer | Technology |
|---|---|
| **Frontend** | Next.js 16, React 19, TypeScript, Tailwind CSS v4, Framer Motion |
| **Backend** | FastAPI, Python, LangChain, LangGraph |
| **LLM** | Google Gemini (via `langchain-google-genai`) |
| **Vector Store** | Pinecone (llama-text-embed-v2, 1024-dim) |
| **Database** | Supabase (PostgreSQL) via SQLAlchemy |
| **Auth** | Supabase Auth + JWT |
| **Reranking** | FlashRank (local) / Reciprocal Rank Fusion (serverless) |
| **Deployment** | Vercel (frontend + backend serverless functions) |

---

## 📁 Project Structure

```
askdoc/
├── client/                  # Next.js 16 frontend
│   ├── app/                 # App Router pages
│   │   ├── chat/            # Chat interface (session-based)
│   │   ├── docs/            # Documentation page
│   │   ├── evaluation/      # RAG evaluation dashboard
│   │   ├── privacy/         # Privacy policy
│   │   └── terms/           # Terms of service
│   ├── components/          # Reusable UI components
│   │   ├── home/            # Landing page sections
│   │   │   ├── HeroSection.tsx
│   │   │   ├── HowItWorksSection.tsx
│   │   │   ├── FeatureCardsSection.tsx
│   │   │   ├── ArchitecturePipelineSection.tsx
│   │   │   └── LiveDemoPreview.tsx
│   │   ├── ChatInterface.tsx
│   │   ├── ChatArea.tsx
│   │   ├── InputArea.tsx
│   │   ├── Sidebar.tsx
│   │   └── SettingsDialog.tsx
│   └── lib/                 # API client, Supabase client, hooks
│
└── server/                  # FastAPI backend
    ├── app/
    │   ├── api/             # REST API routes
    │   │   ├── chat.py      # Chat streaming, regenerate, export
    │   │   ├── documents.py # File upload & ingestion
    │   │   ├── sessions.py  # Session CRUD
    │   │   ├── evaluations.py # RAG evaluation endpoint
    │   │   └── schemas.py   # Pydantic request/response models
    │   ├── core/
    │   │   ├── rag_engine.py    # Main RAG orchestrator
    │   │   ├── graph.py         # LangGraph StateGraph builder
    │   │   ├── graph_nodes.py   # Individual pipeline nodes
    │   │   └── query_cache.py   # LRU response cache
    │   ├── document_processors/ # File parsers (PDF, DOCX, PPTX, XLSX, CSV)
    │   ├── services/
    │   │   ├── embeddings.py    # Pinecone Inference embeddings
    │   │   ├── search.py        # Hybrid search (vector + keyword)
    │   │   └── storage.py       # Supabase file storage
    │   ├── db/              # SQLAlchemy models & database session
    │   ├── auth.py          # Supabase JWT verification
    │   ├── middleware.py    # Rate limiting & error formatting
    │   └── resilience.py    # Retry decorators (tenacity)
    ├── main.py              # FastAPI app entrypoint
    ├── requirements.txt
    └── vercel.json          # Vercel serverless deployment config
```

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** 18+ and npm
- **Python** 3.10+
- A **Pinecone** account (free tier works) → [pinecone.io](https://pinecone.io)
- A **Google Gemini API Key** → [aistudio.google.com](https://aistudio.google.com)
- A **Supabase** project (for auth + database) → [supabase.com](https://supabase.com)

---

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/askdoc.git
cd askdoc
```

---

### 2. Backend Setup (`askdoc/server`)

```bash
cd askdoc/server

# Create a virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

Create a `.env` file inside `askdoc/server/`:

```env
# Google Gemini
GOOGLE_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-2.0-flash

# Pinecone
PINECONE_API_KEY=your_pinecone_api_key_here
PINECONE_INDEX_NAME=askdoc-index

# Supabase / PostgreSQL
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_KEY=your_supabase_service_role_key
DATABASE_URL=postgresql+psycopg2://postgres:password@db.your-project.supabase.co:5432/postgres

# Security
JWT_SECRET=your_supabase_jwt_secret
```

Run the backend:

```bash
uvicorn main:app --reload --port 8000
```

The API will be live at `http://localhost:8000`. Swagger docs at `http://localhost:8000/docs`.

---

### 3. Frontend Setup (`askdoc/client`)

```bash
cd askdoc/client

# Install dependencies
npm install
```

Create a `.env.local` file inside `askdoc/client/`:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

Run the frontend:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 🔄 How It Works (The RAG Pipeline)

1. **Upload** — Drop a PDF, DOCX, PPTX, XLSX, CSV, or TXT file. The backend parses it with structure-aware extractors, chunks it (1800 chars, 300 overlap), embeds every chunk using Pinecone's `llama-text-embed-v2` model, and stores vectors in Pinecone + chunk text in Postgres.

2. **Ask** — Type a question. The LangGraph pipeline kicks in:
   - **Classify Intent**: Is this a general greeting or a document question?
   - **Rephrase Query**: Fix typos, expand abbreviations, normalize numerals (e.g., "7th" → "Seventh" → "VII") for better retrieval recall.
   - **Hybrid Retrieve**: Fetch top-15 chunks via Pinecone vector similarity + keyword BM25 search, then fuse results with Reciprocal Rank Fusion.
   - **Rerank**: Apply FlashRank (locally) or RRF (serverless) to re-order chunks by relevance.
   - **Grade Retrieval**: Check if retrieved docs are actually relevant. If not, rewrite the query and retry (up to 2 times).
   - **Check Ambiguity**: If the question is too vague, ask a clarifying question instead.
   - **Generate Answer**: Stream the response token-by-token using Gemini with inline `[1][2]` source citations.

3. **Evaluate** — Run the built-in evaluation suite: the system auto-generates test questions, queries itself, then uses Gemini-as-judge to score relevance, accuracy, and completeness (0–100).

---

## 🌐 Deployment

Both `server` and `client` are configured for **Vercel** deployment.

### Deploy Backend (Vercel Serverless)

```bash
cd askdoc/server
vercel --prod
```

Set all `.env` variables in the Vercel project dashboard.

### Deploy Frontend

```bash
cd askdoc/client
vercel --prod
```

Set `NEXT_PUBLIC_API_URL` to your deployed backend URL.

---

## 📡 API Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/sessions` | Create a new chat session |
| `GET` | `/sessions` | List all sessions for a user |
| `DELETE` | `/sessions/{id}` | Delete a session and all its vectors |
| `POST` | `/sessions/{id}/upload` | Upload & ingest a document |
| `POST` | `/sessions/{id}/chat` | Stream a chat response (SSE) |
| `POST` | `/sessions/{id}/regenerate` | Regenerate the last AI response |
| `GET` | `/sessions/{id}/export` | Export chat as Markdown |
| `POST` | `/sessions/{id}/evaluate` | Run RAG evaluation on the session |
| `GET` | `/health` | Health check |

---

## 🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

```bash
# Fork the repo, then:
git checkout -b feature/your-feature-name
git commit -m "feat: add your feature"
git push origin feature/your-feature-name
# Open a PR on GitHub
```

---

## 👤 Author

**Tushar Avhad**  
🐙 [GitHub](https://github.com/Tushar260603) · 💼 [LinkedIn](https://www.linkedin.com/in/tushar-avhad-845367438)

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">
  <sub>Built with ❤️ by Tushar Avhad · FastAPI · LangGraph · Pinecone · Google Gemini · Next.js</sub>
</div>
