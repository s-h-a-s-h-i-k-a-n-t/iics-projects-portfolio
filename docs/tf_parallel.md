# Taskflow: `tf_parallel`

## Objective
Demonstrate **parallel task orchestration** in Informatica IICS where multiple mappings execute simultaneously.  
This taskflow showcases how to **reduce overall execution time** using parallel branches while maintaining control through **decisions** and **notifications**.

---

## Taskflow Screenshot
![tf_parallel](https://github.com/s-h-a-s-h-i-k-a-n-t/iics-projects-portfolio/blob/main/CDI/taskflows/tf_parallel.png)

---

## Functional Overview

| Step | Component | Name | Description |
|------|------------|------|--------------|
| 1 | **Start** | `tf_parallel` | Entry point of the taskflow execution. |
| 2 | **Parallel Paths 1** | `prl_SCD2_and_PC` | Splits execution into three parallel mapping tasks. |
| 3 | **Data Task** | `mct_m_SCD_Type2_Date_MD5` | Executes SCD Type 2 logic using MD5 and date comparison. |
| 4 | **Data Task** | `mct_m_SCD_Type2_Flag` | Updates `Is_Active` and `End_Date` fields in SCD target table. |
| 5 | **Data Task** | `mtt_PowerCenter_Task` | Runs a PowerCenter mapping integrated within IICS. |
| 6 | **Decision** | `dec_Equals_Pass` | Evaluates outcome of parallel branches — “Equals pass” indicates success. |
| 7 | **Notification Task 1** | `ntf_Success_Parallel_Pass` | Sends success notification email upon successful completion. |
| 8 | **Mapping Task** | `mtt_Load_4th_record` | Executes dependent mapping (loads every 4th record) after success. |
| 9 | **Notification Task 2** | `ntf_Failure_Parallel_Flag` | Sends failure notification if one or more tasks fail. |
| 10 | **End** | `End` | Terminates taskflow execution gracefully. |

---

## Design Highlights

- **Parallel Execution:** Runs three independent mappings concurrently to improve performance.  
- **Decision Control:** Post-parallel step verifies outcomes before proceeding.  
- **Conditional Notifications:** Sends alerts based on success or failure paths.  
- **Hybrid Integration:** Demonstrates mix of IICS and PowerCenter tasks in one workflow.  
- **Scalable Pattern:** Ideal for dimension or fact loads that can be run simultaneously.

---

## Related Assets

| Type | Name | Purpose |
|------|------|----------|
| Mapping | `mct_m_SCD_Type2_Date_MD5` | Detect and load changes using MD5 hash comparison. |
| Mapping | `mct_m_SCD_Type2_Flag` | Update flag columns (`Is_Active`, `End_Date`) post-insert. |
| Mapping | `mtt_PowerCenter_Task` | Example of a legacy PowerCenter mapping called from IICS. |
| Mapping | `mtt_Load_4th_record` | Demonstration mapping to load every 4th record. |

---

## Download Taskflow Export

[Download Taskflow Export (ZIP)](https://raw.githubusercontent.com/s-h-a-s-h-i-k-a-n-t/iics-projects-portfolio/main/CDI/jobs_exports/tf_parallel-17607314745360.zip)

---

## Summary
The `tf_parallel` taskflow demonstrates a **multi-branch orchestration pattern** — executing multiple mappings in parallel, validating outcomes via a decision step, and sending success or failure notifications.  
This design is ideal for enterprise ETL workflows requiring **parallel loading** and **conditional sequencing** in Informatica IICS.

---

✅ **Outcome:**  
A **parallelized, event-driven taskflow template** that speeds up execution and integrates notification logic for production-grade orchestration.
