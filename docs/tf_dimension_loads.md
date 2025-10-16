#  Taskflow: `tf_dimension_loads`

##  Objective
Automate the **Dimension Load process** using **SCD Type 2 (Date + MD5)** logic in Informatica IICS.  
This taskflow performs *change detection*, *conditional execution*, *retry handling*, and *notifications* to ensure efficient and reliable dimension table updates.

---

##  Functional Overview

| Step | Component | Description |
|------|------------|--------------|
| 1 | **Start** | Entry point of the taskflow execution. |
| 2️ | **Data Task – `dt_Load_Dimension_Staging`** | Runs the mapping `m_data_availability` to verify if source or staging data is available for processing. |
| 3️ | **Decision – `dec_Check_Snapshot_Count`** | Evaluates whether new or changed data exists (e.g., record count ≥ 1). |
| 4️ | **Notification – `ntf_Change_Detected_Alert`** | Sends an email notification confirming that changes are detected. |
| 5️ | **Mapping Task – `mt_SCD2_Date_MD5_Process`** | Executes mapping `m_SCD_Type2_Date_MD5` to handle historical data tracking using MD5 comparison and date-based logic (Start_Date, End_Date, Is_Active). |
| 6️ | **Mapping Task – `mt_SCD2_Flag_Update`** | Executes mapping `m_SCD_Type2_Flag` to update active/inactive flags after new record insertion. |
| 7️ | **Sub-Taskflow – `tf_Load_Fact_Orchestration`** | Triggers the sub-taskflow (`tf_single_task`) to orchestrate dependent loads like `m_Load_every_4th_record` or downstream facts. |
| 8️ | **Assignment – `asg_Set_Default_Counters`** | Initializes retry counters and default variables when no new data is detected. |
| 9️ | **Notification – `ntf_No_Change_Alert`** | Sends a message notifying that no change was detected in the source. |
| 10 | **Decision – `dec_Check_Retry_Count`** | Compares retry attempts with a limit (e.g., `< 4`). |
| 1️1️ | **Wait – `wait_Retry_Interval_5min`** | Waits for a defined duration (e.g., 5 minutes) before retrying. |
| 1️2️ | **Notification – `ntf_Retry_Limit_Reached`** | Sends a final alert when retry limit is reached. |
| 1️3️ | **Throw – `throw_Abort_Dimension_Load`** | Aborts the taskflow gracefully if maximum retries exceeded. |
|  | **End** | Indicates successful or controlled completion. |

---

##  Design Highlights

- **Integrates multiple mappings**: `m_data_availability`, `m_SCD_Type2_Date_MD5`, and `m_SCD_Type2_Flag`.
- **Uses MD5 hash comparison** to detect record-level changes efficiently.
- **Maintains SCD Type 2 history** with Start/End Dates and Active Flags.
- **Implements Retry Logic** with wait intervals to handle transient failures.
- **Includes Notifications** for every major event — success, no change, and failure.
- **Calls Sub-Taskflow** for modular orchestration of dependent loads.

---

##  Related Assets

| Type | Name | Purpose |
|------|------|----------|
| Mapping | `m_data_availability` | Checks data existence before triggering the dimension load. |
| Mapping | `m_SCD_Type2_Date_MD5` | Performs MD5-based change detection and inserts historical versions. |
| Mapping | `m_SCD_Type2_Flag` | Updates active/inactive status for SCD records. |
| Taskflow | `tf_single_task` | Sub-taskflow used to demonstrate sequential mapping orchestration (e.g., `m_Load_every_4th_record`). |

---

##  Export Information

| Asset | File Name | Export Timestamp | Exported On (IST) |
|--------|------------|------------------|------------------|
| Taskflow | `tf_dimension_loads-1760639741312.zip` | `1760639741312` | 🗓 **Oct 17, 2025 – 8:45 PM IST** |

---

##  Folder Location

```
iics-projects-portfolio/
 └── CDI/
     ├── mappings/
     │   ├── m_SCD_Type2_Date_MD5.zip
     │   ├── m_SCD_Type2_Flag.zip
     │   ├── m_data_availability.zip
     │   └── m_Load_every_4th_record.zip
     ├── taskflows/
     │   ├── tf_dimension_loads.png
     │   └── tf_dimension_loads.md
     └── jobs_exports/
         └── tf_dimension_loads-1760639741312.zip
```

---

##  Summary

The `tf_dimension_loads` taskflow is the **controller** for your entire dimension load pipeline.  
It orchestrates multiple mappings to detect changes, load historical records, and update flags in dimension tables.  
It further manages notification alerts, retries, and failure handling to ensure smooth and reliable orchestration.

---

✅ **Outcome:**
A production-grade, fault-tolerant taskflow for dimension data management — integrating **change detection, historical tracking, retry automation, and alerting** in Informatica IICS.
