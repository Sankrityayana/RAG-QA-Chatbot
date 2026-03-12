# RAG QA Chatbot

A Retrieval-Augmented Generation (RAG) powered question-answering chatbot that allows users to upload a PDF document and ask natural language questions about its contents. Built using IBM WatsonX AI, LangChain, ChromaDB, and Gradio.

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

## Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/Sankrityayana/RAG-QA-Chatbot.git
   cd RAG-QA-Chatbot
   ```

2. **Create a virtual environment**

   ```bash
   python -m venv my_env
   # Windows
   my_env\Scripts\activate
   # macOS/Linux
   source my_env/bin/activate
   ```

3. **Install dependencies**

   ```bash
   pip install ibm-watsonx-ai langchain langchain-ibm langchain-community chromadb pypdf gradio
   ```

---

## Configuration

In `qabot.py`, update the following with your IBM WatsonX AI credentials:

```python
# Replace with your actual WatsonX project ID and region URL
project_id = "your-project-id"
url = "https://us-south.ml.cloud.ibm.com"
```

If your WatsonX instance requires an API key, set it via environment variable or pass it to the `Credentials` object:

```bash
export WATSONX_APIKEY="your-api-key"
```

---

## Usage

1. **Run the application**

   ```bash
   python qabot.py
   ```

2. **Open the Gradio UI** in your browser at `http://127.0.0.1:7860`

3. **Upload a PDF** using the file upload input

4. **Type your question** in the query box and submit

5. The chatbot will retrieve relevant context from the PDF and generate an answer

---

## Project Structure

```
QA Bot RAG/
├── qabot.py        # Main application — RAG pipeline and Gradio UI
├── README.md       # Project documentation
└── my_env/         # Python virtual environment (not committed)
```

---

## How It Works

1. **Document Loading** — The uploaded PDF is parsed using `PyPDFLoader`.
2. **Text Splitting** — The document is split into overlapping chunks (1000 chars, 50 overlap) using `RecursiveCharacterTextSplitter`.
3. **Embedding & Indexing** — Each chunk is embedded using IBM WatsonX Embeddings and stored in a ChromaDB vector store.
4. **Retrieval** — On user query, the most semantically similar chunks are retrieved from the vector store.
5. **Generation** — The retrieved context and query are passed to the WatsonX LLM (`mistral-medium-2505`) to generate a grounded answer.
6. **Display** — The answer is shown in the Gradio interface.

---

## License

This project is licensed under the [MIT License](LICENSE).
