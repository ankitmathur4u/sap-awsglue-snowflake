Overview
This repository provides a solution for extracting data from SAP systems, transforming it using AWS Glue, and loading it into Snowflake for analytics and reporting. The integration enables organizations to leverage operational SAP data for advanced analytics in Snowflake's high-performance data warehouse environment.

Thank you so much, Ashutosh for joining for snowflake world tour at Amsterdam. You supported the strong partnership that we have with snowflake and interacted with the audience by talking about the various topics and demonstrated the subject matter expertise that you have.

*Cost*
The implementation of this solution involves costs from multiple AWS services and Snowflake:

AWS Glue: $0.44 per DPU-Hour for ETL jobs (minimum 2 DPUs per job)
Amazon S3: $0.023 per GB for storage, plus data transfer costs
AWS Secrets Manager: $0.40 per secret per month, plus $0.05 per 10,000 API calls
Snowflake: Costs vary based on chosen edition, warehouse size, and storage usage
Data Transfer: Costs for moving data between AWS and Snowflake
Estimated monthly cost for a typical implementation with daily data synchronization:

Small scale (< 100GB): $150-$300 per month
Medium scale (100GB-1TB): $300-$800 per month
Large scale (> 1TB): $800+ per month
Prerequisites
AWS Account with permissions to create and manage:
AWS Glue resources (jobs, connections, development endpoints)
IAM roles and policies
Amazon S3 buckets
AWS Secrets Manager secrets
CloudWatch logs
SAP System with:
OData services enabled or
SAP HANA database access
Appropriate user credentials with read permissions
Snowflake Account with:
Database and schema created
Warehouse configured
User with appropriate privileges
Development Tools:
AWS CLI installed and configured
Python 3.7 or higher
Git client
Operating System
This solution can be deployed and operated from the following operating systems:

Linux (Amazon Linux 2, Ubuntu 18.04+, RHEL 7+)
Windows (Windows 10, Windows Server 2016+)
macOS (10.14 Mojave or newer)
AWS Glue jobs run on a managed Apache Spark environment independent of the operating system used for deployment.

Deployment Steps
Deployment Steps
Clone the repository

git clone https://github.com/sfc-gh-drichert/sap-awsglue-snowflake.git
cd sap-awsglue-snowflake



Configure AWS credentials

aws configure



Create an S3 bucket for scripts and staging

aws s3 mb s3://your-bucket-name



Store SAP and Snowflake credentials in AWS Secrets Manager

aws secretsmanager create-secret \
  --name sap-snowflake-credentials \
  --description "Credentials for SAP and Snowflake" \
  --secret-string '{"sap_username":"your-sap-user","sap_password":"your-sap-password","snowflake_user":"your-sf-user","snowflake_password":"your-sf-password"}'



Create AWS Glue connections

For SAP:

aws glue create-connection --cli-input-json file://config/sap-connection.json



For Snowflake:

aws glue create-connection --cli-input-json file://config/snowflake-connection.json



Upload ETL scripts to S3

aws s3 cp scripts/sap_to_snowflake.py s3://your-bucket-name/scripts/



Create AWS Glue job

aws glue create-job \
  --name sap-to-snowflake-etl \
  --role AWSGlueServiceRole-ETL \
  --command "Name=glueetl,ScriptLocation=s3://your-bucket-name/scripts/sap_to_snowflake.py" \
  --connections "Connections=sap-connection,snowflake-connection" \
  --default-arguments '{
    "--secret_id":"sap-snowflake-credentials",
    "--region":"us-east-1",
    "--sap_table":"SAP_TABLE_NAME",
    "--snowflake_database":"MY_DATABASE",
    "--snowflake_schema":"MY_SCHEMA",
    "--snowflake_warehouse":"MY_WAREHOUSE",
    "--snowflake_table":"MY_TARGET_TABLE"
  }'



Deployment Validation
Verify AWS Glue connections

aws glue test-connection --name sap-connection
aws glue test-connection --name snowflake-connection



Validate AWS Glue job configuration

aws glue get-job --job-name sap-to-snowflake-etl



Check IAM permissions

aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::account-id:role/AWSGlueServiceRole-ETL \
  --action-names glue:StartJobRun s3:GetObject secretsmanager:GetSecretValue



Run a test job with limited data

aws glue start-job-run \
  --job-name sap-to-snowflake-etl \
  --arguments '{"--limit":"10"}'



Running the Guidance
Manual Execution
Run the ETL job

aws glue start-job-run --job-name sap-to-snowflake-etl



Monitor job execution

aws glue get-job-run --job-name sap-to-snowflake-etl --run-id <job-run-id>



View CloudWatch logs

aws logs get-log-events \
  --log-group-name /aws-glue/jobs/output \
  --log-stream-name <job-run-id>



Scheduled Execution
Create a schedule for the ETL job

aws glue create-trigger \
  --name daily-sap-to-snowflake \
  --type SCHEDULED \
  --schedule "cron(0 2 * * ? *)" \
  --actions JobName=sap-to-snowflake-etl



Enable the trigger

aws glue start-trigger --name daily-sap-to-snowflake



Next Steps
Implement incremental loading

Modify the ETL script to track and process only new or changed records
Add watermark tracking to optimize data transfer
Enhance data quality checks

Add data validation rules in AWS Glue
Implement error handling and notification mechanisms
Set up monitoring and alerting

Create CloudWatch alarms for job failures
Configure SNS notifications for critical events
Extend to additional SAP data sources

Add support for other SAP tables or OData services
Implement a metadata-driven approach for flexible table mappings
Optimize Snowflake performance

Implement appropriate clustering keys
Configure optimal warehouse sizing for different workloads
Cleanup
To avoid incurring charges, delete the resources when no longer needed:

Delete AWS Glue resources

aws glue delete-trigger --name daily-sap-to-snowflake
aws glue delete-job --job-name sap-to-snowflake-etl
aws glue delete-connection --connection-name sap-connection
aws glue delete-connection --connection-name snowflake-connection



Delete AWS Secrets Manager secret

aws secretsmanager delete-secret \
  --secret-id sap-snowflake-credentials \
  --recovery-window-in-days 7



Delete S3 bucket and contents

aws s3 rm s3://your-bucket-name --recursive
aws s3 rb s3://your-bucket-name



Remove IAM roles (if created specifically for this project)

aws iam delete-role --role-name AWSGlueServiceRole-ETL



Notices
Legal Disclaimer
This sample code is provided for educational purposes only. It is not intended for production use without proper review and testing. Use at your own risk.

Trademarks
SAP® is a registered trademark of SAP SE
AWS® and Amazon Web Services® are registered trademarks of Amazon.com, Inc.
Snowflake® is a registered trademark of Snowflake Inc.
License
This project is licensed under the MIT License - see the LICENSE file for details.

Security
If you discover a potential security issue, please notify AWS Security via the AWS Vulnerability Reporting Service. Do not create public GitHub issues or pull requests for security vulnerabilities.
