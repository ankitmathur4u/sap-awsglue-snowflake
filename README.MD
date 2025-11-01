# SAP to Snowflake Data Integration Using AWS Glue

This guidance demonstrates how to build an automated data pipeline that extracts data from SAP systems, transforms it using AWS Glue, and loads it into Snowflake for analytics and reporting purposes.

## Table of Contents (required)

1. [Overview](#overview-required)
    - [Cost](#cost)
2. [Prerequisites](#prerequisites-required)
    - [Operating System](#operating-system-required)
3. [Deployment Steps](#deployment-steps-required)
4. [Deployment Validation](#deployment-validation-required)
5. [Running the Guidance](#running-the-guidance-required)
6. [Next Steps](#next-steps-required)
7. [Cleanup](#cleanup-required)
8. [Notices](#notices-required)

## Overview (required)

This guidance demonstrates how to build a scalable data pipeline that extracts operational data from SAP systems and loads it into Snowflake for analytics. Using AWS Glue as the ETL service, the solution automates the transformation and movement of data while maintaining security and performance. Organizations can leverage this integration to unlock business insights from their SAP data through Snowflake's high-performance analytics capabilities, enabling data-driven decision making across the enterprise.

### Cost ( required )

Core Infrastructure Components

AWS Glue (Data Processing)

Pricing: $0.44 per DPU-Hour, billed per second with 1-minute minimum
Estimated Usage: For daily ETL jobs processing SAP data
Assuming 2-hour daily job with 10 DPUs: 10 × 2 × $0.44 = $8.80/day
Monthly Cost: ~$264
Amazon S3 (Data Storage)

Raw Data Storage: $0.023 per GB/month for Standard storage
Estimated Usage:
Initial data load: 100GB-1TB
Daily incremental: 1-10GB
Monthly Cost: $50-300 (depending on data volume)
Snowflake Integration Costs

Data Transfer: Between AWS and Snowflake
Inter-region transfer: $0.02-0.09 per GB
Estimated monthly: $50-200
Snowflake Compute: Based on usage patterns
Standard edition: Base pricing
Enterprise/Business Critical: 1.5x-3x higher for enhanced security
Total Monthly Cost Estimate

Small Implementation (Development/Testing)

AWS Glue: $264
S3 Storage: $50
SAP Infrastructure: $500
Data Transfer: $50
Total: ~$864/month
Medium Implementation (Production)

AWS Glue: $500 (more frequent jobs)
S3 Storage: $150
SAP Infrastructure: $1,200
Data Transfer: $100
Snowflake Integration: $200
Total: ~$2,150/month
Large Implementation (Enterprise)

AWS Glue: $1,000+ (multiple daily jobs, larger DPUs)
S3 Storage: $300
SAP Infrastructure: $2,000+
Data Transfer: $200
Additional AWS services (VPC, CloudWatch, etc.): $100
Total: ~$3,600+/month

### Sample Cost Table ( required )
  
| Component | Description | Pricing Model | Small (Dev/Test) | Medium (Production) | Large (Enterprise) |
|-----------|-------------|---------------|------------------|--------------------|--------------------|
| AWS Glue | Data Processing | $0.44 per DPU-Hour | $264 | $500 | $1,000+ |
| Amazon S3 | Data Storage | $0.023 per GB/month | $50 | $150 | $300 |
| Data Transfer | AWS-Snowflake | $0.02-0.09 per GB | $50 | $100 | $200 |
| Snowflake Integration | Additional Services | Variable | - | $200 | Variable |
| Additional AWS Services | VPC, CloudWatch, etc. | Variable | - | - | $100 |
| **TOTAL MONTHLY COST** | **All Components** | **-** | **~$364** | **~$950** | **~$3,600+** |


## Prerequisites (required)
SAP to Snowflake Data Integration Using AWS Glue

Prerequisites

- AWS Account with permissions to create and manage:
  - AWS Glue resources (jobs, connections, development endpoints)
  - IAM roles and policies
  - Amazon S3 buckets
  - AWS Secrets Manager secrets
  - CloudWatch logs

- SAP System with:
  - OData services enabled or
  - SAP HANA database access
  - Appropriate user credentials with read permissions

- Snowflake Account with:
  - Database and schema created
  - Warehouse configured
  - User with appropriate privileges

- Development Tools:
  - AWS CLI installed and configured
  - Python 3.7 or higher
  - Git client

### Operating System (required)

This solution can be deployed and operated from the following operating systems:

- Linux (Amazon Linux 2, Ubuntu 18.04+, RHEL 7+)
- Windows (Windows 10, Windows Server 2016+)
- macOS (10.14 Mojave or newer)

AWS Glue jobs run on a managed Apache Spark environment independent of the operating system used for deployment.

## Deployment Steps (required)

1. Clone repository and setup environment

    git clone https://github.com/sfc-gh-drichert/sap-awsglue-snowflake.git && cd sap-awsglue-snowflake && python -m venv venv && source venv/bin/activate && pip install -r requirements.txt


2. Configure AWS credentials

    aws configure  # Enter your AWS access key, secret key, region, and output format

3. Create S3 bucket and store name

    export BUCKET_NAME=sap-snowflake-integration-$(aws sts get-caller-identity --query Account --output text) && aws s3 mb s3://$BUCKET_NAME

4. Create IAM role for Glue

    aws iam create-role --role-name GlueETLRole-SAP-Snowflake --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"glue.amazonaws.com"},"Action":"sts:AssumeRole"}]}' && aws iam attach-role-policy --role-name GlueETLRole-SAP-Snowflake --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole

5. Store credentials in Secrets Manager

    aws secretsmanager create-secret --name sap-snowflake-credentials --secret-string '{"sap_username":"your-sap-user","sap_password":"your-sap-pass","snowflake_user":"your-sf-user","snowflake_password":"your-sf-pass"}'

6. Create AWS Glue connections

    aws glue create-connection --cli-input-json file://config/sap-connection.json && aws glue create-connection --cli-input-json file://config/snowflake-connection.json

7. Upload ETL script to S3

    aws s3 cp scripts/sap_to_snowflake.py s3://$BUCKET_NAME/scripts/

8. Create and start AWS Glue job

    aws glue create-job --name sap-to-snowflake-etl --role GlueETLRole-SAP-Snowflake --command "Name=glueetl,ScriptLocation=s3://$BUCKET_NAME/scripts/sap_to_snowflake.py" --connections "Connections=sap-connection,snowflake-connection"

9. Run test job and capture ID

    export JOB_RUN_ID=$(aws glue start-job-run --job-name sap-to-snowflake-etl --arguments '{"--limit":"10"}' --query JobRunId --output text)

10. Monitor job status

aws glue get-job-run --job-name sap-to-snowflake-etl --run-id $JOB_RUN_ID

## Deployment Validation (required)

1. Verify S3 bucket and ETL script

    aws s3 ls s3://$BUCKET_NAME/scripts/sap_to_snowflake.py

2. Check IAM role configuration

    aws iam get-role --role-name GlueETLRole-SAP-Snowflake --query 'Role.[RoleName, Arn]' --output text

3. Validate AWS Secrets Manager

    aws secretsmanager describe-secret --secret-id sap-snowflake-credentials --query 'ARN' --output text

4. Test AWS Glue connections

    aws glue test-connection --name sap-connection
    aws glue test-connection --name snowflake-connection

5. Verify AWS Glue job setup

    aws glue get-job --job-name sap-to-snowflake-etl --query 'Job.[Name, Role, Command.ScriptLocation]' --output text

6. Run test job and capture status

    JOB_RUN_ID=$(aws glue start-job-run --job-name sap-to-snowflake-etl --arguments '{"--limit":"10"}' --query 'JobRunId' --output text)
    aws glue get-job-run --job-name sap-to-snowflake-etl --run-id $JOB_RUN_ID --query 'JobRun.JobRunState' --output text

7. Check CloudWatch logs for errors

    aws logs get-log-events --log-group-name "/aws-glue/jobs/output" --log-stream-name $JOB_RUN_ID --limit 5

8. Verify data in Snowflake (Connect to Snowflake and run):

    SELECT COUNT(*) FROM YOUR_SNOWFLAKE_DB.YOUR_SNOWFLAKE_SCHEMA.YOUR_SNOWFLAKE_TABLE;

9. Generate quick validation report

    echo "Deployment Status:" && \
    echo "S3: $(aws s3 ls s3://$BUCKET_NAME &>/dev/null && echo '✅' || echo '❌')" && \
    echo "IAM: $(aws iam get-role --role-name GlueETLRole-SAP-Snowflake &>/dev/null && echo '✅' || echo '❌')" && \
    echo "Job: $(aws glue get-job --job-name sap-to-snowflake-etl &>/dev/null && echo '✅' || echo '❌')" && \
    echo "Last Run: $(aws glue get-job-run --job-name sap-to-snowflake-etl --run-id $JOB_RUN_ID --query JobRun.JobRunState --output text)"

## Running the Guidance (required)

Input Parameters

    {
      "sap_table": "VBAK",  // SAP Sales Document Header table
      "limit": 10,          // Process 10 records for testing
      "target_table": "SALES_ORDERS"
    }


Run ETL Job

    # Start job with sample parameters
    JOB_RUN_ID=$(aws glue start-job-run \
      --job-name sap-to-snowflake-etl \
      --arguments '{
        "--sap_table":"VBAK",
        "--limit":"10",
        "--snowflake_table":"SALES_ORDERS"
      }' --query JobRunId --output text)

Monitor execution
    aws glue get-job-run --job-name sap-to-snowflake-etl --run-id $JOB_RUN_ID

Expected Output

    Job Run Status: SUCCEEDED
    Records Processed: 10
    Duration: 2m 15s

Verify Results in Snowflake

    -- Connect to Snowflake and run:
    SELECT * FROM SALES_ORDERS LIMIT 5;

Success Criteria

    Job status shows "SUCCEEDED"
    Data appears in Snowflake table
    No errors in CloudWatch logs
    

## Next Steps (required)

1. Scale AWS Glue Workers: Adjust --number-of-workers and --worker-type based on data volume (e.g., increase workers for larger datasets)
2. Optimize Memory Settings: Modify --conf spark.driver.memory and --conf spark.executor.memory for better performance
3. Customize ETL Logic: Adapt sap_to_snowflake.py script to include specific data transformations, filters, or business rules
4. Enhanced Security: Add VPC endpoints, implement column-level encryption, or integrate with AWS KMS for additional security
5. Data Quality Checks: Include custom validation rules in the ETL script to ensure data integrity
6. Monitoring: Set up CloudWatch alarms for job failures, data quality issues, or performance metrics
7. Cost Optimization: Implement job bookmarking for incremental loads to reduce processing time and costs
8. Error Handling: Add custom error handling and notification mechanisms (e.g., SNS topics for job failures)
9. Performance Tuning: Adjust Snowflake warehouse size and implement table clustering based on query patterns
10. Scheduling: Modify the default cron expression (cron(0 2 * * ? *)) to match your business requirements



## Cleanup (required)
    Delete AWS Glue Resources


    1. Delete Glue trigger (if created)

        aws glue delete-trigger —name daily-sap-to-snowflake

    2. Delete Glue job

        aws glue delete-job —job-name sap-to-snowflake-etl

    3. Delete Glue connections

        aws glue delete-connection —connection-name sap-connection
        aws glue delete-connection —connection-name snowflake-connection

    4. Empty and Delete S3 Bucket

        Empty the bucket first (required before deletion)
        aws s3 rm s3://$BUCKET_NAME —recursive

    5. Delete the bucket
        
        aws s3 rb s3://$BUCKET_NAME

    6. Delete AWS Secrets Manager Secret

         Delete secret with recovery window

        aws secretsmanager delete-secret \
          —secret-id sap-snowflake-credentials \
          —recovery-window-in-days 7
    
        Or force delete without recovery window

        aws secretsmanager delete-secret \
        —secret-id sap-snowflake-credentials \
          —force-delete-without-recovery

    7. Remove IAM Role and Policies

        Detach policies

        aws iam detach-role-policy —role-name GlueETLRole-SAP-Snowflake \
          —policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole

    8. Delete role

        aws iam delete-role —role-name GlueETLRole-SAP-Snowflake


    9. Verify Resource Deletion


    10. Run validation checks to ensure cleanup

        echo "Cleanup Validation:" && \
        echo "S3: $(aws s3 ls s3://$BUCKET_NAME 2>&1 | grep -q 'NoSuchBucket' && echo '✅' || echo '❌')" && \
        echo "IAM: $(aws iam get-role —role-name GlueETLRole-SAP-Snowflake 2>&1 | grep -q 'NoSuchEntity' && echo '✅' || echo '❌')" && \
        echo "Job: $(aws glue get-job —job-name sap-to-snowflake-etl 2>&1 | grep -q 'EntityNotFoundException' && echo '✅' || echo '❌')“


## Notices ( required )

Include below mandatory legal disclaimer for Guidance

*Customers are responsible for making their own independent assessment of the information in this Guidance. This Guidance: (a) is for informational purposes only, (b) represents AWS current product offerings and practices, which are subject to change without notice, and (c) does not create any commitments or assurances from AWS and its affiliates, suppliers or licensors. AWS products or services are provided “as is” without warranties, representations, or conditions of any kind, whether express or implied. AWS responsibilities and liabilities to its customers are controlled by AWS agreements, and this Guidance is not part of, nor does it modify, any agreement between AWS and its customers.*


## Authors (optional)

Ankit Mathur, Abhijeet Jangam







