# AI Document Processing with LLMWhisperer

This repository explores OCR and intelligent document extraction using [LLMWhisperer](https://unstract.com/llmwhisperer/) — a document processing API designed to produce LLM-ready text from complex, real-world documents. Each notebook is a self-contained case study that benchmarks LLMWhisperer against [pytesseract](https://github.com/madmaze/pytesseract), the popular open-source OCR engine.

---

## What is LLMWhisperer?

LLMWhisperer converts PDFs, scanned images, Excel files, and other documents into clean, layout-preserved text that can be fed directly into LLMs. Unlike traditional OCR, it handles:

- Handwritten and printed mixed content
- Complex multi-column layouts and tables
- Checkboxes and form fields
- Skewed/misaligned scans
- Multilingual documents with special characters
- Native Excel files with hierarchical headers

**Core API call:**

```python
from unstract.llmwhisperer import LLMWhispererClientV2

client = LLMWhispererClientV2(api_key=LLMWHISPERER_API_KEY)

result = client.whisper(
    file_path="docs/my-document.pdf",
    wait_for_completion=True,
    wait_timeout=300,
    output_mode="layout_preserving",
    mode="form"
)

text = result["extraction"]["result_text"]
```

---

## Case Studies

### Case Study 1 — Scanned Handwritten Insurance Form
**Notebook:** [`llmwhisperer-scanned-handwritten.ipynb`](notebooks/llmwhisperer-scanned-handwritten.ipynb)  
**Document:** ACORD commercial insurance application (scanned + handwritten fill-ins)

#### Challenge
The document combines printed form fields with handwritten entries, making it extremely difficult for standard OCR to distinguish and preserve the structure.

#### Code Snippet
```python
result = client.whisper(
    file_path="docs/accord-insurance-handwritten-scanned-copy.pdf",
    wait_for_completion=True,
    wait_timeout=300,
    output_mode="layout_preserving"
)
text = result["extraction"]["result_text"]
```

#### Results

| Metric | LLMWhisperer | Pytesseract |
|--------|-------------|-------------|
| Characters extracted | **8,573** | 2,791 |
| Lines | **103** | 60 |
| Processing time | — | 3.99s |
| Layout preservation | ✅ Yes | ❌ Poor |
| Handwriting accuracy | ✅ High | ❌ Low |

> **Word overlap between the two outputs: 80.1%** — the remaining 20% is content that only LLMWhisperer captured correctly from the handwritten sections.

---

### Case Study 2 — Income Tax Form & Photographed Hotel Receipt
**Notebook:** [`llmwhisperer-forms-and-receipts.ipynb`](notebooks/llmwhisperer-forms-and-receipts.ipynb)  
**Documents:** Scanned PDF tax form (IRS Form 5500-EZ) + photographed hotel receipt

#### Challenge
Two distinct problems in one notebook:
1. A **scanned tax form** with checkboxes, multi-column sections, and dense layout
2. A **photographed receipt** with uneven lighting, handwritten values, and faded print

#### Code Snippet
```python
# Tax form extraction
result = client.whisper(
    file_path="docs/handwritten-filled-tax-form-photograph.pdf",
    wait_for_completion=True,
    wait_timeout=300,
    output_mode="layout_preserving",
    mode="form"
)
```

#### Extraction Mode Comparison: `form` vs `high_quality`

This notebook also compares two LLMWhisperer extraction modes on the same tax form:

| Mode | Characters | Processing Time | Checkboxes |
|------|-----------|-----------------|------------|
| `form` | 6,327 | 88.26s | ✅ `[X]` / `[ ]` preserved |
| `high_quality` | 6,450 | 57.11s | ❌ Brackets dropped |

**Key finding:** `form` mode correctly preserved checkbox state (`[X]` for checked, `[ ]` for unchecked) — critical for tax forms. `high_quality` mode fixed a date OCR error (`01/02/202 Z` → `01/02/2022`) but lost all checkbox markers. **Use `form` mode for structured forms with checkboxes.**

#### LLMWhisperer vs Pytesseract

| Metric | LLMWhisperer | Pytesseract |
|--------|-------------|-------------|
| **Tax Form** — characters | **7,066** | 3,080 |
| **Tax Form** — processing time | 11.31s | 1.76s |
| **Hotel Receipt** — characters | **2,314** | 1,175 |
| **Hotel Receipt** — processing time | 36.30s | 1.61s |

**Key fields found (Hotel Receipt):**

| Field | LLMWhisperer | Pytesseract |
|-------|-------------|-------------|
| TAX INVOICE | ✅ | ❌ |
| Room No | ✅ | ❌ |
| Guest Name | ✅ | ✅ |
| Total Due | ✅ | ✅ |

---

### Case Study 3 — Multilingual (German–English) Insurance Form
**Notebook:** [`llmwhisperer-multilingual-german.ipynb`](notebooks/llmwhisperer-multilingual-german.ipynb)  
**Document:** Health insurance application form with mixed German and English text

#### Challenge
The document contains German-language labels alongside English values, German umlauts (ä, ö, ü, ß), and dual-language instructions — all in a dense form layout.

#### Code Snippet
```python
result = client.whisper(
    file_path="docs/Insurance-form-german.pdf",
    wait_for_completion=True,
    wait_timeout=300,
    output_mode="layout_preserving"
)
```

#### Results

| Metric | LLMWhisperer | Pytesseract |
|--------|-------------|-------------|
| Characters extracted | **6,691** | 3,059 |
| Processing time | 11.14s | 2.09s |
| Special characters (ä/ö/ü/ß) | **33** | 9 |
| Key fields detected | 7 / 8 | 7 / 8 |
| Currency values found | ✅ High accuracy | ❌ Partial |

> LLMWhisperer correctly rendered German umlauts and preserved the association between German labels and their English-filled values. Pytesseract extracted only 9 special characters vs 33 — many German words were mangled.

---

### Case Study 4 — Scanned Misaligned Logistics Packing List
**Notebook:** [`llmwhisperer-misaligned-packing-list.ipynb`](notebooks/llmwhisperer-misaligned-packing-list.ipynb)  
**Document:** Multi-column packing list scanned at a slight angle

#### Challenge
The document was scanned with page skew, making column alignment unreliable. It contains a dense shipment table with quantities, weights, and dimensions across multiple columns.

#### Code Snippet
```python
result = client.whisper(
    file_path="docs/scanned-misaligned-logistics_packing_list (1).pdf",
    wait_for_completion=True,
    wait_timeout=300,
    output_mode="layout_preserving"
)
```

#### Results

| Metric | LLMWhisperer | Pytesseract |
|--------|-------------|-------------|
| Characters extracted | **2,547** | 1,254 |
| Lines | 36 | 33 |
| Processing time | 11.70s | 1.63s |
| Skew correction | ✅ Automatic | ❌ Manual required |
| Column alignment | ✅ Preserved | ❌ Fragmented |

**Sample LLMWhisperer output (packing list header):**
```
Packing List

Shipper/ Exporter: Faculty of Arts    Ultimate Consignee: Herald Corp    Bill To: Faculty of Arts
           5 Washington                      28 OLD                  5 Washington Square S,
           Square S, New York,               BROMPTON                New York, NY 10012, USA
           NY 10012, USA
```

> Despite the misalignment, LLMWhisperer correctly placed the three-column shipper/consignee header side by side, which pytesseract rendered as a jumbled single column.

---

### Case Study 5 — Excel Insurance Performance Table
**Notebook:** [`llmwhisperer-excel-insurance-performance.ipynb`](notebooks/llmwhisperer-excel-insurance-performance.ipynb)  
**Document:** Excel file with complex hierarchical tables (Singapore Life Insurance P&L)

#### Challenge
The Excel file contains merged cells, hierarchical column headers spanning multiple rows, and numerical data across 106 rows and 26 columns — structures that traditional OCR (image-based) cannot handle natively.

#### Code Snippet
```python
result = client.whisper(
    file_path="docs/insurance-performance (2).xlsx",
    wait_for_completion=True,
    wait_timeout=300,
    output_mode="layout_preserving",
    mode="table"
)
text = result["extraction"]["result_text"]
```

> LLMWhisperer processes Excel files natively (no image conversion needed), preserving merged headers and table structure so the data can be directly consumed by an LLM for analysis or Q&A.

---

## Setup

```bash
# Clone the repo
git clone https://github.com/bhargobdeka/AI-document-processing.git
cd AI-document-processing

# Install dependencies
pip install unstract-llmwhisperer python-dotenv pytesseract Pillow pdf2image matplotlib langchain-openai

# Add your API key
echo "LLMWHISPERER_API_KEY=your_key_here" > .env
```

Get a free LLMWhisperer API key at [unstract.com/llmwhisperer](https://unstract.com/llmwhisperer/).

---

## Overall Findings

Across all five case studies, LLMWhisperer consistently extracted **2–3× more characters** than pytesseract and preserved document structure that pytesseract could not recover. The trade-off is processing time: pytesseract is faster (1–4s) while LLMWhisperer takes 11–90s depending on document complexity.

| Document Type | LLMWhisperer Advantage |
|---------------|----------------------|
| Handwritten forms | Accurate handwriting + layout in one pass |
| Tax / structured forms | Checkbox state detection (`[X]` / `[ ]`) |
| Multilingual documents | Correct special characters and diacritics |
| Misaligned scans | Automatic skew correction |
| Excel / tabular files | Native file parsing, no image conversion |
| Photographed receipts | Handles uneven lighting and faded print |
