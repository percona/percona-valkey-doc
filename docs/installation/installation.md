# Install Valkey

Percona provides Valkey packages for all major active Linux distributions, making it easy to install and run Valkey on your system.

The packages are available for both x86_64 and ARM64 architectures on the following operating systems:

* Oracle Linux 8, Rocky Linux 8 and Alma Linux 8
* Oracle Linux 9, Rocky Linux 9 and Alma Linux 9
* Oracle Linux 10, Rocky Linux 10 and Alma Linux 10
* Ubuntu 22.04
* Ubuntu 24.04
* Ubuntu 26.04
* Debian 11
* Debian 12
* Debian 13

## Before you begin

Before installing Valkey:

1. Subscribe to Percona repositories using the [percona-release repository management tool](https://docs.percona.com/percona-software-repositories/index.html). This tool automatically enables the necessary repository and installs all the necessary dependencies saving you from resolving dependency conflicts during the installation process.
2. Valkey and Redis have conflicting libraries. When you try to install Valkey on the same host with Redis. If Redis is already installed on the host, remove it before installing Valkey, or install Valkey on a separate system.

## Installation methods

Choose the installation method that best fits your environment:

* [Install from Percona repositories](install-package.md)
* [Install using Docker](install-docker.md)

## Next steps

[Connect to Valkey :material-arrow-right:](connect.md){ .md-button }
