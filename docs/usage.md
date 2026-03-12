# Usage Guide

This guide explains how to use the RAG QA Chatbot once it is running.

---

## Launching the App

From the project root with the virtual environment activated:

```bash
python qabot.py
```

The Gradio server starts on `http://127.0.0.1:7860`. Open that URL in any modern browser.

A temporary public Gradio share URL is also printed in the terminal (e.g. `https://xxxx.gradio.live`). This URL is valid for 72 hours and accessible from anywhere — useful for demos. To disable it, set `share=False` in `qabot.py`.

---

## Step-by-Step Walkthrough

### 1. Upload a PDF Document

Click the **"Upload PDF File"** button (or drag and drop) to upload any PDF.

**Supported types:** `.pdf` only  
**File count:** single file per session

![Upload area](../docs/assets/upload.png)

After upload, the file name appears below the upload area confirming receipt.

---

### 2. Enter Your Question

Click into the **"Input Query"** text box and type your question in natural language. You can use multi-line input (Shift+Enter for new line).

**Examples of good questions:**
- *"What is the main conclusion of this document?"*
- *"Summarize the key findings in chapter 3."*
- *"What methodology was used in the study?"*
- *"List all the recommendations mentioned."*
- *"What does the document say about data privacy?"*

---

### 3. Submit and Wait

Click the **Submit** button (or press Enter). The chatbot will:

1. Parse and chunk the PDF
2. Embed chunks and build the vector index
3. Embed the query and retrieve the most relevant chunks
4. Send query + context to WatsonX AI (Mistral Medium)
5. Return the generated answer

Processing time depends on PDF size and network latency to WatsonX AI — typically **5–20 seconds** for a standard document.

---

### 4. Read the Answer

The answer appears in the **"Output"** text box below the inputs. The answer is grounded in the document content retrieved by the RAG pipeline.

---

## Tips for Best Results

| Tip | Details |
|-----|---------|
| **Ask specific questions** | Narrow, specific questions yield better answers than broad ones. |
| **Reference document sections** | Mentioning a chapter, section, or topic helps the retriever focus. |
| **Keep questions concise** | Shorter, clear queries produce more relevant retrievals. |
| **Try rephrasing** | If the answer is poor, rephrase the question differently. |
| **Check the PDF quality** | Scanned/image-only PDFs will not work — text must be selectable. |

---

## Limitations

- **Single document per session**: Each submission re-processes the PDF from scratch. There is no cross-session memory.
- **Max answer length**: Capped at 256 tokens (~200 words). Increase `MAX_NEW_TOKENS` in `qabot.py` for longer answers.
- **PDF must be text-based**: Image PDFs (scans without OCR) are not supported. Use a tool like Adobe Acrobat or `pytesseract` to OCR them first.
- **No conversation history**: Each query is independent. The chatbot does not remember previous questions in the same session.
- **Rate limits**: Responses are subject to IBM WatsonX AI API rate limits based on your subscription plan.
