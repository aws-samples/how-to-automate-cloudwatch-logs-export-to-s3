# CloudWatch-Logs-Automation

Overview

This solution provides automated export of CloudWatch Logs to an S3 bucket with enhanced security features including Object Lock retention and KMS encryption. It uses AWS CloudFormation to deploy the required infrastructure and a Lambda function to manage the exprt process.

The solution address common challenges such as long-term retention for compliance, cost-effective storage of historical data, secure handling of sensitive information, scalability across multiple log groups, and ease of access for analysis. By automating daily exports, it reduces operational overhead while ensuring systematic archiving in S3 with maintained data integrity and security.

![image](https://github.com/user-attachments/assets/93a03949-a270-4616-9d94-6a8a999cb6d8)


# Features:
- Automated export of CloudWatch Logs to S3
- S3 Object Lock for compliance and data retention
- KMS encryption for data at rest
- Optional VPC deployment support
- Configurable retention periods based on environment
- Email alerting system
- Supports multiple CloudWatch Log Groups
- Bucket versioning enabled
- Complete public access blocking
- Lifecycle management for exported logs
Prerequisites
- AWS Account with appropriate permissions
- Valid email address for alerts
- If deploying in VPC: Valid VPC ID, Subnet IDs, and Security Group ID

# Parameters:

Required Parameters:
- AlertEmail: Email address for receiving alerts
- CloudWatchLogGroups: Comma-separated list of CloudWatch Log Groups to export
- Name: Resource name prefix (Default: cwlogs-export)
- Environment: Deployment environment (development/staging-qa/production)
- ObjectLockMode: S3 Object Lock retention mode (GOVERNANCE/COMPLIANCE)
- ObjectLockRetentionDays: Retention period in days (1-3650)
- DeployInVPC: Whether to deploy Lambda in VPC (true/false)
Optional Parameters
- VpcId: VPC ID for Lambda deployment
- SubnetIds: Comma-separated list of Subnet IDs
- SecurityGroupId: Security Group ID for Lambda

# Environment-Specific Configurations:

Development:
- Log Retention: 60 days
Staging/QA:
- Log Retention: 90 days
Production:
- Log Retention: 365 days
Security Features
1.	S3 Bucket:
- Object Lock enabled
- Versioning enabled
- Public access blocked
- KMS encryption
- Bucket policy with principle of least privilege
2.	KMS:
- Automatic key rotation
- Restricted key usage
- Service-specific permissions
3.	IAM:
- Role-based access control
- Minimal required permissions
- Service-specific policies

Resource Components:
1.	S3 Bucket for log storage
2.	KMS key for encryption
3.	Lambda function for export automation
4.	IAM roles and policies
5.	S3 bucket policies
6.	KMS key policies

# Deployment Instructions:

1.	Prepare Parameters:
- Identify CloudWatch Log Groups to export
- Determine Object Lock retention requirements
- Configure VPC settings (if required)
2.	Deploy CloudFormation Stack:
aws cloudformation create-stack \
  --stack-name <stack-name> \
  --template-body file://cloudwatchlogs-to-s3-automation.yaml \
  --parameters \
    ParameterKey=Name,ParameterValue=<name> \
    ParameterKey=Environment,ParameterValue=<environment> \
    ParameterKey=AlertEmail,ParameterValue=<email> \
    ParameterKey=CloudWatchLogGroups,ParameterValue=<log-groups>
3.	Verify Deployment:
- Confirm stack creation completion
- Verify S3 bucket creation
- Check Lambda function deployment
- Validate IAM roles and policies

# Maintenance and Operations:

Monitoring:
- CloudWatch Logs for Lambda function
- S3 bucket metrics
- Export task status
Backup and Recovery
- S3 versioning enabled
- Object Lock protection
- Retention policies enforced

Troubleshooting:
1.	Lambda Function Issues:
- Check CloudWatch Logs
- Verify VPC connectivity (if deployed in VPC)
- Validate IAM permissions
2.	Export Failures:
- Verify S3 bucket permissions
- Check KMS key access
- Validate CloudWatch Log Group existence

Limitations:
- Maximum export task duration: 15 minutes
- Region-specific deployment
- Object Lock mode cannot be changed after bucket creation
- VPC deployment requires existing network infrastructure

Best Practices:
1.	Regular monitoring of export tasks
2.	Periodic review of retention policies
3.	Maintain proper subnet configuration for VPC deployment
4.	Monitor S3 storage costs
5.	Regular validation of email notifications

Clean Up:
To remove the solution:
1.	Empty the S3 bucket (note: Object Lock may prevent immediate deletion)
2.	Delete the CloudFormation stack
3.	Verify resource deletion
4.	Check for any remaining exported logs

- Note: The S3 bucket has a DeletionPolicy of "Retain" to prevent accidental data loss. Manual deletion may be required after stack removal.
