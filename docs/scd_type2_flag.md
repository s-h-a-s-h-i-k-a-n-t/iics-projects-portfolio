# 🧩 SCD Type 2 (Flag Method) — Informatica IICS

## 🎯 Business Objective
To maintain **historical records** of customer data changes using an **Active/Inactive Flag** approach.  
This mapping ensures every change in customer data is tracked by marking the current record as active (`Active_Flag = 1`) and expiring the old record (`Active_Flag = 0`).

---

## ⚙️ Mapping Details

| Parameter | Description |
|------------|-------------|
| **Mapping Name** | `m_SCD_Type2_Flag` |
| **SCD Type** | Type 2 (Flag Method) |
| **Source Table** | `HR.S_CUSTOMER_SCD` |
| **Target Table** | `CORE.T_CUSTOMER_SCD_FLAG` |
| **Primary Key** | `CUSTOMER_ID` |
| **Surrogate Key** | `CUSTOMER_KEY` |
| **Active Flag** | `1 = Active`, `0 = Inactive` |

---

## 🧱 Workflow Overview

SRC_CUSTOMER_SNAPSHOT
│
▼
EXP_ACTIVE_INACTIVE_FLAG
│
▼
LKP_DIM_CUSTOMER_CURR
│
▼
RTR_SCD2_FLAG_CHANGE_DETECT ─────────────┬──────────────┬──────────────┐
│ │ │ │
SEQ_CUSTOMER_SK_GEN TGT_DIM_CUSTOMER_INS_NEW TGT_DIM_CUSTOMER_INS_CHANGE TGT_DIM_CUSTOMER_UPD_EXPIRE

yaml
Copy code

---

## 🔄 Transformation Details

### 🗂️ **1. Source — `SRC_CUSTOMER_SNAPSHOT`**
- Reads active customer data from the source system.
- **Connection:** `Oracle_Src`
- **Object:** `HR.S_CUSTOMER_SCD`
- **Key Fields:** `CUSTOMER_ID`, `FIRST_NAME`, `MOBILE`, `ADDRESS1`, `COUNTRY`, `ZIPCODE`

---

### ⚙️ **2. Expression — `EXP_ACTIVE_INACTIVE_FLAG`**
- Adds two derived fields for flag logic.

| Field Name | Expression | Description |
|-------------|-------------|--------------|
| `o_Active_Flag` | `1` | Marks record as active. |
| `o_Inactive_Flag` | `0` | Marks record as inactive (expired). |

---

### 🔍 **3. Lookup — `LKP_DIM_CUSTOMER_CURR`**
- Looks up the target dimension table `CORE.T_CUSTOMER_SCD_FLAG` to identify existing customers.

| Property | Value |
|-----------|--------|
| **Connection** | `Oracle_Tgt` |
| **Lookup Object** | `CORE.T_CUSTOMER_SCD_FLAG` |
| **Multiple Matches** | Return Any Row |
| **Join Condition** | `CUSTOMER_ID = src_CUSTOMER_ID` |

**Purpose:**  
To determine if the incoming customer record already exists and whether the details have changed.

---

### 🧭 **4. Router — `RTR_SCD2_FLAG_CHANGE_DETECT`**
- Splits data into logical groups: **INSERT_NEW**, **UPDATE_CHANGE**, and **DEFAULT**.

| Group Name | Condition | Description |
|-------------|------------|--------------|
| `INSERT_NEW` | `ISNULL(CUSTOMER_ID)` | Customer not present in target — insert as new record. |
| `UPDATE_CHANGE` | `IIF(CUSTOMER_ID = src_CUSTOMER_ID AND (FIRST_NAME <> src_FIRST_NAME OR MOBILE <> src_MOBILE), TRUE, FALSE)` | Existing customer — data change detected, create new active version. |
| `DEFAULT` | — | Optional pass-through for unchanged records. |

---

### 🔢 **5. Sequence Generator — `SEQ_CUSTOMER_SK_GEN`**
- Generates **surrogate keys** (`CUSTOMER_KEY`) for new or changed customer records.

| Property | Value |
|-----------|--------|
| **Start Value** | 1 |
| **Increment By** | 1 |
| **Connected To** | All Insert Targets |

---

### 🎯 **6. Target — `TGT_DIM_CUSTOMER_INS_NEW`**
- Inserts new customer records not found in the dimension table.

| Column | Mapped From |
|---------|-------------|
| `CUSTOMER_KEY` | `NEXTVAL` |
| `CUSTOMER_ID` | `src_CUSTOMER_ID` |
| `FIRST_NAME` | `src_FIRST_NAME` |
| `MOBILE` | `src_MOBILE` |
| `ADDRESS1` | `src_ADDRESS1` |
| `ZIPCODE` | `src_ZIPCODE` |
| `COUNTRY` | `src_COUNTRY` |
| `FLAG` | `o_Active_Flag` |

---

### 🎯 **7. Target — `TGT_DIM_CUSTOMER_INS_CHANGE`**
- Inserts a **new version** of existing customers whose data has changed.

| Column | Mapped From |
|---------|-------------|
| `CUSTOMER_KEY` | `NEXTVAL` |
| `CUSTOMER_ID` | `src_CUSTOMER_ID` |
| `FIRST_NAME` | `src_FIRST_NAME` |
| `MOBILE` | `src_MOBILE` |
| `ADDRESS1` | `src_ADDRESS1` |
| `ZIPCODE` | `src_ZIPCODE` |
| `COUNTRY` | `src_COUNTRY` |
| `FLAG` | `o_Active_Flag` |

---

### 🧱 **8. Target — `TGT_DIM_CUSTOMER_UPD_EXPIRE`**
- Updates old record versions, marking them as inactive.

| Column | Mapped From |
|---------|-------------|
| `CUSTOMER_KEY` | `CUSTOMER_KEY` |
| `CUSTOMER_ID` | `CUSTOMER_ID` |
| `FLAG` | `o_Inactive_Flag` |

---

## 🧮 Sample Output

| CUSTOMER_KEY | CUSTOMER_ID | FIRST_NAME | MOBILE | FLAG | Remarks |
|---------------|--------------|-------------|---------|--------|----------|
| 1001 | 101 | John | 9001 | 1 | Active record |
| 1002 | 101 | John | 8001 | 0 | Old record (expired) |
| 1003 | 102 | Tina | 7001 | 1 | Active record |

---

## 🌟 Highlights
✅ Implements **SCD Type 2 (Flag Method)** for maintaining historical data.  
✅ Uses **Active/Inactive flags** instead of date ranges.  
✅ Follows **industry-standard naming conventions**.  
✅ Generates **surrogate keys** via sequence generator.  
✅ Works seamlessly across **Customer, Product, or Vendor** dimensions.

---

## 🗂️ Folder Structure Example
/CDI/SCD_Type2_Flag/
│
├── m_SCD_Type2_Flag.png
├── m_SCD_Type2_Flag.zip
├── README.md
└── sample_input_output.xlsx

yaml
Copy code
