<!-- Here’s the **Encrypted AMI Sharing Process** explained in a **clear, visual, real-world style** for deep understanding (and exam prep):

---

## 🔐 Encrypted AMI Sharing – Step-by-Step Flow

### 🎯 Goal:

You want to **share an AMI from Account A to Account B**, but it's **encrypted using a KMS key in Account A**.

---

### 🧭 High-Level Steps

| Step | Action                                           | Who Does It          | Why                                                |
| ---- | ------------------------------------------------ | -------------------- | -------------------------------------------------- |
| 1️⃣  | **Modify AMI launch permissions**                | ✅ Account A          | Allows Account B to see & launch the AMI           |
| 2️⃣  | **Share the KMS key (used to encrypt AMI)**      | ✅ Account A          | Account B needs access to decrypt the EBS snapshot |
| 3️⃣  | **Grant IAM permissions in Account B**           | ✅ Account B          | To actually launch EC2 using that AMI              |
| 4️⃣  | ✅ **Optionally re-encrypt** with its own KMS key | (Optional) Account B | For key control and regional compliance            |

---

## 🔧 Detailed Breakdown

### ✅ Step 1: Share the AMI (Launch Permission)

> 📍 Command/API: `ModifyImageAttribute`

```bash
aws ec2 modify-image-attribute \
  --image-id ami-1234567890abcdef0 \
  --launch-permission "Add=[{UserId=222222222222}]"
```

* This makes the **AMI visible** to Account B.
* But it's still **encrypted with Account A's KMS key** — so Account B can't launch it yet.

---

### ✅ Step 2: Share the KMS Key (Key Policy in Account A)

Update the **KMS key policy** in Account A to grant usage to Account B:

```json
{
  "Sid": "AllowAccountBUseOfKey",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::222222222222:root"
  },
  "Action": [
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:CreateGrant",
    "kms:DescribeKey"
  ],
  "Resource": "*"
}
```

> 🔐 This allows Account B to decrypt the AMI’s underlying snapshot.

---

### ✅ Step 3: Account A's IAM Role assumed by Account B

Make sure Account B assumes the **IAM role** of Account A with permissions like:

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeImages"
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:DescribeKey"
  ],
  "Resource": "*"
}
```

> ✅ Needed to:

* **Decrypt** volumes during boot time using KMS

---

## 📌 Summary Table

| What to Share                | Who Does It | Why                             |
| ---------------------------- | ----------- | ------------------------------- |
| AMI Launch Permission        | Account A   | So Account B can see and launch |
| KMS Key Access (Policy)      | Account A   | So B can decrypt the AMI        |
| IAM Role of Account A with KMS perms | Account B   | So EC2 can be launched          |
| (Optional) Re-encryption     | Account B   | Full ownership of encryption    |

---

## 🧠 Final Tip for Exam

> Sharing an **unencrypted AMI** needs only AMI access.
> But sharing an **encrypted AMI** **always** needs:

* 🔓 AMI access
* 🔐 KMS key access -->


Perfect — here are **clean, redefined, exam-ready notes** with **zero ambiguity**.
You can revise this in **2 minutes before AWS SAA** and you’ll be safe.

---

# 🔐 Encrypted AMI Cross-Account Sharing — **Redefined Notes**

---

## 🎯 Goal

Launch an **encrypted AMI owned by Account A** from **Account B**.

---

## 🧠 Core Rule (MEMORIZE)

> **Encrypted AMI cross-account access requires permission on BOTH sides:**
>
> 🔹 **Resource policies in Account A**
> 🔹 **IAM policies in Account B**

IAM alone is **never sufficient**.

---

## 🏗️ Responsibility Split (VERY IMPORTANT)

### 🅰️ Account A — *Resource Owner*

Controls **what can be shared**

### 🅱️ Account B — *Resource Consumer*

Controls **what actions can be performed**

---

## 🅰️ Account A — REQUIRED Resource Policies

### 1️⃣ AMI Launch Permission (EC2 Resource Policy)

Allows Account B to **see and launch the AMI**.

* API: `ModifyImageAttribute`
* Without this → AMI is **invisible**

---

### 2️⃣ KMS Key Policy (KMS Resource Policy)

Allows Account B to **decrypt the encrypted EBS snapshot**.

Minimum required actions:

* `kms:Decrypt`
* `kms:CreateGrant`
* `kms:DescribeKey`

Without this → AMI visible, **launch fails**

---

## 🅱️ Account B — REQUIRED IAM Policies

### 3️⃣ EC2 Permissions (Identity Policy)

Allows Account B to **launch EC2 instances**.

Required actions:

* `ec2:RunInstances`
* `ec2:DescribeImages`
* `ec2:CreateVolume`
* `ec2:AttachVolume`

---

### 4️⃣ KMS Permissions (Identity Policy)

Allows Account B to **use the shared KMS key**.

Required actions:

* `kms:Decrypt`
* `kms:CreateGrant`

⚠️ These permissions work **only if** Account A’s KMS key policy allows them.

---

## 🚫 NOT Required (EXAM TRAPS)

| Item                              | Required?           |
| --------------------------------- | ------------------- |
| Cross-account IAM role assumption | ❌ No                |
| IAM policy in Account A           | ❌ No                |
| Manual KMS grants                 | ❌ No (auto-created) |
| STS AssumeRole                    | ❌ No                |

---

## 🔄 Optional (Best Practice)

### 🔁 Copy & Re-Encrypt AMI (Account B)

* Account B copies the AMI
* Re-encrypts using **its own KMS key**
* Full encryption ownership
* Common **best-practice exam answer**

---

## ❌ Common Failure Scenarios

| Missing Piece         | Result          |
| --------------------- | --------------- |
| AMI launch permission | AMI not visible |
| KMS key policy        | Launch fails    |
| EC2 IAM permission    | Access denied   |
| KMS IAM permission    | Access denied   |

---

## 🧠 One-Line Exam Answer

> **Cross-account encrypted AMI access requires AMI launch permission and KMS key policy in the owner account, plus EC2 and KMS IAM permissions in the consumer account.**

---