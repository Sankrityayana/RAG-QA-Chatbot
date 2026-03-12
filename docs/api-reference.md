# API & Code Reference

This document describes all functions defined in `qabot.py`, their parameters, return types, and purpose.

---

## Module: `qabot.py`

---

### `get_llm()`

Initializes and returns a WatsonX LLM instance backed by Mistral Medium.

**Parameters:** None

**Returns:** `WatsonxLLM` — a LangChain-compatible LLM wrapper

**Configuration:**

| Parameter         | Value                               | Description                                  |
|-------------------|-------------------------------------|----------------------------------------------|
| `model_id`        | `mistralai/mistral-medium-2505`     | WatsonX model identifier                     |
| `url`             | `https://us-south.ml.cloud.ibm.com` | WatsonX regional endpoint                    |
| `project_id`      | `"skills-network"`                  | WatsonX project ID                           |
| `MAX_NEW_TOKENS`  | `256`                               | Maximum number of tokens in the LLM response |
| `TEMPERATURE`     | `0.5`                               | Generation temperature (0=deterministic, 1=creative) |

**Example:**
```python
llm = get_llm()
response = llm.invoke("What is RAG?")
```

---

### `document_loader(file)`

Loads a PDF file and returns its content as a list of LangChain `Document` objects.

**Parameters:**

| Name  | Type         | Description                              |
|-------|--------------|------------------------------------------|
| `file` | `gr.File` (or any object with `.name`) | Uploaded file object from Gradio |

**Returns:** `List[Document]` — one `Document` per page of the PDF

**Notes:**
- Uses `PyPDFLoader` from `langchain_community`.
- Each `Document` includes `page_content` (text) and `metadata` (page number, source path).

**Example:**
```python
docs = document_loader(file)
print(docs[0].page_content)   # Text of page 1
print(docs[0].metadata)       # {'page': 0, 'source': '/path/to/file.pdf'}
```

---

### `text_splitter(data)`

Splits a list of documents into smaller, overlapping text chunks.

**Parameters:**

| Name   | Type             | Description                                  |
|--------|------------------|----------------------------------------------|
| `data` | `List[Document]` | Documents returned by `document_loader`       |

**Returns:** `List[Document]` — smaller chunk documents

**Configuration:**

| Parameter        | Value  | Description                                         |
|------------------|--------|-----------------------------------------------------|
| `chunk_size`     | `1000` | Maximum characters per chunk                        |
| `chunk_overlap`  | `50`   | Character overlap between consecutive chunks         |
| `length_function`| `len`  | Function used to measure chunk length                |

**Notes:**
- `RecursiveCharacterTextSplitter` splits on `["\n\n", "\n", " ", ""]` in order, preserving sentence boundaries as much as possible.
- Overlap ensures that context at chunk boundaries is not lost during retrieval.

---

### `watsonx_embedding()`

Initializes and returns a WatsonX Embeddings model instance.

**Parameters:** None

**Returns:** `WatsonxEmbeddings` — a LangChain-compatible embeddings wrapper

**Configuration:**

| Parameter               | Value                               | Description                                    |
|-------------------------|-------------------------------------|------------------------------------------------|
| `model_id`              | `ibm/slate-125m-english-rtrvr-v2`   | IBM Slate embedding model                      |
| `url`                   | `https://us-south.ml.cloud.ibm.com` | WatsonX regional endpoint                      |
| `project_id`            | `"skills-network"`                  | WatsonX project ID                             |
| `TRUNCATE_INPUT_TOKENS` | `3`                                 | Truncation setting for tokens exceeding limits |
| `RETURN_OPTIONS`        | `{"input_text": True}`              | Returns original text with embedding response  |

**Notes:**
- IBM Slate 125M is a bi-encoder model optimized for retrieval tasks.
- Produces 768-dimensional vectors.

---

### `vector_database(chunks)`

Creates an in-memory ChromaDB vector store from document chunks and their embeddings.

**Parameters:**

| Name     | Type             | Description                                      |
|----------|------------------|--------------------------------------------------|
| `chunks` | `List[Document]` | Text chunks from `text_splitter`                  |

**Returns:** `Chroma` — a LangChain-compatible vector store with similarity search support

**Notes:**
- ChromaDB is initialized in-memory (not persisted to disk).
- Each call creates a new, empty vector store — there is no cross-call caching.

---

### `retriever(file)`

Full pipeline: loads the PDF, splits it, embeds it, and returns a retriever object.

**Parameters:**

| Name   | Type      | Description                          |
|--------|-----------|--------------------------------------|
| `file` | `gr.File` | Uploaded PDF file from Gradio         |

**Returns:** `VectorStoreRetriever` — LangChain retriever for similarity-based document lookup

**Pipeline:**
```
document_loader(file)
    → text_splitter(splits)         [splits into chunks]
    → vector_database(chunks)       [embeds and indexes]
    → vectordb.as_retriever()       [returns retriever]
```

> **Note:** There is currently a redundancy in `retriever` — `vector_database` is called twice. The second call re-embeds chunks that are already `Document` objects from the first call's output. This does not break functionality but adds unnecessary latency.

---

### `retriever_qa(file, query)`

Main entry point for the RAG QA pipeline. Called by Gradio on each user submission.

**Parameters:**

| Name    | Type      | Description                          |
|---------|-----------|--------------------------------------|
| `file`  | `gr.File` | Uploaded PDF file from Gradio         |
| `query` | `str`     | User's natural language question      |

**Returns:** `str` — the LLM-generated answer

**Pipeline:**
```
get_llm()                           [initialize LLM]
retriever(file)                     [build RAG retriever]
RetrievalQA.from_chain_type(...)    [create QA chain]
qa.invoke(query)                    [run the chain]
return response['result']           [extract answer text]
```

> **Known bug:** The code contains `RetrievalQA.form_chain_type(...)` — note the typo `form` instead of `from`. This will raise an `AttributeError` at runtime. See [setup.md](./setup.md#troubleshooting) for the fix.

---

## Gradio Interface — `rag_application`

The Gradio interface is configured as follows:

| Property        | Value                                                    |
|-----------------|----------------------------------------------------------|
| `fn`            | `retriever_qa`                                           |
| `allow_flagging`| `"never"`                                                |
| `title`         | `"RAG Chatbot"`                                          |
| `description`   | Displayed below the title in the UI                      |
| `inputs[0]`     | `gr.File` — PDF upload, single file, `.pdf` type only    |
| `inputs[1]`     | `gr.Textbox` — 2-line query input with placeholder text  |
| `outputs`       | `gr.Textbox` — displays the generated answer             |

### Launch Configuration

```python
rag_application.launch(
    server_name = "127.0.0.1",   # localhost only
    server_port = 7860,           # port
    share = True                  # creates temporary public Gradio link
)
```
