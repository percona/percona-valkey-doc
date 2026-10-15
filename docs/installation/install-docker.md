# Install from Docker

Run Valkey in a Docker container using the official Percona image.

1. Pull the latest Valkey image:

    ```{.bash data-prompt="$"}
    docker pull percona/valkey:latest
    ```

    To pull a specific release, replace `latest` with the desired version tag, for example:

    ```{.bash data-prompt="$"}
    docker pull percona/valkey:9.1
    ```

2. Run the following command to start a Valkey container and expose the default port:

    ```{.bash data-prompt="$"}
    docker run -d \
    --name valkey \
    -p 6379:6379 \
    percona/valkey:latest
    ```

    Verify that the container is running:

    ```{.bash data-prompt="$"}
    docker ps
    ```

## Next steps

[Connect to Valkey :material-arrow-right:](connect.md#docker){ .md-button }
