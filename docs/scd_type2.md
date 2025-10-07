# SCD Type 2 Mapping – Informatica IICS

## 📌 Business Goal
Maintain full historical data of dimension tables (e.g., Customers, Products) to support analytics and compliance requirements.

Example: If a customer changes their address, the system should keep both **old** and **new** records with validity dates.

---

## 🛠️ Technical Design
- **Source** → Reads data from staging table.
- **Expression** → Derives metadata fields (start date, end date, active flag).
- **Lookup** → Checks if the record already exists in the target.
- **Router** → Splits rows into `new insert`, `historical update`, and `current update`.
- **Sequence Generator** → Generates surrogate keys for new records.
- **Targets** →  
  - `new_insert` → Inserts brand new rows.  
  - `upd_insert` → Inserts changed rows as historical.  
  - `upd_update` → Updates old rows (end date + inactive flag).  

---

## ⚙️ Key Transformations
- **Expression Transformation** → Adds logic for SCD2 fields:
  - `START_DATE` = SYSDATE
  - `END_DATE` = NULL
  - `IS_ACTIVE` = 'Y'
- **Lookup Transformation** → Matches incoming data with target records based on business keys.
- **Router Transformation** → Routes rows for Insert vs Update handling.

---

## ✅ Achievements
- Preserves history of changes for auditing.  
- Supports BI reports to analyze customer/product data across time.  
- Ensures data accuracy and traceability.  

---

## 📸 Mapping Screenshot
![SCD Type 2 Mapping](../mapping_screenshots/scd_type2.png)

---

## 🔗 Related Export
- [Job Export (.zip)](../jobs_exports/mct_m_SCD_Type2_Date_MD5-1759765666795.zip)
