# RAG API

A local retrieval-augmented generation (RAG) API that stores profile text in ChromaDB, retrieves relevant passages, and uses Ollama to answer questions from that context.

[![Contributors](https://img.shields.io/github/contributors/ashfaqfardin/local-rag-api-with-ollama)](https://github.com/ashfaqfardin/local-rag-api-with-ollama/graphs/contributors)
[![Issues](https://img.shields.io/github/issues/ashfaqfardin/local-rag-api-with-ollama)](https://github.com/ashfaqfardin/local-rag-api-with-ollama/issues)
[![Good First Issues](https://img.shields.io/github/issues/ashfaqfardin/local-rag-api-with-ollama/good%20first%20issue)](https://github.com/ashfaqfardin/local-rag-api-with-ollama/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)
[![Last Commit](https://img.shields.io/github/last-commit/ashfaqfardin/local-rag-api-with-ollama)](https://github.com/ashfaqfardin/local-rag-api-with-ollama/commits)
[![Repository Size](https://img.shields.io/github/repo-size/ashfaqfardin/local-rag-api-with-ollama)](https://github.com/ashfaqfardin/local-rag-api-with-ollama)
[![License](https://img.shields.io/github/license/ashfaqfardin/local-rag-api-with-ollama)](https://github.com/ashfaqfardin/local-rag-api-with-ollama/blob/main/LICENSE)

## Features

- **Document ingestion**: Submit profile text through an HTTP endpoint; the API splits it into paragraphs and stores them in ChromaDB.
- **Persistent knowledge base**: ChromaDB saves the collection locally in `chroma_db/`.
- **Context-aware answers**: Retrieve up to two relevant passages and pass them to a local language model.
- **Optional user filtering**: Limit question retrieval to documents associated with a specific user.
- **Local model inference**: Uses Ollama for both embeddings and answer generation.

## Requirements

- Python
- Ollama installed and running
- Ollama models `nomic-embed-text` and `qwen2.5:0.5b`

## Setup

1. Clone the repository and enter the project directory:

   ```bash
   git clone <repository-url>
   cd rag-api
   ```

2. Create and activate a virtual environment:

   ```bash
   python -m venv .venv
   ```

   On Windows:

   ```powershell
   .venv\Scripts\Activate.ps1
   ```

   On macOS or Linux:

   ```bash
   source .venv/bin/activate
   ```

3. Install the dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Download the required Ollama models:

   ```bash
   ollama pull nomic-embed-text
   ollama pull qwen2.5:0.5b
   ```

   Make sure Ollama is running and available at `http://localhost:11434`.

## Build the knowledge base

Create a `profile.txt` file in the project directory with the text to index. The file is local input and is excluded from version control.

Run the ingestion script:

```bash
python build_knowledge_base.py
```

The script splits the file into paragraphs, creates embeddings with Ollama, and stores them in the `personal_profile` ChromaDB collection.

## Run the API

Start the development server from the project directory:

```bash
uvicorn main:app --reload
```

The API is available at `http://127.0.0.1:8000`. Interactive API documentation is available at `http://127.0.0.1:8000/docs`.

## API usage

### Add a document

Send profile text to `POST /documents`. The content is split on blank lines and stored as chunks associated with the supplied user name.

```bash
curl -X POST "http://127.0.0.1:8000/documents" \
  -H "Content-Type: application/json" \
  -d "{\"user_name\":\"alex\",\"content\":\"First paragraph.\\n\\nSecond paragraph.\"}"
```

The response includes the number of chunks added.

### Ask a question

Send a question to `GET /ask`. The optional `user` parameter filters retrieval to that user's chunks.

```bash
curl "http://127.0.0.1:8000/ask?question=What%20experience%20does%20Alex%20have%3F&user=alex"
```

The response includes the generated answer, the retrieved context, and the user filter that was applied.

## Project structure

```text
rag-api/
├── build_knowledge_base.py  # Build the ChromaDB collection from profile.txt
├── main.py                  # FastAPI application and API endpoints
├── requirements.txt         # Python dependencies
├── profile.txt              # Local source text (not committed)
└── chroma_db/               # Persistent vector database (not committed)
```

## Tech stack

- **FastAPI** and **Uvicorn**: HTTP API and ASGI server
- **ChromaDB**: Persistent vector storage and similarity search
- **Ollama**: Local embedding and language models
- **Pydantic**: Request data validation