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
