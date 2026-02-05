# COVER: CISA Observed Vulnerability Exploitation Report

COVER is a Cyber Threat Intelligence (CTI) tool that compares your environment's software vendors (listed in `software.txt`) against CISA's Known Exploited Vulnerabilities (KEV) database. It helps security teams prioritize patching by identifying actively exploited vulnerabilities.

## Key Features
- Automated scans against CISA's KEV catalog.
- Generates reports in JSON and plain text formats.
- Stores reports in an AWS S3 bucket and sends them via email.
- Low-cost, scheduled execution via AWS Lambda.
- Customizable vendor list tailored to your environment.

## Prerequisites
Before deploying, ensure you have:

### AWS Account and Permissions
- An active AWS account.
- IAM permissions for:
  - CloudFormation: `cloudformation:*`.
  - IAM: `iam:CreateRole`, `iam:AttachRolePolicy`, `iam:PutRolePolicy`.
  - Lambda: `lambda:CreateFunction`, `lambda:UpdateFunctionCode`.
  - S3: `s3:CreateBucket`, `s3:PutObject`, `s3:GetObject`.
  - SES: `ses:VerifyEmailIdentity`, `ses:SendEmail`.
- **Tip:** Use an IAM role with `AdministratorAccess` temporarily for setup. Include `--capabilities CAPABILITY_IAM` in CLI commands.

### AWS CLI
- AWS CLI v2 installed (verify with `aws --version`).
- Configured with access keys, region (e.g., `us-east-1`), and output format (`aws configure`).

### SES Email Verification
- Verify sender and recipient emails in the SES console (under Identity Management > Email Addresses).
- If in sandbox mode, verify recipients or request production access to send to unverified addresses.

### S3 Code Bucket
- An existing S3 bucket for storing `cover_lambda.zip`.

**Estimated Costs:** Low—Lambda runs (~$0.000001 per request), S3 storage (~$0.023/GB/month), and SES emails (~$0.10 per 1,000).

## Deployment Instructions
1. Download `COVER-Deploy.yaml`, `cover_lambda.zip`, and `software.txt` from the repository.
2. Edit `software.txt` in a text editor: Add your in-use vendors (one per line) and remove irrelevant ones.
3. Upload `cover_lambda.zip` to your S3 code bucket:
   ```bash
   aws s3 cp cover_lambda.zip s3://your-code-bucket/cover_lambda.zip

   aws cloudformation create-stack --stack-name CoverStack --template-body file://COVER-Deploy.yaml --parameters \
  ParameterKey=CodeS3Bucket,ParameterValue=your-code-bucket \
  ParameterKey=CodeS3Key,ParameterValue=cover_lambda.zip \
  ParameterKey=S3BucketName,ParameterValue=your-cover-bucket-name \
  ParameterKey=EmailRecipient,ParameterValue=recipient@example.com \
  ParameterKey=EmailSender,ParameterValue=sender@example.com \
  --capabilities CAPABILITY_IAM

  aws s3 cp software.txt s3://your-cover-bucket-name/software.txt

  
