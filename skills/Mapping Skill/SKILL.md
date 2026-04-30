---
name: edi-mapping-spec-to-mapping-matrix
description: Transform EDI specification documents (X12 940/945 etc.) from PDF into structured OMS/WMS Mapping Matrix using a two-stage parsing architecture (Index + Lazy Segment Parsing). MUST strictly follow edi-mapping-excel-template.md as the only output format definition.
metadata:
  author: edi-team
  version: "3.0.0"
---

# Skill: EDI Specification to Mapping Matrix Transformer

---

## 0.1 Sequential Page Processing (CORE RULE)

Processing MUST follow strict order:

👉 Page 1 → Process → Output  
👉 Page 2 → Process → Append  
👉 Page 3 → Process → Append  

---

### Execution Rules:

- Only ONE page processed at a time
- Must finish mapping before moving to next page
- Results MUST be appended incrementally
- Maintain global context across pages

---

## 0.2 Visual Extraction Model (CRITICAL)

Each page is treated as a **visual structure (not text PDF)**.

Extraction sources:

- Segment headers (ISA/ GS / N1 / DTM / W05 / LIN / HL)
- Table rows
- Code lists under elements
- Min/Max + Req fields

---

## 0.3 Segment Continuity Rule

Cross-page logic MUST be preserved:

- N1 Loop continues across pages
- REF / LIN / HL relationships must persist
- OMS/WMS field naming must remain consistent

Example:

channelSalesOrderNo must remain identical across all pages

---

## 0.4 Incremental Output Rule (APPEND MODE)

Each page output MUST:

- Be appended to previous result
- Never overwrite previous rows
- Preserve global table continuity


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

## 5. Mapping Rules (UNIFIED RAG + PDF + CONTROL SYSTEM)

This section defines the **single unified mapping decision system**, combining:

- PDF extraction rules (structural source)
- Google Drive historical mapping (SYSTEM OF RECORD - RAG)
- Semantic inference (fallback)
- Hardcode rules (control segments only)

---

## 5.1 Global Mapping Priority (ABSOLUTE ORDER)

All OMS/WMS mapping decisions MUST follow this priority chain:

| Priority | Source | Description |
|----------|--------|-------------|
| 1 | Google Drive Historical Mapping (RAG) | SYSTEM OF RECORD (highest priority) |
| 2 | PDF Explicit Content | Structural truth source |
| 3 | Semantic Inference | Only when no historical mapping exists |
| 4 | Hardcode Rules | Only for control segments (ISA / GS / SE / IEA) |

---

## 5.2 No Hallucination Rule (HIGHEST PRIORITY)

- DO NOT invent fields
- DO NOT create new OMS/WMS schema
- ONLY use:
  - PDF explicit fields
  - Google Drive historical mapping
  - Strict fallback rules (TBD only)

---

## 5.3 Google Drive Mapping Rule (RAG CORE RULE)

Before ANY mapping decision:

### MUST execute:

- Call Google Drive mapping retrieval skill
- Search historical EDI mapping files
- Retrieve similar segment/field mappings

---

### Retrieval Scope:

- 940 / 945 / 856 mapping files
- OMS/WMS historical mappings
- Enterprise mapping templates

---

### Decision Logic:

| Condition | Action |
|----------|--------|
| Exact match found | Use existing mapping directly |
| Partial match found | Adapt existing pattern |
| No match found | Use semantic inference |
| Still unclear | Output "TBD" |

---

👉 Google Drive is treated as SYSTEM OF RECORD

---

## 5.4 Segment-Specific Mapping Rules

### A. Control Segments (STRICT MODE)

Segments:

- ISA
- GS
- SE
- IEA

### Rules:

- ONLY allow Hardcode mapping
- DO NOT output OMS/WMS mapping unless explicitly required

### Format:

Hardcode "VALUE"

Example:

| Segment | Segment Element | OMS/WMS Mapping |
|---------|----------------|------------------|
| ISA | ISA06 | Hardcode "SENDER_ID" |

---

### B. Business Segments (FULL RAG MODE)

Segments:

- REF
- N1
- DTM
- W05
- LIN
- HL

### Rules:

- MUST use Google Drive historical mapping FIRST
- Semantic inference ONLY if no match exists

---

## 5.5 Mapping Normalization Rule

When using Google Drive mappings:

- Normalize naming conventions
- Align with OMS standards
- Reuse canonical field names

Examples:

- shipToAddress[].name  
- orderNo  
- referenceNo  
- channelSalesOrderNo  

---

## 5.6 Hardcode Rule (CONTROL SEGMENTS ONLY)

Applies ONLY to:

- ISA
- GS
- SE
- IEA

Rules:

- ONLY allowed mapping type = Hardcode
- Format strictly:

Hardcode "VALUE"

---

## 5.7 No-Invention Rule (STRICT FALLBACK RULE)

If no mapping exists in:

- Google Drive
- PDF
- Semantic inference

Then MUST return:

"TBD (No historical mapping found)"

---

## 5.8 System Principle (FINAL GOVERNANCE RULE)

- Google Drive = SYSTEM OF RECORD (highest authority)
- PDF = structural truth source
- Semantic = fallback only
- Hardcode = control segments only

👉 Mapping is a **retrieval-first decision system**, not generation

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