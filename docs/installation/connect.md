# Connect to Valkey

Use `valkey-cli` to connect to the Valkey server.

1. Check the connection:

    ```{.bash data-prompt="$"}
    $ valkey-cli ping
    ```

    ??? example "Expected output"

        ```text
        PONG
        ```

2. Run `valkey-cli` without any argument to start interactive mode:

    ```{.bash data-prompt="$"}
    $ valkey-cli
    ```

    ??? example "Expected output"

        ```text
        127.0.0.1:6379>
        ```

## Docker

If Valkey runs in a Docker container, run `valkey-cli` inside the container:

1. Check the connection:

    ```{.bash data-prompt="$"}
    $ docker exec valkey valkey-cli ping
    ```

    ??? example "Expected output"

        ```text
        PONG
        ```

2. Start interactive mode:

    ```{.bash data-prompt="$"}
    $ docker exec -it valkey valkey-cli
    ```

## Useful links

* [Valkey documentation](https://valkey.io/topics/)
* [Valkey command reference](https://valkey.io/commands/)
