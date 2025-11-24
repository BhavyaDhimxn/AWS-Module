# S3 Lab — Basic Bucket Setup & Object Upload (Console)

> **Objective:** Practice the core S3 workflows—creating a bucket, organizing data with a folder (prefix), uploading an object, and attaching a minimal read-only bucket policy.

---

## Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Concept Refresher](#concept-refresher)
4. [Step-by-Step Guide](#step-by-step-guide)
5. [Bucket Policy Example](#bucket-policy-example)
6. [Verification & Testing](#verification--testing)
7. [Cleanup & Next Steps](#cleanup--next-steps)

---

## Overview

Amazon S3 (Simple Storage Service) stores data as objects inside buckets. In this lab I used the **AWS Management Console** to:

- Create a globally unique S3 bucket in my preferred Region.
- Add a logical folder (prefix) for organizing objects.
- Upload a sample object via the console.
- Write a bucket policy that allows public `s3:GetObject` access (useful for static web hosting demos).

These fundamentals unlock many AWS workflows—static hosting, backups, data lakes, and more.

---

## Prerequisites

- AWS account with permissions to manage S3.
- Chosen AWS Region (e.g., `us-east-1`).
- A local file (e.g., `hello.txt` or an image) to upload.
- (Optional) Naming plan for bucket and folders. Example names used below:
  - Bucket: `my-first-lab-bucket-2025`
  - Folder: `lab-artifacts/`
  - Object: `lab-artifacts/welcome.txt`

---

## Concept Refresher

- **Bucket:** Top-level container; names are globally unique.
- **Folder/Prefix:** Visual aid in the Console; technically part of the object key (e.g., `lab-artifacts/welcome.txt`).
- **Object:** Data + metadata. Uploads can be done via Console, CLI, or SDK.
- **Bucket policy:** JSON document applied at the bucket level to control access to bucket resources.

---

## Step-by-Step Guide

### 1. Create the S3 bucket
1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. Search for **S3** and open the service.
3. Click **Create bucket**.
4. Enter a globally unique **Bucket name** (e.g., `my-first-lab-bucket-2025`).
5. Choose your Region.
6. Leave **Block Public Access** ON for production. For this lab, I left the default on and later attached a scoped bucket policy for objects only.
7. Keep remaining defaults and click **Create bucket**.

### 2. Create a folder (prefix)
1. Inside the new bucket, click **Create folder**.
2. Name it `lab-artifacts/`.
3. Leave encryption and storage class default → **Create folder**.

### 3. Upload an object
1. Open the `lab-artifacts/` folder.
2. Click **Upload** → **Add files** → pick your local sample file (e.g., `welcome.txt`).
3. Keep **Storage class** as **Standard** and leave encryption default.
4. Click **Upload**. Wait for the green success banner.

### 4. Attach a basic GetObject bucket policy
1. Navigate to the bucket’s **Permissions** tab.
2. Scroll to **Bucket policy** → **Edit**.
3. Paste the JSON from the next section (replace the bucket name).
4. Save changes. AWS validates the JSON before applying it.

---

## Bucket Policy Example

This policy grants public read access to objects only (not bucket listing). Apply it only to demo buckets; for production, scope to specific principals or CloudFront origins.

```startLine:endLine:s3-lab-basic/README.md
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-first-lab-bucket-2025/*"
    }
  ]
}
```

---

## Verification & Testing

- **Console preview:** Select the uploaded object → **Open**. If the policy is in place, the object loads in a new tab.
- **Public URL test:** Copy the **Object URL** (e.g., `https://my-first-lab-bucket-2025.s3.amazonaws.com/lab-artifacts/welcome.txt`) and open it in an incognito window. Expect the file contents; if you see `AccessDenied`, re-check the policy and Block Public Access settings.
- **Permissions check:** Use the **Permissions** tab → **Access analyzer** to confirm that only `s3:GetObject` is public.

---

## Cleanup & Next Steps

- Delete test objects or the entire bucket when finished to avoid clutter.
- Re-enable stricter Block Public Access settings for future labs.
- Try the same workflow using the AWS CLI (`aws s3 mb`, `aws s3 cp`) or Infrastructure as Code (CloudFormation/Terraform).
- Extend the lab by enabling versioning, lifecycle rules, or static website hosting.

---

*End of S3 Basic Lab.*

