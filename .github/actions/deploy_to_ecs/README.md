# Deploy to ECS (Force New Deployment)
A lightweight composite action that forces a new deployment of an existing AWS ECS services.
Basically: the **restart button** for ECS.

## Usage (single service)
```yaml
permissions:
  id-token: write
  contents: read

steps:
  - name: Restart ECS service
    uses: Geonadir/gh-actions/.github/actions/deploy_to_ecs@v1.0.0
    with:
      AWS_REGION: ap-south-1
      AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcEcsDeployRole
      ECS_CLUSTER: your-cluster
      ECS_SERVICE: your-service
      WAIT_FOR_STABILITY: "true"
```

## Multiple services 
Use workflow matrix to restart multiple services in parallel.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        service:
          - web
          - worker
          - beat

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
## Inputs

| Input | Required | Default | Description |
|------|----------|---------|-------------|
| AWS_REGION | Yes | — | AWS region |
| AWS_ROLE_ARN | Yes | — | IAM Role ARN (OIDC) |
| ECS_CLUSTER | Yes | — | ECS cluster name or ARN |
| ECS_SERVICE | Yes | — | ECS service name or ARN |
| WAIT_FOR_STABILITY | No | true | Wait until service is stable |

## Notes
- This action does **not** update task definitions or images.
- It simply tells ECS: “please re-deploy the tasks, thank you.”
- If it fails, IAM permissions might be the reason. Its always IAM. 😄
