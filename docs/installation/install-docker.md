# Install from Docker

Run Valkey in a Docker container using the [Percona Valkey image](https://hub.docker.com/r/percona/valkey). The image is available for x86_64 and ARM64 architectures.

1. Pull the latest Valkey image:

    ```{.bash data-prompt="$"}
    $ docker pull percona/valkey:latest
    ```

    To pull a specific release, replace `latest` with the version tag, for example:

    ```{.bash data-prompt="$"}
    $ docker pull percona/valkey:9.1
    ```

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
