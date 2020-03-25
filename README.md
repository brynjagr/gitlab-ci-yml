<h1 align="center">
  <img src="https://github.com/SocialGouv/gitlab-ci-yml/raw/master/.github/gitlab.gif" width="250"/>
  <p align="center">.gitlab-ci.yml</p>
  <p align="center" style="font-size: 0.5em">✨✨✨✨✨✨✨✨</p>
</h1>

<p align="center">
  <a href="https://opensource.org/licenses/Apache-2.0"><img src="https://img.shields.io/badge/License-Apache--2.0-yellow.svg" alt="License: Apache-2.0"></a>
  <a href="https://github.com/SocialGouv/gitlab-ci-yml/releases "><img alt="GitHub release (latest SemVer)" src="https://img.shields.io/github/v/release/SocialGouv/gitlab-ci-yml?sort=semver"></a>
  <a href="https://gitlab.factory.social.gouv.fr/SocialGouv/gitlab-ci-yml/commits/master"><img alt="Gitlab Pipeline Status" src="https://gitlab.factory.social.gouv.fr/SocialGouv/gitlab-ci-yml/badges/master/pipeline.svg"></a>
</p>

<br>
<br>
<br>
<br>

## Usage

Use like this in your `.gitlab-ci.yml` :

```yml
---
include:
  - "https://raw.githubusercontent.com/SocialGouv/gitlab-ci-yml/master/github-deployments.yml"
  - "https://raw.githubusercontent.com/SocialGouv/gitlab-ci-yml/master/register-stage.yml"
```

# [.autodevops_simple_app](./autodevops_simple_app.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /autodevops_simple_app.yml
    ref: v10.0.0

variables:
  PROJECT: "sample-next-app"
  RANCHER_PROJECT_ID: "c-gsm8d:p-pwpk6" # "default" project id here
  PORT: 8080
  VALUES_FILE: ./.k8s/app.values.yml # Your values
  ENABLE_AZURE_POSTGRES: 1 # enable postgres deployment using [azure-db](https://github.com/SocialGouv/docker/tree/master/azure-db)
```

# [.base_create_namespace_stage](./base_create_namespace_stage.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_docker_kubectl_image_stage.yml
    ref: v10.0.0
  - project: SocialGouv/gitlab-ci-yml
    file: /base_create_namespace_stage.yml
    ref: v10.0.0

#

Create namespace:
  extends: .base_create_namespace_stage
  variables:
    # The rancher project where the namespaces will be created
    RANCHER_PROJECT_ID: <rancher_project_id>
    # Optional
    REMOTE_URL: "https://github.com/${CI_PROJECT_PATH}.git"
  before_script:
    - K8S_NAMESPACE=my-namespace
    # (re)create to ensure a new namespaces will be created
    # - kubectl delete namespaces ${K8S_NAMESPACE} || true
```

# [.base_delete_useless_k8s_ns_stage](./base_delete_useless_k8s_ns_stage.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_delete_useless_k8s_ns_stage.yml
    ref: v10.0.0
#

Delete useless k8s namespaces:
  extends: .base_delete_useless_k8s_ns_stage
  variables:
    # Optional
    # Filter the namespaces to check for suppression
    K8S_NAMESPACE_PREFIX: "${PROJECT}-${CI_PROJECT_ID}-review"
```

# [.base_deploy_app_chart_stage](./base_deploy_app_chart_stage.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_docker_helm_image_stage.yml
    ref: v10.0.0
  - project: SocialGouv/gitlab-ci-yml
    file: /base_deploy_app_chart_stage.yml
    ref: v10.0.0

#

.deploy_myapp_stage:
  dependencies: []
  stage: Deploy
  extends:
    - .base_deploy_app_chart_stage
  variables:
    CONTEXT: app
    VALUES_FILE: ./.k8s/app.values.yml
    # optional
    HELM_RENDER_ARGS: "--set deployment.port 8080"

#

Deploy myapp (dev):
  extends:
    - .deploy_myapp_stage
  except:
    - master
  variables:
    HOST: ${CI_ENVIRONMENT_SLUG}-${CI_PROJECT_NAME}.${KUBE_INGRESS_BASE_DOMAIN}
  environment:
    name: ${CI_COMMIT_REF_NAME}-dev
    url: https://${CI_ENVIRONMENT_SLUG}-${CI_PROJECT_NAME}.${KUBE_INGRESS_BASE_DOMAIN}

Deploy app (prod):
  extends:
    - .deploy_myapp_stage
  only:
    - master
  variables:
    HOST: ${CI_PROJECT_NAME}.${KUBE_INGRESS_BASE_DOMAIN}
    K8S_NAMESPACE: ${CI_PROJECT_NAME}
    PRODUCTION: "true"
  environment:
    name: prod
    url: https://${CI_PROJECT_NAME}.${KUBE_INGRESS_BASE_DOMAIN}
```

# [.base_docker_helm_image_stage](./base_docker_helm_image_stage.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_docker_kubectl_image_stage.yml
    ref: v10.0.0
  - project: SocialGouv/gitlab-ci-yml
    file: /base_docker_helm_image_stage.yml
    ref: v10.0.0

#

Helm job:
  extends: .base_docker_helm_image_stage
  script:
    - helm version --client-only
```

# [.base_docker_kubectl_image_stage](./base_docker_kubectl_image_stage.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_docker_kubectl_image_stage.yml
    ref: v10.0.0
#

Kubectl job:
  extends: .base_docker_kubectl_image_stage
  script:
    - kubectl version --client
```

# [.base_migrate_azure_db](./base_migrate_azure_db.yml)

This will run the two following scripts for feature-branches deployments :

  - yarn run migrate:latest
  - yarn run seed:run
    
## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_migrate_azure_db.yml
    ref: v10.0.0
```

# [.base_register_stage](./base_register_stage.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_register_stage.yml
    ref: v10.0.0

Register myapp image:
  extends: .base_register_stage
  variables:
    CONTEXT: . # The folder where the Dockerfile is
    IMAGE_NAME: $CI_REGISTRY_IMAGE # The image name
    # optional
    DOCKER_BUILD_ARGS: "--build-arg SENTRY_DSN=https://sentry"
```

# [.base_semantic_release_stage](./base_semantic_release_stage.yml)

## Usage

```yaml
include:
  - project: SocialGouv/gitlab-ci-yml
    file: /base_semantic_release_stage.yml
    ref: v10.0.0

#

Release:
  extends: .base_semantic_release_stage

# or

Release:
  extends: .base_semantic_release_stage
  variables:
    SEMANTIC_RELEASE_PLUGINS: "@semantic-release/changelog @semantic-release/git"

```
