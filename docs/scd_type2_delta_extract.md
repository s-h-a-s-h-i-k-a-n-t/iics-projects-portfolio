# 📘 SCD Type 2 – Date Method (Delta Extract)  
**Mapping Name:** `m_CUSTOMER_SCD_Type_2_date_delta_extract`  
**Tool:** Informatica Cloud Data Integration (IICS)  
**Category:** Cloud Data Warehouse / Dimension Management  

---

## 🎯 Business Goal
Enable **incremental (delta) loading** of customer dimension data using **SCD Type 2 – Date Method**.  
This approach maintains **historical records** efficiently by processing **only changed or new data** since the last ETL run.  

**Example:**  
If a customer’s address or phone number changes, the system updates the existing record’s end date and inserts a new record with a fresh surrogate key and effective date range.

---

## 🛠️ Technical Design Overview
**End-to-End Flow:**  
`Source → Expression → Lookup → Router → Sequence → Targets`

| Step | Transformation | Purpose |
|------|----------------|----------|
| 1️⃣ | **Source (SRC_CUSTOMER_DELTA)** | Reads only delta records (new or changed) from the staging or change-capture layer. |
| 2️⃣ | **Expression (EXP_SCD2_PREP_DATES)** | Derives key metadata fields:<br>• `EFF_START_DATE = SYSDATE`<br>• `EFF_END_DATE = TO_DATE('12/31/9999','MM/DD/YYYY')`<br>• `IS_CURRENT = 'Y'` |
| 3️⃣ | **Lookup (LKP_DIM_CUSTOMER_CURR)** | Compares incoming delta records against current dimension table (`DIM_CUSTOMER`) based on business key (`CUSTOMER_ID`). |
| 4️⃣ | **Router (RTR_SCD2_CHANGE_DETECT)** | Routes rows into: <br>• **Insert_New:** New customers not found in target<br>• **Insert_Change:** Existing customers with data changes<br>• **Update_Expire:** Existing records to be closed (set `EFF_END_DATE = SYSDATE - 1`, `IS_CURRENT = 'N'`) |
| 5️⃣ | **Sequence Generator (SEQ_CUSTOMER_SK_GEN)** | Generates surrogate key for new and changed records. |
| 6️⃣ | **Targets** | Writes data into three target tables:<br>• `TGT_DIM_CUSTOMER_INS_NEW`<br>• `TGT_DIM_CUSTOMER_INS_CHANGE`<br>• `TGT_DIM_CUSTOMER_UPD_EXPIRE` |

---

## ⚙️ Key Logic
```sql
-- Detecting Changed Records
IIF(
  SRC.MOBILE <> LKP.MOBILE OR
  SRC.ADDRESS <> LKP.ADDRESS OR
  SRC.ZIPCODE <> LKP.ZIPCODE,
  TRUE, FALSE
)
