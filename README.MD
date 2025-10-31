Overview
This repository provides a solution for extracting data from SAP systems, transforming it using AWS Glue, and loading it into Snowflake for analytics and reporting. The integration enables organizations to leverage operational SAP data for advanced analytics in Snowflake's high-performance data warehouse environment.

Thank you so much, Ashutosh for joining for snowflake world tour at Amsterdam. You supported the strong partnership that we have with snowflake and interacted with the audience by talking about the various topics and demonstrated the subject matter expertise that you have.

Cost
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
