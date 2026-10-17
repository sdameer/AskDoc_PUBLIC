# AskDoc — AI-Powered PDF Question Answering

AskDoc is an AI-powered document question-answering application that allows users to upload PDF files and ask questions about their contents. It uses **Retrieval-Augmented Generation (RAG)** to retrieve relevant document chunks and generate answers using a Large Language Model (LLM).

**Live Demo:** https://tinyurl.com/sdameer


## Features

- Upload and process PDF documents.
- Ask questions about uploaded documents using natural language.
- Retrieve relevant document chunks using semantic similarity search.
- Generate concise answers grounded in the uploaded document.
- Maintain chat history for follow-up questions.
- Handle greetings and casual conversations without document retrieval.
- Store uploaded PDF files using Cloudinary.
- Store application data in Neon PostgreSQL.
- Filter document retrieval using user and file metadata.

## Tech Stack

| Technology | Purpose |
|---|---|
| Python | Backend programming |
| Django | Backend framework |
| Django REST Framework | REST API development |
| LangChain | RAG pipeline orchestration |
| Groq API | LLM inference |
| `openai/gpt-oss-120b` | Language model |
| Hugging Face | Text embeddings |
| `BAAI/bge-small-en-v1.5` | Dense embedding model |
| Qdrant Cloud | Vector database |
| Neon PostgreSQL | Relational database |
| Cloudinary | PDF file storage |
| Railway | Application deployment |

## How It Works

1. **PDF Upload:** Users upload a PDF document.
2. **Text Extraction:** PyMuPDF extracts text and page information from the PDF.
3. **Text Chunking:** LangChain's `RecursiveCharacterTextSplitter` splits the extracted text into smaller, overlapping chunks.
4. **Embedding Generation:** The Hugging Face embedding model converts text chunks into dense vectors.
5. **Vector Storage:** Qdrant stores the vectors alongside document metadata.
6. **Similarity Search:** When a user asks a question, AskDoc retrieves relevant chunks from Qdrant using semantic similarity search.
7. **Answer Generation:** LangChain passes the retrieved context and relevant chat history to the Groq-hosted language model.
8. **Response:** The model generates a concise answer based on the retrieved document context.

Greetings and other small-talk messages are handled separately without performing document retrieval.

## Project Structure

```text
askdocs/
├── AskDoc/
│   ├── settings.py
│   ├── wsgi.py
│   ├── url.py
│   └── ...
├── authentication/
│   ├── serializers.py
│   ├── views.py
│   ├── url.py
│   └── ...
├── main_app/│  
│   ├── serializers.py
│   ├── views.py
│   ├── url.py
│   └── ...
├── rag/
│   ├── config.py
│   ├── dataloader.py
│   ├── pipeline.py
│   ├── prompts.py
│   └── vector_db.py
├── manage.py
└── requirements.txt
```

This structure highlights the main files in the project. Additional application files, migrations, templates, static assets, and configuration files may also be present.

## RAG Components

- **`config.py`** — Configures Qdrant and provides lazy-loaded Hugging Face embeddings.
- **`dataloader.py`** — Extracts PDF content, splits text into chunks, and attaches metadata.
- **`pipeline.py`** — Implements the RAG pipeline, chat history, small-talk handling, retrieval, and answer generation.
- **`prompts.py`** — Defines prompts for question rewriting and document-based answer generation.
- **`vector_db.py`** — Handles Qdrant collection initialization, document storage, similarity retrieval, and deletion using metadata filters.

## Current Retrieval Approach

AskDoc currently uses **dense-vector semantic similarity search**.

The `BAAI/bge-small-en-v1.5` embedding model converts document chunks and user queries into vectors. Qdrant retrieves relevant chunks based on vector similarity, while metadata filters restrict retrieval to the appropriate user's document.

### Planned Improvement: Hybrid Search

Hybrid search combining dense semantic retrieval and sparse keyword retrieval is planned for a future update. **Hybrid search has not been implemented yet.**

## Limitations

- The current PDF loader processes only the first 15 pages of each PDF.
- Retrieval currently uses dense-vector similarity search.
- Answer quality depends on successful PDF text extraction and relevant retrieval.
- AI-generated responses may be incorrect and should be verified against the original document.


## 📊 RAG Pipeline Evaluation with RAGAS

To evaluate the reliability and answer quality of AskDoc's Retrieval-Augmented Generation (RAG) pipeline, I implemented an automated evaluation workflow using **RAGAS** and a test dataset of 22 questions based on `attention.pdf`.

The evaluation measures answer faithfulness, relevance, retrieval quality, and the system's ability to refuse questions that cannot be answered from the uploaded document.

### Evaluation Setup

- **Evaluation Framework:** RAGAS
- **LLM Judge:** `qwen/qwen3.8-27b` via Groq
- **Test Dataset:** 22 questions
- **Document:** Attention Is All You Need (`attention.pdf`)
- **Similarity Threshold:** Used to filter retrieved chunks based on relevance.
- **Evaluation Metrics:** Faithfulness, Answer Relevancy, Context Precision, and Context Recall.

### 📈 Evaluation Results

| Metric | Score |
|---|---:|
| Faithfulness | 0.711 |
| Answer Relevancy | 0.754 |
| Context Precision | 0.704 |
| Context Recall | 0.727 |

*Scores range from 0 to 1, with higher values generally indicating better performance.*

### 🔍 Retrieval & Reliability Results

| Evaluation | Result |
|---|---:|
| Expected-page hit rate | 16/16 (100%) |
| Top-1 expected-page hit rate | 10/16 (62.5%) |
| Correct refusals for unanswerable questions | 4/4 (100%) |
| False refusals for answerable questions | 2/18 |

- **Retrieval validation:** Relevant document pages were retrieved for all 16 answerable questions.
- **Similarity filtering:** A similarity threshold helps prevent irrelevant chunks from being passed to the generation stage.
- **Hallucination control:** The pipeline correctly refused all four unanswerable questions in this test set.
- **Identified improvement areas:** Retrieval ranking, context precision, faithfulness, and false refusals.

----