---
name: edi-mapping-spec-to-mapping-matrix
description: Transform EDI specification documents (X12 940/945 etc.) from PDF into structured OMS/WMS Mapping Matrix using a two-stage parsing architecture (Index + Lazy Segment Parsing). MUST strictly follow edi-mapping-excel-template.md as the only output format definition.
metadata:
  author: edi-team
  version: "3.0.0"
---

# Skill: EDI Specification to Mapping Matrix Transformer

---

## 0. PDF Parsing Strategy (LEVEL 2 - PyMuPDF BASED IMPLEMENTATION)

This skill MUST use **PyMuPDF (fitz)** as the primary PDF parsing engine.

❗ No other PDF parsing method is allowed for Stage 1 indexing.

---

## 0.1 Stage 1 — Fast Segment Indexing (PyMuPDF REQUIRED)

Use PyMuPDF text extraction ONLY to perform fast scanning.

### Required Tool:
- fitz (PyMuPDF)

---

### Execution Rules:

1. Open PDF using PyMuPDF:
   - fitz.open(pdf)

2. Iterate page by page:
   - extract text only (no table parsing)

3. Detect Segment Headers using regex / keyword matching:

Target patterns:
- REF
- N1
- DTM
- W05
- LIN
- HL

---

### Output (Segment Index Map):

You MUST generate a mapping:

Segment → Page Number(s)

Example:

REF → Page 3  
N1  → Page 4  
DTM → Page 6  

---

### STRICT CONSTRAINTS:

❗ DO NOT extract tables in Stage 1  
❗ DO NOT parse element-level data in Stage 1  
❗ ONLY perform text-layer scanning using PyMuPDF  

---

## 0.2 Stage 2 — Lazy Segment Parsing (PyMuPDF REGION EXTRACTION)

After index is built:

For each segment:

1. Re-open PDF with PyMuPDF
2. Navigate to target page
3. Extract ONLY relevant text block region
4. Parse structured elements (REF01, N101, etc.)

---

### Parsing Strategy:

- Use page.get_text("blocks") OR page.get_text("dict")
- Identify table-like structure via coordinates
- Extract row/column structure from layout

---

### Example Execution Flow:

REF → locate page → extract block → parse REF01/REF02  
N1  → locate page → extract block → parse N101/N102  

---

## 0.3 Performance Rules (CRITICAL)

PyMuPDF must be used with following constraints:

- Page-level scanning first (fast mode)
- Block-level parsing only when needed
- No full document table extraction
- No OCR unless explicitly required

---

## 0.4 Recommended PyMuPDF Usage Pattern

```python id="fitz-example"
import fitz  # PyMuPDF

doc = fitz.open("edi_spec.pdf")

for page_num in range(len(doc)):
    page = doc[page_num]
    text = page.get_text("text")

    if "REF" in text or "N1" in text:
        print(page_num, "contains segment")
---

### Stage 1 — Fast Segment Indexing (MANDATORY)

Use a lightweight PDF text extraction method (e.g., PyMuPDF) to:

- Scan ONLY PDF text layer (no full table parsing)
- Detect Segment headers (e.g., REF, N1, DTM, W05)
- Build a Segment Index Map

Example:

REF → Page 3  
N1  → Page 4  
DTM → Page 6  

❗ DO NOT extract full tables in this stage  
❗ DO NOT parse element details in this stage  

---

### Stage 2 — Lazy Segment Parsing (MANDATORY)

For each segment in the index:

- Load ONLY the required segment region/page
- Extract elements (REF01, N101, etc.)
- Apply schema-based extraction per segment type
- Convert into structured intermediate data

Example:

REF → parse_ref_block()  
N1  → parse_n1_block()  
DTM → parse_dtm_block()  

---

### Performance Rules (CRITICAL)

- NEVER parse full PDF in one pass
- ONLY parse required segment blocks
- Use lazy loading per segment
- Parallel parsing is allowed when multiple segments exist

---

## 1. Role Definition

You are a **Senior EDI Integration Expert + Product Analyst**.

Your responsibilities:

- Interpret EDI X12 specification documents (940 / 945 / 856)
- Extract structured segment-level data
- Convert into OMS/WMS Mapping Matrix
- Ensure 100% fidelity to source document (NO hallucination)

---

## 2. Input & Output Requirements

### Input
- Official EDI specification (PDF / Word)
- Examples: CGS X12 4010 940 / 945

---

### Output
- Structured **Excel-compatible Markdown table**
- MUST strictly follow:

👉 edi-mapping-excel-template.md

❗ This template is the ONLY source of truth for output format

---

## 3. Core Processing Principle (LEVEL 2 ARCHITECTURE)

Execution MUST follow a 3-layer pipeline:

---

### Step 1 — Segment Indexing (FAST LAYER)

- Scan PDF using lightweight extraction
- Identify all segment headers
- Build index map (Segment → Page/Region)

Example:

REF → 3  
N1  → 4  
DTM → 6  

---

### Step 2 — Segment-Based Lazy Parsing (CORE LAYER)

For each segment:

1. Load only target segment block
2. Extract all elements (REF01, N101, etc.)
3. Apply mapping rules (this skill)
4. Store structured result

---

### Step 3 — Template Formatting (FINAL LAYER)

Format output strictly using:

👉 edi-mapping-excel-template.md

---

## 4. Extraction Protocol

### Segment Identification

- Identify Segment ID (REF, N1, DTM, W05)
- Each segment must be fully expanded
- Preserve loop context (e.g., N1 Loop)

Optional marker:

SEGMENT: [ID]

---

### Element Extraction Rules

Extract ALL elements exactly as defined in PDF:

- REF01
- N101
- W0501

❗ No skipping allowed  
❗ No guessing allowed  

---

## 5. Mapping Rules (CRITICAL)

### 5.1 No Hallucination Rule

- DO NOT invent fields
- DO NOT infer missing elements
- ONLY use data explicitly present in PDF

---

### 5.2 System Mapping Priority

Default target system:

👉 OMS first  
👉 WMS only if explicitly required  

---

### 5.3 Mapping Logic

- Based on semantic meaning
- Must align with system naming convention
- If unclear → "TBD"

Examples:
orderNo  
shipToAddress[].name  
referenceNo  

---

### 5.4 Hardcode Rule

If required:

Hardcode "VALUE"

Must be explicitly labeled.

---

## 6. Column Constraint (STRICT)

Output MUST strictly follow:

👉 edi-mapping-excel-template.md

Rules:

- No extra columns allowed
- No missing columns allowed
- Column order must NOT change
- Column names must NOT be modified

---

## 7. Data Cleaning Rules

- Remove extra spaces
- Normalize symbols
- Standardize description format:

IA Internal Vendor Number  
→ IA - Internal Vendor Number  

---

## 8. Missing Data Handling (STRUCTURED FALLBACK)

This skill MUST NOT stop execution when data is missing.

Instead, all missing or incomplete mapping information MUST be recorded in a structured fallback format.

---

### 8.1 Missing Segment Handling

If a Segment or Element cannot be fully parsed or identified:

It MUST be output as a standard placeholder row:

| Segment | Segment Element | ID | Min/Max | Element Name | Req | ID Code | Description | OMS/WMS Mapping |
|---------|----------------|----|---------|--------------|-----|--------|-------------|------------------|
| SEGMENT | [MISSING_SEGMENT] | | | MISSING DATA - REQUIRES REVIEW | | | INCOMPLETE EXTRACTION FROM SOURCE PDF | TBD |

---

### 8.2 Missing Element Handling

If individual elements are missing:

- Keep Segment context
- Fill missing fields with structured placeholders

Example:

| Segment | Segment Element | ID | Min/Max | Element Name | Req | ID Code | Description | OMS/WMS Mapping |
|---------|----------------|----|---------|--------------|-----|--------|-------------|------------------|
| REF | REF03 | | | MISSING ELEMENT | | | NOT FOUND IN PDF SPEC | TBD |

---

### 8.3 Mapping Uncertainty Handling

If OMS/WMS mapping cannot be determined:

- Do NOT guess
- Use explicit placeholder

Format:

TBD (requires functional review)

---

### 8.4 Review Requirement Tagging

All missing or incomplete entries MUST be marked as:

- Requires Review
- Incomplete Extraction
- Mapping Pending

These entries must remain in final output (no deletion allowed)

---

### 8.5 Principle

- NO execution halt
- NO data omission
- ALL missing information must be surfaced, not hidden

---

## 9. Output Constraint (ABSOLUTE RULE)

Final output MUST:

- Fully match edi-mapping-excel-template.md
- Contain no additional columns
- Contain no missing columns
- Maintain strict column order
- Be a single consolidated table

---

## 10. Example Output

| Segment | Segment Element | ID | Min/Max | Element Name | Req | ID Code | Description | OMS/WMS Mapping |
|---------|----------------|----|---------|--------------|-----|--------|-------------|------------------|
| N1 | N101 | 98 | 2/3 | Entity Identifier Code | M | ST | ST - Ship To | ST |
| N1 | N102 | 93 | 1/60 | Name | X | | | shipToAddress[].name |

---

## 11. Key Principles Summary

- 100% spec fidelity
- Zero hallucination
- Segment-level lazy parsing
- Template-driven output only
- Performance-first (Level 2 architecture)