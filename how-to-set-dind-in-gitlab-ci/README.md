# How to setup docker-in-docker environment for Gitlab CI

1. Register to Gitlab

```
sudo gitlab-runner register -n \
  --url https://<internal_gitlab_host>/ \
  --registration-token <runner_token> \
  --executor docker \
  --description "OTTBuilder runner" \
  --docker-image "docker:latest" \
  --docker-privileged \
  --tag-list ottbuilder
```

2. Modify `/etc/gitlab-runner/config.toml` as the following. Ref: https://gitlab.com/gitlab-org/gitlab-runner/issues/1986

```
concurrent = 1
check_interval = 0

[[runners]]
  name = "OTTBuilder runner"
  url = "https://<internal_gitlab_host>/"
  token = "4a11fd.....eb3ac"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "docker:latest"
    privileged = true
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
  [runners.cache]
    Insecure = false
```
3. In your build project, modify `.gitlab-ci.yml`, now you can use docker-in-docker.

```
image: docker:latest
variables:
  DOCKER_DRIVER: overlay2
  
services:
- docker:dind

before_script:
- docker info

build:
  script: 
   - docker build -t bla/bla .
   - docker login -u $DOCKER_USER -p $DOCKER_PASSWORD
   - docker push
```

# How to pull images from AWS ECR as build enviornemnt.
In the runner, you should execute login command for AWS ECS

```
$(aws ecr get-login --no-include-email --region us-east-1)
```
or

Provide the AWS Access key and Secret key in ~/.aws/credentials, then config ECR policy via ECS portal to allow actions for the configured user.
