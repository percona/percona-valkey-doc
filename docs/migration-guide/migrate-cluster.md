# Migrate a Cluster

To migrate a Redis Cluster to Valkey, you have the following options:

* Add the required number of Valkey nodes to the existing cluster as replicas, wait till they replicate the data and promote one Valkey replica for each Redis primary to be a new primary. After the migration, you can remove Redis nodes.
* Use data migration tools like [RIOT](https://redis.github.io/riot/#_introduction), [RedisShake](https://github.com/tair-opensource/RedisShake), [Redis-Migrate-Tool](https://github.com/vipshop/redis-migrate-tool) and the like.

This section demonstrates migration via adding Valkey nodes and replacing Redis ones.

For this scenario, we assume that you have Redis Cluster consisting of 3 primary and 3 replica nodes up and running. Refer to [Create a Redis cluster](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/#create-a-redis-cluster) documentation for steps how to create a Cluster. Also, make sure that both Redis Cluster nodes and Valkey nodes are connected to the same network and can communicate with each other.

1. Check the cluster state

    ```
    $ redis-cli -h 127.0.0.1 -p 6379 -c cluster nodes
    70beedebe43e422b275ee1a7bac0d3819dedca98 172.22.0.3:6379@16379 master - 0 1725450849000 1 connected 0-5460
    8bbe836c59644f7395bbab09c6f8b36bc277e902 172.22.0.5:6379@16379 slave 58061fb2836bdb2f5a0973e1ccfd74a66166f329 0 1725450849510 3 connected
    65061b94da5b481dc35c2df7dae13c233d4b3ad2 172.22.0.4:6379@16379 master - 0 1725450848000 2 connected 5461-10922
    a242941d0e3edad27a954bc14ac3a3413f3040aa 172.22.0.7:6379@16379 slave 65061b94da5b481dc35c2df7dae13c233d4b3ad2 0 1725450849000 2 connected
    3499854656f085ebb77b5b921389a91b7ae9d703 172.22.0.6:6379@16379 slave 70beedebe43e422b275ee1a7bac0d3819dedca98 0 1725450849829 1 connected
    58061fb2836bdb2f5a0973e1ccfd74a66166f329 172.22.0.2:6379@16379 myself,master - 0 1725450846000 3 connected 10923-16383
    ```

2. Enable cluster mode in Valkey configuration. Create a `valkey.conf` configuration file and specify the following directives as the starting point:

    ```
    # valkey.conf file
    port 6379
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
    appendonly yes
    ```

    Read more about Valkey configuration parameters in [the self documented Valkey.conf for 7.2](https://raw.githubusercontent.com/valkey-io/valkey/7.2/valkey.conf)

3. Start a Valkey instance with your custom configuration file. The following command starts Valkey in Docker:

    ```
    $ docker run  -d -v myvalkey/conf:/usr/local/etc/valkey --name valkey-1 --net mynetwork valkey/valkey valkey-server /usr/local/etc/valkey/valkey.conf
    ```

    where `myvalkey/conf` is a local directory containing your `valkey.conf` configuration file, `valkey-1` is the name of your container and `mynetwork` is the name of the network where Redis cluster is running.

4. Retrieve the IP address of the Valkey instance; replace `valkey-1` with the name of your container.

    {% raw %}
    ```
    $ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' valkey-1
    ```
    {% endraw %}

5. Add a Valkey node to the Redis Cluster as a replica. Replace placeholders in angle brackets (`<>`) with your values:

    ``` 
    $ docker exec -it <redis-1> bash
    $ redis-cli --cluster add-node <valkey-node-IP>:6379 <existing-node-IP>:6379 --cluster-slave
    ```

6. Check the cluster status

    ```
    $ redis-cli -c cluster nodes
    ```

    In the output you will see the newly added nodes. For example, we have added a Valkey node with the IP address `172.22.0.8:6379`. The cluster nodes list now includes a new entry as follows:

    ```
    a98d5bac59672597b8509f24970e413002f896b6 172.22.0.8:6379@16379 slave 58061fb2836bdb2f5a0973e1ccfd74a66166f329 0 1725451086000 3 connected
    ```

7. Check that the newly added node is recognized as a replica by running the `INFO REPLICATION` command. The output shows the information about the node's primary.

8. Promote it to be a new primary. Run the following command **from the node you wish to promote**:

    ```
    valkey> cluster failover
    OK
    ```

9. Check the cluster state. You should now see the previous replica to be a new primary.

10. Repeat steps 3-9 to add 2 more Valkey nodes and replace the Redis primary nodes.
11. Repeat steps 3-7 to add 3 Valkey replica nodes.

    To add a replica to a specific primary, do the following:

    a. Filter primary nodes. Connect to any node in the Cluster and run the following command:

       ```
       $ docker exec -it valkey-1 bash
       $ valkey-cli -c cluster nodes | grep master
       70beedebe43e422b275ee1a7bac0d3819dedca98 172.22.0.3:6379@16379 master - 0 1725451135799 1 connected 0-5460
       65061b94da5b481dc35c2df7dae13c233d4b3ad2 172.22.0.4:6379@16379 master - 0 1725451136000 2 connected 5461-10922
       58061fb2836bdb2f5a0973e1ccfd74a66166f329 172.22.0.2:6379@16379 myself,master - 0 1725451136000 3 connected 10923-16383
       ```

    b. Add a new node to a specific primary:

      ```
      $ valkey-cli --cluster add-node 172.22.0.10:6379 172.22.0.2:6379 --cluster-slave --cluster-master-id <node-ID>
      ```
    
12. Remove Redis nodes:

     ```
     redis-cli --cluster del-node 127.0.0.1:7000 `<node-id>`
     ```

     The first argument is just a random node in the cluster, the second argument is the ID of the node you want to remove.

!!! note
    If not for backward compatibility, the Valkey project no longer uses the words "master" and "slave". Unfortunately in the given commands these words are part of the protocol, so we’ll be able to remove such occurrences only when this API will be naturally deprecated.
