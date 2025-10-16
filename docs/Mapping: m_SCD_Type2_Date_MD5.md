# Mapping: m_SCD_Type2_Date_MD5

![SCD Type 2 Date Method MD5 Mapping](https://github.com/s-h-a-s-h-i-k-a-n-t/iics-projects-portfolio/blob/main/CDI/mappings/m_SCD_Type2_Date_MD5.png)

## 🧾 Note
This mapping was built and tested by **Shashi Kant** in the Informatica IICS environment using an Oracle target connection.  
It demonstrates how **MD5-based change detection** combined with **date-based SCD logic** efficiently tracks historical changes in dimension tables.  
The goal is to maintain full history for changed records by inserting new versions and expiring old ones using **Start Date / End Date** columns.

---

## 🎯 Objective
Implement **Slowly Changing Dimension (SCD) Type 2** using the **Date Method** and **MD5 hashing** to:
- Detect changes in source records via MD5 comparison.
- Insert new version rows when changes occur.
- Expire old versions by updating their `END_DATE` and `IS_ACTIVE` flag.

---

## 🧩 Design Overview
### 🔄 Pipeline Flow
```
SRC_CUSTOMER_SNAPSHOT
    ↓
EXP_GEN_MD5_AND_DATES
    ↓
LKP_DIM_CUSTOMER_TARGET
    ↓
RTR_SCD2_CHANGE_DETECT
     ↘︎ SEQ_DIM_CUSTOMER_KEY → TGT_DIM_CUSTOMER_NEW_INSERT
     ↘︎ SEQ_DIM_CUSTOMER_KEY → TGT_DIM_CUSTOMER_UPD_INSERT
     ↘︎ TGT_DIM_CUSTOMER_UPD_UPDATE
```

---

## 🧱 Transformation Details
| Component | Description / Logic |
|------------|--------------------|
| **SRC_CUSTOMER_SNAPSHOT** | Reads the latest customer snapshot from the source/staging system. |
| **EXP_GEN_MD5_AND_DATES** | Generates MD5 hash and initializes `EFF_DATE`, `END_DATE`, and `IS_ACTIVE`. <br> **Expressions:** <br>• `o_in_eff_date = SYSDATE` <br>• `o_in_end_date = TO_DATE('12/31/9999','MM/DD/YYYY')` <br>• `o_in_MD5 = MD5(CUSTOMER_ID || FIRST_NAME || MOBILE || ADDRESS1 || ZIPCODE || COUNTRY)` |
| **LKP_DIM_CUSTOMER_TARGET** | Looks up target table (`CORE.T_CUSTOMER_SCD_DATE_MD5`) by `CUSTOMER_ID` to fetch existing record details (old MD5, surrogate key, end date). |
| **RTR_SCD2_CHANGE_DETECT** | Splits the data flow: <br>• **NEW_INSERT:** record not found in target (`ISNULL(CUSTOMER_ID)`) <br>• **UPD_INSERT:** record exists but MD5 mismatch (`src_o_in_MD5 <> MD5`) <br>• **UPD_UPDATE:** expire old record (set `END_DATE = SYSDATE`, `IS_ACTIVE = 0`). |
| **SEQ_DIM_CUSTOMER_KEY** | Generates surrogate key (`CUSTOMER_KEY = NEXTVAL`). |
| **TGT_DIM_CUSTOMER_NEW_INSERT** | Inserts brand new customers. |
| **TGT_DIM_CUSTOMER_UPD_INSERT** | Inserts new version row for changed customers (Type-2 insert). |
| **TGT_DIM_CUSTOMER_UPD_UPDATE** | Updates existing version — expires it by setting `END_DATE = SYSDATE`. |

---

## 🧮 Router Groups and Conditions
| Group Name | Condition | Purpose |
|-------------|------------|----------|
| `NEW_INSERT` | `ISNULL(CUSTOMER_ID)` | New record → Insert fresh row. |
| `UPD_INSERT` | `src_o_in_MD5 <> MD5` | Data changed → Insert new version row. |
| `UPD_UPDATE` | Derived from `UPD_INSERT` | Expire old record (`END_DATE = SYSDATE`). |

---

## 🧠 MD5 Expression Example
```sql
MD5(
  UPPER(TRIM(CUSTOMER_ID)) ||
  UPPER(TRIM(FIRST_NAME)) ||
  TRIM(MOBILE) ||
  UPPER(TRIM(ADDRESS1)) ||
  TRIM(ZIPCODE) ||
  UPPER(TRIM(COUNTRY))
)
```
This expression ensures that even a single attribute change (like address, mobile, or country) results in a different MD5 hash — enabling accurate change detection.

---

## 🧾 Sample Dataset (Simplified Example)
### Source Data (Incoming Records)
| CUSTOMER_ID | FIRST_NAME | MOBILE | ADDRESS1 | COUNTRY | ZIPCODE |
|--------------|-------------|----------|-----------|----------|----------|
| 101 | Rahul | 9000000000 | Pune | India | 411001 |
| 102 | Sneha | 8123456789 | Delhi | India | 110001 |

### Target Data (Before Mapping Execution)
| CUSTOMER_ID | FIRST_NAME | MOBILE | ADDRESS1 | COUNTRY | ZIPCODE | MD5 | END_DATE | IS_ACTIVE |
|--------------|-------------|----------|-----------|----------|----------|------|-----------|------------|
| 101 | Rahul | 9000000000 | Pune | India | 411001 | 5A1F98C7C4... | 12/31/9999 | 1 |

### Source Change
Record 101 → Address changed from *Pune* → *Mumbai*  
New MD5 ≠ Old MD5 → triggers Type 2 insert.

### Target Data (After Mapping Execution)
| CUSTOMER_ID | FIRST_NAME | MOBILE | ADDRESS1 | COUNTRY | ZIPCODE | MD5 | START_DATE | END_DATE | IS_ACTIVE |
|--------------|-------------|----------|-----------|----------|----------|------|-------------|-----------|------------|
| 101 | Rahul | 9000000000 | Pune | India | 411001 | 5A1F98C7C4... | 01-Jan-2024 | 15-Oct-2025 | 0 |
| 101 | Rahul | 9000000000 | Mumbai | India | 400001 | 9E7C24F1A3... | 15-Oct-2025 | 31-Dec-9999 | 1 |

---

## ✅ Outcome
- **MD5 hash detects changes efficiently.**
- **SCD Type 2 logic** ensures old records are retained with proper end dates.
- **No redundant updates** — only changed records are processed.
- Maintains **complete historical audit trail** in the target dimension.

---

## 💡 Key Takeaway
This mapping demonstrates **real-world SCD Type 2 logic** using:
- **MD5 for change detection**
- **Date-based versioning (EFF_DATE/END_DATE)**
- **Surrogate key tracking**

This is a best-practice design for **data warehouse dimension history management** and ideal for interview demonstrations.

---

## 📦 Export File
You can download the Informatica mapping export file (`.zip`) from the repo:

👉 [Download Mapping Export (ZIP)](../jobs_exports/m_SCD_Type2_Date_MD5.zip)

---

🧠 *Author: Shashi Kant | Built & Tested in Informatica IICS*  
📁 *Part of IICS Projects Portfolio (SCD Series)*
