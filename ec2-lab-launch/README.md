# 🧠 AWS Lab — Understanding JSON Policy Elements & Launching an EC2 Instance (Console)

> **Date:** 2025-11-10  
> **Objective:** Learn the **elements of an IAM JSON policy** and understand how to **launch an EC2 instance** using the AWS Management Console.

---

## 📚 Table of Contents
1. [Overview](#overview)
2. [Part 1 — IAM JSON Policy Elements](#part-1--iam-json-policy-elements)
   - [Structure of a JSON Policy](#structure-of-a-json-policy)
   - [Key Elements Explained](#key-elements-explained)
   - [Example Policy](#example-policy)
3. [Part 2 — Launching an EC2 Instance (Console)](#part-2--launching-an-ec2-instance-console)
   - [Steps to Launch an Instance](#steps-to-launch-an-instance)
   - [Verifying the Instance](#verifying-the-instance)
   - [Terminating the Instance](#terminating-the-instance)
4. [Key Takeaways](#key-takeaways)
5. [References](#references)

---

## 🧩 Overview

In this class, you learned two core AWS skills:
1. **How IAM policies work** — specifically, the internal structure and meaning of elements within a JSON policy document.  
2. **How to launch an Amazon EC2 instance** — the process of provisioning a virtual server using the AWS Management Console.

These two topics are foundational for managing permissions and compute resources in AWS.

---

## 🛡️ Part 1 — IAM JSON Policy Elements

### Structure of a JSON Policy

A standard AWS IAM policy is a **JSON-formatted document** that defines *who* can perform *what actions* on *which resources* under *which conditions*.

Basic structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow or Deny",
      "Action": "service:operation",
      "Resource": "resource-arn",
      "Condition": {
        "key": "value"
      }
    }
  ]
}

## 💻 Part 2 — Launching an EC2 Instance (Console)

Amazon EC2 (**Elastic Compute Cloud**) lets you run virtual servers (called **instances**) in the AWS cloud.  
Follow the steps below to launch your first EC2 instance using the **AWS Management Console**.

---

### 🚀 Steps to Launch an Instance

1. **Sign in** to the [AWS Management Console](https://console.aws.amazon.com/).  
2. Navigate to the **EC2** service using the search bar at the top.  
3. Click **Launch Instance**.  
4. Enter an instance name (e.g., `MyFirstEC2Instance`).  
5. Under **Application and OS Images (Amazon Machine Image - AMI):**  
   - Choose **Amazon Linux 2** or **Ubuntu 22.04 LTS**.  
6. Under **Instance Type:**  
   - Select **t2.micro** *(Free Tier eligible)*.  
7. Under **Key pair (login):**  
   - Create a new key pair if you don’t already have one.  
   - Download the `.pem` file securely and keep it safe.  
8. Under **Network settings:**  
   - Choose **Allow SSH traffic from anywhere (0.0.0.0/0)** *(for testing only)*.  
   - For production, **restrict SSH** to your IP address.  
9. Under **Storage:**  
   - Keep default settings (usually an 8 GB gp3 volume).  
10. Click **Launch Instance** to create your instance.

✅ Wait until the **Instance state** changes to **Running**.

---

### 🔍 Verifying the Instance

1. Go to **EC2 → Instances → YourInstanceName**.  
2. Verify the following details:
   - **Instance state:** Running  
   - **Status checks:** 2/2 checks passed  
   - **Public IPv4 address:** Visible (used for SSH access)  
3. Click **Connect** → choose **EC2 Instance Connect** → click **Open Console Shell**.  
4. Run the command below to confirm your instance is active:

   ```bash
   uname -a