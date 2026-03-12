# RAG QA Chatbot

A Retrieval-Augmented Generation (RAG) powered question-answering chatbot that allows users to upload a PDF document and ask natural language questions about its contents. Built using IBM WatsonX AI, LangChain, ChromaDB, and Gradio.

---

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | System design, data flow diagram, and component breakdown |
| [Setup & Installation](docs/setup.md) | Full installation and credential configuration guide |
| [Usage Guide](docs/usage.md) | How to use the chatbot UI, tips, and limitations |
| [API Reference](docs/api-reference.md) | All functions, parameters, and return types documented |
| [Configuration Reference](docs/configuration.md) | Every tunable parameter and its options |

---

## Features

- Upload any PDF document and query its contents interactively
- Retrieval-Augmented Generation pipeline for context-aware answers
- IBM WatsonX AI LLM (`mistralai/mistral-medium-2505`) for response generation
- IBM WatsonX Embeddings (`ibm/slate-125m-english-rtrvr-v2`) for semantic search
- ChromaDB as the in-memory vector store
- Simple and intuitive Gradio web interface

---

## Architecture

```
PDF Upload
    │
    ▼
PyPDFLoader  ──►  RecursiveCharacterTextSplitter  ──►  ChromaDB (Vector Store)
                                                              │
                                                             ▼
User Query  ──────────────────────────────────►  Retriever (Similarity Search)
                                                              │
                                                             ▼
                                               WatsonX LLM (Mistral Medium)
                                                              │
                                                             ▼
                                                       Answer Output
```

> See [docs/architecture.md](docs/architecture.md) for the full detailed architecture with component descriptions.

---

## Tech Stack

| Component         | Technology                                      |
|-------------------|-------------------------------------------------|
| LLM               | IBM WatsonX — `mistralai/mistral-medium-2505`   |
| Embeddings        | IBM WatsonX — `ibm/slate-125m-english-rtrvr-v2` |
| Vector Store      | ChromaDB                                        |
| PDF Loader        | LangChain `PyPDFLoader`                         |
| Text Splitter     | LangChain `RecursiveCharacterTextSplitter`      |
| RAG Chain         | LangChain `RetrievalQA`                         |
| UI Framework      | Gradio                                          |
| Orchestration     | LangChain                                       |

---

## Prerequisites

- Python 3.10+
- Access to [IBM WatsonX AI](https://www.ibm.com/products/watsonx-ai) with a valid `project_id` and API credentials

---

## Quick Start

1. **Clone the repository**

   ```bash
   git clone https://github.com/Sankrityayana/RAG-QA-Chatbot.git
   cd RAG-QA-Chatbot
   ```

2. **Create and activate a virtual environment**

   ```bash
   python -m venv my_env
   # Windows
   my_env\Scripts\activate
   # macOS/Linux
   source my_env/bin/activate
   ```

3. **Install dependencies**

   ```bash
   pip install ibm-watsonx-ai langchain langchain-ibm langchain-community langchain-text-splitters chromadb pypdf gradio
   ```

4. **Set your WatsonX API key**

   ```bash
   # Windows
   set WATSONX_APIKEY=your-api-key-here
   # macOS/Linux
   export WATSONX_APIKEY=your-api-key-here
   ```

5. **Run the app**

   ```bash
   python qabot.py
   ```

   Open `http://127.0.0.1:7860` in your browser.

> For detailed setup instructions, see [docs/setup.md](docs/setup.md).

---

## Usage

1. **Upload a PDF** using the file upload area
2. **Type your question** in the query box
3. **Click Submit** and wait for the answer
4. The chatbot retrieves relevant context from the PDF and generates a grounded answer

> For tips, examples, and limitations, see [docs/usage.md](docs/usage.md).

---

## Project Structure

```
RAG-QA-Chatbot/
├── qabot.py                    # Main application — RAG pipeline and Gradio UI
├── README.md                   # Project overview (this file)
├── .gitignore                  # Git ignore rules
└── docs/
    ├── architecture.md         # System architecture and data flow
    ├── setup.md                # Installation and configuration guide
    ├── usage.md                # Usage guide and tips
    ├── api-reference.md        # Function and interface documentation
    └── configuration.md        # All configurable parameters
```

---

## How It Works

1. **Document Loading** — The uploaded PDF is parsed using `PyPDFLoader`.
2. **Text Splitting** — The document is split into overlapping chunks (1000 chars, 50 char overlap) using `RecursiveCharacterTextSplitter`.
3. **Embedding & Indexing** — Each chunk is embedded using IBM WatsonX Embeddings and stored in a ChromaDB vector store.
4. **Retrieval** — On user query, the most semantically similar chunks are retrieved from the vector store.
5. **Generation** — The retrieved context and query are passed to the WatsonX LLM (`mistral-medium-2505`) to generate a grounded answer.
6. **Display** — The answer is shown in the Gradio interface.

> For a deep dive into each component, see [docs/architecture.md](docs/architecture.md).

---

## License

This project is licensed under the [MIT License](LICENSE).
