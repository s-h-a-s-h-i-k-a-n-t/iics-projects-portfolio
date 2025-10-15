# 🧩 SCD Type 2 (Flag Method) — Informatica IICS

## 🎯 Business Objective
This mapping implements **Slowly Changing Dimension Type 2 (Flag Method)** to maintain a **complete change history** of customer records.  
Instead of using date ranges, this method uses simple **Active (1)** and **Inactive (0)** flags to identify current vs. expired records in the target dimension.

It helps business users track how a customer’s profile evolved over time while keeping the latest version active.

---

## ⚙️ Mapping Overview
**Mapping Name:** `m_SCD_Type2_Flag`  
**Type:** SCD Type 2 — Flag Method  
**Source Table:** `HR.S_CUSTOMER_SCD`  
**Target Table:** `CORE.T_CUSTOMER_SCD_FLAG`  
**Keys Used:**  
- **Primary Key:** `CUSTOMER_ID`  
- **Surrogate Key:** `CUSTOMER_KEY`

---

## 🧱 Workflow Logic

SRC_CUSTOMER_SNAPSHOT
│
▼
EXP_ACTIVE_INACTIVE_FLAG
│
▼
LKP_DIM_CUSTOMER_CURR
│
▼
RTR_SCD2_FLAG_CHANGE_DETECT ─────┬────────────┬────────────┐
│ │ │ │
SEQ_CUSTOMER_SK_GEN TGT_DIM_CUSTOMER_INS_NEW TGT_DIM_CUSTOMER_INS_CHANGE TGT_DIM_CUSTOMER_UPD_EXPIRE

yaml
Copy code

---

## 🔄 Transformation Summary

### 1️⃣ Source — `SRC_CUSTOMER_SNAPSHOT`
Reads active customer data from the **HR schema** (Oracle source).  
The input snapshot represents the most recent extract of customer information.

---

### 2️⃣ Expression — `EXP_ACTIVE_INACTIVE_FLAG`
Adds two flag fields to identify record states:
- `Active_Flag = 1` → Current valid record  
- `Inactive_Flag = 0` → Expired record  

These flags are later used in target update logic to maintain the correct record status.

---

### 3️⃣ Lookup — `LKP_DIM_CUSTOMER_CURR`
Performs a lookup on the **target dimension table** (`CORE.T_CUSTOMER_SCD_FLAG`)  
to check if the incoming record already exists.  
It helps in identifying:
- New customers  
- Changed customers (based on business key & attribute comparison)

---

### 4️⃣ Router — `RTR_SCD2_FLAG_CHANGE_DETECT`
Splits the data into three routes:
- **INSERT_NEW:** For customers not existing in target (new entries)  
- **UPDATE_CHANGE:** For customers whose attributes have changed  
- **DEFAULT:** For unchanged records (ignored downstream)

---

### 5️⃣ Sequence Generator — `SEQ_CUSTOMER_SK_GEN`
Generates surrogate keys (`CUSTOMER_KEY`) for new and changed records.  
This ensures each version of the same customer has a **unique dimension key**.

---

### 6️⃣ Target — Dimension Table
Three write paths handle insert and update operations:

- **`TGT_DIM_CUSTOMER_INS_NEW`** → Inserts brand-new customer records.  
- **`TGT_DIM_CUSTOMER_INS_CHANGE`** → Inserts a new active version for changed customers.  
- **`TGT_DIM_CUSTOMER_UPD_EXPIRE`** → Updates the previous version, marking it inactive.

---

## 🧮 Example Scenario

| Step | Action | Active_Flag | Remarks |
|------|---------|--------------|----------|
| 1 | First time customer `101` appears | 1 | New record inserted |
| 2 | Customer `101` changes phone number | 0 / 1 | Old record flagged inactive; new version inserted |
| 3 | No change detected in next load | 1 | Only active record retained |

---

## 🌟 Key Highlights
✅ Simplified **flag-based SCD tracking** (no date fields).  
✅ Works efficiently for **large-volume incremental loads**.  
✅ **Industry-standard naming** for all transformations.  
✅ Compatible with other dimensions (e.g., Product, Vendor, Region).  
✅ Lightweight, fast, and easy to maintain.

---

## 🗂 Folder Example
/CDI/SCD_Type2_Flag/
│
├── m_SCD_Type2_Flag.png
├── m_SCD_Type2_Flag.zip
├── README.md
└── sample_input_output.xlsx


---

## 👤 Author
**Shashi Kant**  
ETL Developer | Informatica IICS | Oracle PL/SQL | Snowflake  
📧 shashikant.dev.informatica@gmail.com  
🌐 [GitHub Portfolio](https://github.com/s-h-a-s-h-i-k-a-n-t/iics-projects-portfolio)

---

### 🧾 Summary
This mapping demonstrates a **real-world SCD Type 2 implementation** using the **Flag Method**.  
It’s ideal for systems that prefer minimal date handling while maintaining full change traceability.  
Designed with clear logic, reusability, and easy debugging in mind — perfect for enterprise-grade ETL pipelines.
