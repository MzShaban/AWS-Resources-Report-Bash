# AWS Resources Report Script

## Overview
This script automates the process of discovering AWS resources and generates a report with details about EC2 instances, S3 buckets, IAM users, and Lambda functions.

## Step-by-Step Execution

### Step 1: Set Up AWS Credentials
Before running the script, ensure your AWS CLI is configured with the correct credentials:

```sh
aws configure
```

Provide the following details:
- **AWS Access Key ID**
- **AWS Secret Access Key**
- **Default region** (e.g., `us-east-1`)
- **Default output format** (`json`, `table`, or `text`)

---

### Step 2: Create the Bash Script
Create a new file:

```sh
nano aws_resources_report.sh
```

Paste the following script:

```sh
#!/bin/bash

# Author: Moaz Shaban
# Date: 04.02.2025

# Description:
# This script performs the following steps:
# 1. Connects to an EC2 instance via SSH
# 2. Installs AWS CLI on the EC2 instance
# 3. Configures AWS CLI with credentials and region
# 4. Retrieves AWS resources: EC2 instances, S3 buckets, IAM users, and Lambda functions
# 5. Stores the results in a text file with a summary of counts and details
# 6. Mentions encountered issues (e.g., region mismatch) and how to resolve them

# Variables
EC2_INSTANCE_USER="ubuntu"  # Change to "ec2-user" for Amazon Linux
EC2_INSTANCE_IP="your-ec2-ip"  # Replace with your EC2 instance public IP
SSH_KEY_PATH="/path/to/your/private-key.pem"  # Replace with your private key path
REPORT_FILE="aws_resources_report.txt"

# Clear previous report
> "$REPORT_FILE"

echo "Generating AWS resources report..." > "$REPORT_FILE"

# Connect to EC2 and install AWS CLI
echo "Connecting to EC2 instance and setting up AWS CLI..." >> "$REPORT_FILE"
ssh -i "$SSH_KEY_PATH" "$EC2_INSTANCE_USER"@"$EC2_INSTANCE_IP" << 'EOF'
sudo apt update -y
sudo apt install -y awscli
aws --version
EOF

echo "AWS CLI installed on EC2." >> "$REPORT_FILE"

# Ensure AWS CLI is configured correctly
echo "Configuring AWS CLI..." >> "$REPORT_FILE"
aws configure set region us-east-1  # Change this if needed
aws configure set output table

# Fetch EC2 instances
echo -e "
==== EC2 Instances ====" >> "$REPORT_FILE"
EC2_LIST=$(aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`].Value|[0],State.Name]' --output table)
EC2_COUNT=$(echo "$EC2_LIST" | grep -c "i-")  # Count instances by filtering for instance IDs
echo "We have $EC2_COUNT EC2 instances." >> "$REPORT_FILE"
echo "$EC2_LIST" >> "$REPORT_FILE"

# Fetch S3 Buckets
echo -e "
==== S3 Buckets ====" >> "$REPORT_FILE"
S3_LIST=$(aws s3 ls)
S3_COUNT=$(echo "$S3_LIST" | wc -l)
echo "We have $S3_COUNT S3 buckets." >> "$REPORT_FILE"
echo "$S3_LIST" >> "$REPORT_FILE"

# Fetch IAM Users
echo -e "
==== IAM Users ====" >> "$REPORT_FILE"
IAM_LIST=$(aws iam list-users --query 'Users[*].[UserName]' --output table)
IAM_COUNT=$(echo "$IAM_LIST" | grep -c "UserName")
echo "We have $IAM_COUNT IAM users." >> "$REPORT_FILE"
echo "$IAM_LIST" >> "$REPORT_FILE"

# Fetch Lambda Functions
echo -e "
==== Lambda Functions ====" >> "$REPORT_FILE"
LAMBDA_LIST=$(aws lambda list-functions --query 'Functions[*].[FunctionName]' --output table)
LAMBDA_COUNT=$(echo "$LAMBDA_LIST" | grep -c "FunctionName")
echo "We have $LAMBDA_COUNT Lambda functions." >> "$REPORT_FILE"
echo "$LAMBDA_LIST" >> "$REPORT_FILE"

# Mention encountered issues and solutions
echo -e "
==== Issues and Fixes ====" >> "$REPORT_FILE"
echo "Issue: AWS region was incorrect, causing empty results." >> "$REPORT_FILE"
echo "Fix: Ensure the correct region is set using: aws configure set region us-east-1" >> "$REPORT_FILE"
echo "Verify the region using: aws configure get region" >> "$REPORT_FILE"

echo "AWS resources report generated successfully: $REPORT_FILE"
```

---

### Step 3: Make the Script Executable
After saving the script, give it execution permissions:

```sh
chmod +x aws_resources_report.sh
```

---

### Step 4: Run the Script
Execute the script:

```sh
./aws_resources_report.sh
```

---

## Troubleshooting Issues

### 1. "Permission denied" when executing the script
Run:
```sh
chmod +x aws_resources_report.sh
```
Try running it with:
```sh
sudo ./aws_resources_report.sh
```

### 2. "AWS CLI not found" inside EC2
Ensure AWS CLI is installed inside the EC2 instance:

```sh
ssh -i "/path/to/private-key.pem" ubuntu@your-ec2-ip
sudo apt install -y awscli
aws --version
```

### 3. "No EC2 instances found"
Check if the correct AWS region is set:

```sh
aws configure get region
```
If it's incorrect, change it:

```sh
aws configure set region us-east-1
```

---

## Final Thoughts
This script automates AWS resource discovery, making it easier to track EC2 instances, S3 buckets, IAM users, and Lambda functions in your AWS account. It also includes error handling and troubleshooting steps.

---


