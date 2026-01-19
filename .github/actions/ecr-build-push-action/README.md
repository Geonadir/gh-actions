# Build & Push Docker Image to AWS ECR

A **GitHub composite action** that builds a Docker image using **Buildx** and pushes it to **Amazon ECR** using **GitHub OIDC**.  

## Usage

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - name: Build & Push to ECR
    id: ecrbuild
    uses: Geonadir/gh-actions/.github/actions/ecr-build-push-action@v1.0.0
    with:
      AWS_REGION: your-aws-region
      AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcEcrPushRole #Your role here
      ECR_REPOSITORY: your-ecr-repo-here
      IMAGE_TAG: latest
      DOCKERFILE_PATH: Dockerfile
      BUILD_CONTEXT: .
      PIP_MODULES_FILE: requirements.txt
      APT_MODULES_FILE: apt_requirements.txt #DO NOT PASS INPUT if not needed

  - name: Print outputs
    run: |
      echo "IMAGE_URI=${{ steps.ecrbuild.outputs.IMAGE_URI }}"
      echo "REGISTRY=${{ steps.ecrbuild.outputs.REGISTRY }}"
      echo "IMAGE_DIGEST=${{ steps.ecrbuild.outputs.IMAGE_DIGEST }}"
```

## Inputs

| Input Name         | Required | Default                              | Description                                      |
|--------------------|----------|--------------------------------------|--------------------------------------------------|
| `AWS_REGION`       | Yes      | —                                    | AWS region where the ECR repository exists        |
| `AWS_ROLE_ARN`     | Yes      | —                                    | IAM Role ARN to assume via GitHub OIDC            |
| `ECR_REPOSITORY`   | Yes      | —                                    | Name of the Amazon ECR repository                 |
| `IMAGE_TAG`        | No       | `latest`                             | Docker image tag to push                          |
| `DOCKERFILE_PATH`  | No       | `Dockerfile`                  | Path to the Dockerfile                            |
| `BUILD_CONTEXT`    | No       | `.`                                  | Docker build context                              |
| `PIP_MODULES_FILE` | No       | `requirements.txt`    | Path to pip requirements file                    |
| `APT_MODULES_FILE` | No       | `apt_requirements.txt`| Path to apt dependencies requirements file       |

## Requirements
- Add this on your `workflow.yml`
  ```yaml
  permissions:
    id-token: write
    contents: read
  ```
- GitHub OIDC enabled in AWS
- IAM role with ECR push permissions
- Existing ECR repository
