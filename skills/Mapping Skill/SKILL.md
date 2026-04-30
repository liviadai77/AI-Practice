# EDI Mapping Skill & Standards (Spec → Excel)

## 1. Purpose

This document defines the standard workflow to convert **Trading Partner EDI Specifications (PDF)** into a **structured Excel Mapping File**.

Goal:
- Ensure consistency across mappings
- Avoid missing or incorrect fields
- Standardize mapping output format

---

## 2. Input & Output

### 2.1 Input
- Trading Partner EDI Specification (PDF)
- Document Types:
  - 940 (Warehouse Shipping Order)
  - 945 (Warehouse Shipping Advice)
  - Others (850 / 856 / etc.)

---

### 2.2 Output
- Excel Mapping File (.xlsx)

Each segment must be expanded into rows with structured columns.

---

## 3. Extraction Rules (PDF → Structured Data)

For each **Segment (e.g., REF, N1, W05)**:

### 3.1 Identify Segment
- Segment Name (e.g., REF)
- Segment Description

---

### 3.2 Extract Elements

Each segment contains multiple elements:

Example:
REF01, REF02, REF03...

---

### 3.3 Required Fields to Extract

| Column | Description | Source in PDF |
|--------|------------|--------------|
| Segment | Segment Name (e.g., REF) | Segment header |
| Segment Element | e.g., REF01 | Element table |
| Element ID | e.g., 128 | "ID" column |
| Min/Max Length | e.g., 2/30 | Min/Max |
| Element Name | e.g., Reference Identification Qualifier | Name column |
| Required | M / O | Req column |
| Data Type | ID / AN / N | Type column |
| Description | Explanation text | Notes/Description |
| Mapping Field | OMS/WMS field | Derived |

---

## 4. Excel Output Structure

Each row represents ONE element.

### Standard Columns:

| Column Name | Description |
|------------|------------|
| Segment |
| Segment Element |
| Element ID |
| Min/Max |
| Element Name |
| Required |
| Type |
| Description |
| OMS/WMS Mapping |

---

## 5. Description Formatting Rule

If PDF provides value explanation like:

IA Internal Vendor Number

Must be formatted as:

IA - Internal Vendor Number

---

## 6. Mapping Rules (Critical)

### 6.1 No Fabrication Rule ❗
- DO NOT invent fields
- DO NOT skip fields
- MUST follow PDF exactly

---

### 6.2 Full Coverage Rule
- Every element (e.g., REF01, REF02...) must be included
- Even optional fields must be listed

---

### 6.3 Mapping Field Assignment

Mapping field must be:
- Based on business meaning
- Based on system knowledge (OMS/WMS)

Example:
REF02 → referenceNo  
N102 → shipToName  

---

### 6.4 Unknown Fields

If mapping is unclear:

- Leave blank OR mark as:
  TBD

DO NOT guess.

---

## 7. Segment Handling Rules

### 7.1 Repeat Segments
- Must still list all elements once
- Do not duplicate rows unnecessarily

---

### 7.2 Loop Awareness
- Keep segment context (e.g., N1 loop vs header)
- Add notes if needed

---

## 8. Validation Checklist

Before finalizing Excel:

- [ ] All segments covered  
- [ ] All elements extracted  
- [ ] No missing required fields  
- [ ] No fabricated data  
- [ ] Description formatted correctly  
- [ ] Mapping fields reviewed  

---

## 9. Common Mistakes (Anti-Patterns)

❌ Skipping optional fields  
❌ Guessing mapping fields  
❌ Ignoring Min/Max  
❌ Mixing segment contexts  
❌ Not standardizing descriptions  

---

## 10. Example

Input (PDF):
Segment: REF

| Ref | ID | Req | Type | Min/Max | Name |
|-----|----|-----|------|---------|------|
| REF01 | 128 | M | ID | 2/3 | Reference Identification Qualifier |

Output (Excel Row):

| Segment | Element | ID | Min/Max | Name | Req | Type | Description | Mapping |
|--------|--------|----|---------|------|-----|------|------------|--------|
| REF | REF01 | 128 | 2/3 | Reference Identification Qualifier | M | ID | IA - Internal Vendor Number | vendorCode |

---

## 11. Summary

This process ensures:
- 100% spec coverage
- Zero ambiguity
- Standardized mapping output

Mapping quality = Spec understanding + Structured extraction + Controlled interpretation