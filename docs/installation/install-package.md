# Install from Percona repositories

## Prerequisites

1. You need access to the Percona repositories. Use the [`percona-release`](https://docs.percona.com/percona-software-repositories/index.html) repository management tool to enable the required repository.
2. Valkey and Redis use conflicting libraries. If Redis is already installed on the host, remove it before installing Valkey, or install Valkey on a separate host.

## Install Valkey

The following steps install Valkey {{ valkey_version }} from the `{{ valkey_repo }}` repository. Each Valkey major and minor version has its own repository, named `valkey-<major><minor>`. To install a different version, replace `{{ valkey_repo }}` with the matching repository name, for example, `valkey-91` for Valkey 9.1.

Run the following commands as a root user or with `sudo`. Select your operating system:

=== "Debian / Ubuntu"

    1. Install the tools required to fetch and install `percona-release`:

        ```{.bash data-prompt="$"}
        $ sudo apt update
        $ sudo apt install -y wget gnupg2 lsb-release
        ```

    2. Fetch the `percona-release` package:

        ```{.bash data-prompt="$"}
        $ wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
        ```

    3. Install the downloaded package with `dpkg`:

        ```{.bash data-prompt="$"}
        $ sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
        ```

    4. Enable the repository:

        ```{.bash data-prompt="$"}
        $ sudo percona-release enable {{ valkey_repo }} release
        ```

    5. Update the local cache:

        ```{.bash data-prompt="$"}
        $ sudo apt update
        ```

    6. Install Valkey:

        ```{.bash data-prompt="$"}
        $ sudo apt install percona-valkey-server
        ```

        The installation enables and starts the `valkey-server` service.

    7. Check the Valkey service:

        ```{.bash data-prompt="$"}
        $ sudo systemctl status valkey-server
        ```

=== "RHEL-based distributions"

    Use these steps on Oracle Linux, Rocky Linux, Alma Linux, and Amazon Linux 2023.

    1. Install `percona-release`:

        ```{.bash data-prompt="$"}
        $ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
        ```

    2. Enable the repository:

        ```{.bash data-prompt="$"}
        $ sudo percona-release enable {{ valkey_repo }} release
        ```

    3. Install Valkey:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-valkey
        ```

    4. Start the Valkey service and enable it to start at boot:

        ```{.bash data-prompt="$"}
        $ sudo systemctl enable --now valkey@default
        ```

    5. Check the Valkey service:

        ```{.bash data-prompt="$"}
        $ sudo systemctl status valkey@default
        ```

## Next steps

[Connect to Valkey :material-arrow-right:](connect.md){ .md-button }
