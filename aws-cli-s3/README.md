# AWS CLI & S3 Lab — Command-Line Operations & Advanced Concepts

> **Objective:** Learn how to configure AWS CLI, manage S3 buckets via command line, understand bucket versioning and cross-region replication, and get introduced to AWS Lambda and its differences from EC2.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Part 1: AWS CLI Configuration](#part-1-aws-cli-configuration)
   - Installing AWS CLI
   - Creating IAM Access Keys
   - Configuring AWS CLI
   - Verifying Configuration
4. [Part 2: S3 CLI Commands](#part-2-s3-cli-commands)
   - Common S3 Operations
   - Command Examples
5. [Part 3: S3 Bucket Versioning](#part-3-s3-bucket-versioning)
   - What is Versioning?
   - Enabling Versioning via CLI
   - Working with Versions
6. [Part 4: Cross-Region Replication (CRR)](#part-4-cross-region-replication-crr)
   - Understanding CRR
   - Setting Up Replication
7. [Part 5: Introduction to AWS Lambda](#part-5-introduction-to-aws-lambda)
   - What is Lambda?
   - Key Characteristics
8. [Part 6: EC2 vs Lambda Comparison](#part-6-ec2-vs-lambda-comparison)
   - Differences Summary
   - When to Use Which
9. [Key Takeaways](#key-takeaways)
10. [Next Steps](#next-steps)

---

## Overview

This lab covers multiple AWS concepts:

- **AWS CLI Setup:** Configure command-line access to AWS services.
- **S3 Operations:** Perform bucket and object management using terminal commands.
- **Advanced S3 Features:** Versioning and cross-region replication for data protection and compliance.
- **Serverless Computing:** Introduction to AWS Lambda and comparison with EC2.

These skills enable automation, scripting, and efficient cloud resource management.

---

## Prerequisites

- AWS account with appropriate permissions.
- Terminal/Command Prompt access (macOS, Linux, or Windows).
- AWS CLI installed (or instructions to install it).
- Basic familiarity with command-line interfaces.

---

## Part 1: AWS CLI Configuration

### Installing AWS CLI

**macOS (using Homebrew):**
```bash
brew install awscli
```

**Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Windows:**
Download the MSI installer from [AWS CLI Downloads](https://aws.amazon.com/cli/).

Verify installation:
```bash
aws --version
```

### Creating IAM Access Keys

Access keys are required for programmatic access via CLI. **Never share or commit access keys to version control.**

**Steps to create access keys:**

1. Sign in to [AWS Management Console](https://console.aws.amazon.com/).
2. Navigate to **IAM** → **Users**.
3. Select your user (or create a new one).
4. Go to the **Security credentials** tab.
5. Scroll to **Access keys** → click **Create access key**.
6. Choose **Command Line Interface (CLI)** as the use case.
7. Review and click **Create access key**.
8. **Important:** Download the CSV file or copy both:
   - **Access Key ID** (e.g., `AKIAIOSFODNN7EXAMPLE`)
   - **Secret Access Key** (e.g., `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`)

> ⚠️ **Security Note:** The secret access key is shown only once. Store it securely. If lost, delete the key and create a new one.

### Configuring AWS CLI

Run the configure command:
```bash
aws configure
```

You'll be prompted for:
1. **AWS Access Key ID:** Paste your access key ID.
2. **AWS Secret Access Key:** Paste your secret access key.
3. **Default region name:** Enter your preferred region (e.g., `us-east-1`, `eu-west-1`).
4. **Default output format:** Choose `json`, `yaml`, `text`, or `table` (recommended: `json`).

Example session:
```bash
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: json
```

Configuration is stored in:
- **macOS/Linux:** `~/.aws/credentials` and `~/.aws/config`
- **Windows:** `C:\Users\USERNAME\.aws\credentials` and `C:\Users\USERNAME\.aws\config`

### Verifying Configuration

Test your setup:
```bash
aws sts get-caller-identity
```

Expected output:
```json
{
    "UserId": "AIDAIOSFODNN7EXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}
```

---

## Part 2: S3 CLI Commands

The AWS CLI provides powerful commands for managing S3 buckets and objects. Here are the most common operations:

### Common S3 Operations

#### Create a Bucket
```bash
aws s3 mb s3://bucket-name --region us-east-1
```
Note: Bucket names must be globally unique. Use `--region` to specify the region.

#### List Buckets
```bash
aws s3 ls
```

#### List Objects in a Bucket
```bash
aws s3 ls s3://bucket-name/
```

#### List Objects with Prefix (Folder)
```bash
aws s3 ls s3://bucket-name/folder-name/
```

#### Upload a File
```bash
aws s3 cp local-file.txt s3://bucket-name/
```

#### Upload a Directory (Recursive)
```bash
aws s3 cp ./local-folder/ s3://bucket-name/folder-name/ --recursive
```

#### Download a File
```bash
aws s3 cp s3://bucket-name/file.txt ./local-file.txt
```

#### Download a Directory (Recursive)
```bash
aws s3 cp s3://bucket-name/folder-name/ ./local-folder/ --recursive
```

#### Sync Directories
```bash
aws s3 sync ./local-folder/ s3://bucket-name/folder-name/
```
Syncs only changed/new files, more efficient than recursive copy.

#### Delete an Object
```bash
aws s3 rm s3://bucket-name/file.txt
```

#### Delete a Directory (Recursive)
```bash
aws s3 rm s3://bucket-name/folder-name/ --recursive
```

#### Delete a Bucket (must be empty)
```bash
aws s3 rb s3://bucket-name
```

#### Delete a Bucket and All Contents
```bash
aws s3 rb s3://bucket-name --force
```

#### Copy Object Between Buckets
```bash
aws s3 cp s3://source-bucket/file.txt s3://destination-bucket/file.txt
```

#### Move Object (Copy + Delete)
```bash
aws s3 mv s3://bucket-name/old-name.txt s3://bucket-name/new-name.txt
```

### Command Examples

**Example: Upload a file with metadata**
```bash
aws s3 cp document.pdf s3://my-bucket/documents/ \
  --metadata "author=John Doe,project=Lab"
```

**Example: Upload with specific storage class**
```bash
aws s3 cp large-file.zip s3://my-bucket/archive/ \
  --storage-class STANDARD_IA
```

**Example: Exclude files during sync**
```bash
aws s3 sync ./source/ s3://my-bucket/ \
  --exclude "*.tmp" --exclude "*.log"
```

**Example: Set ACL during upload**
```bash
aws s3 cp public-file.html s3://my-bucket/ \
  --acl public-read
```

---

## Part 3: S3 Bucket Versioning

### What is Versioning?

**Bucket versioning** keeps multiple variants of an object in the same bucket. When versioning is enabled:

- Each time you upload an object with the same key, S3 creates a new version.
- Previous versions are preserved and can be retrieved.
- This protects against accidental overwrites and deletions.
- Useful for compliance, backup, and recovery scenarios.

**Important points:**
- Once enabled, versioning can be **suspended** but not disabled.
- Each version consumes storage (costs apply).
- Use lifecycle policies to manage old versions.

### Enabling Versioning via CLI

**Enable versioning:**
```bash
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Enabled
```

**Suspend versioning:**
```bash
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Suspended
```

**Check versioning status:**
```bash
aws s3api get-bucket-versioning --bucket my-bucket-name
```

### Working with Versions

**List all versions of objects:**
```bash
aws s3api list-object-versions --bucket my-bucket-name
```

**Download a specific version:**
```bash
aws s3api get-object \
  --bucket my-bucket-name \
  --key file.txt \
  --version-id VERSION_ID \
  local-file.txt
```

**Delete a specific version:**
```bash
aws s3api delete-object \
  --bucket my-bucket-name \
  --key file.txt \
  --version-id VERSION_ID
```

**Restore a previous version (make it current):**
```bash
# Copy old version to current
aws s3api copy-object \
  --bucket my-bucket-name \
  --copy-source my-bucket-name/file.txt?versionId=VERSION_ID \
  --key file.txt
```

---

## Part 4: Cross-Region Replication (CRR)

### Understanding CRR

**Cross-Region Replication (CRR)** automatically replicates objects from a source bucket in one region to a destination bucket in another region.

**Use cases:**
- **Compliance:** Store data in multiple regions for regulatory requirements.
- **Disaster Recovery:** Backup data to a different geographic location.
- **Latency Reduction:** Keep copies closer to end users.
- **Data Sovereignty:** Meet data residency requirements.

**Requirements:**
- Versioning must be enabled on both source and destination buckets.
- Source and destination buckets must be in different AWS regions.
- IAM role with replication permissions must be created.

### Setting Up Replication

**Step 1: Enable versioning on both buckets**
```bash
# Source bucket
aws s3api put-bucket-versioning \
  --bucket source-bucket-name \
  --versioning-configuration Status=Enabled

# Destination bucket
aws s3api put-bucket-versioning \
  --bucket destination-bucket-name \
  --versioning-configuration Status=Enabled
```

**Step 2: Create IAM role for replication**

Create a policy document `replication-policy.json`:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetReplicationConfiguration",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::source-bucket-name"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersionForReplication",
        "s3:GetObjectVersionAcl",
        "s3:GetObjectVersionTagging"
      ],
      "Resource": [
        "arn:aws:s3:::source-bucket-name/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete",
        "s3:ReplicateTags"
      ],
      "Resource": [
        "arn:aws:s3:::destination-bucket-name/*"
      ]
    }
  ]
}
```

**Step 3: Configure replication rule**

Create `replication-config.json`:
```json
{
  "Role": "arn:aws:iam::ACCOUNT_ID:role/replication-role",
  "Rules": [
    {
      "Status": "Enabled",
      "Priority": 1,
      "Filter": {},
      "Destination": {
        "Bucket": "arn:aws:s3:::destination-bucket-name",
        "StorageClass": "STANDARD"
      }
    }
  ]
}
```

Apply the replication configuration:
```bash
aws s3api put-bucket-replication \
  --bucket source-bucket-name \
  --replication-configuration file://replication-config.json
```

**Verify replication:**
```bash
aws s3api get-bucket-replication --bucket source-bucket-name
```

---

## Part 5: Introduction to AWS Lambda

### What is Lambda?

**AWS Lambda** is a serverless compute service that runs code in response to events without provisioning or managing servers.

**Key characteristics:**
- **Serverless:** No servers to manage—AWS handles infrastructure.
- **Event-driven:** Executes code in response to triggers (S3 uploads, API calls, scheduled events, etc.).
- **Pay-per-use:** Charged only for compute time consumed (per 100ms).
- **Auto-scaling:** Automatically scales from zero to thousands of concurrent executions.
- **Supported languages:** Node.js, Python, Java, Go, .NET, Ruby, and more.

**Common use cases:**
- Processing files uploaded to S3.
- Building REST APIs with API Gateway.
- Scheduled tasks (cron jobs).
- Real-time data transformation.
- Backend for mobile/web applications.

### Key Characteristics

- **Function:** Your code packaged as a Lambda function.
- **Trigger:** Event source that invokes the function (S3, API Gateway, CloudWatch Events, etc.).
- **Runtime:** Language and version environment (e.g., Python 3.11).
- **Memory:** Configurable from 128 MB to 10 GB (affects CPU and cost).
- **Timeout:** Maximum execution time (up to 15 minutes).
- **Concurrency:** Number of simultaneous executions (default: 1000 per region).

**Example Lambda trigger scenario:**
```
S3 Bucket → Object Upload Event → Lambda Function → Process Image/Data
```

---

## Part 6: EC2 vs Lambda Comparison

### Differences Summary

| Aspect | EC2 | Lambda |
|--------|-----|--------|
| **Infrastructure Management** | You manage servers, OS, patches, scaling | Fully managed by AWS |
| **Provisioning** | Manual or Auto Scaling Groups | Automatic, instant |
| **Scaling** | Manual configuration required | Automatic, scales to zero |
| **Billing Model** | Pay for running instances (hourly) | Pay per request and compute time (100ms increments) |
| **Startup Time** | Minutes (instance launch) | Milliseconds (cold start: ~100ms-5s) |
| **Use Case** | Long-running applications, web servers, databases | Event-driven, short-lived tasks, microservices |
| **Runtime Control** | Full control over OS and environment | Limited to supported runtimes |
| **State** | Can maintain state | Stateless (use external storage) |
| **Maximum Execution Time** | Unlimited (as long as instance runs) | 15 minutes |
| **Concurrent Executions** | Limited by instance capacity | Up to 1000 concurrent (configurable) |
| **Cost Efficiency** | Good for predictable, constant workloads | Excellent for sporadic, event-driven workloads |

### When to Use Which

**Choose EC2 when:**
- You need full control over the operating system and runtime environment.
- Running long-running applications (web servers, databases, batch jobs).
- Predictable, constant traffic patterns.
- Need to install custom software or dependencies not supported by Lambda.
- Applications require persistent connections or stateful operations.

**Choose Lambda when:**
- Event-driven workloads (file processing, API backends, scheduled tasks).
- Sporadic or unpredictable traffic patterns.
- Short-duration tasks (under 15 minutes).
- Want to minimize operational overhead (no server management).
- Building microservices or serverless architectures.
- Cost optimization for low-to-medium traffic applications.

**Hybrid Approach:**
Many applications use both:
- **EC2** for main application servers or databases.
- **Lambda** for event processing, background jobs, or API endpoints.

---

## Key Takeaways

1. **AWS CLI** enables automation and scripting of AWS operations from the command line.
2. **Access keys** must be created securely and never shared or committed to version control.
3. **S3 CLI commands** (`cp`, `sync`, `rm`, `mb`, `rb`) provide efficient bucket and object management.
4. **Bucket versioning** protects against accidental deletions and enables point-in-time recovery.
5. **Cross-Region Replication** ensures data redundancy and compliance across geographic regions.
6. **AWS Lambda** offers serverless, event-driven compute with automatic scaling and pay-per-use pricing.
7. **EC2 vs Lambda:** Choose EC2 for long-running, predictable workloads; choose Lambda for event-driven, short-duration tasks.

---

## Next Steps

- Practice S3 CLI commands with real buckets and objects.
- Enable versioning on a test bucket and experiment with version management.
- Set up cross-region replication between two buckets in different regions.
- Create your first Lambda function triggered by an S3 upload event.
- Explore AWS CLI help: `aws s3 help` or `aws lambda help`.
- Learn about S3 lifecycle policies to automate version management.
- Study Lambda function deployment using AWS CLI (`aws lambda create-function`).
- Compare costs between EC2 and Lambda for your specific use cases.

---

*End of AWS CLI & S3 Lab.*

