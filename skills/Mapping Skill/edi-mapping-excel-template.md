---
name: edi-mapping-excel-template
description: Standard output template for converting EDI specification (X12 940/945 etc.) into structured OMS/WMS mapping matrix. This template defines the ONLY allowed output format for all EDI mapping transformations.
metadata:
  author: edi-team
  version: "1.0.0"
---

# EDI Mapping Excel Template (Standard Output Format)

---

## 1. Purpose

This template defines the **strict output structure** for all EDI mapping results.

It is the **single source of truth** for:
- Column definitions
- Column order
- Formatting rules
- Output consistency

---

## 2. Output Format Rules (STRICT)

All EDI mapping outputs MUST:

- Follow this exact column structure
- Maintain column order strictly
- Include ALL columns (no omission allowed)
- Avoid any additional columns
- Use one row per EDI element

❗ No deviation is allowed

---

## 3. Column Definitions (MANDATORY STRUCTURE)

| Column Name | Description |
|------------|-------------|
| Segment | EDI Segment ID (e.g., REF, N1, DTM) |
| Segment Element | Element identifier (e.g., REF01, N101) |
| ID | Data Element ID from EDI spec |
| Min/Max | Field length constraint (e.g., 2/3, 1/60) |
| Element Name | Official EDI field name |
| Req | Requirement flag (M / O / X) |
| ID Code | Code list value (if applicable) |
| Description | Normalized description |
| OMS/WMS Mapping | Target system mapping field |

---

## 4. Formatting Rules

### 4.1 Segment Column
- Must be uppercase (REF, N1, DTM)
- Must match EDI spec exactly

---

### 4.2 Segment Element
- Must follow original spec format
- Examples:
  - REF01
  - N101
  - W0501

---

### 4.3 Description Normalization Rule

If source text is:

IA Internal Vendor Number  

It MUST be formatted as:

IA - Internal Vendor Number  

---

### 4.4 Mapping Field Rules

- Use camelCase for system fields
- Prefer OMS fields by default
- If unclear → use "TBD"
- No guessing allowed

Examples:
- orderNo
- shipToAddress[].name
- referenceNo

---

### 4.5 Hardcode Rule

If fixed value is required:

Must be written as:

Hardcode "VALUE"

---

## 5. Data Integrity Rules

- One row per element only
- No merging of elements
- No skipping optional fields
- Must preserve full EDI structure

---

## 6. Missing Data Handling

If any field is missing:

| Column | Value |
|--------|------|
| Description | MISSING DATA - REQUIRES REVIEW |
| OMS/WMS Mapping | TBD |

❗ Missing data MUST still be included in output

---

## 7. Segment Grouping Rule

- Segments must be grouped logically
- Loop context (e.g., N1 Loop) must be preserved via segment order

---

## 8. Example Output

| Segment | Segment Element | ID | Min/Max | Element Name | Req | ID Code | Description | OMS/WMS Mapping |
|---------|----------------|----|---------|--------------|-----|--------|-------------|------------------|
| REF | REF01 | 128 | 2/3 | Reference Identification Qualifier | M | IA | IA - Internal Vendor Number | referenceType |
| REF | REF02 | 127 | 1/30 | Reference Identification | O | | | referenceNo |
| N1 | N101 | 98 | 2/3 | Entity Identifier Code | M | ST | ST - Ship To | ST |
| N1 | N102 | 93 | 1/60 | Name | X | | | shipToAddress[].name |

---

## 9. Validation Checklist

Before final output:

- [ ] All columns exist
- [ ] Column order is correct
- [ ] No extra columns exist
- [ ] No missing elements
- [ ] Description normalized
- [ ] Mapping fields validated