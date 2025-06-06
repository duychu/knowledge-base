---
sidebar_position: 1
---

# Buildkit

Buildkit is great replacement for kaniko which is simply dead because of no maintainer

The great thing about it is it support rootless setup so we can improve our build process without compromises security

## Setup in homelab

Because in homelab we use custom ca-certificate, so to make it works properly, I found that the easies way is to update ca-certiciate.crt
from base image moby/buildkit:rootless

I have tried multiple way to avoid doing that, but result is not good at all. That is why I ended up with below Dockerfile to build my own rootless base image for my docker pipeline

```bash
FROM moby/buildkit:v0.22.0-rootless

# REFER: https://github.com/moby/buildkit/blob/v0.22.0/Dockerfile
# this ca is for homelab

USER root
COPY --chown=1000:1000 ca.crt ca.crt
RUN cat ca.crt >> /etc/ssl/certs/ca-certificates.crt
USER 1000:1000
```

In gitlab ci, you can setup build stage

```yaml
variables:
  CONTEXT_PATH: ""  # in case we need specify different context path than default $CI_PROJECT_DIR  
  DOCKER_FILE: "Dockerfile" # declare DOCKER_FILE variable in case the location is different than default
  DOCKER_TAG: NA
  BUILD_ARGS: "" # in case we need build-args we define empty variable that could be reused later and used for kaniko/buildkit build

.docker_build: 
  image:
    name: harbor.homelab.duychu.link/library/moby/buildkit:v0.22.0-rootless  # <-- customized from buildkit
    entrypoint: [""]
  stage: build
  variables:
    # enable this when using rootless buildkit
    BUILDKITD_FLAGS: --oci-worker-no-process-sandbox --trace
  before_script:
    - env
    - mkdir -p ~/.docker
    - |
      cat > ~/.docker/config.json << EOF
      {
        "auths": {
          "": {
            "auth": "$(printf "%s:%s" "${CI_REGISTRY_USER}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')"
          },
          "$(echo -n $CI_DEPENDENCY_PROXY_SERVER | awk -F[:] '{print $1}')": {
            "auth": "$(printf "%s:%s" ${CI_DEPENDENCY_PROXY_USER} "${CI_DEPENDENCY_PROXY_PASSWORD}" | base64 | tr -d '\n')"
          },
          "$HARBOR": {
            "auth": "$(printf "%s:%s" ${HARBOR_USER} "${HARBOR_PASSWORD}" | base64 | tr -d '\n')"
          }
        }
      }
      EOF
    - cat ~/.docker/config.json
  script:
    # [Buildkit configuration]                  https://docs.docker.com/build/buildkit/toml-configuration/
    # [Specify name of dockerfile in buildkit]  https://github.com/moby/buildkit/issues/684
    - export BUILDKITD_FLAGS="$BUILDKITD_FLAGS"
    - |
      buildctl-daemonless.sh build \
        --frontend dockerfile.v0 \
        --local context=$CI_PROJECT_DIR/$CONTEXT_PATH \
        --local dockerfile=$CI_PROJECT_DIR \
        --opt filename=$DOCKER_FILE \
        --output type=image,name=$DOCKER_IMAGE:$DOCKER_TAG,push=true \
        ${BUILD_ARGS}
```

then in main pipeline, you can reuse it by extending the `.docker_build`

```yaml
build-staging:
  extends: .docker_build
  rules:
    ...
```
