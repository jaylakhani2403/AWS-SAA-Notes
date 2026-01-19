# AWS Database Services - Detailed Quick Reference

## **RDS (Relational Database Service)**
**✅ Managed Relational SQL Databases**
- **Engines Supported**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- **Core Features**:
  - Multi-AZ deployment for high availability (sync replication)
  - Automated backups with PITR (Point-in-Time Recovery) up to 35 days
  - Read replicas (async replication) for scaling read operations
  - Automatic minor version patching
  - Storage auto-scaling (except SQL Server)
  - IAM database authentication
  - Performance Insights dashboard
  - RDS Proxy for connection pooling
- **Use Cases**:
  - Traditional web applications
  - E-commerce platforms
  - Business applications (ERP, CRM)
  - Legacy system migrations
- **Best For**: When you need managed traditional SQL databases with minimal operational overhead

## **Aurora**
**✅ High-performance MySQL/PostgreSQL compatible**
- **Performance**: 5x faster than MySQL, 3x faster than PostgreSQL
- **Architecture**:
  - Shared storage architecture (6 copies across 3 AZs)
  - Up to 15 read replicas with low latency
  - Multi-master writes (limited regions)
- **Core Features**:
  - Aurora Global Database (cross-region replication, <1 sec latency)
  - Backtrack - rewind DB cluster to specific time
  - Serverless option (auto-scales based on load)
  - Aurora Machine Learning (in-database ML)
  - Parallel Query for analytics workloads
- **Use Cases**:
  - High-traffic web applications
  - SaaS platforms
  - Gaming leaderboards
  - Real-time bidding systems
- **Best For**: Enterprise applications needing high performance and availability

## **ElastiCache**
**✅ Managed In-memory Caching**
- **Two Engines**: Redis & Memcached
- **Redis Features**:
  - Multi-AZ with auto-failover
  - Read replicas
  - Backup and restore
  - Redis AUTH for security
  - Data persistence options
- **Memcached Features**:
  - Multi-node for partitioning data
  - Auto-discovery of nodes
  - Simple key-value model
- **Use Cases**:
  - Session store (Redis)
  - Real-time analytics dashboard
  - Gaming leaderboards
  - Database query caching
  - Message broker (Redis Pub/Sub)
- **Performance**: Sub-millisecond latency
- **Best For**: Reducing database load, speeding up applications

## **DynamoDB**
**✅ Managed NoSQL Database (Key-Value & Document)**
- **Core Features**:
  - Serverless, auto-scaling
  - Single-digit millisecond latency at any scale
  - DynamoDB Accelerator (DAX) for microsecond reads
  - Global Tables for multi-region apps
  - Point-in-Time Recovery up to 35 days
  - On-demand backup (no performance impact)
  - Transactions support (ACID compliance)
- **Data Model**:
  - Partition key (required)
  - Sort key (optional)
  - GSI (Global Secondary Indexes) - up to 20
  - LSI (Local Secondary Indexes) - up to 5
- **Use Cases**:
  - Serverless applications
  - Mobile backends
  - Gaming platforms
  - Ad tech real-time bidding
  - Shopping carts
- **Best For**: Applications with unpredictable workloads needing seamless scaling

## **S3 (Simple Storage Service)**
**✅ Object Storage (Not a traditional database but used for data)**
- **Storage Classes**:
  - Standard (frequent access)
  - Standard-IA (infrequent)
  - One Zone-IA
  - Intelligent Tiering
  - Glacier (archive)
  - Glacier Deep Archive
- **Core Features**:
  - 99.999999999% durability (11 9's)
  - Lifecycle policies for automatic tiering
  - Versioning
  - Cross-region replication
  - S3 Select (retrieve subset of data)
  - Glacier Select
  - Batch operations
- **Use Cases**:
  - Data lakes and analytics
  - Backup and archive
  - Static website hosting
  - Big data storage
  - Disaster recovery
- **Best For**: Storing massive amounts of unstructured data

## **DocumentDB**
**✅ MongoDB-Compatible Document Database**
- **Compatibility**: MongoDB 3.6, 4.0 API
- **Core Features**:
  - Storage auto-scales up to 64TB
  - 15 read replicas with low latency
  - Automated backups
  - Encryption at rest and in transit
  - Fault-tolerant, self-healing storage
- **Architecture**:
  - Distributed, durable storage
  - 6 copies across 3 AZs
  - SSD-backed
- **Use Cases**:
  - Content management systems
  - Catalogs and user profiles
  - Real-time analytics
- **Best For**: MongoDB workloads needing AWS integration

## **Neptune**
**✅ Fully Managed Graph Database**
- **Graph Models**: Property Graph & RDF
- **Query Languages**: Gremlin, SPARQL, openCypher
- **Core Features**:
  - Can store billions of relations
  - Traverses 100,000s queries/sec
  - Replication across 3 AZs
  - Point-in-time recovery
  - Encryption at rest
- **Performance**:
  - Millisecond latency for graph queries
  - Optimized for complex joins
- **Use Cases**:
  - Fraud detection
  - Social networks
  - Recommendation engines
  - Knowledge graphs
  - Network security
- **Best For**: Applications with highly connected data

## **Keyspaces (for Apache Cassandra)**
**✅ Managed Apache Cassandra Service**
- **Core Features**:
  - Serverless (auto-scales)
  - Apache Cassandra 3.11 CQL API compatible
  - Replication across 3 AZs
  - Encryption at rest by default
  - Point-in-time recovery (35 days)
- **Performance**:
  - Single-digit millisecond latency
  - 1000s of requests per second
- **Use Cases**:
  - IoT applications
  - Time series data
  - Industrial telemetry
  - Retail applications
- **Best For**: Cassandra workloads with serverless preference

## **Timestream**
**✅ Time Series Database**
- **Core Features**:
  - Serverless (auto-scales)
  - Built-in time series analytics
  - SQL query support
  - Data tiering (memory vs magnetic)
  - Encryption at rest and in transit
- **Performance**:
  - 1000x faster than relational databases for time series
  - 1/10th the cost of relational databases
- **Use Cases**:
  - IoT applications
  - DevOps monitoring
  - Industrial telemetry
  - Application metrics
- **Best For**: Time-stamped data with analytics needs

---

## **Quick Decision Matrix**
| **When you need...** | **Choose...** |
|---------------------|--------------|
| Traditional SQL with HA | RDS Multi-AZ |
| Highest performance SQL | Aurora |
| In-memory caching | ElastiCache |
| Predictable schema | RDS |
| Unpredictable schema | DynamoDB |
| Key-value with massive scale | DynamoDB |
| MongoDB compatibility | DocumentDB |
| Graph relationships | Neptune |
| Cassandra compatibility | Keyspaces |
| Time series data | Timestream |
| Massive object storage | S3 |
| Serverless SQL | Aurora Serverless |
| Cross-region replication | Aurora Global or DynamoDB Global Tables |
| Sub-millisecond reads | ElastiCache or DynamoDB with DAX |

## **Key Differences**
- **RDS vs Aurora**: Aurora offers better performance, more replicas, global databases
- **DynamoDB vs MongoDB**: DynamoDB is key-value/document, MongoDB has richer querying
- **Redis vs Memcached**: Redis has more features (persistence, data types), Memcached is simpler
- **S3 vs Databases**: S3 for storage, databases for querying and transactions