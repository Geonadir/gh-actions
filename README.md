# gh-actions

Reusable GitHub composite actions for building and deploying containerized applications to AWS.

## Actions

| Action | Path | Description |
|--------|------|-------------|
| [ecr-build-push-action](/.github/actions/ecr-build-push-action/README.md) | `.github/actions/ecr-build-push-action` | Build a Docker image with Buildx and push it to Amazon ECR |
| [deploy_to_lambda](/.github/actions/deploy_to_lambda/README.md) | `.github/actions/deploy_to_lambda` | Deploy a container image from ECR to an AWS Lambda function |
| [deploy_to_ecs](/.github/actions/deploy_to_ecs/README.md) | `.github/actions/deploy_to_ecs` | Force a new deployment of an existing ECS service |

All actions authenticate with AWS via **GitHub OIDC** — no long-lived credentials stored in secrets.

## Prerequisites

- GitHub OIDC configured in your AWS account
- An IAM role with a trust policy that allows the GitHub Actions OIDC provider to assume it
- The required AWS resources already exist (ECR repository, Lambda function, or ECS cluster/service)

Add the following permissions block to any workflow that uses these actions:

```yaml
permissions:
  id-token: write   # required for OIDC token exchange
  contents: read
```

## Common Workflows

### Build and deploy to Lambda

```yaml
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - uses: actions/checkout@v7

      - name: Build & Push to ECR
        id: build
        uses: Geonadir/gh-actions/.github/actions/ecr-build-push-action@v1.0.0
        with:
          AWS_REGION: ap-south-1
          AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcEcrPushRole
          ECR_REPOSITORY: my-service
          IMAGE_TAG: ${{ github.sha }}

      - name: Deploy to Lambda
        uses: Geonadir/gh-actions/.github/actions/deploy_to_lambda@v1.0.0
        with:
          AWS_REGION: ap-south-1
          AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcLambdaDeployRole
          ECR_REGISTRY: ${{ steps.build.outputs.REGISTRY }}
          ECR_REPOSITORY: my-service
          IMAGE_TAG: ${{ github.sha }}
          LAMBDA_FUNCTION_NAME: my-lambda-function
```

### Build and deploy to ECS

```yaml
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - uses: actions/checkout@v7

      - name: Build & Push to ECR
        uses: Geonadir/gh-actions/.github/actions/ecr-build-push-action@v1.0.0
        with:
          AWS_REGION: ap-south-1
          AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcEcrPushRole
          ECR_REPOSITORY: my-service
          IMAGE_TAG: ${{ github.sha }}

      - name: Force new ECS deployment
        uses: Geonadir/gh-actions/.github/actions/deploy_to_ecs@v1.0.0
        with:
          AWS_REGION: ap-south-1
          AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcEcsDeployRole
          ECS_CLUSTER: my-cluster
          ECS_SERVICE: my-service
          WAIT_FOR_STABILITY: "true"
```

### Restart multiple ECS services in parallel

Use a workflow matrix to redeploy several services simultaneously:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        service: [web, worker, beat]
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Force new deployment (${{ matrix.service }})
        uses: Geonadir/gh-actions/.github/actions/deploy_to_ecs@v1.0.0
        with:
          AWS_REGION: ap-south-1
          AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcEcsDeployRole
          ECS_CLUSTER: my-cluster
          ECS_SERVICE: ${{ matrix.service }}
          WAIT_FOR_STABILITY: "true"
```

## Versioning

Always reference a specific tag rather than a branch to avoid unexpected changes:

```yaml
# Good — pinned to a release
uses: Geonadir/gh-actions/.github/actions/ecr-build-push-action@v1.0.0

# Avoid — floats with every commit to master
uses: Geonadir/gh-actions/.github/actions/ecr-build-push-action@master
```
