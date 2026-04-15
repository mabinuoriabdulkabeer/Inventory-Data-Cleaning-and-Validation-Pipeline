# 📊 Case Study: Building a Data Validation Pipeline for SKU Inventory Onboarding

> **28% of merchant data was not usable without validation.**

---

## 📸 Screenshots

### Raw vs Clean Data
| Before | After |
|--------|-------|
| ![Raw Dataset](https://github.com/mabinuoriabdulkabeer/Inventory-Data-Cleaning-and-Validation-Pipeline/blob/main/Raw%20Dataset.png) | ![Clean Dataset](https://github.com/mabinuoriabdulkabeer/Inventory-Data-Cleaning-and-Validation-Pipeline/blob/main/Cleaned%20Data.png) |

> *Left: Raw merchant data as received. Right: Validated, system-ready output.*

---

### Error Table

> *Sneakpeak of what the error table looks like with validation columns.*
![Error TAble](https://github.com/mabinuoriabdulkabeer/Inventory-Data-Cleaning-and-Validation-Pipeline/blob/main/Error%20Table.png)

---

### Error Breakdown
![Error Breakdown](https://github.com/mabinuoriabdulkabeer/Inventory-Data-Cleaning-and-Validation-Pipeline/blob/main/Error%20Reason.png)
> *Distribution of error types across the 2,000-row dataset.*

---

## 📊 By The Numbers

| Metric | Value |
|--------|-------|
| Total rows processed | **2,000** |
| Rows with at least one error | **~560 (28%)** |
| Duplicate SKUs | **~15–20% of rows** |
| Missing Price values | **~12% of rows** |
| Missing Stock values | **~7% of rows** |
| Missing SKUs | **~2.5% of rows** |
| Rows with Price = 0 | **~50 rows** |
| Fully blank rows | **~20 rows** |
| Distinct date formats found | **6 formats** |
| Supplier name variants collapsed | **14 variants → 6 clean names** |
| Category variants collapsed | **20+ variants → 7 clean categories** |

> Out of 2,000 rows, **28% had at least one data quality issue** that would have caused a failure, a wrong value, or a broken record if uploaded directly into a system.

---

## 🧭 Context

In retail and fintech systems, product data is the foundation of everything — from transactions to inventory tracking and reporting.

However, merchant-provided data is often inconsistent, incomplete, and error-prone. Without proper validation, this leads to:

- Failed product onboarding
- Incorrect inventory levels
- Broken reporting pipelines
- Poor customer experience

This project simulates a real-world onboarding scenario where messy SKU-level data must be cleaned, validated, and prepared for system upload.

---

## 🎯 Problem Statement

Given a messy SKU inventory dataset, how can we:

- Ensure data accuracy before system upload
- Identify and classify data quality issues
- Prevent operational failures caused by bad data
- Create a scalable validation process

---

## 📂 Dataset Overview

The dataset contains **2,000 rows** of product-level records across 7 fields:

| Field | Description |
|-------|-------------|
| `Product_Name` | Name of the retail product |
| `SKU` | Unique product identifier |
| `Category` | Product category (e.g. Electronics, Clothing) |
| `Price` | Product price in Naira |
| `Stock_Quantity` | Units available in inventory |
| `Supplier` | Name of the supplying merchant |
| `Date_Added` | Date the product was onboarded |

### Observed Issues

Initial data profiling revealed **15 distinct data quality problems** across all 7 fields, spanning structural errors, formatting inconsistencies, and invalid values:

| # | Issue | Volume / Example |
|---|-------|-----------------|
| 1 | Duplicate SKUs | ~15–20% of rows |
| 2 | Missing Price | ~12% of rows |
| 3 | Missing Stock Quantity | ~7% of rows |
| 4 | Missing SKU | ~2.5% of rows |
| 5 | Corrupt Price formats | `₦5,000`, `5k`, `abc`, `ten thousand`, `TBD`, `5,000` |
| 6 | Corrupt Stock values | `many`, `low`, `-`, `in stock` |
| 7 | Negative Stock values | e.g. `-5`, `-10` |
| 8 | Extreme Stock values | e.g. `9999` (likely placeholder entry) |
| 9 | Price = 0 | ~50 rows |
| 10 | Product name casing chaos | `T-Shirt` / `tshirt` / `T SHIRT` / `Tee Shirt` |
| 11 | Category name variants | `Electronics` / `ELECTRONICS` / `Gadgets` / `gadgets` |
| 12 | Supplier name variants | `TechSource Ltd` / `techsource` / `Tech Source` / `TechSource Ltd.` |
| 13 | Mixed date formats | 6 formats including `May 16, 2022`, `03/25/2024`, `2023-11-02` |
| 14 | Future and pre-2020 dates | Invalid onboarding dates mixed across rows |
| 15 | Blank rows | ~20 fully blank rows, ~10 partially blank rows |

---

## 🧠 Approach

Instead of directly cleaning the data, I designed a **structured validation pipeline** to simulate real operational workflows.

---

### 🔹 Step 1: Data Profiling

Before making any changes, I assessed the dataset to understand:

- Distribution of missing values
- Presence of invalid records
- Structure and consistency of key fields

This ensured that cleaning decisions were **data-driven**, not arbitrary.

---

### 🔹 Step 2: Data Standardisation

To eliminate inconsistencies:

- Trimmed and cleaned all text fields
- Standardised text casing across all columns
- Mapped all name variants to a single canonical value per category, supplier, and product name
- Unified 6 different date formats into a single `YYYY-MM-DD` standard

---

### 🔹 Step 3: Data Cleaning

Key transformations included:

- Replacing missing SKUs with a placeholder (`MISSING_SKU`)
- Converting Price field: removed `₦` symbols, commas, and shorthand values like `5k` → `5000`
- Converting Stock field: replaced text values (`many`, `low`, `-`) with `null`
- Handling invalid and mixed data types across numeric columns
- Parsing and standardising all date formats using multi-format detection

---

### 🔹 Step 4: Validation Framework

A validation layer was implemented to enforce business rules:

#### Validation Rules

| Rule | Condition |
|------|-----------|
| SKU must exist | `SKU ≠ null` |
| Price must be valid | `Price > 0` and numeric |
| Stock must be non-negative | `Stock_Quantity ≥ 0` |
| Date must be valid | Parseable and within expected range |

#### Validation Columns Created

- `SKU_Status`
- `Price_Status`
- `Stock_Status`
- `Date_Status`
- `Data_Quality_Status`

This transformed the dataset from "clean-looking" to **system-controlled**.

---

### 🔹 Step 5: Error Classification

Errors were categorised into actionable groups:

- Missing SKU
- Missing Price
- Missing Stock
- Negative Stock
- Zero Price
- Future Date
- Pre-2020 Date

This allowed for targeted analysis and prioritisation.

---

### 🔹 Step 6: Data Segmentation

The dataset was split into two outputs:

#### ✅ Clean Dataset
- Contains only validated records
- Ready for system upload

#### ❌ Error Log
- Contains all invalid records
- Includes error reasons per row
- Supports merchant correction workflows

---

## 📊 Key Insights

Analysis of error distribution revealed:

### 🔴 1. Missing SKU — Highest Priority
- Breaks product identification at the system level
- Prevents inventory tracking, reporting, and transaction matching
- Must be resolved before any onboarding can proceed

### 🔴 2. Missing Price
- Blocks transaction processing
- Prevents revenue tracking and reporting

### 🟠 3. Missing Stock
- Leads to inaccurate inventory availability
- Can cause overselling or missed restocking triggers

### 🟠 4. Negative Stock
- Indicates logical or data entry errors
- Requires validation controls at the point of entry

### 🟡 5. Naming & Format Inconsistencies
- Silently breaks grouping, filtering, and reporting
- Often overlooked but causes significant downstream errors

---

## ⚙️ Operational Thinking

Instead of treating all errors equally, I prioritised them based on business impact:

| Priority | Issue | Impact |
|----------|-------|--------|
| 1 | Missing SKU | System-breaking |
| 2 | Missing Price | Revenue-blocking |
| 3 | Missing Stock | Operational risk |
| 4 | Negative Stock | Validation issue |
| 5 | Naming inconsistencies | Reporting integrity |
| 6 | Date errors | Audit and compliance risk |

This reflects how real onboarding and operations teams triage data issues — by business consequence, not by volume.

---

## 🚀 Outcome

The final solution delivered:

- A structured **data validation pipeline** built entirely in Power Query
- A clean, system-ready dataset with consistent formatting across all fields
- An error log for merchant correction workflows
- Clear visibility into data quality issues by type and volume

---

## 🧾 Business Impact

If implemented in a real system, this approach would:

- Reduce merchant onboarding delays caused by bad data
- Improve data accuracy and reliability across inventory and reporting systems
- Prevent transaction and stock errors from reaching production
- Enable scalable, repeatable data operations without manual intervention

---

## 🧠 Key Takeaways

- Cleaning data is not enough — **validation is critical**
- Missing identifiers (SKU) are the most dangerous data issue in any product system
- Error classification enables better operational decisions than bulk deletion
- Structured pipelines outperform ad-hoc fixes at scale
- Merchant-submitted data should never be trusted at face value — every field requires a defined validation rule before system ingestion

---

## 🔧 Tools Used

| Tool | Usage |
|------|-------|
| Microsoft Excel | Data inspection and output formatting |
| Power Query (M language) | Transformation, standardisation, and validation pipeline |

---

## 👤 Author

**Abdulkabeer Mabinuori**

- 📍 Nigeria
- 📧 [mabinuoriabdulkabeer2005@gmail.com](mailto:mabinuoriabdulkabeer2005@gmail.com)
- 🔗 [LinkedIn — Abdulkabeer Mabinuori](http://www.linkedin.com/in/abdulkabeer-mabinuori-144068386)
