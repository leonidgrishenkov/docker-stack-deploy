[![GitHub Actions Marketplace](https://img.shields.io/badge/action-marketplace-blue.svg?logo=github&color=orange)](https://github.com/marketplace/actions/docker-stack-deploy-action)
[![Release version badge](https://img.shields.io/github/v/release/leonidgrishenkov/docker-stack-deploy)](https://github.com/leonidgrishenkov/docker-stack-deploy/releases)

GitHub Action and Docker image used to deploy a Docker stack on a Docker Swarm.


## Configuration options

| GitHub Action Input | Environment Variable | Summary | Required | Default Value |
| --- | --- | --- | --- | --- |
| `registry` | `REGISTRY` | Specify which container registry to login to. | |
| `username` | `USERNAME` | Container registry username. | | |
| `password` | `PASSWORD` | Container registry password. | | |
| `remote_host` | `REMOTE_HOST` | Hostname or address of the machine running the Docker Swarm manager node | ✅ | |
| `remote_port` | `REMOTE_PORT` | SSH port to connect on the the machine running the Docker Swarm manager node. | | **22** |
| `remote_user` | `REMOTE_USER` | User with SSH and Docker privileges on the machine running the Docker Swarm manager node. | ✅ | |
| `remote_private_key` | `REMOTE_PRIVATE_KEY` | Private key used for ssh authentication. | ✅ | |
| `deploy_timeout` | `DEPLOY_TIMEOUT` | Seconds, to wait until the deploy finishes | | **600** |
| `stack_file` | `STACK_FILE` | Path to the stack file used in the deploy. | ✅ | |
| `stack_name` | `STACK_NAME` | Name of the stack to be deployed. | ✅ | |
| `stack_param` | `STACK_PARAM` | Additional parameter (env var) to be passed to the stack. | | |
| `env_file` | `ENV_FILE` | Additional environment variables to be passed to the stack. | | |
| `debug` | `DEBUG` | Verbose logging | | **0** |


## Using the GitHub Action

Add, or edit an existing, `yaml` file inside `.github/actions` and use the configuration options listed above.

### Examples

#### Deploying public images


```yaml
name: Deploy Staging

on:
  push:
    branches:
      - main

jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout codebase
        uses: actions/checkout@v2

      - name: Deploy
        uses: kitconcept/docker-stack-deploy@v1.0.1
        with:
          remote_host: ${{ secrets.REMOTE_HOST }}
          remote_user: ${{ secrets.REMOTE_USER }}
          remote_private_key: ${{ secrets.REMOTE_PRIVATE_KEY }}
          stack_file: "stacks/plone.yml"
          stack_name: "plone-staging"
```

#### Deploying private images from GitHub Container Registry

First, follow the steps to [create a Personal Access Token](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-to-the-container-registry).

```yaml
name: Deploy Live

on:
  push:
    tags:
      - '*.*.*'

jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout codebase
        uses: actions/checkout@v2

      - name: Deploy
        uses: kitconcept/docker-stack-deploy@v1.0.1
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

## Using the Docker Image

It is possible to directly use the `ghcr.io/kitconcept/docker-stack-deploy` Docker image, passing the configuration options as environment variables.

### Examples

#### Local machine

Considering you have a local file named `.env_deploy` with content:

```
REGISTRY=hub.docker.com
USERNAME=foo_usr
PASSWORD=averylargepasswordortoken
REMOTE_HOST=192.168.17.2
REMOTE_PORT=22
REMOTE_USER=user
STACK_FILE=path/to/stack.yml
STACK_NAME=mystack
DEBUG=1
```

Run the following command:
```shell
docker run --rm
  -v "$(pwd)":/github/workspace
  -v /var/run/docker.sock:/var/run/docker.sock
  --env-file=.env_deploy
  -e REMOTE_PRIVATE_KEY="$(cat ~/.ssh/id_rsa)"
  ghcr.io/kitconcept/docker-stack-deploy:latest
```

#### GitLab CI

On your GitLab project, go to  `Settings -> CI/CD` and add the environment variables under **Variables**.

Then edit your `.gitlab-cy.yml` to include the `deploy` step:

```yaml
image: busybox:latest

services:
  - docker:20.10.16-dind

before_script:
  - docker info

deploy:
  stage: deploy
  varibles:
    REGISTRY: ${REGISTRY}
    USERNAME: ${REGISTRY_USER}
    PASSWORD: ${REGISTRY_PASSWORD}
    REMOTE_HOST: ${DEPLOY_HOST}
    REMOTE_PORT: 22
    REMOTE_USER: ${DEPLOY_USER}
    REMOTE_PRIVATE_KEY: "${DEPLOY_KEY}"
    STACK_FILE: stacks/app.yml
    STACK_NAME: app
    DEPLOY_IMAGE: ghcr.io/kitconcept/docker-stack-deploy:latest
  script:
    - docker pull ${DEPLOY_IMAGE}
    - docker run --rm
       -v "$(pwd)":/github/workspace
       -v /var/run/docker.sock:/var/run/docker.sock
       -e REGISTRY=${REGISTRY}
       -e USERNAME=${USERNAME}
       -e PASSWORD=${PASSWORD}
       -e REMOTE_HOST=${REMOTE_HOST}
       -e REMOTE_PORT=${REMOTE_PORT}
       -e REMOTE_USER=${REMOTE_USER}
       -e REMOTE_PRIVATE_KEY="${REMOTE_PRIVATE_KEY}"
       -e STACK_FILE=${STACK_FILE}
       -e STACK_NAME=${STACK_NAME}
       -e DEBUG=1
       ${DEPLOY_IMAGE}

```
## License

The project is licensed under [MIT License](./LICENSE)
