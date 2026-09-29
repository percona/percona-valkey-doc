# Install Valkey from Percona repositories

Percona provides Valkey packages for the following Linux distributions:

* Oracle Linux 8, Rocky Linux 8, and Alma Linux 8
* Oracle Linux 9, Rocky Linux 9, and Alma Linux 9
* Oracle Linux 10, Rocky Linux 10, and Alma Linux 10
* Ubuntu 22.04
* Ubuntu 24.04
* Ubuntu 26.04
* Debian 11
* Debian 12
* Debian 13

The packages are available for x86_64 and ARM64 architectures.

## Prerequisites

1. To install Valkey, you need access to the Percona repositories. Use the [`percona-release`](https://docs.percona.com/percona-software-repositories/index.html) repository management tool to enable the required repository.
2. Both Valkey and Redis use the same libraries that conflict with each other when you try to install Valkey on the same host with Redis. Therefore, either install Valkey on another host or remove Redis first before you install Valkey.

## Install Valkey

=== "Install on Debian / Ubuntu"

    Run the following commands as a root user or with `sudo`.

    1. Install `percona-release`:
        
        a. Fetch `percona-release` packages from Percona web:
        
           ```{.bash data-prompt="$"}
           $ wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
           ```    

        b. Install the downloaded package with `dpkg`:    

           ```{.bash data-prompt="$"}
           $ sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
           ```    

        After you install this package, you can access the Percona repositories. 
    
    2.  Enable the repository:    

         ```{.bash data-prompt="$"}
         $ sudo percona-release enable valkey-91 release
         ```    

    3. Remember to update the local cache:    

        ```{.bash data-prompt="$"}
        $ sudo apt update
        ```

    4. Install Valkey:
     
        ```{.bash data-prompt="$"}
        $ sudo apt install percona-valkey-server
        ```

=== "Install on Oracle Linux"

    1. Install `percona-release`:

        ```{.bash data-prompt="$"}
        $ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
        ```

    2. Enable the repository:

        ```{.bash data-prompt="$"}
        $ sudo percona-release enable valkey-91 release
        ```

    3. Install Valkey:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-valkey
        ```

    4. Start the Valkey service:

        ```{.bash data-prompt="$"}
        $ sudo systemctl start valkey
        ```

    5. Check the Valkey service:

        ```{.bash data-prompt="$"}
        $ sudo systemctl status valkey
        ```

## Connect to Valkey

Use `valkey-cli` to connect to the Valkey server.

Check the connection:

```bash
$ valkey-cli ping
PONG
```

Run `valkey-cli` without any argument to start interactive mode:

```bash 
$ valkey-cli
127.0.0.1:6379>
```

For information about Valkey, see the [Valkey documentation website](https://valkey.io/topics/).

For information about Valkey commands, see the [Valkey command reference](https://valkey.io/commands/).
