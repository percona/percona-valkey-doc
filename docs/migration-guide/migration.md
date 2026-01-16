# Migration from Redis to Valkey

This is a migration guide from Redis Community Edition to Valkey.
You will learn how to migrate a standalone Redis server instance and a Redis Cluster.

This guide provides migration steps for Redis server and Valkey deployed in Docker; however, the principles are the same for any deployment method.

!!! note
    For more information on installation options, see [install Valkey](../get-started/installation.md).

## Why to migrate to Valkey?

* Valkey is the vendor-neutral and open-source software
* Enhanced performance with multi-threading and dual-channel replication
* Improved memory efficiency by using one dictionary per slot in cluster mode and embedding keys in dictionaries.

### Migration compatibility matrix

You can migrate Redis server to Valkey.
The following table provides migration options depending on the Redis version you run:

| Redis | Valkey |
|-------|--------|
| 6.x   | 7.2.x  |
| 7.2.x | 7.2.x  |
| 7.2.x | 8.0.x  |
| 7.4   | n/a    |

## Migrate a standalone instance

To migrate a standalone Redis server to Valkey, you have the following options:

* [Physical migration](#physical-migration) by copying the most recent on-disk snapshot from the Redis server to Valkey and starting Valkey server with it
* [Setting up replication](#replication) between Redis and Valkey
* [Migrating specific keys](#migrate-specific-keys)

The example migration steps are provided for Redis 7.2.5 and Valkey version 7.2.6.

### Physical migration

This is the easiest and fastest migration method. You make a fresh snapshot of your Redis instance and copy it over to Valkey. Valkey reads the data from the snapshot on startup and restores its contents into memory. The tradeoffs for this method are:

* The downtime to shutdown Redis and wait for Valkey to load the data.
* Potential risk of data loss on instances with heavy writes. To prevent it, you must disconnect all active connections before the migration start.

The migration steps are the following:

1. Disconnect all active connections to the Redis instance.

2. Make a backup of your Redis instance.

    a. Connect to Redis and check the number of keys:
        ```
        $ redis-cli -h 127.0.0.1 -p 6379
        redis> INFO KEYSPACE
        # Keyspace
        db0:keys=6286,expires=0,avg_ttl=0
        ```

    b. Check the configuration for the directory (`dir`) where Redis stores its database files and the name of the database file (`dbfilename`):
       ```
       redis> CONFIG GET dir dbfilename
       1) "dir"
       2) "/data"
       3) "dbfilename"
       4) "dump.rdb"
       ```

    c. Check the timestamp of the last save operation
       ```
       redis> LASTSAVE
       (integer) 1724764878
       ```

    d. Start a backup
       ```
       redis> BGSAVE
       Background saving started
       ```

    e. Check if the backup succeeded by running `LASTSAVE` periodically. The backup ended when the timestamp changes.

    f. Exit the `redis-cli` by pressing `Ctrl-D`.

2. Stop the Redis server.
3. Copy the RDB file to the Valkey's data directory and start Valkey.

    >NOTE: If you enabled AOF in your Valkey configuration, disable it on the first start. Otherwise, the copied RDB file will not be imported into Valkey.

    For Docker deployments, copy the RDB file to your host and start a Valkey container mounting this file to the container's data directory. Replace the `<container-name>` and `<path/on/host>` placeholders with your values.

    ```
    $ docker cp <container-name>:/data/dump.rdb <path/on/host>
    ```

    Start Valkey:

    ```
    $ docker run -d --name somevalkey -v <path/on/host>:/data valkey/valkey
    ```

4. Check the keyspace on Valkey to verify that the data is migrated:

    ```
    $ docker exec -it somevalkey valkey-cli
    valkey> INFO KEYSPACE
    # Keyspace
    db0:keys=6286,expires=0,avg_ttl=0
    ```

5. To exit `valkey-cli`, press `Ctrl-D`.

### Replication

To minimize the downtime during migration, you can use replication. Both Redis and Valkey allow replaying data on another server to handle the workload.

In this scenario we will configure Valkey to be the replica of Redis. For illustrative purposes, both Redis and Valkey are running in separate Docker containers connected to the same network.

1. Retrieve the IP address of Redis container. Replace the `myredis` placeholder with the name of your container.

    {% raw %}
    ```
    $ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' myredis
    ```
    {% endraw %}

2. Connect to Valkey and configure replication. Replace the IP address and port with the ones of your Redis container, retrieved at the previous step:

    ```
    $ docker exec -it myvalkey valkey-cli
    valkey> REPLICAOF 172.17.0.2 6379
    ```

3. Check the replication status in Valkey.

    ```
    valkey> INFO REPLICATION
    # Replication
    role:slave
    master_host:172.21.0.4
    master_port:6379
    master_link_status:up
    master_last_io_seconds_ago:-1
    master_sync_in_progress:0
    ....
    ```

4. Now that the two are in sync, check that your applications connect to Valkey and shut down your Redis instance.

   You can shut down Redis in one of the following ways:

   * Using `redis-cli`:

      ```
      $ redis-cli shutdown
      ```

   * If Redis was started directly in the foreground (using redis-server), you can simply stop it by pressing `Ctrl-C` in the terminal where it is running.

5. Halt Valkey replication:

    ```
    valkey> REPLICAOF NO ONE
    OK
    ```

!!! note
    If not for backward compatibility, the Valkey project no longer uses the words "master" and "slave". Unfortunately in this command these words are part of the protocol, so we’ll be able to remove such occurrences only when this API will be naturally deprecated.

### Migrate specific keys

Both physical migration and replication migrate the entire keyspace over to Valkey.

There may be cases when you need to migrate only specific set of critical keys.
To do that, use the `MIGRATE` command.

For the following steps, we assume that both Redis and Valkey Docker containers are connected to the same network and can communicate with each other.
For simplicity, we are running both instances without authentication.

1. Connect to Redis and set the keys you wish to migrate over. Replace `myredis` with the name of your container.
   For example, let's use the keys `message` and  `mydata`.

    ```
    $ docker exec -it myredis redis-cli
    redis> SET message "Hello Valkey!"
    redis> HSET  mydata name Alice age 33 country Brazil "favorite food" beans
    (integer) 4
    ```

2. Retrieve the IP address of your Valkey container
    
    {% raw %}
    ```
    $ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' myvalkey
    ```
    {% endraw %}

3. From the Redis server, run the `MIGRATE` command and specify the following information:

     * The IP address and post of the Valkey instance
     * Authentication password. For no authentication, set the empty value 
     * The database number. To get the number, run the `INFO keyspace` command
     * Timeout for the operation, in milliseconds
     * The `COPY` option ensures that the key is not removed from the source instance after the migration.
     * The `REPLACE` option allows the key on the destination instance to be replaced if it already exists.
     * `KEYS`. Specify the key(s) to migrate

     For example, to migrate the `message` and `mydata` keys to the Valkey instance with the IP address 172.21.0.3, the command looks as follows:

    ```
    redis> MIGRATE 172.21.0.3 6379 "" 0 10 COPY REPLACE KEYS message mydata
    ```

4. Exit `redis-cli` by pressing `Ctrl-D`

4. Connect to Valkey and check the migrated keys. replace `myvalkey` with the name of your Valkey container.

    ```
    $ docker exec -it myvalkey valkey-cli
    valkey> GET message
    "Hello Valkey"
    valkey> HGETALL mydata
    1) "name"
    2) "Alice"
    3) "age"
    4) "33"
    5) "country"
    6) "Brazil"
    7) "favorite food"
    8) "beans"
    ```
