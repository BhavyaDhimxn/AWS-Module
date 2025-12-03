# Event-Driven Architecture Lab — S3 + Lambda Integration

> **Objective:** Build a simple event-driven architecture where an S3 bucket triggers a Lambda function (Python) whenever a new object is uploaded. The Lambda function logs the file name and bucket name. Also learn about S3 storage classes and their use cases.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Concept Refresher](#concept-refresher)
4. [Part 1: Create S3 Bucket](#part-1-create-s3-bucket)
5. [Part 2: Create Lambda Function](#part-2-create-lambda-function)
6. [Part 3: Configure S3 Event Trigger](#part-3-configure-s3-event-trigger)
7. [Part 4: Test the Event-Driven Architecture](#part-4-test-the-event-driven-architecture)
8. [Part 5: S3 Storage Classes](#part-5-s3-storage-classes)
9. [Verification & Monitoring](#verification--monitoring)
10. [Troubleshooting](#troubleshooting)
11. [Cleanup & Next Steps](#cleanup--next-steps)

---

## Overview

This lab demonstrates a fundamental **event-driven architecture** pattern using AWS services:

- **S3 Bucket:** Source of events (object uploads)
- **Lambda Function:** Event handler that processes uploads
- **Event Bridge:** S3 automatically invokes Lambda when objects are created

**What happens:**
1. User uploads a file to the S3 bucket
2. S3 generates an event notification
3. Lambda function is automatically triggered
4. Lambda function processes the event and logs file details

This pattern is widely used for:
- Image processing and resizing
- Data transformation pipelines
- Automated backups and archiving
- Log file processing
- Real-time data analytics

---

## Prerequisites

- AWS account with permissions to create S3 buckets and Lambda functions
- Basic understanding of Python (for Lambda function code)
- Access to AWS Management Console
- (Optional) AWS CLI configured for testing

---

## Concept Refresher

### Event-Driven Architecture

**Event-driven architecture** is a design pattern where components communicate through events. In AWS:

- **Event Source:** Service that generates events (S3, DynamoDB, API Gateway, etc.)
- **Event Handler:** Service that responds to events (Lambda, SQS, SNS, etc.)
- **Decoupling:** Services don't need to know about each other directly

### S3 Event Notifications

S3 can send notifications when certain events occur:
- `s3:ObjectCreated:*` — Object is created (PUT, POST, Copy, Multipart Upload)
- `s3:ObjectCreated:Put` — Object is uploaded via PUT
- `s3:ObjectCreated:Post` — Object is uploaded via POST
- `s3:ObjectRemoved:*` — Object is deleted
- `s3:ObjectRemoved:Delete` — Object is deleted

### Lambda Function Basics

- **Runtime:** Programming language environment (Python, Node.js, Java, etc.)
- **Handler:** Entry point function that processes events
- **Event Object:** JSON payload containing event details
- **Context Object:** Runtime information about the invocation

---

## Part 1: Create S3 Bucket

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. Navigate to **S3** service.
3. Click **Create bucket**.
4. Enter a globally unique **Bucket name** (e.g., `event-driven-lab-bucket-2025`).
5. Choose your **Region** (e.g., `us-east-1`).
6. Keep default settings for:
   - **Object Ownership:** ACLs disabled (recommended)
   - **Block Public Access:** Keep enabled (unless needed)
   - **Bucket Versioning:** Disabled (for simplicity)
   - **Default encryption:** Server-side encryption (optional)
7. Click **Create bucket**.

**Note:** Remember the bucket name—you'll need it for the Lambda function and event configuration.

---

## Part 2: Create Lambda Function

### Step 1: Create the Function

1. Navigate to **Lambda** service in the AWS Console.
2. Click **Create function**.
3. Choose **Author from scratch**.
4. Configure:
   - **Function name:** `s3-upload-logger` (or your preferred name)
   - **Runtime:** `Python 3.11` or `Python 3.12` (latest stable)
   - **Architecture:** `x86_64`
5. Click **Create function**.

### Step 2: Write the Python Handler Code

In the Lambda function code editor, replace the default code with:

```python
import json

def lambda_handler(event, context):
    """
    Lambda function triggered by S3 object upload events.
    Logs the bucket name and object key (file name) for each uploaded file.
    """
    
    # Process each record in the event
    for record in event['Records']:
        # Extract S3 event details
        bucket_name = record['s3']['bucket']['name']
        object_key = record['s3']['object']['key']
        
        # Log the information
        print(f"New file uploaded!")
        print(f"Bucket: {bucket_name}")
        print(f"File name: {object_key}")
        
        # Optional: Return response
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'File processed successfully',
                'bucket': bucket_name,
                'file': object_key
            })
        }
```

**Code Explanation:**
- `lambda_handler`: Entry point function (required name)
- `event`: Contains S3 event data in JSON format
- `context`: Runtime information (not used here)
- `event['Records']`: Array of event records (one per object)
- `record['s3']['bucket']['name']`: Bucket name from event
- `record['s3']['object']['key']`: Object key (file path/name)

### Step 3: Configure Basic Settings

1. Scroll to **General configuration** → **Edit**.
2. Set:
   - **Timeout:** `30 seconds` (sufficient for logging)
   - **Memory:** `128 MB` (minimum, sufficient for this function)
3. Click **Save**.

### Step 4: Set Up IAM Permissions

Lambda needs permission to read from S3 and write logs to CloudWatch.

1. Go to the **Configuration** tab → **Permissions**.
2. Click on the **Execution role** name (e.g., `s3-upload-logger-role-xxxxx`).
3. This opens IAM. Click **Add permissions** → **Attach policies**.
4. Search and attach:
   - `AmazonS3ReadOnlyAccess` (if you want Lambda to read objects)
   - `CloudWatchLogsFullAccess` (for logging)
   
   **Note:** For this lab, CloudWatch logging happens automatically. The default execution role already has basic CloudWatch permissions.

5. Return to the Lambda function.

---

## Part 3: Configure S3 Event Trigger

Now connect S3 bucket events to the Lambda function.

### Option A: Configure from S3 Console

1. Go back to your S3 bucket.
2. Open the **Properties** tab.
3. Scroll to **Event notifications** → **Create event notification**.
4. Configure:
   - **Event name:** `trigger-lambda-on-upload`
   - **Prefix:** (optional) Filter by folder/prefix, e.g., `uploads/`
   - **Suffix:** (optional) Filter by file extension, e.g., `.jpg`
   - **Event types:** Select `All object create events` or specifically `Put`
5. Under **Destination**, select **Lambda function**.
6. Choose your Lambda function: `s3-upload-logger`.
7. Click **Save changes**.

### Option B: Configure from Lambda Console

1. In the Lambda function page, go to **Configuration** tab → **Triggers**.
2. Click **Add trigger**.
3. Select **S3** as the source.
4. Configure:
   - **Bucket:** Select your bucket (`event-driven-lab-bucket-2025`)
   - **Event type:** `All object create events` or `Put`
   - **Prefix/Suffix:** (optional) Leave blank for all files
5. Check **Recursive invocation** warning acknowledgment (if shown).
6. Click **Add**.

**Important:** When you add the trigger, AWS automatically adds the necessary resource-based policy to allow S3 to invoke your Lambda function.

---

## Part 4: Test the Event-Driven Architecture

### Test via Console Upload

1. Go to your S3 bucket.
2. Click **Upload** → **Add files** → select a test file (e.g., `test.txt`).
3. Click **Upload**.
4. Wait a few seconds for the Lambda function to execute.

### View Lambda Execution Results

1. Go to **Lambda** → your function (`s3-upload-logger`).
2. Open the **Monitor** tab → **View CloudWatch logs**.
3. Click on the latest **Log stream**.
4. You should see output like:

```
START RequestId: abc123-456-def-789
New file uploaded!
Bucket: event-driven-lab-bucket-2025
File name: test.txt
END RequestId: abc123-456-def-789
```

### Test via AWS CLI (Optional)

```bash
# Upload a file to trigger the event
aws s3 cp test-file.txt s3://event-driven-lab-bucket-2025/

# Check Lambda logs
aws logs tail /aws/lambda/s3-upload-logger --follow
```

### Expected Behavior

- Upload a file → Lambda executes automatically
- Check CloudWatch logs → See bucket name and file name printed
- Upload multiple files → Each triggers a separate Lambda invocation
- Upload to a subfolder → Lambda still triggers (unless filtered by prefix)

---

## Part 5: S3 Storage Classes

S3 offers different storage classes optimized for various access patterns and cost requirements.

### Storage Classes Overview

| Storage Class | Use Case | Durability | Availability | Cost |
|---------------|----------|------------|--------------|------|
| **S3 Standard** | Frequently accessed data | 99.999999999% (11 9's) | 99.99% | Highest |
| **S3 Intelligent-Tiering** | Unknown or changing access patterns | 11 9's | 99.9% | Automatic cost optimization |
| **S3 Standard-IA** | Infrequently accessed, rapid retrieval | 11 9's | 99.9% | Lower than Standard |
| **S3 One Zone-IA** | Infrequently accessed, non-critical data | 11 9's | 99.5% | Lower than Standard-IA |
| **S3 Glacier Instant Retrieval** | Archive with instant access | 11 9's | 99.9% | Very low |
| **S3 Glacier Flexible Retrieval** | Archive, retrieval in 1-5 minutes | 11 9's | 99.99% | Very low |
| **S3 Glacier Deep Archive** | Long-term archive, 12-hour retrieval | 11 9's | 99.99% | Lowest |

### Key Characteristics

**S3 Standard:**
- Default storage class
- Best for frequently accessed data
- Low latency, high throughput
- Use for: Active websites, content distribution, big data analytics

**S3 Intelligent-Tiering:**
- Automatically moves objects between access tiers
- No retrieval fees or operational overhead
- Small monthly monitoring fee per object
- Use for: Data with unknown or changing access patterns

**S3 Standard-IA (Infrequent Access):**
- Lower storage cost than Standard
- Higher retrieval cost
- Minimum billable object size: 128 KB
- Minimum storage duration: 30 days
- Use for: Backups, disaster recovery, long-term storage

**S3 One Zone-IA:**
- Similar to Standard-IA but stored in single Availability Zone
- 20% cheaper than Standard-IA
- Lower availability (data loss risk if AZ fails)
- Use for: Secondary backup copies, recreatable data

**S3 Glacier Instant Retrieval:**
- Archive storage with millisecond retrieval
- Lower cost than Standard-IA
- Minimum billable object size: 128 KB
- Minimum storage duration: 90 days
- Use for: Archive data that needs instant access

**S3 Glacier Flexible Retrieval:**
- Three retrieval options:
  - **Expedited:** 1-5 minutes
  - **Standard:** 3-5 hours
  - **Bulk:** 5-12 hours
- Use for: Long-term backups, compliance archives

**S3 Glacier Deep Archive:**
- Lowest cost storage class
- Retrieval time: 12 hours
- Use for: Compliance archives, long-term retention (7-10+ years)

### Setting Storage Class During Upload

**Via Console:**
1. Upload file → **Properties** → **Storage class**
2. Select desired class (e.g., `Standard-IA`)

**Via CLI:**
```bash
# Upload with Standard-IA storage class
aws s3 cp file.txt s3://bucket-name/ \
  --storage-class STANDARD_IA

# Upload with Glacier Instant Retrieval
aws s3 cp archive.zip s3://bucket-name/ \
  --storage-class GLACIER_IR
```

### Lifecycle Policies for Automatic Transitions

Automatically move objects between storage classes based on age:

**Example: Move to Standard-IA after 30 days**
```json
{
  "Rules": [
    {
      "Id": "MoveToStandardIA",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        }
      ]
    }
  ]
}
```

**Example: Move to Glacier after 90 days, then Deep Archive after 365 days**
```json
{
  "Rules": [
    {
      "Id": "ArchiveRule",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ]
    }
  ]
}
```

---

## Verification & Monitoring

### CloudWatch Logs

Monitor Lambda executions:
1. **Lambda Console** → **Monitor** → **View CloudWatch logs**
2. Or directly: **CloudWatch** → **Log groups** → `/aws/lambda/s3-upload-logger`

### CloudWatch Metrics

View function metrics:
- **Invocations:** Number of times function executed
- **Duration:** Execution time
- **Errors:** Failed invocations
- **Throttles:** When function hits concurrency limits

### Test Scenarios

1. **Single file upload:** Verify one log entry
2. **Multiple files:** Verify multiple invocations
3. **Large file:** Check if timeout occurs (increase timeout if needed)
4. **Different file types:** Test with `.txt`, `.jpg`, `.pdf`, etc.

---

## Troubleshooting

### Lambda Not Triggering

**Check:**
- S3 event notification is configured correctly
- Lambda function has correct permissions
- Bucket and Lambda are in the same region (recommended)
- Event notification prefix/suffix filters aren't excluding your files

### Lambda Function Errors

**Common issues:**
- **Timeout:** Increase function timeout in configuration
- **Memory:** Increase memory allocation if processing large files
- **Permissions:** Ensure execution role has CloudWatch logs permission
- **Code errors:** Check CloudWatch logs for Python exceptions

### Viewing Error Details

1. **Lambda Console** → **Monitor** → **View CloudWatch logs**
2. Look for `ERROR` or exception stack traces
3. Common errors:
   - `KeyError`: Missing event field (check event structure)
   - `TimeoutError`: Function exceeded timeout
   - `AccessDenied`: Missing IAM permissions

### Testing Lambda Manually

Test the function with a sample S3 event:

1. **Lambda Console** → **Test** tab
2. Create a new test event with this JSON:

```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "us-east-1",
      "eventTime": "2025-01-15T10:00:00.000Z",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "event-driven-lab-bucket-2025"
        },
        "object": {
          "key": "test-file.txt",
          "size": 1024
        }
      }
    }
  ]
}
```

3. Click **Test** → Verify output matches expected format

---

## Cleanup & Next Steps

### Cleanup Steps

1. **Delete S3 objects:**
   - Go to S3 bucket → Select all objects → **Delete**
   - Or delete the entire bucket

2. **Remove Lambda trigger:**
   - Lambda → **Configuration** → **Triggers** → Delete S3 trigger

3. **Delete Lambda function:**
   - Lambda → Select function → **Delete**

4. **Clean up CloudWatch logs (optional):**
   - CloudWatch → **Log groups** → `/aws/lambda/s3-upload-logger` → **Delete log group**

### Next Steps & Extensions

- **Process file contents:** Modify Lambda to read and process uploaded files (e.g., image resizing, text extraction)
- **Multiple triggers:** Add triggers for `ObjectRemoved` events
- **Error handling:** Add SQS Dead Letter Queue for failed Lambda invocations
- **Filtering:** Use prefix/suffix filters to trigger Lambda only for specific file types
- **Storage class automation:** Create lifecycle policies to automatically transition objects
- **Cost optimization:** Analyze access patterns and move infrequently accessed data to cheaper storage classes
- **Monitoring:** Set up CloudWatch alarms for Lambda errors or S3 bucket metrics
- **Multi-region:** Replicate events to Lambda functions in multiple regions

### Real-World Use Cases

- **Image processing:** Resize images on upload, generate thumbnails
- **Data pipeline:** Transform CSV files, load into databases
- **Log aggregation:** Process log files, send to analytics services
- **Backup validation:** Verify backup files after upload
- **Content moderation:** Scan uploaded content for inappropriate material

---

*End of Event-Driven Architecture Lab.*

