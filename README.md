COVER (CISA Observed Vulnerability Exploitation Report)

The purpose of this tool is to compare a list of software vendors you have present in your environment (software.txt) against CISA's KEV (Known Exploited Vulnerabilities) database in order to help teams prioritize patching.

COVER is deployed using AWS CloudFormation via the AWS CLI. The COVER deployment creates a low-cost Lambda function that runs on the cadence you desire. Every time the COVER function runs, reports are sent to an AWS S3 bucket (Created in your environment) as well as emailed to the email address you provide upon deployment. These reports are in both JSON and plain text formats.

REQUIREMENTS
You will need the following prior to deployment of COVER:

  AWS Account and Permissions:
  * Active AWS Account
  * IAM Permissions:
    * Permissions for CloudFormation: cloudformation:* (to create/update stacks).
    * IAM: iam:CreateRole, iam:AttachRolePolicy, iam:PutRolePolicy (the template creates an execution role for Lambda).
    * Lambda: lambda:CreateFunction, lambda:UpdateFunctionCode.
    * S3: s3:CreateBucket, s3:PutObject, s3:GetObject.
    * SES: ses:VerifyEmailIdentity, ses:SendEmail (for verification and sending).

    * Use an IAM administrator role or a custom policy with these actions. If using the AWS CLI, attach the AdministratorAccess policy temporarily for setup. The --capabilities CAPABILITY_IAM flag in the CLI command acknowledges IAM resource        creation.

  AWS CLI:
  * I nstall AWS CLI v2: Required for CLI-based deployment (recommended for automation). Download and install from AWS CLI documentation. Verify with aws --version.
  * Configure AWS CLI: Run aws configure to set your Access Key ID, Secret Access Key, default region (e.g., us-east-1), and output format (e.g., json). Use IAM credentials with the permissions above.

  SES Email Verification:
  * Verified Identities: Before the Lambda can send emails, verify the sender (EMAIL_SENDER) and recipient (EMAIL_RECIPIENT) in Amazon SES (Simple Email Service).
    * Go to the SES console in your region.
    * Under "Identity Management" > "Email Addresses", create and verify each email (AWS sends a verification link to the inbox).
    * If your account is in SES sandbox mode (default for new accounts), you must also verify recipient addresses to receive emails. Request production access via the SES console to send to unverified addresses.
   
  Code Bucket:
  * An existing S3 bucket to store the cover_lambda.zip

INSTRUCTIONS
1. Download COVER-Deploy.yaml, cover_lambda.zip, and software.txt.
2. Using a the text editor of your choice, edit the software vendors in software.txt. Add vendors (one per line) that are in-use in your environment and remove any that are not.
3. Upload cover_lambda.zip to your AWS S3 Code Bucket.
  3a. aws s3 cp cover_lambda.zip s3://your-code-bucket/cover_lambda.zip
4. Deploy the stack
  4a. aws cloudformation create-stack
   --stack-name CoverStack
   --template-body file://COVER-Deploy.yaml
   --parameters \
     ParameterKey=CodeS3Bucket,ParameterValue=your-code-bucket
     ParameterKey=CodeS3Key,ParameterValue=cover_lambda.zip
     ParameterKey=S3BucketName,ParameterValue=your-cover-bucket-name
     ParameterKey=EmailRecipient,ParameterValue=recipient@example.com
     ParameterKey=EmailSender,ParameterValue=sender@example.com
   --capabilities CAPABILITY_IAM \
   4b. Ensure to change the CodeS3Bucket, S3BucketName, EmailRecipient, and EmailSender parameter values prior to executing the command above.
  5. Upload software.txt to S3BucketName (The bucket created by the CloudFormation template)
     5a. aws s3 cp software.txt s3://your-cover-bucket-name/software.txt
  6. Once the deployment completes, test the Lambda function.
  7. Configure the Lambda to execute on your desired cadence using an AWS EventBridge rule
