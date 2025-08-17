# Dokploy Deployer Action

A GitHub Action that triggers deployments on Dokploy and optionally polls for deployment status completion.

## Features

-   ✅ Deploy applications and compose projects on Dokploy using direct IDs
-   🔄 Optional polling for deployment status using deployment tracking APIs
-   ⏱️ Configurable polling intervals and timeouts
-   📊 Detailed status reporting with deployment IDs and logs

## Usage

### Basic Usage (Fire and Forget)

```yaml
- name: Deploy to Dokploy
  uses: your-username/dokploy-deployer-action@v1
  with:
      dokploy_url: ${{ secrets.DOKPLOY_URL }}
      api_key: ${{ secrets.DOKPLOY_API_KEY }}
      type: application
      application_id: ${{ secrets.DOKPLOY_APPLICATION_ID }}
```

### With Status Polling

```yaml
- name: Deploy to Dokploy with Status Check
  uses: your-username/dokploy-deployer-action@v1
  with:
      dokploy_url: ${{ secrets.DOKPLOY_URL }}
      api_key: ${{ secrets.DOKPLOY_API_KEY }}
      type: application
      application_id: ${{ secrets.DOKPLOY_APPLICATION_ID }}
      wait_for_deployment: true
      deployment_check_interval: 30
      deployment_timeout: 1200
```

### Using Direct IDs

```yaml
- name: Deploy Application by ID
  uses: your-username/dokploy-deployer-action@v1
  with:
      dokploy_url: ${{ secrets.DOKPLOY_URL }}
      api_key: ${{ secrets.DOKPLOY_API_KEY }}
      type: application
      application_id: ${{ secrets.DOKPLOY_APPLICATION_ID }}
      wait_for_deployment: true
```

### Compose Deployment

```yaml
- name: Deploy Compose Project
  uses: your-username/dokploy-deployer-action@v1
  with:
      dokploy_url: ${{ secrets.DOKPLOY_URL }}
      api_key: ${{ secrets.DOKPLOY_API_KEY }}
      type: compose
      compose_id: ${{ secrets.DOKPLOY_COMPOSE_ID }}
      wait_for_deployment: true
```

## Inputs

| Input                       | Description                                  | Required | Default       |
| --------------------------- | -------------------------------------------- | -------- | ------------- |
| `dokploy_url`               | Dokploy base URL with API access             | ✅       | -             |
| `api_key`                   | Dokploy API key                              | ✅       | -             |
| `type`                      | Deployment type (`application` or `compose`) | ❌       | `application` |
| `application_id`            | Dokploy application ID                       | ❌       | `""`          |
| `compose_id`                | Dokploy compose ID                           | ❌       | `""`          |
| `wait_for_deployment`       | Wait for deployment completion               | ❌       | `false`       |
| `deployment_check_interval` | Status check interval in seconds             | ❌       | `30`          |
| `deployment_timeout`        | Max deployment wait time in seconds          | ❌       | `1200`        |

## Outputs

| Output              | Description                                                   |
| ------------------- | ------------------------------------------------------------- |
| `deployment_status` | Final deployment status (when wait_for_deployment is enabled) |

## Status Values

When `wait_for_deployment` is enabled, the `deployment_status` output will contain one of:

-   `done` - Deployment completed successfully
-   `error` - Deployment failed
-   `timeout` - Polling timed out before completion

## Secrets Setup

### Required Secrets

Add these secrets to your GitHub repository (Settings → Secrets and variables → Actions):

| Secret Name              | Description                                | Example                            |
| ------------------------ | ------------------------------------------ | ---------------------------------- |
| `DOKPLOY_URL`            | Your Dokploy instance URL                  | `https://your-dokploy.example.com` |
| `DOKPLOY_API_KEY`        | API key from Dokploy dashboard             | `xxxxxxxxxxxxxxxxxxxxxx`           |
| `DOKPLOY_APPLICATION_ID` | Application ID from Dokploy                | `xxxxxxxxxxxxxxxxxxxxxx`           |
| `DOKPLOY_COMPOSE_ID`     | Compose ID from Dokploy (if using compose) | `xxxxxxxxxxxxxxxxxxxxxx`           |

### Getting Your Dokploy Credentials

1. **API Key**: Go to your Dokploy dashboard → Settings → Profile → Generate API key
2. **Application/Compose ID**: Go to your project → Select your app → Copy the ID from the URL or app settings
3. **Dokploy URL**: Your Dokploy instance base URL (without `/api` suffix)

## Examples

### Complete CI/CD Pipeline

```yaml
name: Deploy to Dokploy

on:
    push:
        branches: [master]

jobs:
    deploy:
        runs-on: ubuntu-latest

        steps:
                         - name: Deploy Application
               id: deploy
               uses: your-username/dokploy-deployer-action@v1
               with:
                   dokploy_url: ${{ secrets.DOKPLOY_URL }}
                   api_key: ${{ secrets.DOKPLOY_API_KEY }}
                   type: application
                   application_id: ${{ secrets.DOKPLOY_APPLICATION_ID }}
                   wait_for_deployment: true
                   deployment_check_interval: 2
                   deployment_timeout: 120

            - name: Check Deployment Result
              if: always()
              run: |
                  echo "Deployment status: ${{ steps.deploy.outputs.deployment_status }}"
                  if [ "${{ steps.deploy.outputs.deployment_status }}" = "done" ]; then
                    echo "🎉 Deployment successful!"
                  elif [ "${{ steps.deploy.outputs.deployment_status }}" = "error" ]; then
                    echo "❌ Deployment failed!"
                    exit 1
                  elif [ "${{ steps.deploy.outputs.deployment_status }}" = "timeout" ]; then
                    echo "⏰ Deployment timed out!"
                    exit 1
                  fi
```

### Parallel Deployments

```yaml
jobs:
    deploy:
        runs-on: ubuntu-latest
        strategy:
            matrix:
                environment: [staging, production]

        steps:
                         - name: Deploy to ${{ matrix.environment }}
               uses: your-username/dokploy-deployer-action@v1
               with:
                   dokploy_url: ${{ secrets.DOKPLOY_URL }}
                   api_key: ${{ secrets.DOKPLOY_API_KEY }}
                   type: application
                   application_id: ${{ secrets[format('DOKPLOY_APPLICATION_ID_{0}', matrix.environment)] }}
                   wait_for_deployment: true
```

## Error Handling

The action will:

-   ✅ Validate inputs before deployment
-   🔄 Retry API calls on network failures (when polling)
-   ⏰ Timeout gracefully if deployment takes too long
-   📋 Provide detailed logs for troubleshooting
-   ❌ Exit with non-zero code on deployment failures

## Requirements

-   Dokploy instance with API access
-   Valid API key with deployment permissions
-   `jq` (pre-installed on GitHub runners)
-   `curl` (pre-installed on GitHub runners)

## License

MIT License - see LICENSE file for details.
