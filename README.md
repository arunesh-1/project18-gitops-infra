# Project 18: GitOps infrastructure

A pull request to `main` validates Terraform. After a pull request is merged,
the apply job waits for approval in the `production` GitHub environment, then
uses AWS OIDC credentials to apply the merged commit. The workflow runs on a
push to `main` and checks that its commit is the merge commit of a pull request.

The example creates a private S3 bucket in `ap-south-1`. Terraform state is
stored in a separate, pre-existing S3 bucket. Both buckets can incur charges.

## Prerequisites

- The `production` environment must require a reviewer and allow deployments
  only from `main`. Verify that the GitHub plan enforces these rules.
- Create an S3 state bucket in `ap-south-1`, with versioning and public access
  blocked. It must not be the bucket managed by this Terraform configuration.
- Configure an AWS IAM OIDC provider for `https://token.actions.githubusercontent.com`
  and a role whose trust policy limits `aud` to `sts.amazonaws.com` and `sub`
  to `repo:arunesh-1/project18-gitops-infra:environment:production`.
- Grant that role S3 permissions only for the state bucket, its lock file, and
  the example bucket. The example bucket is named
  `project18-gitops-demo-<AWS account ID>`.
- In the repository's `production` environment, set variables `AWS_ROLE_ARN`
  to the role ARN and `TF_STATE_BUCKET` to the state bucket name.

Only trusted maintainers should be able to change the workflow, environment,
and AWS role: these can affect deployment security. No long-lived AWS access
keys are needed in GitHub.

## Deployment

Open a pull request against `main`. Once validation succeeds, merge it and
approve the `production` deployment in the Actions run. The job plans and
applies that merge commit. Direct pushes and unmerged pull requests do not
trigger apply. A deployment fails rather than apply an older commit if `main`
has advanced while approval was pending.

S3 state locking serializes Terraform operations. Do not delete the state
bucket while the managed bucket exists. To clean up, use a reviewed Terraform
change; deleting the state bucket alone does not delete managed infrastructure.
