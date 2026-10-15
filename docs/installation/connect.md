# Connect to Valkey

After installing and starting Valkey, verify that the server is running:

1. Check connectivity:

    ```{.bash data-prompt="$"}
    valkey-cli ping
    PONG
    ```

    ```bash {.no-copy}
    PONG
    ```

2. Start an interactive session:

    ```{.bash data-prompt="$"}
    valkey-cli
    ```

You can pass `valkey-cli` without any argument. You should see the following prompt:

```bash {.no-copy}
127.0.0.1:6379> 
```

## Docker

If Valkey is running in a Docker container:

1. Run `valkey-cli` inside the container:

    ```{.bash data-prompt="$"}
    docker exec valkey valkey-cli ping
    ```
  
2. Start an interactive session:

    ```{.bash data-prompt="$"}
    docker exec -it valkey valkey-cli
    ```

Congratulations, you are now connected to your Valkey server!
