[![GitHub Actions Marketplace](https://img.shields.io/badge/action-marketplace-blue.svg?logo=github&color=blue)](https://github.com/marketplace/actions/docker-stack-swarm-deploy)
[![Release version badge](https://img.shields.io/github/v/release/leonidgrishenkov/docker-stack-deploy)](https://github.com/leonidgrishenkov/docker-stack-deploy/releases)

# About

GitHub Action and Docker image used to deploy a Docker stack on a Docker Swarm.

This is a fork from [Action made by kitconcept](https://github.com/kitconcept/docker-stack-deploy) with some fixes and updates.

# Configuration options

| GitHub Action Input  | Environment Variable | Summary                                                                                      | Required | Default Value |
| -------------------- | -------------------- | -------------------------------------------------------------------------------------------- | -------- | ------------- |
| `registry`           | `REGISTRY`           | Specify which container registry to login to.                                                |          |
| `username`           | `USERNAME`           | Container registry username.                                                                 |          |               |
| `password`           | `PASSWORD`           | Container registry password.                                                                 |          |               |
| `remote_host`        | `REMOTE_HOST`        | Hostname or address of the machine running the Docker Swarm manager node                     | ✅       |               |
| `remote_port`        | `REMOTE_PORT`        | SSH port to connect on the the machine running the Docker Swarm manager node.                |          | **22**        |
| `remote_user`        | `REMOTE_USER`        | User with SSH and Docker privileges on the machine running the Docker Swarm manager node.    | ✅       |               |
| `remote_private_key` | `REMOTE_PRIVATE_KEY` | Private key used for ssh authentication.                                                     | ✅       |               |
| `deploy_timeout`     | `DEPLOY_TIMEOUT`     | Seconds, to wait until the deploy finishes                                                   |          | **600**       |
| `stack_file`         | `STACK_FILE`         | Path to the stack file used in the deploy.                                                   | ✅       |               |
| `stack_name`         | `STACK_NAME`         | Name of the stack to be deployed.                                                            | ✅       |               |
| `stack_param`        | `STACK_PARAM`        | Additional parameter (env var) to be passed to the stack.                                    |          |               |
| `env_file`           | `ENV_FILE`           | Additional environment variables to be passed to the stack in format: VAR1=value\nVAR2=value |          |               |
| `debug`              | `DEBUG`              | Verbose logging                                                                              |          | **0**         |

# Examples

## Deploying public images

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Deploy
        uses: leonidgrishenkov/docker-stack-deploy@v1.2.3
        with:
          remote_host: ${{ secrets.REMOTE_HOST }}
          remote_user: ${{ secrets.REMOTE_USER }}
          remote_private_key: ${{ secrets.REMOTE_PRIVATE_KEY }}
          stack_file: "stacks/plone.yml"
          stack_name: "plone-staging"
```

## Deploying private images from GitHub Container Registry

First, follow the steps to [create a Personal Access Token](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-to-the-container-registry).

```yaml
name: Deploy

on:
  push:
    tags:
      - "*.*.*"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Deploy
        uses: leonidgrishenkov/docker-stack-deploy@v1.2.3
        with:
          registry: "ghcr.io"
          username: ${{ secrets.GHCR_USERNAME }}
          password: ${{ secrets.GHCR_TOKEN }}
          remote_host: ${{ secrets.REMOTE_HOST }}
          remote_user: ${{ secrets.REMOTE_USER }}
          remote_private_key: ${{ secrets.REMOTE_PRIVATE_KEY }}
          stack_file: "stacks/plone.yml"
          stack_name: "plone-live"
          stack_param: "foo"
```

## Deploying private images from Yandex Cloud Container Registry

Make sure you have Yandex Cloud OAuth or IAM token to login into Container Registry. For more details see [Authentication in Container Registry](https://yandex.cloud/en/docs/container-registry/operations/authentication).

Here also you can see how to use `env_file` option for additional environment variables.

```yaml
name: Deploy

on:
  push:
    tags:
      - "*.*.*"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Deploy
        uses: leonidgrishenkov/docker-stack-deploy@v1.2.3
        with:
          registry: cr.yandex
          username: oauth
          password: ${{ secrets.YC_OAUTH_TOKEN }}
          remote_host: ${{ secrets.REMOTE_HOST }}
          remote_user: ${{ secrets.REMOTE_USER }}
          remote_private_key: ${{ secrets.REMOTE_PRIVATE_KEY }}
          stack_file: ./compose.yaml
          stack_name: bot
          env_file: |
            IMAGE=mysuperimage
            TAG=v1.0.0
            VAR1=value
            VAR2=value
```
