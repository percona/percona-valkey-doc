# Run in Docker

Run Valkey in a Docker container using the [Percona Valkey images](https://hub.docker.com/r/percona/valkey) on Docker Hub.

## Available images

Both images are available for x86_64 and ARM64 architectures.

| Image | Example tags | Description |
|-------|--------------|-------------|
| Standard | `latest`, `9`, `9.1`, `9.1.2` | Built on Red Hat Universal Base Image 9 Minimal. |
| Hardened | `9-hardened`, `9.1-hardened`, `9.1.2-hardened` | Minimal image built on a hardened Debian base, with a reduced attack surface. |

Both images expose port `6379`, store data in `/data`, and run as the non-root `valkey` user.

## Start a container

1. Pull the Valkey image:

    === "Standard image"

        ```{.bash data-prompt="$"}
        $ docker pull percona/valkey:latest
        ```

        To pull a specific release, replace `latest` with the version tag, for example, `9.1`.

    === "Hardened image"

        ```{.bash data-prompt="$"}
        $ docker pull percona/valkey:9.1-hardened
        ```

        There is no `latest` tag for the hardened image. In the following steps, replace `percona/valkey:latest` with `percona/valkey:9.1-hardened`.

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
