
# 📊 **AWS QuickSight – In-Depth (with Modes & Embedding)**

---

## 🔍 What is QuickSight?

**AWS QuickSight** is a **serverless**, **scalable** Business Intelligence (BI) tool to create:

* Dashboards
* Reports
* Ad-hoc visual analysis

It supports **interactive dashboards**, **ML-powered insights**, and **embedding** into applications.

> Think of it as **AWS-native Tableau / Power BI**, but cloud-native and pay-as-you-go.

---

## 🎯 Why Use QuickSight?

| Feature                       | Benefit                                                            |
| ----------------------------- | ------------------------------------------------------------------ |
| ✅ **Serverless**              | No infrastructure to manage (auto-scales to 1000s of users)        |
| ⚡ **SPICE engine**            | provides fast, in-memory querying for big data (only for data imported in quicksight)                              |
| 🔗 **Native AWS integration** | Seamless access to S3, Athena, RDS, Redshift, Glue, etc.           |
| 💸 **Pay-per-session**        | Cost-efficient for large reader audiences                          |
| 🧠 **ML Insights**            | Built-in anomaly detection, forecasting, and natural language Q\&A |
| 🧩 **Embedding support**      | Integrate dashboards into web apps or SaaS platforms               |

---
Here’s a clear breakdown of the **two modes** in AWS QuickSight for dashboards, focusing on:

* **Caching vs Live querying**
* **Whether they support embedding**

---
Absolutely! Here's an extended and refined version of your QuickSight Dashboard Modes table, giving you a **more detailed comparison**—perfect for revision, interviews, or documentation.

---

### ⚙️ **QuickSight Dashboard Modes**

| Mode                  | 🧠 **SPICE (In-Memory Engine)**                                                              | 🔄 **Direct Query (Live Connection)**                                               |
| --------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Definition**        | QuickSight **copies and caches** your dataset in a **high-speed, in-memory engine (SPICE)**. | QuickSight connects **live** to the data source and runs **queries on demand**.     |
| **Data Freshness**    | ❌ **Static** – Data remains the same **until explicitly refreshed** (manual or scheduled).   | ✅ **Real-time** – Always reflects the **latest data** from the underlying database. |
| **Performance**       | ⚡ **Very fast** – Ideal for large datasets and high concurrency dashboards.                  | 🐢 Depends on database performance; **may introduce latency** under load.           |
| **Query Cost**        | ✅ Minimal – No load on your database after caching.                                          | 💸 Higher – Each user/dashboard access **queries the source**, increasing costs.    |
| **Scaling**           | 🌐 Excellent – Handles thousands of users without DB strain.                                 | 🚫 Limited by source DB scalability and connection limits.                          |
| **Data Size Limits**  | \~250 million rows / 500 GB per dataset (limits vary by region and edition).                 | No ingestion limit (but bound by your source system's limits).                      |
| **Refresh Options**   | ⏱️ Manual refresh, scheduled (hourly/daily), or via API trigger.                             | 🔁 Always up-to-date – no need for refresh schedules.                               |
| **Offline Tolerance** | ✅ Yes – Dashboards still work even if source DB is temporarily unavailable.                  | ❌ No – Dashboard fails if the data source is down or unreachable.                   |
| **Ideal Use Cases**   | - Executive dashboards<br>- Periodic reporting<br>- Cost-sensitive BI<br>- High-speed needs  | - Monitoring dashboards<br>- Real-time analytics<br>- Operational decision-making   |
| **Embedding Support** | ✅ Yes – Can embed dashboards using IAM or anonymous embedding.                               | ✅ Yes – Embedding works seamlessly with dynamic data access.                        |
| **Security & RLS**    | ✅ Full support for IAM + Row-level security.                                                 | ✅ Full support for IAM + Row-level security.                                        |
| **Pricing Impact**    | 💰 Pay for SPICE capacity (\$0.25/GB/month after 10 GB free).                                | 💰 Pay for each query (e.g., Athena costs, RDS CPU, etc.).                          |

---

> ✅ **Both modes support embedding**, but:

* Use **SPICE** for speed and cost efficiency with hourly/daily refreshes.
* Use **Direct Query** when **real-time data** is critical (e.g., monitoring dashboards).

---

## 🧩 Embedding QuickSight Dashboards

You can embed dashboards into:

* Your **SaaS app**
* **Internal tools** or customer portals
* **White-label** analytics for clients

### 🔐 Access Options

| Type                    | Explanation                                                             |
| ----------------------- | ----------------------------------------------------------------------- |
| **User-based (IAM)**    | Authenticated AWS users see dashboards via roles                        |
| **Anonymous embedding** | View-only dashboards for unauthenticated users (reader pricing applies) |
| **Row-level security**  | Embed one dashboard, but restrict content per user                      |

---

### 🧠 Is Embedded Dashboard Real-Time?

| Backing Mode     | Behavior in Website Embed                                     |
| ---------------- | ------------------------------------------------------------- |
| **SPICE**        | ❌ Not real-time; users see latest cached data at last refresh |
| **Direct Query** | ✅ Fully dynamic — reflects live database changes on each view |

✅ You can **embed both SPICE and Direct Query dashboards** — the mode determines whether it's real-time.

> **With regular SPICE refreshes (e.g., every 15 mins)**, you can get "almost real-time" with fast performance.

---

## 🔗 Supported Data Sources

| AWS Native        | External & Other                    |
| ----------------- | ----------------------------------- |
| S3 (via Athena)   | Snowflake, SQL Server, Teradata     |
| Athena            | Salesforce                          |
| Amazon Redshift   | MySQL/PostgreSQL (on-prem or cloud) |
| Amazon RDS        | Excel, CSV, local uploads           |
| Aurora            | Google Sheets (via connectors)      |
| Glue Data Catalog |                                     |

---

## 📈 How QuickSight Works (Flow)

```
1. Connect to Data Source (RDS, Athena, etc.)
      ↓
2. Create a Dataset
      ↓
3. Choose SPICE or Direct Query
      ↓
4. Build Analysis (charts, tables)
      ↓
5. Publish Dashboard
      ↓
6. Embed in apps or share via URL
```

---

## 🛡️ Security & Access

| Feature            | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| IAM-based control  | Use IAM policies to manage authors/readers                   |
| Row-Level Security | Use filtering to show personalized data in shared dashboards |
| KMS Integration    | Encryption at rest using AWS KMS                             |
| VPC Access         | Private access to RDS/Redshift inside VPC                    |

---

## 🧾 Pricing Overview

| Component         | Price                                                               |
| ----------------- | ------------------------------------------------------------------- |
| **Authors**       | \~\$18/month per user                                               |
| **Readers**       | \~\$0.30 per session (max 1 daily session charge)                   |
| **SPICE storage** | First 10 GB free, then \$0.25/GB/month                              |
| **Direct Query**  | You pay for the actual data source access (e.g., Athena query cost) |

---

## 🧠 ML Insights (Enterprise Edition)

1. **Anomaly Detection** – Auto-detect outliers in time series
2. **Forecasting** – Predict values (linear/regression)
3. **Natural Language (Q bar)** – Ask questions like “What were sales in June?”

---

## 🔧 Real-World Architecture Example

### Use Case: BI Dashboard for IoT Data

**S3 + Glue + Athena + QuickSight**

1. IoT logs land in **S3**
2. **AWS Glue Crawler** builds schema in **Glue Data Catalog**
3. **Athena** queries the data
4. **QuickSight** connects to Athena → dashboard created
5. Dashboard is embedded into internal tool

> ✅ Fully serverless, scalable, and cost-efficient

---

## 🎯 Exam Tips (SAA-C03)

| Tip                                                | Why it matters                                              |
| -------------------------------------------------- | ----------------------------------------------------------- |
| Use SPICE to improve performance                   | Avoids excessive load on RDS/Redshift                       |
| Know difference: SPICE vs Direct                   | Key exam scenario for dashboard freshness vs speed tradeoff |
| Use Row-Level Security in shared apps              | To restrict per-user data without duplicating dashboards    |
| Embedded dashboards use IAM or anonymous embedding | Control access & cost effectively                           |

---

## ✅ Summary

| Question                           | Answer                                                                   |
| ---------------------------------- | ------------------------------------------------------------------------ |
| Can I embed QuickSight dashboards? | ✅ Yes – with IAM or anonymous options                                    |
| Are embedded dashboards real-time? | ✅ Only if using **Direct Query**                                         |
| Does SPICE lose data?              | ❌ No – it's durable until refreshed or deleted                           |
| Can I use both modes together?     | ✅ Yes – one dataset can be SPICE, another Direct Query in same dashboard |
| Should I always use SPICE?         | Use when speed & cost matter more than up-to-the-second freshness        |

---

Let me know if you want:

* 📌 Architecture diagrams
* 🧠 SPICE vs Direct visual
* 🔁 SPICE refresh flow
* 📚 Comparison: QuickSight vs Tableau/PowerBI

Ready for the next service from Stephane's SAA course?
