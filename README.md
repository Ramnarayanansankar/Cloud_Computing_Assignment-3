# AWS Data Processing Pipeline

This repository details the end-to-end serverless data processing pipeline created on AWS. The process involved ingesting raw data into S3, using a Lambda function for automated processing, cataloging the processed data with AWS Glue, running analytical queries with Amazon Athena, and visualizing the results on a dynamic webpage hosted on an EC2 instance.
Each section below includes an explanation of the approach and a screenshot.

## 1. Amazon S3 Bucket Structure 🪣

### Approach
Created a central Amazon S3 bucket to serve as the foundation for the entire data pipeline. To organize the workflow, established three distinct folders within this bucket:

* **`raw/`**: The folder acts as the ingestion point. All input CSV files are uploaded here.
* **`processed/`**: The folder stores the clean, filtered data output by the Lambda function after it processes the files from the `raw/` folder.
* **`enriched/`**: The folder is the designated output location for Amazon Athena. It stores the `.csv` files generated as results from the SQL queries.

### Output Screenshot: 
<img width="780" height="620" alt="S3BucketStructure" src="https://github.com/user-attachments/assets/1c25c723-296d-413a-8b51-38394415cae7" />

---
## 2. IAM Roles and Permissions 🔐

### Approach
To ensure secure, permission-based interaction between AWS services, created three separate IAM roles:
1.  **Lambda Execution Role (`Lambda-S3-Processing-Role`)**: The role is attached to the Lambda function. It grants permissions for Lambda to read objects from the `raw/` S3 folder, write new objects to the `processed/` S3 folder, and write logs to CloudWatch. `AWSLambdaBasicExecutionRole` and `AmazonS3FullAccess` are the policies granted for this role.
2.  **Glue Service Role (`Glue-S3-Crawler-Role`)**: This role is used by the AWS Glue Crawler. It provides permissions to access the `processed/` data in S3. `AWSGlueServiceRole`, `AWSGlueConsoleFullAccess` and `AmazonS3FullAccess` are the policies granted to manage the creation of databases and tables.
3.  **EC2 Instance Profile (`EC2-Athena-Dashboard-Role`)**: The role is attached to the EC2 instance. It grants the instance permissions to run queries with `AmazonAthenaFullAccess` and to read/write query results from S3 using `AmazonS3FullAccess`.

### Output Screenshot:
<img width="780" height="620" alt="IamRolesCreated" src="https://github.com/user-attachments/assets/8c1a970d-7c0b-403d-bfb7-d3c1d80d2b88" />

---
## 3. AWS Lambda Function ⚙️

### Approach
Created a serverless compute function using AWS Lambda to automate the data cleaning process. The function, named `FilterAndProcessOrders`, uses the Python 3.13 runtime. It is configured to use the existing `Lambda-S3-Processing-Role` created in the previous step. The function's code is designed to be triggered by an S3 event, read the uploaded CSV file, perform data filtering and transformations, and then save the cleaned data back to the `processed/` S3 folder.

### Output Screenshot:
<img width="780" height="620" alt="LambdaFunction" src="https://github.com/user-attachments/assets/1590b2a9-f537-4fcc-95e7-59992f02bb0e" />

---
## 4. S3 Trigger Configuration ⚡

### Approach
To make the pipeline event-driven, added an S3 trigger to the `FilterAndProcessOrders` Lambda function. This trigger is configured to monitor the S3 bucket for **All object create events**. To ensure the function only runs on relevant files, specified a **prefix** of `raw/` and a **suffix** of `.csv`. This configuration automatically invokes the Lambda function *only* when a new CSV file is uploaded to the `raw/` folder.

### Output Screenshot:
<img width="780" height="620" alt="S3TriggerInLambdaFunction" src="https://github.com/user-attachments/assets/6f0133ac-62df-4ef4-9fec-cc822923b135" />

---
## 5. Processed Data in S3

### Approach
After configuring the Lambda function and its S3 trigger, tested the pipeline by uploading the `Orders.csv` file to the `raw/` folder of the S3 bucket. The action successfully triggered the Lambda function, which executed its processing logic. As a result, a new, cleaned CSV file appeared in the `processed/` folder, confirming the automated workflow was successful.

### Output Screenshot: 
<img width="780" height="620" alt="S3BucketProcessedFolder" src="https://github.com/user-attachments/assets/9a00871b-2d8b-4d96-9a9b-1a99e018e1cb" />

---
## 6. AWS Glue Crawler 🕸️

### Approach
To make the processed data queryable by Athena, used the AWS Glue to create a data catalog. Created a new crawler named `orders_processed_crawler` and pointed it to the `processed/` S3 folder. Assigned the `Glue-S3-Crawler-Role` to it. The crawler was configured to output its findings to a new database named `orders_db`. Upon running the crawler, it successfully scanned the processed CSV, inferred its schema, and created a new table within the `orders_db` database.

### Output Screenshot:
<img width="780" height="620" alt="CloudWatchLogs" src="https://github.com/user-attachments/assets/78607b07-da9c-49df-97e1-c486186cb705" />

---

## 7. Amazon Athena Query Results in S3

### Approach
With the data cataloged, navigated to the Amazon Athena service. Set the query results location to the `enriched/` S3 folder. Using the Trino SQL editor, I successfully ran analytical queries against the `orders_db` database and its table. The web application hosted on EC2 also runs these queries, and the results of each query are automatically saved as new CSV and metadata files in the `enriched/` S3 folder.

### Output Screenshot:
<img width="780" height="620" alt="S3BucketEnrichedFolder" src="https://github.com/user-attachments/assets/cf98412d-a18b-4432-88c1-483bfda72a3e" />

---

## 8. EC2 Webpage Visualization 🖥️

### Approach
To display the query results, launched a `t3.micro` EC2 instance. Used the `EC2-Athena-Dashboard-Role` and configured the security group to allow SSH (port 22) and HTTP access on port 5000. After connecting via SSH, installed Python, Flask, and Boto3. Configured and ran the `app.py` script, which starts a Flask web server. The server executes the Athena queries in real-time and renders the results in a web browser.

### Output Screenshots:
<img width="780" height="620" alt="FinalWebpagePhoto1" src="https://github.com/user-attachments/assets/9ce5477d-bac4-4610-a6d6-318e33b899bd" /> <br >

<img width="780" height="620" alt="FinalWebpagePhoto2" src="https://github.com/user-attachments/assets/08d4875b-c26e-4ce7-89f3-1fb18dd816c3" />
