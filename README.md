# AWS IAM Permissions Boundary for AI Pipeline Developer

This CloudFormation template deploys a permissions boundary designed for developers building AI pipelines with AWS services like Lambda, S3, SageMaker, and Bedrock. It’s located in the boundary-policies/ directory and named ai-boundary.yaml.

## Overview

In AWS, permissions boundaries allow admins to delegate IAM role creation while capping privileges. This template enables a developer to create roles for AI workflows—such as Lambda triggers, S3 data retrieval, SageMaker training, or Bedrock inference—while enforcing limits. It scopes S3 access to specific buckets, permits key AI pipeline actions, and conditionally allows IAM management only when a boundary is attached to created roles.

Key features:
- Scoped S3 access to ai-pipeline-* buckets.
- Core AI actions for Lambda, SageMaker, and Bedrock.
- IAM actions permitted only with this boundary attached.
- Protection against boundary tampering.

## Parameters

# AllowedBucketPrefix
- Type: String
- Default: ai-pipeline-
- Description: Defines the prefix for S3 buckets the developer can access (e.g., ai-pipeline-data). This ensures S3 actions are limited to pipeline-relevant buckets.

## Deployment Steps

Assuming AWS credentials are configured in your environment (e.g., via aws configure), follow these steps to deploy ai-boundary.yaml from the boundary-policies/ directory.

1. **Clone the Repository**
   Clone the menashi-samples repo to your local machine:
   git clone https://github.com/dinaodum/menashi-samples.git
   Navigate to the boundary-policies/ directory:
   cd menashi-samples/boundary-policies

2. **Deploy the Stack**
   Use the AWS CLI to deploy the CloudFormation stack:
   aws cloudformation deploy --template-file ai-boundary.yaml --stack-name ai-boundary-test --capabilities CAPABILITY_NAMED_IAM
   # Note: --capabilities CAPABILITY_NAMED_IAM is required because the template creates named IAM resources.
   # Optional: Customize the S3 bucket prefix:
   aws cloudformation deploy --template-file ai-boundary.yaml --stack-name ai-boundary-test --capabilities CAPABILITY_NAMED_IAM --parameter-overrides AllowedBucketPrefix=my-ai-

3. **Verify the Deployment**
   Check the stack outputs to get the boundary and role ARNs:
   aws cloudformation describe-stacks --stack-name ai-boundary-test --query "Stacks[0].Outputs[*].[OutputKey, OutputValue]" --output table
   Verify the role in the AWS CLI:
   aws iam get-role --role-name AIPipelineDeveloperRole
   # Note: Look for the PermissionsBoundary field to confirm it’s attached.

4. **Test the Boundary**
   # Create a Role with the Boundary
   Use the boundary ARN from the output to create a new role:
   aws iam create-role --role-name TestAIPipelineRole --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}' --permissions-boundary $(aws cloudformation describe-stacks --stack-name ai-boundary-test --query "Stacks[0].Outputs[?OutputKey=='BoundaryArn'].OutputValue" --output text)
   # Result: This succeeds because the boundary is attached.
   
   # Create a Role without a Boundary
   aws iam create-role --role-name TestNoBoundary --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}'
   # Result: This fails with AccessDenied due to the IAM condition.
   
   # Test AI Actions
   From a Lambda or CLI with AIPipelineDeveloperRole:
   - aws s3 ls s3://ai-pipeline-data works (if the bucket exists).
   - aws lambda invoke --function-name some-ai-function works (if within permissions).

5. **Cleanup**
   Delete the stack to remove all resources:
   aws cloudformation delete-stack --stack-name ai-boundary-test
   Confirm it’s gone:
   aws cloudformation describe-stacks --stack-name ai-boundary-test
   # Note: Expect a “stack not found” error if successful.

## Notes

- Prerequisites: AWS CLI installed, credentials set up, and IAM permissions to create roles/policies (iam:CreateRole, iam:CreatePolicy, etc.).
- Security: The conditional IAM logic ensures roles created by the developer inherit this boundary, aligning with the AWS Security Pillar.
- Use Case: Ideal for admins delegating AI pipeline development while maintaining control.

Explore this pattern and more at [menashiconsulting.com/blog](https://menashiconsulting.com/blog)!