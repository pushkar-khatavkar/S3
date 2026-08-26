# Amazon S3 Masterclass: Complete Hands-On Lab Guide

## Lab Overview

| Item | Details |
| --- | --- |
| **Objective** | Master Amazon S3 from core hosting to advanced transfer, security, and developer integrations |
| **Duration** | 120 minutes (2 Hours) |
| **Target Audience** | Beginner Cloud Students |
| **Cost** | Free Tier Eligible |
| **AWS Region** | US East (N. Virginia) `us-east-1` |

## Architecture & Roadmap

```text
User (Web Browser) ──► [ S3 Static Website Endpoint ]
                            ├─ Bucket Policy (Public Read)
                            ├─ Versioning (V1, V2, Delete Markers)
                            └─ CORS Configuration
Cross-Origin App   ──► [ S3 Bucket ] ◄── HTTP OPTIONS Preflight
AWS CLI / SDK      ──► [ S3 Access Point (Read-Only Alias) ]
Global User (WAN)  ──► [ S3 Transfer Acceleration Endpoint ]
                            └─ Multipart Upload (Chunked Data)

```

---

## Pre-requisites: Setup Your Workspace

Open your terminal and prepare your local directory. Ensure you follow the paths exactly as written.

```bash
mkdir -p ~/aws-s3-workshop
cd ~/aws-s3-workshop

cat << 'EOF' > index.html
<!DOCTYPE html>
<html>
<head><title>S3 Masterclass</title></head>
<body style="text-align:center; padding:50px; font-family:sans-serif; background:#0f172a; color:white;">
    <h2>Amazon S3 Hands-On Workshop</h2>
    <p>Static Website Hosted Successfully!</p>
    <p>Version: 1.0</p>
</body>
</html>
EOF

dd if=/dev/zero of=large-file.dat bs=1M count=25

```

---

## Part 1: Provisioning & Static Website Hosting

**Step 1: Create the Bucket**
Go to AWS Console -> S3 -> Create bucket. Name it `cloudkida-<your-name>` (must be globally unique). Select the US East (N. Virginia) `us-east-1` region. Uncheck "Block all public access" and acknowledge the warning. Click Create bucket.

**Step 2: Enable Static Website Hosting**
Click your newly created bucket and go to the Properties tab. Scroll to the bottom to Static website hosting and click Edit. Select Enable and choose Host a static website. Set the Index document to `index.html`. Click Save changes. Copy the Bucket website endpoint.

**Step 3: Add Bucket Policy**
Go to the Permissions tab. Scroll to Bucket policy and click Edit. Paste the following JSON, replacing `YOUR-BUCKET-NAME`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        }
    ]
}

```

Click Save changes.

**Step 4: Upload & Verify**
Upload `index.html` directly to the root of your bucket via the AWS Console or CLI.

**Verification Commands:**

```bash
aws s3api get-bucket-policy \
    --bucket cloudkida-<your-name> \
    --output json

curl -I http://cloudkida-<your-name>.s3-website-us-east-1.amazonaws.com

```

---

## Part 2: Security & Cross-Origin Resource Sharing (CORS)

**Step 1: Configure CORS**
In your bucket, go to the Permissions tab. Scroll down to Cross-origin resource sharing (CORS) and click Edit. Paste this configuration to allow external web applications to execute GET requests:

```json
[
    {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET"],
        "AllowedOrigins": ["https://myapp.com"],
        "ExposeHeaders": []
    }
]

```

Click Save changes.

**Step 2: Verify CORS via CLI**

**Verification Command:**

```bash
curl -i -X OPTIONS \
    -H "Origin: https://myapp.com" \
    -H "Access-Control-Request-Method: GET" \
    http://cloudkida-<your-name>.s3.us-east-1.amazonaws.com/index.html

```

*Expected Output: The response headers must explicitly include `Access-Control-Allow-Origin: [https://myapp.com](https://myapp.com)`.*

---

## Part 3: S3 Object Versioning

**Step 1: Enable & Test Versioning**
Go to the Properties tab -> Bucket Versioning -> Click Edit -> Enable -> Save. Modify your local `index.html` file to say `Version: 2.0`. Upload the modified file to S3. Refresh your website in the browser to see the update.

**Verification Command:**

```bash
aws s3api get-bucket-versioning \
    --bucket cloudkida-<your-name> \
    --query "Status" \
    --output text

```

**Step 2: Test Delete Markers**
In the Objects tab, toggle Show versions to OFF. Delete `index.html`. Your website will now show a 404 Not Found error. Toggle Show versions to ON. You will see a Delete marker placed on top of your previous versions. Delete only the Delete marker object. Refresh your website. Version 2.0 is instantly restored.

---

## Part 4: Access Points & CLI Integration

**Step 1: Create a Read-Only Access Point**
In the S3 Console's left-hand menu, click Access Points -> Create access point. Name it `workshop-read-ap`. Select your bucket and set the Network Origin to Internet.

To ensure the `aws s3 ls` command works in the next step, we must grant `s3:ListBucket` permissions on the Access Point itself, alongside `s3:GetObject` for the objects. Paste the following Access Point Policy (Replace `YOUR-ACCOUNT-ID`):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowReadOnlyListAndGet",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:us-east-1:YOUR-ACCOUNT-ID:accesspoint/workshop-read-ap",
                "arn:aws:s3:us-east-1:YOUR-ACCOUNT-ID:accesspoint/workshop-read-ap/object/*"
            ]
        },
        {
            "Sid": "ExplicitDenyWrite",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:us-east-1:YOUR-ACCOUNT-ID:accesspoint/workshop-read-ap/object/*"
        }
    ]
}

```

Click Create access point.

**Step 2: Verify Access Point Constraints**

**Verification Commands (Replace with your Access Point Alias):**

```bash
# Test Read Access (List contents)
aws s3 ls s3://workshop-read-ap-<alias-id>-s3alias/

# Test Write Access (Upload attempt)
aws s3 cp ./index.html s3://workshop-read-ap-<alias-id>-s3alias/

```

*Expected Output: The `ls` command will successfully list the bucket contents, but the `cp` command will fail with `AccessDenied`.*

---

## Part 5: Advanced Uploads (Multipart & Acceleration)

**Step 1: Trigger a Multipart Upload**
Run this command. The AWS CLI handles large file chunking automatically (e.g., 12%, 47%, 90%).

```bash
aws s3 cp ~/aws-s3-workshop/large-file.dat s3://cloudkida-<your-name>/

```

**Step 2: Visual Proof of Multipart Assembly**

**Verification Command:**

```bash
aws s3api head-object \
    --bucket cloudkida-<your-name> \
    --key large-file.dat \
    --query 'ETag'

```

*Expected Output: An ETag containing a hyphen and a part count, such as `"d41d8cd98f00b204e9800998ecf8427e-4"`.*

**Step 3: Transfer Acceleration Speed Test**
Go to Bucket Properties -> Transfer acceleration -> Enable. We will now upload the same `.dat` file using the standard endpoint, and then using the accelerated endpoint via the AWS CLI to observe the time difference.

**Verification Commands:**

```bash
# 1. Standard Upload Test
time aws s3 cp ~/aws-s3-workshop/large-file.dat s3://cloudkida-<your-name>/standard-upload.dat --region us-east-1

# 2. Enable AWS CLI to use the accelerate endpoint
aws configure set default.s3.use_accelerate_endpoint true

# 3. Accelerated Upload Test
time aws s3 cp ~/aws-s3-workshop/large-file.dat s3://cloudkida-<your-name>/accelerated-upload.dat --region us-east-1

# 4. Disable accelerate endpoint in CLI config to return to normal
aws configure set default.s3.use_accelerate_endpoint false

```

*Expected Output: The `time` command will output the real, user, and sys times for both transfers, allowing you to compare the speed improvement provided by the CloudFront edge network.*

---

## Part 6: Developer Integration (Python SDK)

**Step 1: Create and Run the Script to List Objects**
Ensure Boto3 is installed (`pip3 install boto3`). Create the script:

```bash
cat << 'EOF' > s3_list.py
import boto3

bucket_name = "cloudkida-<your-name>"
s3 = boto3.client('s3')

print(f"Listing objects in bucket: {bucket_name}...\n")
response = s3.list_objects_v2(Bucket=bucket_name)

if 'Contents' in response:
    for obj in response['Contents']:
        print(f"- File: {obj['Key']} | Size: {obj['Size']} bytes")
else:
    print("Bucket is completely empty.")
EOF

```

Run the script: `python3 s3_list.py`

**Step 2: Verify Python Execution**

**Verification Command:**

```bash
echo $?

```

*Expected Output: `0` (indicates the Python script executed successfully without errors).*
