Here are the notes on **AWS Athena** based on Stephane Maarek’s AWS SAA-C03 slides:

---

## 🔍 **AWS Athena – Querying Data in S3 with SQL**

---

### ✅ What is Athena?

* **Serverless** interactive query service.
* Allows you to **analyze data in Amazon S3** using **standard SQL**.
* federated query allows custom data sources.
* **No infrastructure** to manage.
* Based on **Presto** (open-source SQL engine) under the hood.

---

### 🗂️ How It Works

1. **Store raw data in S3** (CSV, JSON, Parquet, etc.).
2. Define a **schema** using the AWS Glue Data Catalog (or manually).
3. Run **SQL queries** on the data directly in S3 using Athena.
4. **Pay-per-query**: You pay for the **amount of data scanned**.

---

## Athena Federated Query

Run SQL queries with data source other than s3 using data source connector (lambda).

**What it is:**
- Query external data sources **directly** from Athena
- Data stays in its original location (RDS, DynamoDB, Elasticsearch, JDBC databases, etc.)
- Uses **connectors** (Lambda functions) to translate queries and fetch data

**How it works:**
1. Athena receives SQL query
2. Delegates to appropriate **data source connector** (Lambda function)
3. Connector fetches data from source system
4. Results returned to Athena for processing

**Key benefits:**
- ✅ Query multiple data sources with single SQL
- ✅ No ETL needed for one-off queries
- ✅ Join data across S3 and external sources

**Limitations:**
- ❌ Slower than native Athena on S3
- ❌ Connector Lambda costs apply
- ❌ Limited pushdown optimization

**Use case example:** 
```sql
-- Query joining S3 data with RDS data
SELECT s3.sales_id, rds.customer_name
FROM s3_sales_data s3
JOIN postgres_customers rds ON s3.cust_id = rds.id;
```

---

### 💡 Key Features

| Feature                  | Description                                       |
| ------------------------ | ------------------------------------------------- |
| **Serverless**           | No servers or clusters to provision or manage     |
| **SQL Support**          | Standard ANSI SQL syntax for querying             |
| **Fast & Interactive**   | Ideal for ad-hoc querying and data exploration    |
| **Integrates with Glue** | Uses AWS Glue as a schema/catalog manager         |
| **Secure**               | Supports encryption (S3 SSE), IAM, VPC, KMS       |
| **Output to S3**         | Query results are stored in a specified S3 bucket |

---

### 📦 Supported Formats

* CSV, JSON, ORC, Parquet, Avro, and more.

---

### 💰 Pricing

* Charged **per TB of data scanned**.
* **Optimize cost**: Convert to **columnar formats** (Parquet, ORC) and **partition your data**.

---

### 📌 Common Use Cases

* Ad-hoc data analysis
* Analyzing logs (e.g., CloudTrail, ALB logs)
* Quick exploration of S3-stored data
* Business intelligence reporting
* ETL pre-processing or validation

---

### 🧠 Optimization Tips

| Tip                      | Benefit                           |
| ------------------------ | --------------------------------- |
| **Partition data**       | Reduces scanned data, lowers cost |
| **Use columnar formats** | Parquet/ORC → faster + cheaper    |
| **Use Glue Catalog**     | Simplifies schema management      |

---

### 🆚 When to Use Athena vs. Redshift

| Scenario                     | Use                                  |
| ---------------------------- | ------------------------------------ |
| Ad-hoc SQL queries on S3     | **Athena**                           |
| Complex analytics, joins, BI | **Redshift (OLAP)**                  |
| Real-time dashboards on logs | **Athena + QuickSight / OpenSearch** |

---

Let me know if you'd like Athena use case examples, Glue integration, or comparison with Redshift Spectrum.
