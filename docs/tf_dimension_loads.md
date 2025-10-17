# Taskflow: tf_dimension_loads

## Objective
Automate daily dimension loads using Slowly Changing Dimension Type-2 (SCD2) logic.  
This taskflow orchestrates multiple mappings to detect data changes between daily snapshots,  
expire old versions, and insert new versions while maintaining historical records.

---

## Folder Location
```
iics-projects-portfolio/
├── CDI/
│   ├── mappings/
│   │   ├── m_SCD_Type2_Date_MD5.zip
│   │   ├── m_SCD_Type2_Flag.zip
│   └── taskflows/
│       └── tf_dimension_loads/
│           ├── tf_dimension_loads.png
│           └── tf_dimension_loads.zip
└── docs/
    └── tf_dimension_loads.md
```

---

## Taskflow Screenshot
Below is the canvas view for this Taskflow:

![tf_dimension_loads](../CDI/taskflows/tf_dimension_loads/tf_dimension_loads.png)

---

## Download Taskflow Export
[Download Taskflow Export (ZIP)](../CDI/taskflows/tf_dimension_loads/tf_dimension_loads.zip)

---

## Step-by-Step Taskflow Logic

| Step | Task Name | Type | Description |
|------|------------|------|--------------|
| 1 | Start | — | Initializes runtime parameters such as `$$RUN_DATE` |
| 2 | dt_Load_Dimension_Staging | Data Task | Loads current snapshot into staging and checks record count |
| 3 | dec_Check_Snapshot_Count | Decision | Verifies if snapshot has new or updated records |
| 4 | ntf_Change_Detected_Alert | Notification | Sends alert when change is detected in source snapshot |
| 5 | mt_SCD2_Date_MD5_Process | Mapping Task | Compares MD5 hash to detect data changes (Type-2 with start/end dates) |
| 6 | mt_SCD2_Flag_Update | Mapping Task | Updates `IS_ACTIVE` flag to maintain current vs expired records |
| 7 | Audit_Run_Summary (optional) | Mapping Task | Logs inserted, updated, and expired row counts |
| 8 | End | — | Ends taskflow and clears runtime variables |

---

## Sample Dataset

### Day-1 Snapshot
| CUSTOMER_ID | CUSTOMER_NAME | CITY   | EMAIL                  | LOAD_DATE |
|-------------:|---------------|--------|------------------------|------------|
| 101 | Rahul Mehta | Mumbai | rahul.m@example.com | 2025-10-15 |
| 102 | Sneha Iyer  | Pune   | sneha.iyer@example.com | 2025-10-15 |
| 103 | Rajiv Soni  | Delhi  | rajiv.soni@example.com | 2025-10-15 |

---

### Day-2 Snapshot
| CUSTOMER_ID | CUSTOMER_NAME | CITY       | EMAIL                   | LOAD_DATE |
|-------------:|---------------|------------|--------------------------|------------|
| 101 | Rahul Mehta | Bangalore | rahul.m@example.com | 2025-10-16 |
| 102 | Sneha Iyer  | Pune       | sneha.iyer@example.com | 2025-10-16 |
| 103 | Rajiv Soni  | Delhi      | rajiv.soni@example.com | 2025-10-16 |
| 104 | Neha Gupta  | Chennai    | neha.gupta@example.com | 2025-10-16 |

---

## Expected Target After Taskflow Run (SCD2)

| DIM_KEY | CUSTOMER_ID | CUSTOMER_NAME | CITY      | EMAIL                   | START_DATE | END_DATE   | IS_ACTIVE |
|---------:|-------------:|---------------|-----------|--------------------------|-------------|-------------|-----------:|
| 1 | 101 | Rahul Mehta | Mumbai    | rahul.m@example.com     | 2025-10-15 | 2025-10-16 | 0 |
| 2 | 101 | Rahul Mehta | Bangalore | rahul.m@example.com     | 2025-10-16 | 9999-12-31 | 1 |
| 3 | 102 | Sneha Iyer  | Pune      | sneha.iyer@example.com  | 2025-10-15 | 9999-12-31 | 1 |
| 4 | 103 | Rajiv Soni  | Delhi     | rajiv.soni@example.com  | 2025-10-15 | 9999-12-31 | 1 |
| 5 | 104 | Neha Gupta  | Chennai   | neha.gupta@example.com  | 2025-10-16 | 9999-12-31 | 1 |

---

## Notes
- Implements complete SCD Type-2 (Date + MD5) logic.  
- Handles change detection, versioning, and historical preservation.  
- Maintains `START_DATE`, `END_DATE`, and `IS_ACTIVE` for tracking active vs expired rows.  
- Tested using Informatica IICS Cloud Data Integration with Oracle Target.  
- Includes retry handling and notification branching for production readiness.

---

## Related Assets
- Mappings:  
  - `m_SCD_Type2_Date_MD5`  
  - `m_SCD_Type2_Flag`  
- Taskflow: `tf_dimension_loads`  
- Target Table: `DIM_CUSTOMER`

---

© Shashi Kant — IICS Projects Portfolio
