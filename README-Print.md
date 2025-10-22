## Data Governance - Data Asset Score

**🔹 Background & Motivation**

> Rapid growth of payments business exposed chaos in our Hive/Spark data layer: inconsistent table names, missing comments, unmanaged dependencies, quality checks, security compliance, or cost inefficiencies.

1. Table Standards (35%): naming, comments, dependency hygiene
2. Data Quality Checks (35%): SLA‑driven timeliness, DQC rule coverage, alert management
3. Security (15%): sensitive‑field encryption & owner compliance
4. Cost (15%): compute and storage cost

Data Governance for 🚀 SLA Optimisation

| No. | ✨ Optimisation Area                 | 📌 Description                                                                                                          |
|-----|--------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| 1️⃣ | 🔗 **Workflow Dependency**           | Removed non-critical and redundant dependencies to streamline DAG execution.                                           |
| 2️⃣ | ⏱️ **Trigger-Based Scheduling**      | Replaced fixed-time triggers with dependency-based scheduling.<br>Tasks now auto-execute upon upstream success.        |
| 3️⃣ | 🚨 **Monitoring & Alerting**         | Added alerting for job failures and delays, enabling early detection and faster troubleshooting.                       |
| 4️⃣ | 🧩 **Spark Job Optimization**        | Prioritized optimization of long-running (1h+) critical path jobs and de-emphasized low-impact ones.                   |

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
    DAS_TITLE --> STD_HDR
    DAS_TITLE --> DQ_HDR
    DAS_TITLE --> SECO_HDR

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

**SECURITY Standards**

**COST Standards**
