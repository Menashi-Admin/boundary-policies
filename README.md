# AWS IAM Permissions Boundary for AI Pipeline Developer

Deploys a permissions boundary and sample role for developers building AI pipelines with Lambda, S3, SageMaker, and Bedrock.

## Overview
This template creates a boundary allowing scoped access to AI-related AWS services while denying IAM and admin actions. It’s attached to a sample role, capping its privileges even with broader policies.

## Usage
1. Clone: `git clone https://github.com/dinaodum/menashi-samples.git`.
2. Navigate: `cd menashi-samples/ai-boundary`.
3. Deploy: `aws cloudformation deploy --template-file ai-boundary.yaml --stack-name ai-boundary-test --capabilities CAPABILITY_NAMED_IAM`.
   - Optional: `--parameter-overrides AllowedBucketPrefix=my-ai-`.
4. Check outputs: `aws cloudformation describe-stacks --stack-name ai-boundary-test --query "Stacks[0].Outputs[*].[OutputKey, OutputValue]" --output table`.
5. Verify: `aws iam get-role --role-name AIPipelineDeveloperRole`.
6. Clean up: `aws cloudformation delete-stack --stack-name ai-boundary-test`.

## Testing
- Attach `AmazonS3FullAccess` to the role and test:
  - S3 access to `ai-pipeline-*` works.
  - IAM actions are blocked.

## Notes
- Requires IAM permissions to deploy.
- Ideal for admins securing developer workflows.
