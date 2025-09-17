
> 🛡️ **Disclaimer:**  
> The following content represents generalized industry knowledge and anonymized case practices.  
> It does **not contain any confidential, proprietary, or internal information** from any specific company.

---

# Data Warehouse Solutions

The core purpose of a data warehouse is to integrate and store large amounts of internal and external data, providing accurate, reliable data for analysis, reporting, and decision-making, while addressing issues like **fragmentation**, and difficult historical data management.

## 1. Data Warehouse Architecture - Hourglass

Built a layered data warehouse (ODS > DIL > DML > DAL) to ingest, clean, and transform data into fact and dimension tables. Defined data domains, granularity, metrics, and embedded business logic for subject-oriented, multi-dimensional analysis

<details>
<summary>💡Why Layered Design?</summary>

Dimensional Modeling
  It follows the principle of data layering (e.g., DIL, DML), which enables clear separation of concerns between raw data integration and subject-oriented analysis.

> - Clarifies responsibilities across layers (e.g., raw events vs. analysis-ready data)  
> - Supports atomic and aggregated metrics  
> - Improves reusability and maintainability  
> - Aligns with modern data warehouse best practices (e.g., Kimball methodology)
> - 🧭 Industry Terminology — DIL and DML follow the same layered logic as DWD/DWS in other companies. Naming may different, but all follow Kimball-style dimensional modeling.


</details>

<div align="center">
  <img src="docs/dwh-1.jpg" alt="Diagram" width="700">
</div>

---

**Data Warehouse Planning :**  

> **End‑to‑end-from Planning → Dimension Management → Metric Definition → Physical Schema Modeling → Subject‑Area Delivery—featuring clear separation of layers and single‑responsibility.**

<details>
<summary>Professional Term Explanation</summary>

| No. | Term                            | Description                                                                 |
|-----|---------------------------------|-----------------------------------------------------------------------------|
| 1   | Data Warehouse Planning         | High-level planning of the warehouse, including domains, granularity, and load strategies. |
| 2   | Business Domain                 | High-level business categories such as `Cross-border Payment` and `Credit Card`. |
| 3   | Business Process                | Specific workflows describing how data flows through business operations.   |
| 4   | Data Domain                     | Logical grouping of data, e.g., user, product, funds, contract.             |
| 5   | Granularity                     | Level of detail in data (e.g., per transaction, per day, per user).         |
| 6   | Constraints                     | External requirements such as SLA, compliance, or system limitations.       |
| 7   | Load Strategy                   | Full or incremental data ingestion approach.                                |
| 8   | Dimension Management            | Design and governance of dimensions and their hierarchies.                  |
| 9   | Dimension Tables                | Tables that describe entities used for slicing facts, such as user. |
| 10  | Metric Definition               | Systematic definition and classification of metrics.                        |
| 11  | Atomic Metrics                  | Direct metrics from raw events with no transformation (stored in DIL).      |
| 12  | Simple Derived Metrics          | Lightly transformed fields like `age_group`, can exist in DIL or DML.       |
| 13  | Complex Derived Metrics         | Aggregated metrics involving logic or multiple tables (mainly in DML).      |
| 14  | Horizontal Metrics              | Timepoint or milestone fields (e.g., first_payment_time), stored horizontally, one row per entity. |
| 15  | Vertical Metrics                | Aggregated tags or metrics stacked by type (e.g., trd_cnt_30d, trd_amt_month). |
| 16  | Physical Schema Modeling        | The process of creating actual dimension and fact tables in the warehouse.  |
| 17  | Fact Tables                     | Tables that store measurable events, often with foreign keys to dimensions. |
| 18  | Data-Mart / Subject Modeling    | Design of wide analytical tables for multi-metric, multi-perspective analysis, focused on a specific entity such as user, merchant, or order. |

data development process：
1. Defined business goals and requirements.
2. Collected data into ODS and integrated into fact and dimension tables (DIL/DIM).
3. Organised data domains, determined data granularity, and designed key metrics.
4. Abstracted business and data subject analyses into DML tables.
5. Delivered reporting, supporting subject-specific and multi-dimensional analysis
</details>

```mermaid
flowchart TB

%% Planning
subgraph PL["PL"]
  direction TB
  Data_Warehouse_Planning["📦 Data Warehouse Planning"]:::topNode
  PL1["Business Domain"]:::subgroupNode
  PL2["Business Processe"]:::subgroupNode
  PL3["Data Domain"]:::subgroupNode
  PL5["Granularity"]:::subgroupNode
  PL6["Constraints"]:::subgroupNode
  PL7["Load Strategy"]:::subgroupNode
end

%% Dimension Management
subgraph DM["DM"]
  direction TB
  Dimension_Management["📘 Dimension Management"]:::secondNode
  DM1["Dimension Classification"]:::yellowNode
  DM2["Dimension Tables"]:::yellowNode
  DM3["Dimension Attributes"]:::yellowNode
end

%% Metric Definition
subgraph MD["MD: Metric Definition"]
  direction TB
  Metric_Definition["📊 Metric Definition"]:::secondNode
  AM["Atomic Metrics<br>(DIL / Fact Tables)"]:::purpleNode
  SM["Simple Derived Metrics<br>(DIL or DML)"]:::purpleNode
  CM["Complex Derived Metrics<br>(Mainly in DML)"]:::purpleNode
end

%% Physical Schema Modeling
subgraph PH["PH"]
  direction TB
  Schema_Physical["📐 Physical Schema Modeling"]:::secondNode
  SM1["Dimension Tables"]:::blueNode
  SM2["Fact Tables"]:::blueNode
end

%% Data‑Mart / Subject Modeling
subgraph DMART["D-MART"]
  direction TB
  Data_Mart["🎯 Data‑Mart Modeling"]:::secondNode
  SM3["Subject Tables"]:::blueNode
  WAT3["Wide / Aggregated Tables"]:::blueNode
end

%% End‑to‑end Flow
Data_Warehouse_Planning --> Dimension_Management
Dimension_Management   --> Metric_Definition
Metric_Definition      --> Schema_Physical
Schema_Physical        --> Data_Mart

%% Styling
classDef topNode      fill:#D1F2EB,stroke:#117A65,stroke-width:2px;
classDef secondNode   fill:#EAF2F8,stroke:#2874A6,stroke-width:2px;
classDef subgroupNode fill:#FCF3CF,stroke:#B7950B,stroke-width:1px;
classDef yellowNode   fill:#FDEBD0,stroke:#CA6F1E,stroke-width:1px;
classDef purpleNode   fill:#EBDEF0,stroke:#884EA0,stroke-width:1px;
classDef blueNode     fill:#D6EAF8,stroke:#2E86C1,stroke-width:1px;
```

## 2. Data Governance - Data Asset Score

**🔹 Background & Motivation**

> Rapid growth of payments business exposed chaos in our Hive/Spark data layer: inconsistent table names, missing comments, unmanaged dependencies, quality checks, security compliance, or cost inefficiencies.

> Previously, only Data Quality Checks (DQC) were used to evaluate data assets. In this project, expansion of the Data Asset Scoring mechanism by introducing 3 new dimensions:
> Table Standards (35%) & Security (15%) & Cost (15%)  **Combined with DQC (35%)**, we built a comprehensive 100-point scoring system that evaluates the usability, reliability, compliance, and cost-efficiency of data assets.

<details>
<summary><strong>🎯 Goals & Expected Benefits</strong></summary>  
Updating the Data Asset Scoring framework (0–100 points) to quantify each table’s:

1. Table Standards (35%): naming, comments, dependency hygiene
2. Data Quality Checks (35%): SLA‑driven timeliness, DQC rule coverage, alert management
3. Security (15%): sensitive‑field encryption & owner compliance
4. Cost (15%): compute and storage cost

</details>
  
<details>
<summary><strong>⚙️ Design & Implementation</strong></summary>

1. Scoring Rules automated via SparkSQL jobs running daily;
2. Table (names, comments, dependencies) extracted from Hive Meta Table & Lineage Relationship Table.
3. DQC rules stored and versioned in a rule table, evaluation output is written into a DQC_Score table.
4. Security – Perform sensitive‑field encryption checks using the scan results supplied by the data-security‑platform team
5. Cost – Implemented by our Data Platform team via daily scans for stale/“garbage” tables and by defining table lifecycle stages. Each day’s cost evaluation output is written into a Cost_Score table.
6. Whitelist Mechanism allows table owners to apply for temporary exemptions.
7. Finally, together the scores from all four dimensions, applied our weighted formula, and loaded the consolidated score into the central Data Asset Score table.

| Field Name    | Description          |
| ------------- | -------------------- |
| fdate  | Date           |
| fetl_time  | ETL time           |
| ftable\_name  | Table name           |
| fowner        | Table owner          |
| fbusiness     | Business Domain   |
| fstd\_score   | Standards score      |
| fdqc\_score   | Data quality score   |
| fsecu\_score  | Security score       |
| fcost\_score  | Cost score           |
| ftotal\_score | Total score          |
| fscore\_time  | Scoring timestamp    |
| fexempt\_flag | Exemption flag (Y/N) |
</details>

<details>
<summary><strong>🚀 Push via Platform & Automation & Manual configuration</strong></summary>

- ✅ Enforced **naming conventions** for all new tables through platform-level **constraints** in the visual table-creation process
- ✅ **Auto-configured** zero-record checks and primary key uniqueness constraints  
- ✅ Enhanced **whitelist governance**, preventing exempt tables from impacting scoring  
- ✅ **Excluded** temporary tables prefixed with `temp_`, `tmp_`, or `check_` from evaluation scope

</details>

<details>
<summary><strong>📈 Results : Average asset score improved from 77+ → 86+</strong></summary>

> Overall Health Improvement
> On a 100‑point scale, portfolio of tables has moved from the “C+” range up into the “B+” range—meaning that, on average, assets now meet governance criteria (naming standards, DQC coverage, security and cost controls).

</details>

> 🌱 Future Extensions: Incorporate data‑usage heatmaps & Add partition‑level DQC quality checks.

<details>
<summary>Data Governance for 🚀 SLA Optimisation</summary>

| No. | ✨ Optimisation Area                 | 📌 Description                                                                                                          |
|-----|--------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| 1️⃣ | 🔗 **Workflow Dependency**           | Removed non-critical and redundant dependencies to streamline DAG execution.                                           |
| 2️⃣ | ⏱️ **Trigger-Based Scheduling**      | Replaced fixed-time triggers with dependency-based scheduling.<br>Tasks now auto-execute upon upstream success.        |
| 3️⃣ | 🚨 **Monitoring & Alerting**         | Added alerting for job failures and delays, enabling early detection and faster troubleshooting.                       |
| 4️⃣ | 🧩 **Spark Job Optimization**        | Prioritized optimization of long-running (1h+) critical path jobs and de-emphasized low-impact ones.                   |

</details>

```mermaid
flowchart LR

    %% ============ OUTER CONTAINER ============
    subgraph DAS[" "]
      direction TB

      %% --- Big Title Node (prominent) ---
      DAS_TITLE["<br/>**DATA ASSET SCORE**<br/>"]:::title

      %% --- 1) STANDARD (35%) ---
      subgraph STDG[" "]
        direction TB
        STD_HDR["📚 **Standard (35%)**"]:::group
        ST_A["Naming Conventions<br>50%"]:::item
        ST_B["Comment Standards<br>37.5%"]:::item
        ST_C["Dependency Standards<br>12.5%"]:::item
        STD_HDR --> ST_A & ST_B & ST_C
      end

      %% --- 2) DQC (35%) ---
      subgraph DQG[" "]
        direction TB
        DQ_HDR["✅ **Data Quality Check (35%)**"]:::group
        DQ_A["Timely Monitoring<br>20%"]:::item
        DQ_B["DQC Coverage<br>50%"]:::item
        DQ_C["Alert Management<br>30%"]:::item
        DQ_HDR --> DQ_A & DQ_B & DQ_C
      end

      %% --- 3) SECURITY + COST (15% + 15%) ---
      subgraph SECOG[" "]
        direction TB
        SECO_HDR["🔐💰 **Security + Cost (15% + 15%)**"]:::group
        SECO_A["Sensitive Field Encryption<br>Owner Compliance"]:::item
        SECO_B["Compute Cost<br>Storage Cost"]:::item
        SECO_HDR --> SECO_A & SECO_B
      end
    end

    %% ============ FLOW / ORDER ============
    DAS_TITLE --> STD_HDR --> DQ_HDR --> SECO_HDR

    %% ============ STYLES ============
    %% Title: strong orange, thick border, bold
    classDef title fill:#FFB74D,stroke:#333,stroke-width:3px,font-weight:bold;

    %% Group headers: teal/blue-green with white text for strong contrast
    classDef group fill:#26A69A,stroke:#0D5C55,stroke-width:2px,color:#FFFFFF;

    %% Items: light blue with slightly darker blue border
    classDef item fill:#E3F2FD,stroke:#1565C0,stroke-width:1.5px;
```

**Table Standards**

```mermaid
flowchart TB
  %% Top-level: Standards
  STD["Table Standards (35%)"]
  STD --> NS["Naming Convention 50%"]
  STD --> CS["Comment Standard 37.5%"]
  STD --> DS["Dependency Standard 12.5%"]

  %% Naming Convention Subgroup
  subgraph NS-Group
    direction TB
    NS1["a. prefix"]
    NS2["b. suffix"]
    NS3["c. business domain"]
    NS4["d. data domain"]
  end
  NS --> NS-Group

  %% Comment Standard Subgroup
  subgraph CS-Group
    direction TB
    CS1["a. table comment"]
    CS2["b. column comment "]
  end
  CS --> CS-Group

  %% Dependency Standard Subgroup
  subgraph DS-Group
    direction TB
    DS1["a. Backward dependency <br> b. ODS-layer dependency"]
  end
  DS --> DS-Group

  %% Styling
  classDef topNode fill:#98FB98,stroke:#333,stroke-width:2px
  classDef secondNode fill:#ADD8E6,stroke:#333,stroke-width:2px
  classDef subgroupNode fill:#F5B7B1,stroke:#333,stroke-width:1px
  classDef purpleNode fill:#A9CCE3,stroke:#333,stroke-width:1px
  classDef yellowNode fill:#F9E79F,stroke:#333,stroke-width:1px

  class STD topNode
  class NS,CS,DS secondNode
  class NS1,NS2,NS3,NS4 subgroupNode
  class CS1,CS2,CS3,CS4 purpleNode
  class DS1,DS2 yellowNode
```

**Data Quality Check**

```mermaid
flowchart TB
  %% Top-level: Data Quality
  DQ["Data Quality (35%)"]
  DQ --> TM["Timeliness Monitoring Coverage 20%"]
  DQ --> DQC["DQC Coverage 50%"]
  DQ --> QAM["Quality Alert Ticket 30%"]

  %% Timeliness Monitoring Subgroup
  subgraph TM-Group
    direction TB
    TM1["a. Task time > 1 day"]
    TM2["b. Task time exceeds <br> Layer SLA commitment"]
  end
  TM --> TM-Group

  %% DQC Assurance Subgroup
  subgraph DQC-Group
    direction TB
    DQC1["a. zero records in partition"]
    DQC2["b. Uniqueness rule"]
  end
  DQC --> DQC-Group

  %% Quality Alert Subgroup
  subgraph QAM-Group
    direction TB
    QAM1["a. ≥3 unresolved tickets"]
    QAM2["b. ≥1 unresolved tickets"]
  end
  QAM --> QAM-Group

  %% Styling
  classDef topNode fill:#98FB98,stroke:#333,stroke-width:2px
  classDef secondNode fill:#ADD8E6,stroke:#333,stroke-width:2px
  classDef pinkNode fill:#F5B7B1,stroke:#333,stroke-width:1px
  classDef purpleNode fill:#A9CCE3,stroke:#333,stroke-width:1px
  classDef yellowNode fill:#F9E79F,stroke:#333,stroke-width:1px

  class DQ topNode
  class TM,DQC,QAM secondNode
  class TM1,TM2 pinkNode
  class DQC1,DQC2,DQC3,DQC4 purpleNode
  class QAM1,QAM2,QAM3,QAM4 yellowNode
```
