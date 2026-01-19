Here are **in-depth notes on Amazon DynamoDB**, covering **all core concepts**, **internal architecture**, and specifically everything **Stephane Maarek teaches in Module 19** of the AWS Solutions Architect Associate (SAA-C03) course:

---

## 🧠 **Amazon DynamoDB – In-Depth Notes**

---

### 📌 What is DynamoDB?

* **Fully managed NoSQL database** service offered by AWS.
* Designed for **single-digit millisecond** performance at any scale.
* Supports **key-value** and **document-based** models.
* **Serverless**: No infrastructure to manage.
* Used for **real-time apps**, **IoT**, **gaming**, **mobile apps**, **e-commerce carts**, etc.

---

### ⚙️ Core Concepts

#### 1. **Tables**

* Collection of **items** (like rows in RDBMS).
* Created with:

  * **Primary Key**
  * **Read/Write capacity mode** (On-demand or provisioned)

#### 2. **Items**

* Analogous to rows.
* Each item is uniquely identified by a **primary key**.
* Stored as **JSON-like structure**.

#### 3. **Attributes**

* Analogous to columns.
* Flexible: Items in the same table can have **different attributes** (No schema enforcement).

---

### 🔑 Primary Keys (VERY IMPORTANT)

* All items must have a **unique Primary Key (Partition Key + Sort Key(Optional))**.

#### ➤ **Partition Key (PK)** (a.k.a. Hash Key)

* Uniquely identifies an item.
* DynamoDB uses it to **determine the partition** (internal storage location).
* All items must have a **unique Partition Key** as there is no Sort key.

#### ➤ **Partition Key + Sort Key** (a.k.a. Composite Key)

* Enables **storing multiple related items** with the same PK (with unique Sort keys per PK).
* Sort Key allows **querying based on range/sorting**, e.g., timestamps.

🧠 *Use Cases:*

* PK only: Simple lookups (e.g., Users table).
* PK + SK: Grouped items (e.g., Orders per Customer).

---

### 🧭 Data Access Patterns

* 🔍 **GetItem**: Fast lookup by primary key.
* 🔍 **Query**: Retrieve items using PK and optional SK conditions.
* 🔍 **Scan**: Goes through **entire table** – costly & slow. Use with caution.

---

### ⚡️ Performance Internals

#### 🔸 **Partitions**

* Internal storage units.
* DynamoDB automatically **splits and rebalances partitions** as data grows.
* Partition key determines where data lives (hashing).

#### 🔸 **Hot Partitions**

* Occurs when many requests target the same PK → **bottleneck**.
* Avoid by:

  * Using **good PK distribution**
  * Adding **random suffixes/prefixes**
  * Using **write sharding**

---

### 🔄 Read/Write Capacity Modes

#### 1. **Provisioned Mode**

* You specify:

  * `RCU` (Read Capacity Units)
  * `WCU` (Write Capacity Units)

* Auto scaling also available for RCU and WCU(optional).
* Cost-effective if you know traffic pattern.

#### 2. **On-Demand Mode**

* Auto-scales based on usage.
* No RCU and WCU, cost per read/write.
* No capacity planning.
* Slightly **more expensive**, ideal for unpredictable workloads.

---


### 📏 **DynamoDB Throughput Internals**

#### ✅ **RCU (Read Capacity Unit)**

* **1 RCU** =
  🔸 **1 strongly consistent read/sec** of item **up to 4 KB**
  🔸 **OR 2 eventually consistent reads/sec** of item **up to 4 KB**

➡️ If item > 4 KB, divide size by 4 and **round up** to get RCUs needed.
eg. 400kb item needs 100 rcu to read strongly consistent.

---

#### ✅ **WCU (Write Capacity Unit)**

* **1 WCU** =
  🔸 **1 write/sec** of item **up to 1 KB**

➡️ If item > 1 KB, divide size by 1 and **round up** to get WCUs needed.
eg. 400kb item needs 400 wcu to write.


---

## **Indexes**

### **1. Local Secondary Index (LSI)**

### **Core Concept**
Think of LSI as creating **multiple sorted views** of the same data within each partition.

### **How It Works Under the Hood**
```javascript
// Base Table: User orders sorted by date
Partition [User#123]:
PK=User#123, SK=2024-01-01, Amount=100
PK=User#123, SK=2024-01-02, Amount=250
PK=User#123, SK=2024-01-03, Amount=150

// LSI-Amount: Same data, sorted differently
PK=User#123, LSI-SK=100  → points to SK=2024-01-01
PK=User#123, LSI-SK=150  → points to SK=2024-01-03  
PK=User#123, LSI-SK=250  → points to SK=2024-01-02
```

### **Storage Architecture**
- **Collocated**: LSI entries live **within the same physical partition** as base data
- **References**: Contains pointers to base items, not full copies (unless projecting ALL)
- **Synchronous Updates**: Write to base table and LSI happens atomically

---

### **2. Global Secondary Index (GSI)**

### **Core Concept**
GSI is essentially **a completely separate table** that DynamoDB maintains automatically, with its own partition structure.

### **How It Works Under the Hood**
```javascript
// Base Table: User-centric view
[Partition A] PK=User#123, SK=ORDER#001, Status=SHIPPED
[Partition B] PK=User#456, SK=ORDER#002, Status=PENDING

// GSI-Status: Status-centric view (separate physical table)
[Partition X] PK=SHIPPED, GSI-SK=ORDER#001, user_id=123
[Partition Y] PK=PENDING, GSI-SK=ORDER#002, user_id=456
```

### **Storage Architecture**
- **Separate Partitions**: GSI lives in **completely different physical storage**
- **Asynchronous Replication**: Changes propagate async (typically < 1s)
- **Independent Scaling**: Can provision different throughput than base table

---

### **Key Distinctions in Practice**

#### **When to Use Which:**
- **LSI**: "Show me this user's data in different sort orders"
- **GSI**: "Show me all data across users filtered/sorted differently"

---

🧠 *Index Caveats:*

* Indexes consume **RCU/WCU**
* Indexes can **lag behind base table**

---

### 🔐 Security

* **Encryption at rest** with KMS (enabled by default)
* **IAM policies** for access control
* **Fine-grained access control** using IAM and `Condition` keys (e.g., limit access by `userId` attribute)

---

## **DynamoDB Stream Processing**

create, update, delete stream. can be used directly or via kinesis data streams (routed to).

### **Two Options**

#### **DynamoDB Streams**
- 24-hour retention
- Limited consumers (2-3)
- Native Lambda triggers
- **Best for**: Simple event reactions

#### **Kinesis Data Streams**
- Up to 1-year retention
- Unlimited consumers
- Full Kinesis ecosystem
- **Best for**: Complex pipelines

### **How It Works**
```
DynamoDB Table → Stream → Process → Destinations
   (CUD ops)     ↓
           Real-time processing
```

### **Architecture Paths**

#### **DynamoDB Streams:**
- → Lambda (automatic)
- → EC2/KCL (custom)

**Use**: Welcome emails, replication, simple transforms

#### **Kinesis Data Streams:**
- → Kinesis Analytics/Firehose
- → S3/Redshift/OpenSearch
- → Multiple Lambdas/EC2

**Use**: Analytics pipelines, data lakes, multi-consumer systems

### **Quick Comparison**
- **Retention**: 24h vs 1 year
- **Scalability**: Limited vs Unlimited consumers
- **Integration**: Lambda-native vs Manual setup
- **Complexity**: Simple vs Enterprise pipelines

**Choose DynamoDB Streams** for Lambda-driven workflows.  
**Choose Kinesis** for complex, multi-destination processing.

---

### ☁️ Global Tables (Multi-Region)

* **Active-active replication** across AWS regions (two way replication)
* Enable DynamoDB Streams first.
* Built on **Streams + Replication**
* Supports **multi-region writes and reads**
* Conflict resolution: **Last writer wins**

🧠 *Use Case:* Low-latency multi-region apps with automatic replication.

---

### 📦 Backup & Restore

* **On-Demand Backup** (manual) and **Point-in-time recovery (PITR)**
* PITR allows restoring to **any second in the last 35 days**
* Backup/restore does **not impact performance**

---

### 🧪 Transactions

* **ACID transactions** supported for:

  * Multiple Put/Update/Delete/Get operations
  * Across **multiple items**
* Uses `TransactWriteItems` and `TransactGetItems`
* **Up to 25 items**, max 4MB
* Ideal for **banking apps**, **leaderboards**, etc.

---

### ⏳ TTL (Time to Live)

* Automatically expire and delete items based on a TTL attribute (timestamp)
* Good for **temporary sessions**, **caches**, etc.

---

### 🔍 PartiQL Support

* SQL-like syntax to query DynamoDB:

```sql
SELECT * FROM Users WHERE userId = '123';
```

* Makes migration/testing easier
* Internally uses same API as normal operations

---

## **DynamoDB Accelerator (DAX)**

### **What is DAX?**
- **In-memory cache** for DynamoDB - reduces read load by **10x**
- **Microsecond latency** for cached reads (vs DynamoDB's millisecond)
- **Fully managed** - no application changes needed

### **Key Features**
```python
# Same API as DynamoDB - just change endpoint
dax_client = AmazonDaxClient(endpoint='dax-cluster.amazonaws.com')

# Automatic cache warming
# Reads populate cache automatically

# Write-through cache
# Writes go to both DAX and DynamoDB
```

### **When to Use DAX**
✅ **Read-heavy workloads** (e.g., user profiles, product catalogs)  
✅ **Repeat queries** with same parameters  
✅ **Microsecond latency** requirements  
✅ **Reduce DynamoDB read costs** (cache hits don't consume RCU)

### **Limitations**
❌ **Cache size limited** by node type (up to 237GB per cluster)  
❌ **Not for write-heavy** workloads  
❌ **Default TTL**: 5 minutes (configurable)  
❌ **Additional cost** on top of DynamoDB

### **DAX vs ElastiCache**
- **DAX**: Optimized for DynamoDB objects/queries
- **ElastiCache**: General-purpose cache (Redis/Memcached)
- **Use both**: DAX for single objects + ElastiCache for computed aggregations

### **Architecture**
```
App → DAX Cluster → DynamoDB
      ↓ Cache Hit   ↓ Cache Miss
    μs latency     ms latency
```

**TLDR**: DAX = In-memory read cache for DynamoDB, 10x faster reads, no code changes.

---

## 🚨 DynamoDB Best Practices (as taught by Stephane)

* Always prefer **Query** over **Scan**
* Design tables based on **access patterns**, not relational modeling
* Avoid **hot partitions**
* Use **On-Demand** for spiky or unpredictable workloads
* Use **GSIs** for alternative access patterns
* Use **Streams** for event-driven use cases
* Enable **PITR** for mission-critical tables
* Use **Transactions** only when needed (adds latency)

---

## 🧮 Real-World Example

#### **App: EduConnect Messaging Table (PK + SK)**

| Partition Key (PK) | Sort Key (SK)          | Message              |
| ------------------ | ---------------------- | -------------------- |
| `chat#123`         | `timestamp#2025-07-22` | “Hello there!”       |
| `chat#123`         | `timestamp#2025-07-23` | “How are you?”       |
| `chat#456`         | `timestamp#2025-07-22` | “Let's meet at 5 PM” |

* Retrieve all messages in a chat:

```js
Query where PK = 'chat#123' ORDER BY SK
```

---

## 🧠 Recap Table

| Feature          | Type / Purpose                     |
| ---------------- | ---------------------------------- |
| PK Only          | Unique lookup                      |
| PK + SK          | Grouped + Range query              |
| LSI              | Alt SK on same PK                  |
| GSI              | Totally different PK/SK            |
| Streams          | Trigger Lambdas on changes         |
| TTL              | Auto delete after time             |
| PITR             | Restore any second in past 35 days |
| Transactions     | ACID on multiple items             |
| On-Demand Mode   | Auto scale, easy but costly        |
| Provisioned Mode | Manual scale, cost effective       |
| Global Tables    | Multi-region active-active sync    |


---

Would you like visual diagrams for partitions, indexes, or streams next?
