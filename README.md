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
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 10 53 00 PM" src="https://github.com/user-attachments/assets/9b406944-a3ff-4864-9df4-aa242e1245b6" />


---
## 2. IAM Roles and Permissions 🔐

### Approach
To ensure secure, permission-based interaction between AWS services, created three separate IAM roles:
1.  **Lambda Execution Role (`Lambda-S3-Processing-Role`)**: The role is attached to the Lambda function. It grants permissions for Lambda to read objects from the `raw/` S3 folder, write new objects to the `processed/` S3 folder, and write logs to CloudWatch. `AWSLambdaBasicExecutionRole` and `AmazonS3FullAccess` are the policies granted for this role.
2.  **Glue Service Role (`Glue-S3-Crawler-Role`)**: This role is used by the AWS Glue Crawler. It provides permissions to access the `processed/` data in S3. `AWSGlueServiceRole`, `AWSGlueConsoleFullAccess` and `AmazonS3FullAccess` are the policies granted to manage the creation of databases and tables.
3.  **EC2 Instance Profile (`EC2-Athena-Dashboard-Role`)**: The role is attached to the EC2 instance. It grants the instance permissions to run queries with `AmazonAthenaFullAccess` and to read/write query results from S3 using `AmazonS3FullAccess`.

### Output Screenshot:
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 10 55 44 PM" src="https://github.com/user-attachments/assets/ad44b419-f6ea-4940-92b9-2a93c68bd17c" />


---
## 3. AWS Lambda Function ⚙️

### Approach
Created a serverless compute function using AWS Lambda to automate the data cleaning process. The function, named `FilterAndProcessOrders`, uses the Python 3.13 runtime. It is configured to use the existing `Lambda-S3-Processing-Role` created in the previous step. The function's code is designed to be triggered by an S3 event, read the uploaded CSV file, perform data filtering and transformations, and then save the cleaned data back to the `processed/` S3 folder.

### Output Screenshot:
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 10 56 54 PM" src="https://github.com/user-attachments/assets/48ca749e-4d04-4823-97c4-c994c103b473" />

---
## 4. S3 Trigger Configuration ⚡

### Approach
To make the pipeline event-driven, added an S3 trigger to the `FilterAndProcessOrders` Lambda function. This trigger is configured to monitor the S3 bucket for **All object create events**. To ensure the function only runs on relevant files, specified a **prefix** of `raw/` and a **suffix** of `.csv`. This configuration automatically invokes the Lambda function *only* when a new CSV file is uploaded to the `raw/` folder.

### Output Screenshot:
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 10 58 00 PM" src="https://github.com/user-attachments/assets/59f38ae4-4ba1-4d66-866d-6172cc6cdd91" />


---
## 5. Processed Data in S3

### Approach
After configuring the Lambda function and its S3 trigger, tested the pipeline by uploading the `Orders.csv` file to the `raw/` folder of the S3 bucket. The action successfully triggered the Lambda function, which executed its processing logic. As a result, a new, cleaned CSV file appeared in the `processed/` folder, confirming the automated workflow was successful.

### Output Screenshot: 
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 10 58 46 PM" src="https://github.com/user-attachments/assets/7712d83a-42ba-47e1-acaa-4a4311f9f878" />


---
## 6. AWS Glue Crawler 🕸️

### Approach
To make the processed data queryable by Athena, used the AWS Glue to create a data catalog. Created a new crawler named `orders_processed_crawler` and pointed it to the `processed/` S3 folder. Assigned the `Glue-S3-Crawler-Role` to it. The crawler was configured to output its findings to a new database named `orders_db`. Upon running the crawler, it successfully scanned the processed CSV, inferred its schema, and created a new table within the `orders_db` database.

### Output Screenshot:
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 10 59 36 PM" src="https://github.com/user-attachments/assets/1e2dcb74-600b-4359-80fb-c391ad19fd9c" />


---

## 7. Amazon Athena Query Results in S3

### Approach
With the data cataloged, navigated to the Amazon Athena service. Set the query results location to the `enriched/` S3 folder. Using the Trino SQL editor, I successfully ran analytical queries against the `orders_db` database and its table. The web application hosted on EC2 also runs these queries, and the results of each query are automatically saved as new CSV and metadata files in the `enriched/` S3 folder.

### Output Screenshot:
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 11 00 38 PM" src="https://github.com/user-attachments/assets/7cdf70a2-7e20-4777-a840-6cc096fe756f" />


---

## 8. EC2 Webpage Visualization 🖥️

### Approach
To display the query results, launched a `t3.micro` EC2 instance. Used the `EC2-Athena-Dashboard-Role` and configured the security group to allow SSH (port 22) and HTTP access on port 5000. After connecting via SSH, installed Python, Flask, and Boto3. Configured and ran the `app.py` script, which starts a Flask web server. The server executes the Athena queries in real-time and renders the results in a web browser.

### Output Screenshots:
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 11 01 03 PM" src="https://github.com/user-attachments/assets/b6937542-4e46-4d5c-acd6-fb13c603b840" />
<img width="1440" height="900" alt="Screenshot 2025-11-12 at 11 01 14 PM" src="https://github.com/user-attachments/assets/1d7227c3-abfc-4a64-9405-62edd9e09142" />
<img width="1440" height="839" alt="Screenshot 2025-11-12 at 11 01 24 PM" src="https://github.com/user-attachments/assets/4d53be6e-e16d-46d9-baf5-9c85bfc0e987" />
<img width="1440" height="613" alt="Screenshot 2025-11-12 at 11 01 33 PM" src="https://github.com/user-attachments/assets/22ca4e0d-19ba-4927-97d8-809e9824c5d5" />

