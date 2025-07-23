# SLA Improvement ? 95% to 99%+.

| No. | ✨ Optimization Area                     | 📌 Description                                                                                                                        |
| ---------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 1️⃣ | **🔗 Workflow_Dependency** | Removed non-critical and redundant dependencies to streamline execution flow.                                                         |
| 2️⃣ | **⏱️ Trigger-Based Scheduling**         |  — Replaced time-based triggers with dependency-based scheduling <br> — tasks auto-trigger upon upstream success.                             |
| 3️⃣ | **⚙️ Spark\_Job\_Performance\_Tuning**     | Tuned long-running jobs (>1h): optimized `groupBy`, joins, partitioning, memory usage; reduced shuffles and improved execution speed. |
| 4️⃣ | **🚨 Monitoring & Alerting**            | Set up alerts for job failures and delays to ensure quick response and SLA adherence.                                                 |

## 🧠 SparkSQL - Optimization Case

> **The following are simplified examples. The real situation is more complicated.**

### init SQL

```sql
SELECT
  t.institution,
  t.sender_country,
  t.currency,
  COUNT(DISTINCT t.transaction_id) AS tx_count,
  COUNT(DISTINCT t.sender_id) AS sender_count,
  SUM(t.amount * fx.fx_to_cnh) AS total_cnh,
  SUM(t.amount * fx.fx_to_usd) AS total_usd
FROM transactions t
JOIN exchange_rates fx ON t.currency = fx.currency AND t.fdate = fx.fdate
WHERE t.fdate BETWEEN '20240101' AND '20241231'
GROUP BY t.institution, t.sender_country, t.currency;
```

| Issue                            | Description                                                                                                               |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Data Skew in Remitting Countries | For example, transactions from the US exceed 12 million, accounting for 40%, causing severe skew in `group by + shuffle`. |
| Large Exchange Rate Table        | `exchange_rates` contains millions of daily records, causing high I/O during joins.                                       |
| COUNT DISTINCT Memory Pressure   | Multi-dimensional and multiple `COUNT DISTINCT` aggregations lead to memory spills and GC issues.                         |
| Improper Aggregate Type          | Default use of `HashAggregate` is prone to OOM under skewed data conditions.                                              |

