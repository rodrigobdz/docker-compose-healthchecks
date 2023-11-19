# Docker Compose Healthchecks

> Collection of Docker Compose `healthcheck` examples

## Collection

### Postgres

```yml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 30s
  timeout: 30s
  retries: 3
```

[Source](https://til.codes/health-check-option-in-docker-to-wait-for-dependent-containers-to-be-healthy/)

### Redis

```yml
redis:
  image: redis
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 1s
    timeout: 3s
    retries: 30
```

[Source](https://gist.github.com/phuysmans/4f67a7fa1b0c6809a86f014694ac6c3a)

### Elasticsearch

```yml
 healthcheck:
  test: curl --silent http://localhost:9200 >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi
  interval: 30s
  timeout: 10s
  retries: 5
```

[Source](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-docker.html)

### Minio RELEASE.2023-11-01T18-37-25Z and older

```yml
healthcheck:
  test: timeout 5s bash -c ':> /dev/tcp/127.0.0.1/9000' || exit 1
  start_period: 5s
  interval: 10s
  timeout: 5s
  retries: 2
```

Source [[1]](https://github.com/minio/minio/issues/18373)

### Minio RELEASE.2023-10-25T06-33-25Z and earlier

```yml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
  interval: 30s
  timeout: 20s
  retries: 3
```

Source [[1]](https://github.com/minio/minio/blob/c9d502e6fadf2c2cbb1e2ce7a9dc8de7dfed1d34/docs/orchestration/docker-swarm/docker-compose-secrets.yaml#L25-L29) [[2]](https://github.com/minio/minio/blob/bdd094bc39275a0543484c09f4fb99ce3fe14787/docs/metrics/healthcheck/README.md)
### Mongo

```yml
healthcheck:
  test:
    [
      "CMD",
      "mongo",
      "--quiet",
      "127.0.0.1/test",
      "--eval",
      "'quit(db.runCommand({ ping: 1 }).ok ? 0 : 2)'",
    ]
  interval: 10s
  timeout: 10s
  retries: 5
  start_period: 40s
```

## Scripting

To wait for a Docker container until its health state is `healthy` before executing a command inside the container, use the following function in a shell script:

```sh
readonly DOCKER_CONTAINER='my_postgres'

wait_until_container_healthy() {
  until
    [ "$(docker inspect --format "{{.State.Health.Status}}" "$DOCKER_CONTAINER")" = 'healthy' ]
  do
    # Log container status
    local container_status
    container_status="$(docker inspect --format "{{.State.Health.Status}}" "$DOCKER_CONTAINER")"

    echo "Waiting for container $DOCKER_CONTAINER to be 'healthy', current status is '$container_status'. Sleeping for five seconds..."
    sleep 5
  done
}
```

This snippet is an improved version of [this](https://stackoverflow.com/a/33520390/2227405) StackOverflow answer.

## Related

- [`docker/awesome-compose`](https://github.com/docker/awesome-compose) - Official Docker Compose examples (includes healthchecks)
- [`docker-library/healthcheck`](https://github.com/docker-library/healthcheck) - Examples for Dockerfiles
