# Install from Percona Repositories

To install Valkey from Percona Repositories, select your operating system:

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
         $ sudo percona-release enable valkey-91 release
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
        $ sudo percona-release enable valkey-91 release
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

## Next steps

[Connect to Valkey :material-arrow-right:](connect.md){ .md-button }
