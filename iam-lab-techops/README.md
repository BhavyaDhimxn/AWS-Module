# IAM Lab — TechOps Ltd (Console-only)

> **Goal:** Using the **AWS Management Console only** (no CLI), create an IAM configuration for TechOps Ltd that gives appropriate permissions to three teams: **Developers**, **Auditors**, and **Interns**. This README explains IAM fundamentals, the lab problem, design decisions, and a **step-by-step console-only solution** you can follow and copy into your repository.

---

## Table of contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [IAM fundamentals (high-level)](#iam-fundamentals-high-level)
4. [Problem statement](#problem-statement)
5. [Design & approach](#design--approach)
6. [Step-by-step solution — Console ONLY](#step-by-step-solution--console-only)

   * Create groups
   * Create policies (visual editor / JSON)
   * Attach policies to groups
   * Create users and add users to groups
   * Test permissions in the Console
   * Cleanup
7. [Policy JSON examples (for Console policy editor)](#policy-json-examples-for-console-policy-editor)
8. [Testing & verification (Console methods)](#testing--verification-console-methods)
9. [Security best practices and notes](#security-best-practices-and-notes)
10. [Next steps / Extensions](#next-steps--extensions)

---

## Overview

This lab shows how to implement Identity and Access Management (IAM) configurations in the AWS Console without using the CLI. It focuses on:

* Creating **IAM groups** for team-based permission management.
* Creating **customer-managed policies** using the Console policy editor (visual or JSON mode).
* Assigning users to groups and verifying access via the Console.

The setup follows least-privilege principles and is designed for a development/test environment — tighten scopes before applying in production.

---

## Prerequisites

* An AWS account with **Administrator** or equivalent IAM permissions to manage IAM resources.
* Access to the **AWS Management Console** ([https://console.aws.amazon.com/iam](https://console.aws.amazon.com/iam)).
* Optional: A list of S3 bucket names to scope S3 permissions (if you want to avoid `*`). Replace `example-bucket` with your bucket name(s) wherever indicated.

---

## IAM fundamentals (high-level)

* **Users:** Represent individual people or services who sign in or use access keys.
* **Groups:** Collections of users. Attach policies to groups to manage many users together.
* **Roles:** Entities assumed by services or users to obtain temporary credentials.
* **Policies:** JSON documents that grant or deny permissions. In the Console you can use a visual policy editor or paste JSON directly.

Important policy elements: `Effect` (Allow/Deny), `Action` (service API operations), `Resource` (ARNs), and `Condition` (optional constraints).

---

## Problem statement

TechOps Ltd has three teams with the following access needs:

* **Developers** — need access to **EC2** and **S3** (create, modify, manage dev resources).
* **Auditors** — need **read-only** access to all AWS services.
* **Interns** — should **only view S3 bucket contents** (list buckets and objects; read-only for objects).

Task: Use the AWS Console to create IAM groups and policies so each team can do precisely what they need and no more.

---

## Design & approach

1. Create three groups: `Developers`, `Auditors`, `Interns`.
2. For `Auditors`, use AWS-managed **ReadOnlyAccess** to simplify setup.
3. Create two customer-managed policies (via Console Policy Editor):

   * `DevelopersPolicy` — allow appropriate EC2 & S3 actions (prefer scoping to dev resources). For simplicity in the lab we will allow EC2 `*` actions but scope S3 to specific buckets.
   * `InternsPolicy` — allow S3 `ListBucket` and `GetObject` on specific buckets only.
4. Attach policies to groups, create users, add users to groups, and test via the Console.

---

## Step-by-step solution — Console ONLY

Follow these steps in the AWS Management Console. Each step includes exact menu navigation and recommended options.

> **Open the IAM Console:** [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/)

### A. Create IAM Groups

1. In the left sidebar, click **Groups** (older consoles) or **User groups** (newer consoles).
2. Click **Create group**.
3. For **Group name**, enter `Developers` and click **Create group** (do not attach policies now — we’ll create and attach custom policies in the next steps).
4. Repeat steps to create `Auditors` and `Interns` groups.

### B. Create Policies (Console policy editor)

We will create two customer-managed policies: `DevelopersPolicy` and `InternsPolicy`.

#### B.1 Create `DevelopersPolicy`

1. In the left sidebar, click **Policies**.
2. Click **Create policy**.
3. Choose **JSON** or **Visual editor**. (JSON is faster if you have policy text; Visual editor can help build permissions step-by-step). Below are two options — **Visual** for guidance and **JSON** to paste.

**Using Visual editor — Developers**

* Service: **EC2** → Actions: check **All EC2 actions** (or select fine-grained actions like `DescribeInstances`, `RunInstances`, `StartInstances`, `StopInstances`, `TerminateInstances` — for a tighter policy select specific actions).
* Add another service: **S3** → Actions: check the actions developers need (e.g., `ListBucket`, `GetObject`, `PutObject`, `DeleteObject`, `GetBucketLocation`, `PutBucketPolicy`)
* Resources for EC2: by default select **All resources** (EC2 resource-level permissions are often `*` for many actions). If you have resource ARNs, limit them.
* Resources for S3: click **Add ARN** and enter the bucket ARN(s) you want to permit, for example:

  * `arn:aws:s3:::example-bucket` (bucket-level actions)
  * `arn:aws:s3:::example-bucket/*` (object-level actions)
* Optional: Add tags or conditions (e.g., allow only from certain IP range).
* Click **Next: Tags** (optional) → **Next: Review**.
* Policy name: `DevelopersPolicy`.
* Description: `Allow EC2 management and S3 access for development resources (Console-only) — scope S3 to specific buckets.`
* Click **Create policy**.

#### B.2 Create `InternsPolicy`

1. Click **Create policy** again.
2. Choose **Visual editor**.

* Service: **S3** → Actions: expand **Read** and select **ListBucket**, **GetObject**. Also add `GetBucketLocation` under **Other actions** if shown.
* Under **Resources** add the specific bucket ARN(s): `arn:aws:s3:::example-bucket` and `arn:aws:s3:::example-bucket/*`.
* Review → Name the policy `InternsPolicy`.
* Description: `Allow list and read access to specific S3 buckets for interns (Console-only).`
* Click **Create policy**.

> Tip: If you want interns to view multiple buckets, add each bucket ARN under Resources.

### C. Attach Policies to Groups

1. Navigate to **User groups**.
2. Click the `Auditors` group → **Permissions** tab → click **Attach policies**.

   * Search for **ReadOnlyAccess** (AWS managed) and check it. Click **Attach policies**.
3. Click the `Developers` group → **Permissions** → **Attach policies** → search for `DevelopersPolicy` and attach it.
4. Click the `Interns` group → **Permissions** → **Attach policies** → search for `InternsPolicy` and attach it.

### D. Create Users and Add to Groups

1. In the IAM Console, click **Users** → **Add users**.
2. **User name:** enter a name (e.g., `alice-dev`, `bob-audit`, `charlie-intern`).
3. **Select AWS access type:** For this lab, enable **AWS Management Console access**. Optionally enable **Programmatic access** if desired (but your instruction is Console-only; so we recommend enabling only console access).
4. For console password, choose **Autogenerated password** or **Custom password**; you can require **User must create a new password at next sign-in**.
5. Click **Next: Permissions**.
6. Choose **Add user to group** and check the group the user belongs to (e.g., `Developers` for `alice-dev`).
7. Continue through **Tags** (optional) → **Review** → **Create user**.
8. Repeat for each user you want to test.

> For service accounts or programmatic-only users, you would normally create access keys — but since this lab is Console-only, we skip that.

### E. Test Permissions (Console flows)

Sign in as each user (open an incognito/private window to avoid session conflicts) to verify allowed and denied actions.

**Developer user (alice-dev)**

* Confirm you can navigate to **EC2** → Launch an instance (or attempt to). If your DevelopersPolicy allowed `ec2:RunInstances`, the console will let you proceed to instance configuration. If some EC2 actions are denied, the console will show an error on action attempt.
* Confirm you can open **S3** → Open `example-bucket` → Upload / Delete objects (if `PutObject`/`DeleteObject` allowed).

**Auditor user (bob-audit)**

* Confirm you can see services across the Console but cannot modify resources. For example, go to **EC2** → Attempt to modify an instance — you should see an **Access denied** message or disabled UI elements for restricted actions.

**Intern user (charlie-intern)**

* Go to **S3** → Attempt to list buckets and open `example-bucket` → Try to view objects (should succeed). Attempt to upload or delete objects — you should receive **Access denied**.

### F. Cleanup (Optional)

If this is a temporary lab, clean up to avoid leaving IAM entities:

1. Remove users from groups via **Users** → select user → **Groups** → **Remove from group**.
2. Delete the users: **Users** → select user → **Delete user**.
3. Detach and delete customer-managed policies: **Policies** → search `DevelopersPolicy` / `InternsPolicy` → **Detach** from any principals if needed → **Delete policy**.
4. Delete groups: **User groups** → select group → **Delete group**.

---

## Policy JSON examples (paste these into the Console policy editor JSON tab if you prefer JSON over the visual editor)

> Replace `example-bucket` with your actual bucket name(s). These JSON snippets are safe to paste into the Console **Create policy → JSON** tab.

### DevelopersPolicy (EC2: all actions; S3: scoped to example-bucket)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2FullAccess",
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    },
    {
      "Sid": "S3DevBucketAccess",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:PutBucketPolicy"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket",
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```

### InternsPolicy (S3 read-only for specified buckets)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucket",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket"
      ]
    },
    {
      "Sid": "GetObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```

> Note: `Auditors` uses the AWS-managed policy **ReadOnlyAccess** (`arn:aws:iam::aws:policy/ReadOnlyAccess`) so no custom JSON is required.

---

## Testing & verification (Console methods)

* **Sign-in simulation:** Create test users and sign into the Console (use private browser sessions). Try allowed and disallowed workflows.
* **IAM Policy Simulator (Console):** In IAM Console, open **Policy Simulator** → choose the user or group and simulate API calls (e.g., `s3:PutObject`, `ec2:RunInstances`) to confirm Allow/Deny results.
* **CloudTrail:** If CloudTrail is enabled, review logs for events and `AccessDenied` results.

---

## Security best practices and notes

* **Least privilege:** Narrow actions and resources where possible. Replace `*` with specific ARNs.
* **Temporary credentials:** Prefer IAM Roles and STS for programmatic access in production.
* **MFA:** Require MFA for privileged accounts.
* **Password policy:** Enforce strong passwords and rotation rules.
* **Audit & logging:** Enable CloudTrail and S3 access logs for monitoring.
* **Use separate accounts/environments:** Isolate production from development.

---

## Next steps / Extensions

* Convert this Console process into **CloudFormation** or **Terraform** for repeatable automation.
* Add permission boundaries and IAM role assumption patterns for cross-account auditing.
* Create separate `Dev` and `Prod` S3 buckets and show cross-account or cross-role access patterns.

---

*End of IAM Lab — TechOps Ltd (Console-only).*
