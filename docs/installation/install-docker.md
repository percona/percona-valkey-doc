# Run in Docker

Run Valkey in a Docker container using the [Percona Valkey images](https://hub.docker.com/r/percona/valkey) on Docker Hub.

## Available images

Both images are available for x86_64 and ARM64 architectures.

| Image | Tags | Description |
|-------|------|-------------|
| Standard | `latest`, `<major>`, `<major>.<minor>`, `<major>.<minor>.<patch>` | Built on Red Hat Universal Base Image 9 Minimal. |
| Hardened | `<major>-hardened`, `<major>.<minor>-hardened`, `<major>.<minor>.<patch>-hardened` | Minimal image built on a hardened Debian base, with a reduced attack surface. |

The `<major>` and `<major>.<minor>` tags always point to the newest release in that series. For example, `{{ valkey_major }}` points to the latest Valkey {{ valkey_major }}.x release. For the full list of tags, see [Docker Hub](https://hub.docker.com/r/percona/valkey/tags).

Both images expose port `6379`, store data in `/data`, and run as the non-root `valkey` user.

## Start a container

1. Pull the Valkey image:

    === "Standard image"

        ```{.bash data-prompt="$"}
        $ docker pull percona/valkey:latest
        ```

        To pull a specific release, replace `latest` with a version tag, for example, `{{ valkey_version }}`.

    === "Hardened image"

        ```{.bash data-prompt="$"}
        $ docker pull percona/valkey:{{ valkey_major }}-hardened
        ```

        There is no `latest` tag for the hardened image. In the following steps, replace `percona/valkey:latest` with `percona/valkey:{{ valkey_major }}-hardened`.

2. Start a Valkey container and expose the default port:

    ```{.bash data-prompt="$"}
    $ docker run -d \
      --name valkey \
      -p 6379:6379 \
      percona/valkey:latest
    ```

    To keep data when you remove the container, mount a volume to `/data`:

    ```{.bash data-prompt="$"}
    $ docker run -d \
      --name valkey \
      -p 6379:6379 \
      -v valkey-data:/data \
      percona/valkey:latest
    ```

3. Verify that the container is running:

    ```{.bash data-prompt="$"}
    $ docker ps --filter name=valkey
    ```

## Next steps

[Connect to Valkey :material-arrow-right:](connect.md#docker){ .md-button }
