# Deploy Docker Image to AWS Lambda

A **GitHub composite action** that deploys a container image from **Amazon ECR** to an **AWS Lambda function** using **GitHub OIDC authentication**.

This action updates the Lambda function code by pointing it to a new image tag in ECR.

## Usage

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - name: Deploy image to Lambda
    id: lambdadeploy
    uses: Geonadir/gh-actions/.github/actions/deploy_to_lambda@v1.0.0
    with:
      AWS_REGION: your-aws-region
      AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcLambdaDeployRole # Your role here
      ECR_REGISTRY: 123456789012.dkr.ecr.us-east-1.amazonaws.com
      ECR_REPOSITORY: your-ecr-repo-here
      IMAGE_TAG: latest #USE YOURS
      LAMBDA_FUNCTION_NAME: your-lambda-function-name

  - name: Print outputs
    run: |
      echo "IMAGE_URI=${{ steps.lambdadeploy.outputs.IMAGE_URI }}"
```

---

## Inputs

| Input Name             | Required | Default | Description                                                            |
| ---------------------- | -------- | ------- | ---------------------------------------------------------------------- |
| `AWS_REGION`           | Yes      | —       | AWS region where the Lambda function exists                            |
| `AWS_ROLE_ARN`         | Yes      | —       | IAM Role ARN to assume via GitHub OIDC                                 |
| `ECR_REGISTRY`         | Yes      | —       | ECR registry URI (e.g. `123456789012.dkr.ecr.us-east-1.amazonaws.com`) |
| `ECR_REPOSITORY`       | Yes      | —       | Name of the Amazon ECR repository                                      |
| `IMAGE_TAG`            | Yes      | —       | Docker image tag to deploy                                             |
| `LAMBDA_FUNCTION_NAME` | Yes      | —       | Name of the AWS Lambda function                                        |

## Outputs

| Output Name | Description                       |
| ----------- | --------------------------------- |
| `IMAGE_URI` | Full image URI deployed to Lambda |


## Requirements

* Add this to your `workflow.yml`

  ```yaml
  permissions:
    id-token: write
    contents: read
  ```
* GitHub OIDC enabled in AWS
* IAM role with permission to:

  * `lambda:UpdateFunctionCode`
* Existing:

  * ECR repository
  * Lambda function configured to use **container images**

## Notes

* This action **does not build or push images** — pair it with your ECR build action.
* The Lambda function must already exist and be configured for container-based deployment.
* The image must be in the **same region** as the Lambda function.
