# Setup & Installation Guide

This guide walks through everything needed to get the RAG QA Chatbot running locally from scratch.

---

## Prerequisites

| Requirement           | Version / Notes                                                   |
|-----------------------|-------------------------------------------------------------------|
| Python                | 3.10 or higher                                                    |
| pip                   | Latest recommended (`pip install --upgrade pip`)                  |
| IBM WatsonX AI access | Active account with a project ID and API key                      |
| Internet connection   | Required for WatsonX AI API calls                                 |

---

## Step 1 — Clone the Repository

```bash
git clone https://github.com/Sankrityayana/RAG-QA-Chatbot.git
cd RAG-QA-Chatbot
```

---

## Step 2 — Create a Virtual Environment

It is strongly recommended to use a virtual environment to avoid dependency conflicts.

**Windows:**

```cmd
python -m venv my_env
my_env\Scripts\activate
```

**macOS / Linux:**

```bash
python3 -m venv my_env
source my_env/bin/activate
```

You should see `(my_env)` prefixed to your terminal prompt once activated.

---

## Step 3 — Install Dependencies

```bash
pip install --upgrade pip
pip install ibm-watsonx-ai langchain langchain-ibm langchain-community langchain-text-splitters chromadb pypdf gradio
```

### Full dependency list

| Package                    | Purpose                                      |
|----------------------------|----------------------------------------------|
| `ibm-watsonx-ai`           | IBM WatsonX AI SDK (LLM + Embeddings client) |
| `langchain`                | Core LangChain framework (chains, prompts)   |
| `langchain-ibm`            | LangChain integration for IBM WatsonX        |
| `langchain-community`      | Community loaders and vector stores          |
| `langchain-text-splitters` | Text chunking utilities                      |
| `chromadb`                 | In-memory vector store                       |
| `pypdf`                    | PDF parsing backend for PyPDFLoader          |
| `gradio`                   | Web UI framework                             |

---

## Step 4 — Configure IBM WatsonX AI Credentials

Open `qabot.py` and update the following fields with your own IBM WatsonX AI credentials:

```python
# In get_llm()
project_id = "your-watsonx-project-id"
url = "https://us-south.ml.cloud.ibm.com"   # change region if needed

# In watsonx_embedding()
project_id = "your-watsonx-project-id"
url = "https://us-south.ml.cloud.ibm.com"
```

### Setting the API Key

**Option A — Environment variable (recommended):**

```bash
# Windows cmd
set WATSONX_APIKEY=your-api-key-here

# Windows PowerShell
$env:WATSONX_APIKEY="your-api-key-here"

# macOS / Linux
export WATSONX_APIKEY="your-api-key-here"
```

**Option B — `.env` file:**

Create a `.env` file in the project root:

```text
WATSONX_APIKEY=your-api-key-here
```

Then load it at the top of `qabot.py`:

```python
from dotenv import load_dotenv
load_dotenv()
```

> **Security note:** Never commit your API key to source control. The `.gitignore` already excludes `.env` files.

---

## Step 5 — Run the Application

```bash
python qabot.py
```

Expected output:

```text
Running on local URL:  http://127.0.0.1:7860
Running on public URL: https://xxxx.gradio.live  (temporary share link)
```

Open `http://127.0.0.1:7860` in your browser.

---

## Troubleshooting

| Problem                              | Likely Cause & Fix                                                                |
|--------------------------------------|-----------------------------------------------------------------------------------|
| `ModuleNotFoundError`                | Dependency not installed. Run `pip install <package-name>`.                       |
| `401 Unauthorized` from WatsonX      | Invalid or missing API key. Check `WATSONX_APIKEY` env variable.                  |
| `404` on model ID                    | Model not available in your region/plan. Verify model ID on WatsonX AI console.   |
| Port 7860 already in use             | Change `server_port` in `rag_application.launch()` to another port (e.g. `7861`). |
| `AttributeError: form_chain_type`    | Typo in source — should be `from_chain_type`. Fix in `qabot.py`.                  |
| Slow responses                       | Normal for first request — ChromaDB is built fresh each time.                     |
