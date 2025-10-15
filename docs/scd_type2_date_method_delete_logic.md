#  SCD Type 2 (Date Method with Delete Logic) — Informatica IICS

##  Business Objective
To maintain **historical customer data** in the data warehouse with proper handling of:
- **New records** (insert)
- **Changed records** (expire old + insert new)
- **Deleted records** (soft delete logic)

This mapping ensures every change in customer data is **tracked over time** using effective and end dates.

---

##  Mapping Details
**Mapping Name:** `m_SCD_Type_2_Date_method_delete_logic`  
**SCD Type:** Type 2 (Date Method)  
**Source Table:** `HR.S_CUSTOMER_SCD`  
**Target Table:** `CORE.T_CUSTOMER_SCD_DATE`  

---

##  Workflow Overview
SRC_CUSTOMER_SNAPSHOT ─┐
│
▼
JNR_SRC_DIM_CUSTOMER
│
▼
EXP_SCD2_PREP_DATES_DELETE
│
▼
RTR_SCD2_CHANGE_DELETE_DETECT
├─► SEQ_CUSTOMER_SK_GEN ─► new_insert
├────────────────────────► upd_insert
├────────────────────────► upd_update
└────────────────────────► DELETE


---

##  Transformation Logic

### 🔹 Source – `SRC_CUSTOMER_SNAPSHOT`
- Fetches active customer data from the source.
- Connection: **Oracle_Src**
- Object: `HR.S_CUSTOMER_SCD`

### 🔹 Target Reference – `SRC_DIM_CUSTOMER_CURR`
- Reads existing dimension data for comparison.
- Connection: **Oracle_Tgt**
- Object: `CORE.T_CUSTOMER_SCD_DATE`

---

### 🔹 Joiner – `JNR_SRC_DIM_CUSTOMER`
| Property | Value |
|-----------|--------|
| Master | Source |
| Detail | Target |
| Join Condition | `CUSTOMER_ID = src_CUSTOMER_ID` |
| Join Type | Full Outer Join |

**Purpose:**  
To compare incoming source data with the target dimension and identify:
- New customers  
- Changed customers  
- Deleted customers  

---

### 🔹 Expression – `EXP_SCD2_PREP_DATES_DELETE`
Adds new fields for managing SCD valid dates.  

| Field | Expression | Description |
|--------|-------------|-------------|
| `in_eff_date` | `SYSDATE()` | Start date of record validity |
| `in_end_date` | `TO_DATE('12/31/9999','MM/DD/YYYY')` | Default end date for active record |

---

### 🔹 Router – `RTR_SCD2_CHANGE_DELETE_DETECT`
Splits data into different groups:

| Group Name | Condition | Description |
|-------------|------------|-------------|
| **INSERT** | `ISNULL(CUSTOMER_ID)` | New customer, insert as a new record |
| **UPDATE** | `IIF(src_CUSTOMER_ID=CUSTOMER_ID AND (src_FIRST_NAME<>FIRST_NAME OR src_MOBILE<>MOBILE), TRUE, FALSE)` | Data change detected — expire old + insert new |
| **DELETE** | `ISNULL(src_CUSTOMER_ID)` | Record deleted from source — mark as inactive |

---

### 🔹 Sequence – `SEQ_CUSTOMER_SK_GEN`
Generates **surrogate keys** for new and changed customer records.  
Used by `new_insert` and `upd_insert` targets.

---

##  Target Definitions

| Target Name | Operation | Purpose |
|--------------|------------|----------|
| **`new_insert`** | Insert | Adds completely new customers. |
| **`upd_insert`** | Insert | Inserts a new version of an existing record when data changes. |
| **`upd_update`** | Update | Expires the old version by setting `END_DATE = SYSDATE()-1`. |
| **`DELETE`** | Update | Marks records as soft-deleted (`DELETE_FLAG='Y'`, `IS_CURRENT='N'`). |

---

##  Sample Output

| CUSTOMER_KEY | CUSTOMER_ID | NAME | MOBILE | EFF_DATE | END_DATE | IS_CURRENT | DELETE_FLAG |
|---------------|--------------|------|---------|-----------|-----------|-------------|--------------|
| 1001 | 101 | John | 9001 | 15-Oct-2025 | 31-Dec-9999 | Y | N |
| 1002 | 102 | David | 7001 | 01-Sep-2024 | 14-Oct-2025 | N | N |
| 1003 | 102 | David | 8001 | 15-Oct-2025 | 31-Dec-9999 | Y | N |
| 1004 | 103 | Tina | 7001 | 10-Aug-2024 | 14-Oct-2025 | N | Y |

---

##  Highlights
✅ Combines **SCD Type 2** + **Soft Delete** logic  
✅ Ensures **full history tracking** using `EFF_DATE` and `END_DATE`  
✅ Generates surrogate keys using sequence  
✅ Follows **industry-standard transformation naming**  
✅ Works seamlessly for any dimension table (Customer, Product, Vendor)

---

##  Folder Structure Example


/CDI/SCD_Type_2_Date_Method_Delete_Logic/
│
├── m_SCD_Type_2_Date_method_delete_logic.png
├── m_SCD_Type_2_Date_method_delete_logic.zip
├── README.md
└── sample_input_output.xlsx


---

###  Summary
> This mapping implements a **real-world Slowly Changing Dimension Type 2 (Date Method)** with built-in delete detection and surrogate key generation in Informatica IICS.  
It ensures **data accuracy, history preservation, and clean auditability** — making it enterprise-ready.
