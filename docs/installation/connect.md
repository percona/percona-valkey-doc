# Connect to Valkey

After installing and starting Valkey, verify that the server is running:

1. Check connectivity:

    ```bash
    valkey-cli ping
    PONG
    ```

2. Start an interactive session:

    ```bash
    valkey-cli
    ```

You can pass `valkey-cli` without any argument. You should see the following prompt:

```bash {.no-copy}
127.0.0.1:6379> 
```

## Docker

If Valkey is running in a Docker container:

1. Run `valkey-cli` inside the container:

    ```bash
    docker exec -it <container-name> valkey-cli ping
    ```

2. Start an interactive session:

    ```bash
    docker exec -it <container-name> valkey-cli
    ```

Congratulations, you are now connected to your Valkey server!
