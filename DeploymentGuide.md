Deployment Guide
Option 1: AWS Management Console Deployment
Step 1: Launch Stack
Sign in to AWS Management Console

Open CloudFormation

Click "Create stack" (with new resources)

Step 2: Specify Template
Select "Upload a template file"

Upload cloudwatchlogs-to-s3-automation.yaml

Click "Next"

Step 3: Configure Stack Parameters
Stack Name and Basic Settings:

Stack name: cwlogs-export-stack
Name: cwlogs-export
Environment: [development|staging-qa|production]


Log Export Configuration:

CloudWatchLogGroups: /aws/lambda/function1,/aws/lambda/function2
AlertEmail: your.email@domain.com


Security Settings:

ObjectLockMode: [GOVERNANCE|COMPLIANCE]
ObjectLockRetentionDays: [1-3650]


VPC Configuration (if required):

DeployInVPC: [true|false]
VpcId: vpc-xxxxxxxx
SubnetIds: subnet-xxxxxx,subnet-xxxxxx
SecurityGroupId: sg-xxxxxxxx


Step 4: Stack Options
Configure stack options (optional):

Tags

Permissions

Stack failure options

Advanced options

Click "Next"

Step 5: Review and Create
Review all parameters

Check acknowledgment boxes for:

IAM resource creation

Custom names

Click "Create stack"

Step 6: Monitor Deployment
Watch "Events" tab for progress

Wait for CREATE_COMPLETE status

Check "Outputs" tab for resource information

Option 2: AWS CLI Deployment
Step 1: Prepare Parameters File [1]
Create parameters.json:

[
  {
    "ParameterKey": "Name",
    "ParameterValue": "cwlogs-export"
  },
  {
    "ParameterKey": "Environment",
    "ParameterValue": "production"
  },
  {
    "ParameterKey": "AlertEmail",
    "ParameterValue": "your.email@domain.com"
  },
  {
    "ParameterKey": "CloudWatchLogGroups",
    "ParameterValue": "/aws/lambda/function1,/aws/lambda/function2"
  },
  {
    "ParameterKey": "ObjectLockMode",
    "ParameterValue": "GOVERNANCE"
  },
  {
    "ParameterKey": "ObjectLockRetentionDays",
    "ParameterValue": "365"
  },
  {
    "ParameterKey": "DeployInVPC",
    "ParameterValue": "false"
  }
]


Step 2: Deploy Stack
Basic deployment:

aws cloudformation create-stack \
  --stack-name cwlogs-export-stack \
  --template-body file://cloudwatchlogs-to-s3-automation.yaml \
  --parameters file://parameters.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM


VPC deployment:

aws cloudformation create-stack \
  --stack-name cwlogs-export-stack \
  --template-body file://cloudwatchlogs-to-s3-automation.yaml \
  --parameters file://parameters.json \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
  --parameters \
      ParameterKey=DeployInVPC,ParameterValue=true \
      ParameterKey=VpcId,ParameterValue=vpc-xxxxxxxx \
      ParameterKey=SubnetIds,ParameterValue=\"subnet-xxxxx,subnet-xxxxx\" \
      ParameterKey=SecurityGroupId,ParameterValue=sg-xxxxxxxx


Step 3: Monitor Deployment
Check stack status:

aws cloudformation describe-stacks \
  --stack-name cwlogs-export-stack \
  --query 'Stacks[0].StackStatus'


List stack events:

aws cloudformation describe-stack-events \
  --stack-name cwlogs-export-stack


Get stack outputs:

aws cloudformation describe-stacks \
  --stack-name cwlogs-export-stack \
  --query 'Stacks[0].Outputs'


Step 4: Verify Resources
Check S3 bucket creation:

aws s3 ls | grep cwlogs-export


Verify Lambda function:

aws lambda list-functions \
  --query 'Functions[?contains(FunctionName, `cwlogs-export`)]'


Step 5: Post-Deployment
Confirm SNS subscription email

Test initial log export

Monitor CloudWatch Logs for Lambda execution

Troubleshooting Commands
Check stack failure reasons:

aws cloudformation describe-stack-events \
  --stack-name cwlogs-export-stack \
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`]'


View Lambda logs:

aws logs get-log-events \
  --log-group-name /aws/lambda/cwlogs-export-function \
  --log-stream-name $(aws logs describe-log-streams \
    --log-group-name /aws/lambda/cwlogs-export-function \
    --order-by LastEventTime \
    --descending \
    --limit 1 \
    --query 'logStreams[0].logStreamName' \
    --output text)

Delete stack if needed:

aws cloudformation delete-stack \
  --stack-name cwlogs-export-stack
