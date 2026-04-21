
---

# ITCS-6190 Assignment 3: AWS Data Processing Pipeline

This project demonstrates an end-to-end serverless data pipeline using AWS services. The workflow includes data ingestion into S3, processing using AWS Lambda, cataloging with AWS Glue, querying using Amazon Athena, and visualizing results using a web dashboard hosted on EC2.

---

# 1. Amazon S3 Bucket Structure 🪣

Create an S3 bucket to organize the data pipeline workflow.

### Structure:

```
bucket-name/
│
├── raw/         → Stores raw input data (CSV files)
├── processed/   → Stores cleaned and transformed data from Lambda
└── enriched/    → Stores Athena query outputs
```

### Purpose:

* **raw/** → Input dataset upload
* **processed/** → Lambda output after transformation
* **enriched/** → Query results storage

📸 **Screenshot 1: S3 Bucket Structure showing raw, processed, enriched folders**
<img width="1919" height="866" alt="Screenshot 2026-04-20 222713" src="https://github.com/user-attachments/assets/f3d0c554-460c-4d39-8adc-b2cd5e2a26cb" />


---

# 2. IAM Roles and Permissions 🔐

Create IAM roles to allow AWS services to communicate securely.

## Lambda Execution Role

Attach:

* AWSLambdaBasicExecutionRole
* AmazonS3FullAccess

---

## Glue Service Role

Attach:

* AWSGlueConsoleFullAccess
* AWSGlueServiceRole
* AmazonS3FullAccess

---

## EC2 Instance Role

Attach:

* AmazonS3FullAccess
* AmazonAthenaFullAccess

<img width="1919" height="866" alt="Screenshot 2026-04-20 222752" src="https://github.com/user-attachments/assets/f67f2b8a-6f39-4100-a7dc-e039634cea74" />


---

# 3. Create Lambda Function ⚙️

Create a Lambda function to process raw data.

* Name: `FilterAndProcessOrders`
* Runtime: Python 3.9+
* Role: Lambda Execution Role

### Function Purpose:

* Reads CSV from `raw/`
* Processes and filters data
* Writes output to `processed/`

📸 **Lambda Function Creation Page**
<img width="1919" height="868" alt="Screenshot 2026-04-20 222826" src="https://github.com/user-attachments/assets/7a3365c6-8d90-46a8-b17b-5f5075344ba0" />

---

# 4. Configure S3 Trigger ⚡

Configure trigger to automatically invoke Lambda.

### Settings:

* Bucket: your S3 bucket
* Event: All object create events
* Prefix: `raw/`
* Suffix: `.csv`

📸 **S3 Trigger Configuration**
<img width="1919" height="870" alt="Screenshot 2026-04-20 222933" src="https://github.com/user-attachments/assets/28c1a7b2-a828-425d-a773-74bd6eb92ab8" />

---

### Test Step:

Processed Folder

```
processed/
```

This triggers Lambda automatically.
<img width="1919" height="880" alt="Screenshot 2026-04-20 222957" src="https://github.com/user-attachments/assets/0f5ef048-5ac2-4fe0-a80d-47d4061f83af" />

---

# 5. AWS Glue Crawler 🕸️

Create a crawler to catalog processed data.

* Name: `orders_processed_crawler`
* Data Source: `s3://bucket-name/processed/`
* IAM Role: Glue Service Role
* Database: `orders_db`

Run crawler to create table.

📸 **Glue Crawler Setup**
<img width="1919" height="872" alt="Screenshot 2026-04-20 223032" src="https://github.com/user-attachments/assets/772ecd57-9055-416a-8c69-c12fc84d60e4" />

---

# 6. Amazon Athena Queries 🔍

Use Athena to query processed data.

Set:

* Data source: AwsDataCatalog
* Database: `orders_db`

### Queries:

1. Total Sales by Customer
2. Monthly Order Volume and Revenue
3. Order Status Dashboard
4. Average Order Value (AOV)
5. Top 10 Orders (Feb 2025)


---

### Output Storage:

Query results are stored in:

```
s3://bucket-name/enriched/
```

📸 **Athena outputs**
<img width="1919" height="865" alt="Screenshot 2026-04-20 223146" src="https://github.com/user-attachments/assets/8a005b5f-4f55-4952-a112-dc2bd36f5803" />

---

# 7. Launch EC2 Web Server 🖥️

Launch EC2 instance to host dashboard.

### Configuration:

* AMI: Amazon Linux 2023
* Instance Type: t2.micro
* Ports:

  * SSH (22)
  * Custom TCP (5000)

Attach EC2 IAM Role.

📸 **EC2 Instance Running**

---

# 8. Connect to EC2 Instance 🔗

SSH into instance:

```bash
ssh -i key.pem ec2-user@<public-ip>
```

Install dependencies:

```bash
sudo yum update -y
sudo yum install python3-pip -y
pip3 install Flask boto3
```

---

# 9. Create Web Application 🌐

Create Flask app:

```bash
nano app.py
```

Run dashboard using Athena query results.

---

# 10. Run Dashboard 🚀

Start server:

```bash
python3 app.py
```

Open:

```
http://<EC2-PUBLIC-IP>:5000
```
<img width="1919" height="868" alt="Screenshot 2026-04-20 223218" src="https://github.com/user-attachments/assets/c9a7c0f7-d89f-435c-9ff0-918f26736fff" />

---

# Important Notes ⚠️

* Stop EC2 when not in use to avoid charges
* Ensure Lambda successfully writes to `processed/`
* Ensure Glue crawler creates table before Athena queries

---

# Final Architecture Summary

```
S3 (raw)
   ↓
Lambda (processing)
   ↓
S3 (processed)
   ↓
Glue Crawler → orders_db
   ↓
Athena Queries
   ↓
S3 (enriched)
   ↓
EC2 Dashboard (Flask)
```

---
