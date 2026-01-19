Here are **in-depth notes on AWS Lambda**, fully aligned with Module 19 of Stephane Maarek’s SAA-C03 course, including all core topics, internals, integrations, pricing, and real-world examples.

---

# 🧠 AWS Lambda – In-Depth Notes (Complete for SAA-C03)

---

## ✅ What is AWS Lambda?

AWS Lambda is a **serverless compute service** that lets you run code **without provisioning or managing servers**.

You simply upload your code, and Lambda **executes it only when needed** — you **pay per request** and **duration**, not uptime.

---

## ⚙️ Lambda Core Concepts

| Feature                   | Details                                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Runtime**               | Supports many languages: Node.js, Python, Java, Go, Ruby, .NET, etc.                                   |
| **Trigger-based**         | Invoked by events: API Gateway, S3, DynamoDB, SNS, SQS, etc.                                           |
| **Stateless**             | Each invocation is isolated. No persistent storage between calls.                                      |
| **Fully managed**         | AWS handles provisioning, scaling, fault tolerance, OS patching.                                       |
| **Automatic scaling**     | Scales up automatically based on traffic. No warm-up needed after first time (except for cold starts). |
| **Short-lived execution** | Max execution time: **15 minutes**. Designed for short compute tasks.                                  |

---

## 📦 Lambda Deployment Models

### 1. **Direct upload**

* Upload .zip file via AWS Console or CLI.


### 2. Lambda Layers

**Reusable dependencies (e.g., libraries, runtimes, configs)** shared across multiple Lambda functions.
Mounted at **`/opt/`** during the **init (cold start)** phase.
Stored in AWS (region/account scoped) — not visible like S3.
Each function can use **up to 5 layers** (max 250 MB unzipped per layer).
Used to avoid duplicating code across functions.


### 3. **Container Image Support**

* Build Lambda functions as container images (max size 10 GB).
* Supports Docker images with custom runtimes or complex dependencies.

---

## **Lambda SnapStart** - Performance optimization:

- **10x faster cold starts** at no extra cost
- **Pre-initializes** functions and captures snapshots of memory/disk state
- **Skips initialization phase** during invocation → direct to invoke phase
- **Triggered** when publishing new function versions
- **Reduces latency** especially beneficial for Java workloads with heavy initialization

*(Traditional flow: Initialize → Invoke → Shutdown. With SnapStart: Snapshot → Invoke → Shutdown)*

---

## ⚙️ Lambda Triggers

| Trigger Source                      | What it does                                                       |
| ----------------------------------- | ------------------------------------------------------------------ |
| **API Gateway**                     | Invoke Lambda on HTTP request — build serverless APIs              |
| **S3**                              | Trigger on object create/delete (e.g., image resize pipeline)      |
| **DynamoDB Streams**                | Trigger Lambda for real-time DB change handling                    |
| **SNS**                             | Process incoming notifications                                     |
| **SQS**                             | Poll messages and process them in Lambda (fan-out)                 |
| **CloudWatch Events / EventBridge** | Schedule jobs like CRON or respond to system events                |
| **Cognito**                         | Invoke during sign-up/sign-in (e.g., custom auth logic)            |
| **ALB**                             | Lambda can now be a target for **Application Load Balancer** (ALB) |

---

## 🔄 Lambda Execution Model (Internals)

### Cold Start vs Warm Start

| Type           | Explanation                                                                                                      |
| -------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Cold Start** | First invocation spins up a new container: takes longer (\~100ms–1s). Happens after idle time or version change. |
| **Warm Start** | If the same Lambda is invoked again soon, the container is reused — faster response (\~ms).                      |

Lambda functions run in **Firecracker microVMs** for security and isolation.

---

## ⚙️ Environment Variables & Config

* You can set key-value pairs in Lambda.
* Use `AWS_SECRETS_MANAGER` or `AWS_SSM` for secure config data.

---

## 🔐 IAM & Lambda

* Lambda needs **IAM Execution Role** to:

  * Access AWS services (e.g., S3, DynamoDB)
  * Write logs to CloudWatch
* Use **Least Privilege** for security best practice.
* Example:

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject"],
  "Resource": ["arn:aws:s3:::my-bucket/*"]
}
```

---

## 📊 Monitoring and Logging

| Tool                   | Purpose                                                   |
| ---------------------- | --------------------------------------------------------- |
| **CloudWatch Logs**    | Automatic logging of stdout/stderr                        |
| **CloudWatch Metrics** | Tracks invocation count, duration, errors, throttles      |
| **X-Ray**              | Distributed tracing — visualize flow across microservices |

---

## ⏱️ Concurrency and Throttling

| Term                        | Description                                                            |
| --------------------------- | ---------------------------------------------------------------------- |
| **Default Concurrency**     | Each region has a quota (e.g., 1000 concurrent executions per account).            |
| **Reserved Concurrency**    | Limit a function’s concurrency to reserve capacity(limits maximum concurrency).                    |
| **Provisioned Concurrency** | Keep N warm instances ready to avoid cold starts (costs extra).        |
| **Burst Concurrency**       | Initial burst of 500 (varies per region), then adds 500/s until limit. |

---

## 🔄 Asynchronous vs Synchronous Invocation

| Type                               | Example              | Behavior                                   |
| ---------------------------------- | -------------------- | ------------------------------------------ |
| **Synchronous**                    | API Gateway → Lambda | Client waits for response                  |
| **Asynchronous**                   | S3/SNS → Lambda      | Lambda queues request, returns immediately |
| **Event Source Mapping (polling)** | SQS/DynamoDB Streams | Lambda pulls messages and processes them   |

---

## 🧱 Lambda Destinations (Asynchronous Flow Handling)

Lambda can route results to a destination after async execution:

| Outcome     | Destination                   |
| ----------- | ----------------------------- |
| **Success** | SNS, SQS, Lambda, EventBridge |
| **Failure** | SNS, SQS, Lambda, EventBridge |

This improves **resilience**, replacing DLQs in modern use cases.

---

## 🛠️ Error Handling & Retries

| Invocation                | Retry?                     | DLQ Support? |
| ------------------------- | -------------------------- | ------------ |
| Sync                      | No                         | No           |
| Async                     | Yes (2 retries by default) | Yes          |
| Poll-based (SQS/DynamoDB) | Configurable               | Yes          |

Use **dead-letter queues** (DLQ) for catching failed events.

---

## 💰 Lambda Pricing (As of 2025)

### Charges based on:

* **Requests**: First 1M/month free, then \$0.20 per million
* **Duration**: Charged in **ms** of execution
* **Memory size**: You choose from 128MB to 10GB
* **Provisioned Concurrency**: Extra charge

> 🧮 Cost = (Number of requests × request charge) + (Duration in GB-seconds × compute charge)

---

## 🧩 Lambda + VPC Integration

By default, Lambda **runs inside AWS owned VPC**. But if it needs to access:

* RDS in private subnet
* Internal APIs
  → You must **attach VPC** + subnets + security groups.

⚠️ Remember:

* If placed inside VPC, configure **NAT Gateway** to let Lambda reach the internet.
* VPC-enabled Lambda now uses **Hyperplane ENIs** (no cold ENI issues like before).

---

## 🧪 Real-World Use Cases

| Use Case                   | Lambda Role                                    |
| -------------------------- | ---------------------------------------------- |
| Image thumbnail generation | Triggered by S3 upload                         |
| REST APIs                  | Backend for API Gateway                        |
| Serverless CRON jobs       | Triggered by EventBridge (CloudWatch Events)   |
| Real-time file processing  | Event from S3/DynamoDB                         |
| Auth flows                 | Custom login/signup using Cognito triggers     |
| Stream processing          | Read messages from Kinesis or DynamoDB Streams |
| Event fan-out              | SNS → Lambda → multiple destinations           |

---

## 🆚 Lambda vs Other Compute

| Service     | When to Use                                                  |
| ----------- | ------------------------------------------------------------ |
| **Lambda**  | Short-lived, event-driven tasks, no infrastructure to manage |
| **Fargate** | Containerized workloads, background jobs, custom runtimes    |
| **EC2**     | Full OS-level control, long-running processes, legacy apps   |

---

## 📌 Best Practices

* Use **short execution time** (under 5s) for responsiveness
* **Separate logic into multiple functions** (microservices style)
* Use **Environment Variables** for config
* **Monitor metrics** for throttling, duration spikes
* Use **provisioned concurrency** for critical low-latency apps
* **Don’t store state in Lambda** — use S3, DynamoDB, or ElastiCache instead

---

Let me know if you want:

* 📁 Folder structure and example code for a Lambda function project
* 💡 Practice scenarios or sample SAA questions from Module 19
* 🧪 A comparison between Lambda and Fargate or Step Functions

Would you like me to summarize this into flashcards or cheatsheet style as well?
