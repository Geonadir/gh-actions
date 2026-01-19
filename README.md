# gh-actions

A tiny toolbox of GitHub composite actions.

## What’s inside

- **`.github/actions/ecr-build-push-action`**  
  Build a Docker image and push it to **AWS ECR**.
  Docs: see the action’s own `README.md`.

## Usage

```yaml
- uses: zite-io/gh-actions/.github/actions/action-name@v1.0.0
  with:
    AWS_REGION: ap-south-1
    AWS_ROLE_ARN: arn:aws:iam::123456789012:role/GitHubOidcEcrPushRole
    ECR_REPOSITORY: my-service
    IMAGE_TAG: latest
## Other inputs as needed
```

## Tip
Always use a tag (e.g.`@v1.0.0`) instead of branch `@master`