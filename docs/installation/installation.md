# Install Valkey

Percona provides Valkey for all major RPM and DEB distributions, making it easy to install and run Valkey on your system.

## Before you begin

Before installing Valkey:

1. Subscribe to Percona repositories using the [percona-release repository management tool](https://docs.percona.com/percona-software-repositories/index.html). This tool automatically enables the necessary repository and installs all the necessary dependencies saving you from resolving dependency conflicts during the installation process.
2. Valkey and Redis use conflicting libraries. If Redis is already installed on the host, remove it before installing Valkey, or install Valkey on a separate system.

## Installation methods

Choose the installation method that best fits your environment:

* [Install from Percona repositories](install-package.md)
* [Install using Docker](install-docker.md)
* [Install from Package Repository](install-repo-pack.md)

## Next steps

[Connect to Valkey :material-arrow-right:](connect.md){ .md-button }
