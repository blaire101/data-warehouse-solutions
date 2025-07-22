# 📊 Data Governance – Data Quality Overview

## 🔍 Background

- **Total Tables**: 30,000+  
- **Average Asset Score**: 78+  
- **Naming Convention Compliance Score**: 55+  
  > Indicates poor adherence to table naming standards  
- **DQC (Data Quality Check) Coverage Score**: 47+  
  > Reflects insufficient data quality assurance across tables

---

## ⚙️ Data Quality Operations

Data quality is evaluated across **four dimensions**:

| Dimension | Weight | Description |
|-----------|--------|-------------|
| **Standards** | 35% | Naming and domain consistency |
| **DQC Rules** | 35% | Monitoring rule configuration and coverage |
| **Security** | 15% | Access control and privacy compliance |
| **Cost** | 15% | Resource consumption and table lifecycle management |

---

### 🧩 Standards (35%)

**Scoring Rule**:  
If **any** of the following violations occur, the table receives **0 points**:

- **a.** Table prefix does not follow naming conventions  
- **b.** Table suffix (update frequency) is non-compliant  
- **c.** Business domain in table name is invalid  
- **d.** Data domain in table name is invalid  

---

### 🛠️ DQC Rules (35%)

**Scoring Rule**:

| Violation | Score |
|-----------|-------|
| a. Null-drop rules missing | 0 points |
| b. Row count fluctuation rules missing | 60 points |
| c. Uniqueness rules missing | 60 points |

---

### 🔐 Security & 💰 Cost (15% + 15%)

> Ensures that tables comply with security practices and are managed efficiently in terms of storage and compute costs.

---

## 🚀 Push via Platform & Automation

- ✅ Enforced **naming conventions** and **dependency configuration** for all new tables through platform-level upgrades  
- ✅ **Auto-configured** zero-record checks and primary key uniqueness constraints  
- ✅ Enhanced **whitelist governance**, preventing exempt tables from impacting scoring  
- ✅ **Excluded** temporary tables prefixed with `temp_`, `tmp_`, or `check_` from evaluation scope

---

## 📈 Results

- ✅ **Average asset score improved** from **77 → 86**  
- ✅ **SLA compliance increased** from **95% → 99%+**, driven by workflow optimization and scheduling improvements
