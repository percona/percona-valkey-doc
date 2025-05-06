# Install Valkey from Percona repositories

Valkey offers packages for a select number of Linux distributions. Specifically, you can install it from Fedora/EPEL yum repositories and as binary tarballs for both Ubuntu Bionic and Ubuntu Focal. Additionally, it is available as a Docker container.

To support Valkey’s development and adoption, Percona provides packages for all major active Linux distributions, making it easy for you to install Valkey on your system.

The packages are available for both x86_64 and ARM64 architectures for the following operating systems:

* Oracle Linux 8, Rocky Linux 8 and Alma Linux 8
* Oracle Linux 9, Rocky Linux 9 and Alma Linux 9
* Ubuntu 20.04
* Ubuntu 22.04
* Ubuntu 24.04
* Debian 11
* Debian 12

## Preconditions

1. To install the software, you will need to subscribe to Percona repositories. To do this, use the [percona-release repository management tool](https://docs.percona.com/percona-software-repositories/index.html) that automatically enables the necessary repository. The software and all required dependencies are available in this repository, saving you from resolving dependency conflicts during the installation process.
2. Both Valkey and Redis use the same libraries that conflict with each other when you try to install Valkey on the same host with Redis. Therefore, either install Valkey on another host or remove Redis first before you install Valkey.

## Install Valkey

=== "Install on Debian / Ubuntu"

    Run the following command as a root user or using the sudo privileges.

    1. Install `percona-release`:
        
        a. Fetch `percona-release` packages from Percona web:
        
           ```{.bash data-prompt="$"}
           $ wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
           ```    

        b. Install the downloaded package with `dpkg`:    

           ```{.bash data-prompt="$"}
           $ sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
           ```    

           After you install this package, you have the access to Percona repositories.    
    
    2.  Enable the repository:    

         ```{.bash data-prompt="$"}
         $ sudo percona-release enable valkey experimental
         ```    

    3. Remember to update the local cache:    

        ```{.bash data-prompt="$"}
        $ sudo apt update
        ```

    4. Install Valkey
     
        ```{.bash data-prompt="$"}
        $ sudo apt install valkey
        ```

=== "Install on Oracle Linux"

    1. Install `percona-release`

        ```{.bash data-prompt="$"}
        $ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
        ```
    
    2. Enable the repository

        ```{.bash data-prompt="$"}
        $ sudo percona-release enable valkey experimental
        ```
    
    3. Install Valkey

        ```{.bash data-prompt="$"}
        $ sudo yum install valkey
        ```
    
    4. Upon installation, Valkey is not started automatically. To start it, run the following command:

        ```{.bash data-prompt="$"}
        $ sudo systemctl start valkey
        ```
    
    5. Check Valkey's status:

        ```{.bash data-prompt="$"}
        $ sudo systemctl status valkey
        ```

## Connect to Valkey

With Valkey up and running, you can now connect to it using the `valkey-cli` interface. Check if Valkey is correctly running by passing the following command:

```bash
    $ valkey-cli ping
    PONG
```

You can pass `valkey-cli` without any argument. You should see the following prompt:

```
127.0.0.1:6379> 
```

This way you connect to Valkey in interactive mode.  
