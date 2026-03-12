# Architecture — RAG QA Chatbot

This document describes the end-to-end architecture of the RAG QA Chatbot, including the data flow, component responsibilities, and integration points.

---

## Overview

The application follows a **Retrieval-Augmented Generation (RAG)** pattern. Instead of relying solely on a large language model's pre-trained knowledge, it grounds its answers in the actual content of a user-supplied PDF document.

At a high level:
1. A PDF is loaded and split into manageable text chunks.
2. Each chunk is embedded into a high-dimensional vector and stored in a vector database.
3. When a user asks a question, the most semantically similar chunks are retrieved.
4. Those chunks (context) plus the user's question are passed to an LLM, which generates a grounded answer.

---

## Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Gradio UI                               │
│        (browser at http://127.0.0.1:7860)                       │
└──────────────┬───────────────────────────┬──────────────────────┘
               │ PDF File Upload            │ Text Query
               ▼                           ▼
     ┌──────────────────┐        ┌──────────────────────┐
     │  PyPDFLoader     │        │   WatsonX LLM        │
     │  (document_      │        │   mistral-medium-2505│
     │   loader)        │        │   (get_llm)          │
     └────────┬─────────┘        └──────────┬───────────┘
              │ List[Document]               │
              ▼                             │
     ┌──────────────────┐                   │
     │ RecursiveCharacter│                  │
     │ TextSplitter      │                  │
     │ chunk_size=1000   │                  │
     │ chunk_overlap=50  │                  │
     └────────┬──────────┘                  │
              │ List[Document chunks]        │
              ▼                             │
     ┌──────────────────────────┐           │
     │  WatsonxEmbeddings       │           │
     │  ibm/slate-125m-english- │           │
     │  rtrvr-v2                │           │
     └────────┬─────────────────┘           │
              │ Embedding vectors            │
              ▼                             │
     ┌──────────────────┐                   │
     │   ChromaDB       │◄──── Similarity ──┘
     │  (Vector Store)  │      Retrieval
     │  (in-memory)     │      (Top-K chunks)
     └──────────────────┘           │
                                    ▼
                         ┌──────────────────────┐
                         │  RetrievalQA Chain   │
                         │  (chain_type="stuff")│
                         │  context + query     │
                         │   → LLM → answer     │
                         └──────────┬───────────┘
                                    │ answer text
                                    ▼
                              Gradio Output
```

---

## Component Breakdown

### 1. Gradio Interface (`rag_application`)
- **Role**: Front-end web UI. Accepts a PDF file and a text query from the user and displays the LLM's answer.
- **Inputs**: `gr.File` (PDF), `gr.Textbox` (question)
- **Output**: `gr.Textbox` (answer)
- **Served at**: `http://127.0.0.1:7860` with optional public Gradio share link.

### 2. Document Loader (`document_loader`)
- **Library**: `langchain_community.document_loaders.PyPDFLoader`
- **Role**: Reads the uploaded PDF and returns a list of `Document` objects, one per page.
- **Output type**: `List[Document]`

### 3. Text Splitter (`text_splitter`)
- **Library**: `langchain_text_splitters.RecursiveCharacterTextSplitter`
- **Role**: Splits large documents into overlapping chunks to fit within embedding and LLM context windows.
- **Parameters**:
  - `chunk_size = 1000` characters
  - `chunk_overlap = 50` characters
  - `length_function = len` (character-based length)

### 4. Embedding Model (`watsonx_embedding`)
- **Library**: `langchain_ibm.WatsonxEmbeddings`
- **Model**: `ibm/slate-125m-english-rtrvr-v2`
- **Role**: Converts text chunks into dense numerical vectors for semantic similarity search.
- **Parameters**:
  - `TRUNCATE_INPUT_TOKENS: 3` — truncates input to avoid token overflow
  - `RETURN_OPTIONS: {"input_text": True}` — returns original text alongside embeddings

### 5. Vector Store (`vector_database`)
- **Library**: `langchain_community.vectorstores.Chroma`
- **Role**: Stores document chunk embeddings in-memory. Provides similarity search to retrieve the most relevant chunks for a given query.
- **Persistence**: In-memory only (no disk persistence between sessions).

### 6. Retriever (`retriever`)
- **Role**: Orchestrates document loading → text splitting → embedding → vector database indexing, then exposes a retriever interface (`vectordb.as_retriever()`) for similarity-based chunk lookup.

### 7. LLM (`get_llm`)
- **Library**: `langchain_ibm.WatsonxLLM`
- **Model**: `mistralai/mistral-medium-2505`
- **Role**: Generates a natural language answer given a query and retrieved context.
- **Parameters**:
  - `MAX_NEW_TOKENS: 256` — maximum length of generated response
  - `TEMPERATURE: 0.5` — balanced between determinism and creativity

### 8. RetrievalQA Chain (`retriever_qa`)
- **Library**: `langchain.chains.RetrievalQA`
- **Chain type**: `"stuff"` — concatenates all retrieved chunks into a single prompt context.
- **Role**: Combines the retriever and LLM into an end-to-end QA chain. Invokes the retriever for context, constructs a prompt, and calls the LLM for the final answer.

---

## IBM WatsonX AI Integration

Both the LLM and the embedding model are hosted on **IBM WatsonX AI** (`us-south.ml.cloud.ibm.com`).

| Service       | Endpoint                              | Model ID                          |
|---------------|---------------------------------------|-----------------------------------|
| LLM Inference | `https://us-south.ml.cloud.ibm.com`   | `mistralai/mistral-medium-2505`   |
| Embeddings    | `https://us-south.ml.cloud.ibm.com`   | `ibm/slate-125m-english-rtrvr-v2` |

Authentication is handled via IBM WatsonX AI credentials bound to `project_id = "skills-network"`.

---

## Limitations & Notes

- **In-memory vector store**: ChromaDB is initialized fresh on every request. There is no caching between different PDF uploads.
- **Context window**: The `"stuff"` chain type concatenates all retrieved chunks. For very large documents with many relevant sections, this may approach the LLM's context limit.
- **Token truncation**: `TRUNCATE_INPUT_TOKENS: 3` on the embedding model is a minimal truncation setting; adjust based on your document language and chunk size.
- **Gradio share link**: `share=True` in `rag_application.launch()` creates a temporary public Gradio link — disable this in production.
