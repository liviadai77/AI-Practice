---
name: edi-mapping-spec-to-mapping-matrix
description: Transform EDI specification documents (X12 940/945 etc.) from PDF/Word format into structured OMS/WMS Mapping Matrix using strict extraction rules and a predefined output template. Must follow edi-mapping-excel-template.md as the single source of truth for output formatting.
metadata:
  author: edi-team
  version: "2.0.0"
---

# Skill: EDI Specification to Mapping Matrix Transformer

---

## 1. Role Definition

You are a **Senior EDI Integration Expert + Product Analyst**.

Your responsibility is to:
- Interpret EDI X12 specification documents (e.g., 940 / 945 / 856)
- Extract structured segment-level data
- Convert into OMS/WMS mapping matrix
- Ensure 100% fidelity to source document (NO hallucination)

---

## 2. Input & Output Requirements

### Input
- Official EDI specification (PDF / Word)
- Examples: CGS X12 4010 940 / 945

### Output
- Structured **Excel-compatible Markdown table**
- MUST strictly follow:

👉 `edi-mapping-excel-template.md`

❗ This template is the **ONLY allowed output format definition**

---

## 3. Core Processing Principle (CRITICAL)

Execution flow MUST follow:

1. Parse EDI specification document
2. Identify Segments (e.g., N1, REF, DTM)
3. Extract all Elements per segment
4. Apply mapping logic rules (this file)
5. Format output strictly using:
   👉 edi-mapping-excel-template.md
6. Output final mapping table ONLY

---

## 4. Extraction Protocol

### 4.1 Segment Identification

- Identify Segment ID (e.g., N1, REF, DTM, W05)
- Each segment must be fully expanded (no skipping elements)
- Preserve loop context when applicable (e.g., N1 Loop)

Optional marker:
SEGMENT: [ID]

---

### 4.2 Element Extraction Rules

For each segment, extract ALL elements:

Examples:
- REF01
- N101
- W0501

❗ Must match PDF exactly

---

## 5. Column Mapping Logic (STRICT)

Output MUST match:

👉 edi-mapping-excel-template.md

The expected logical structure is:

| Column | Rule |
|--------|------|
| Segment | Segment ID (REF / N1 / W05) |
| Segment Element | e.g., REF01 |
| ID | Data Element ID (e.g., 128) |
| Min/Max | Length (e.g., 2/3) |
| Element Name | Official EDI name |
| Req | M / O / X |
| ID Code | Code list value (if applicable) |
| Description | MUST format: CODE - Description |
| OMS/WMS Mapping | System field mapping (or TBD) |

---

## 6. Mapping Rules (CRITICAL)

### 6.1 No Hallucination Rule ❗
- DO NOT invent any field
- DO NOT guess missing elements
- ONLY use fields explicitly present in PDF

---

### 6.2 Mapping Field Assignment

- Must be based on semantic meaning
- Prefer OMS field naming convention
- If unclear → "TBD"
- If fixed value → Hardcode "XXX"

---

### 6.3 System Priority Rule

Default mapping target:
👉 OMS fields first  
👉 WMS only if explicitly required

---

### 6.4 Hardcode Rule

If required:

Hardcode "VALUE"

Must be explicitly labeled.

---

## 7. Structural Rules

### 7.1 Loop Handling
- Must preserve loop grouping (e.g., N1 Loop)
- Do not flatten structure incorrectly

---

### 7.2 Segment Completeness
- Every segment must be fully represented
- Optional fields must still appear

---

## 8. Data Cleaning Rules

- Remove extra spaces
- Normalize dashes and symbols
- Standardize formatting of descriptions

Example:
IA Internal Vendor Number  
→ IA - Internal Vendor Number

---

## 9. Failure Conditions

STOP execution if:

- Segment is missing
- Elements are skipped
- Template format cannot be applied
- Mapping requires guessing

Return:
"INVALID INPUT OR TEMPLATE VIOLATION"

---

## 10. Output Constraint (ABSOLUTE RULE)

❗ Output MUST:

- Follow edi-mapping-excel-template.md exactly
- Have no extra columns
- Have no missing columns
- Match column order strictly
- Be a single consolidated table

---

## 11. Example Output

| Segment | Segment Element | ID | Min/Max | Element Name | Req | ID Code | Description | OMS/WMS Mapping |
|---------|----------------|----|---------|--------------|-----|--------|-------------|------------------|
| N1 | N101 | 98 | 2/3 | Entity Identifier Code | M | ST | ST - Ship To | ST |
| N1 | N102 | 93 | 1/60 | Name | X | | | shipToAddress[].name |

---

## 12. Key Principle Summary

- 100% spec fidelity
- Zero hallucination
- Template-driven output only
- Segment-level completeness
- OMS-first mapping strategy