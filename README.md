# Serverless Flight Data Processing Pipeline with AWS

Description: Unlock valuable insights from flight data. This repository contains the source code for a serverless data pipeline that ingests flight data in CSV format from Amazon S3, processes it using AWS Glue's powerful data transformation capabilities, and loads it into a Redshift data warehouse for in-depth analysis and business intelligence.
 
## Architecture

![architecture](https://github.com/diegovillatoromx/s3-redshift-pipeline-flights/blob/main/architecture.png)

## Data Pipeline Workflow

### 1. Data Ingestion
- **Data Source**: Flight data, which includes information such as schedules, routes, passenger numbers, and other operational details, is generated daily.
- **Upload to S3**: This data is automatically uploaded to an Amazon S3 bucket. The files are organized in a Hive-style partitioning scheme (e.g., `/year/month/day/`), which facilitates future queries and optimizes performance.
- **Automatic Detection (EventBridge)**: Amazon EventBridge monitors the S3 bucket for the arrival of new files. When a new file is detected, EventBridge triggers a predefined rule that initiates the next step in the workflow.

### 2. Workflow Coordination with Step Functions
- **Workflow Initialization (Step Functions)**: AWS Step Functions is responsible for coordinating and managing the entire workflow. When the EventBridge rule is triggered, Step Functions starts a series of automated tasks.
- **Data Scanning (Glue Crawler)**: The first step in the workflow is to run an AWS Glue Crawler. This crawler explores the newly arrived data in S3, updating the Glue Data Catalog with the new data structure and schema.
- **Data Processing (Glue Job)**: Once the crawler finishes and if the process was successful, Step Functions invokes a Glue Job that processes the data. This can include tasks such as data cleansing, transformations, aggregations, or data enrichment based on business requirements.

### 3. Loading into Redshift
- **Transfer to Redshift**: After the data has been successfully processed by the Glue Job, it is loaded into Amazon Redshift, a data warehouse where further analysis will take place.
- **Error Handling and Notifications (SNS)**:
  - **Glue Job Failure**: If the Glue Job fails at any point, Step Functions uses Amazon SNS (Simple Notification Service) to send a failure notification to administrators or the data team, indicating the need for intervention.
  - **Process Success**: If the entire process is successful, a notification of success is also sent via SNS.
