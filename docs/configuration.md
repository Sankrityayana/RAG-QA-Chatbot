# Configuration Reference

All configurable values in the RAG QA Chatbot are currently hardcoded in `qabot.py`. This document lists every configuration parameter, its default value, and guidance on customizing it.

---

## LLM Configuration (`get_llm`)

| Parameter          | Default Value                         | How to Change                                              |
|--------------------|---------------------------------------|------------------------------------------------------------|
| `model_id`         | `mistralai/mistral-medium-2505`       | Replace with any WatsonX-compatible LLM model ID           |
| `url`              | `https://us-south.ml.cloud.ibm.com`   | Change to your WatsonX region endpoint                     |
| `project_id`       | `"skills-network"`                    | Set to your IBM WatsonX project ID                         |
| `MAX_NEW_TOKENS`   | `256`                                 | Increase for longer answers (up to the model's max limit)  |
| `TEMPERATURE`      | `0.5`                                 | `0.0` = deterministic, `1.0` = very creative               |

### Available WatsonX LLM Models (examples)

| Model ID                               | Notes                            |
|----------------------------------------|----------------------------------|
| `mistralai/mistral-medium-2505`        | Default — balanced capability    |
| `mistralai/mistral-large`              | More capable, higher latency     |
| `ibm/granite-13b-chat-v2`             | IBM Granite chat model           |
| `meta-llama/llama-3-3-70b-instruct`   | Llama 3.3 70B instruction-tuned  |

Check the [WatsonX AI Model Catalog](https://dataplatform.cloud.ibm.com/wx/samples/models) for the full list of available models.

---

## Embedding Configuration (`watsonx_embedding`)

| Parameter               | Default Value                         | How to Change                                        |
|-------------------------|---------------------------------------|------------------------------------------------------|
| `model_id`              | `ibm/slate-125m-english-rtrvr-v2`     | Replace with another WatsonX embedding model ID      |
| `url`                   | `https://us-south.ml.cloud.ibm.com`   | Change to your WatsonX region endpoint               |
| `project_id`            | `"skills-network"`                    | Set to your IBM WatsonX project ID                   |
| `TRUNCATE_INPUT_TOKENS` | `3`                                   | Increase to avoid truncating longer text chunks      |
| `RETURN_OPTIONS`        | `{"input_text": True}`                | Set to `False` to skip returning original text       |

### Available WatsonX Embedding Models

| Model ID                             | Dims | Notes                           |
|--------------------------------------|------|---------------------------------|
| `ibm/slate-125m-english-rtrvr-v2`   | 768  | Default — English retrieval     |
| `ibm/slate-30m-english-rtrvr-v2`    | 384  | Smaller/faster variant          |

---

## Text Splitter Configuration (`text_splitter`)

| Parameter         | Default | Guidance                                                             |
|-------------------|---------|----------------------------------------------------------------------|
| `chunk_size`      | `1000`  | Characters per chunk. Smaller = more granular retrieval. Larger = more context per chunk. |
| `chunk_overlap`   | `50`    | Characters of overlap between chunks. Increase to avoid losing context at boundaries. |
| `length_function` | `len`   | Use `len` for character-based splitting (default and recommended).    |

**Tuning guidance:**

- For short, dense technical documents: `chunk_size=500`, `chunk_overlap=100`
- For long narrative documents: `chunk_size=1500`, `chunk_overlap=150`
- Overlap should be ~5–10% of chunk size.

---

## Vector Store Configuration (`vector_database`)

ChromaDB is used in in-memory mode. No persistent configuration is required.

To enable persistence across sessions, modify `vector_database` to use a directory:

```python
from chromadb.config import Settings

def vector_database(chunks):
    embedding_model = watsonx_embedding()
    vectordb = Chroma.from_documents(
        chunks,
        embedding_model,
        persist_directory="./chroma_store"   # persist to disk
    )
    return vectordb
```

---

## Retriever Configuration

The retriever uses default similarity search (`cosine` distance). To customize the number of retrieved chunks:

```python
retriever = vectordb.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 5}   # return top 5 most similar chunks (default is 4)
)
```

---

## RetrievalQA Chain Configuration (`retriever_qa`)

| Parameter               | Default   | Notes                                                             |
|-------------------------|-----------|-------------------------------------------------------------------|
| `chain_type`            | `"stuff"` | Concatenates all chunks into one prompt. See alternatives below.  |
| `return_source_documents` | `False` | Set to `True` to also return the source chunks used.              |

### Chain Type Options

| Chain Type      | Description                                                             |
|-----------------|-------------------------------------------------------------------------|
| `"stuff"`       | All retrieved chunks stuffed into one prompt. Simple, works for short docs. |
| `"map_reduce"` | Each chunk answered separately, then combined. Better for long docs.    |
| `"refine"`      | Iteratively refines the answer chunk by chunk. Most thorough.           |
| `"map_rerank"` | Scores and ranks chunk answers, returns the best one.                   |

---

## Gradio Server Configuration

In `rag_application.launch()`:

| Parameter     | Default       | Description                                            |
|---------------|---------------|--------------------------------------------------------|
| `server_name` | `"127.0.0.1"` | Bind to localhost. Use `"0.0.0.0"` to expose on LAN.  |
| `server_port` | `7860`        | HTTP port. Change if 7860 is in use.                   |
| `share`       | `True`        | Creates a temporary public Gradio link. Set `False` in production. |

---

## WatsonX Regions

If you are not in `us-south`, use the appropriate endpoint:

| Region          | URL                                    |
|-----------------|----------------------------------------|
| US South        | `https://us-south.ml.cloud.ibm.com`    |
| EU (Frankfurt)  | `https://eu-de.ml.cloud.ibm.com`       |
| UK (London)     | `https://eu-gb.ml.cloud.ibm.com`       |
| JP (Tokyo)      | `https://jp-tok.ml.cloud.ibm.com`      |
| US East         | `https://us-east.ml.cloud.ibm.com`     |
