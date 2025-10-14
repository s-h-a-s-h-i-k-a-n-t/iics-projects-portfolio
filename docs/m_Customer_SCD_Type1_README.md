# Mapping: m_Customer_SCD_Type1_MD5

### Note
This mapping was **built and tested by me (Shashi Kant)** in my Informatica IICS lab environment using sample customer data.  
I designed it to demonstrate how **MD5 hashing** can simplify change detection logic in Slowly Changing Dimensions.  
The goal was to show real-world ETL handling — detecting changed customer records and updating them seamlessly.

---

## Objective
Implement **Slowly Changing Dimension (SCD) Type 1** using **MD5 hash comparison** to detect changes in customer attributes and overwrite the existing record with the latest values.

---

## Design Overview
Pipeline flow:
1. **SRC_CUSTOMER** → reads source rows  
2. **EXP_GENERATE_MD5** → builds MD5 across business columns  
3. **LKP_CUSTOMER_TARGET** → checks existing target row and retrieves stored MD5  
4. **RTR_SCD_ROUTER** → splits into **INSERT** vs **UPDATE**  
5. **TGT_INSERT_CUSTOMERS / TGT_UPDATE_CUSTOMERS** → loads final data  

---

## Mapping Diagram
![Customer SCD Type 1 Mapping](../CDI/mappings/m_Customer_SCD_Type1_MD5.png)

---

## Export File
You can download the Informatica mapping export file (.zip) from the link below:

[Download Mapping Export (ZIP)](../jobs_exports/m_Customer_SCD_Type1_MD5-1759953821141.zip)

---

## Transformations & Purpose

| Component | Key Logic / Purpose |
|------------|--------------------|
| **SRC_CUSTOMER** | Reads active customers from source system/table |
| **EXP_GENERATE_MD5** | Creates a change-detection hash |
| **LKP_CUSTOMER_TARGET** | Finds existing customer and retrieves stored MD5 |
| **RTR_SCD_ROUTER** | Routes records → **INSERT** (new) / **UPDATE** (changed) |
| **TGT_INSERT_CUSTOMERS** | Inserts brand-new customers |
| **TGT_UPDATE_CUSTOMERS** | Updates attributes for existing customers |

---

### MD5 Expression (Example)

```text
MD5(
  UPPER(TRIM(CUST_NAME)) ||
  UPPER(TRIM(EMAIL)) ||
  TRIM(PHONE) ||
  UPPER(TRIM(ADDRESS1)) ||
  UPPER(TRIM(CITY))
)

# 📊 Sample Dataset – SCD Type 1 (MD5 Hash Comparison)

### Purpose
To demonstrate how **MD5-based change detection** identifies modified customer records and updates them in place (without retaining history).

---

## 🧩 Scenario
A customer updates their **email** and **phone number** in the source system.  
The mapping detects these changes by comparing the **MD5 hash** of source data with that of the target table.

---

## 📥 Source Data (Incoming Records)

| CUSTOMER_ID | CUSTOMER_NAME | EMAIL              | PHONE       | CITY  | HASH_BEFORE_RUN |
|--------------|----------------|--------------------|--------------|-------|-----------------|
| 101          | Rahul Mehta     | rahul@gnail.com    | 9000000000   | Pune  | 5A1F98C7C4…     |
| 102          | Sneha Rao       | sneha@gmail.com    | 8123456789   | Delhi | 3C7F45D9E2…     |
| 103          | Amit Sharma     | amit@yahoo.com     | 9123456789   | Mumbai | 7E9F13B2D1…     |

---

## 📊 Target Data (Before Mapping Execution)

| CUSTOMER_ID | CUSTOMER_NAME | EMAIL              | PHONE       | CITY  | HASH_STORED |
|--------------|----------------|--------------------|--------------|-------|--------------|
| 101          | Rahul Mehta     | rahul@gnail.com    | 9000000000   | Pune  | 5A1F98C7C4… |
| 102          | Sneha Rao       | sneha@gmail.com    | 8123456789   | Delhi | 3C7F45D9E2… |
| 103          | Amit Sharma     | amit@yahoo.com     | 9123456789   | Mumbai | 7E9F13B2D1… |

---

## 🔍 Source Changes Detected
Record **101** has a modified email and phone number.  
New MD5 hash ≠ old MD5 hash → record routed to **UPDATE**.

---

## ✅ Target Data (After Mapping Execution)

| CUSTOMER_ID | CUSTOMER_NAME | EMAIL              | PHONE          | CITY  | HASH_UPDATED |
|--------------|----------------|--------------------|----------------|-------|---------------|
| 101          | Rahul Mehta     | rahul@gmail.com    | +91-9000000000 | Pune  | 9E7C24F1A3…  |
| 102          | Sneha Rao       | sneha@gmail.com    | 8123456789     | Delhi | 3C7F45D9E2…  |
| 103          | Amit Sharma     | amit@yahoo.com     | 9123456789     | Mumbai | 7E9F13B2D1…  |

---

## 💡 Outcome
✅ **MD5 hash detects changes** — no need for multiple column-by-column comparisons.  
✅ **Only changed record (101)** is updated; unchanged rows remain intact.  
✅ **SCD Type 1 behavior** — data is overwritten (no version history).  
✅ Ensures data freshness while keeping dimension size compact.

---

## 🧠 Key Takeaway
This dataset helps recruiters visualize how **MD5-based SCD Type 1 mapping** keeps the dimension table updated efficiently —  
ideal for **real-time ETL pipelines**, **incremental loads**, and **data warehouse synchronization**.
