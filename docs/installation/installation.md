# Install Valkey

Percona provides Valkey packages for the following Linux distributions:

* Oracle Linux 8, Rocky Linux 8, and Alma Linux 8
* Oracle Linux 9, Rocky Linux 9, and Alma Linux 9
* Oracle Linux 10, Rocky Linux 10, and Alma Linux 10
* Amazon Linux 2023
* Ubuntu 22.04
* Ubuntu 24.04
* Ubuntu 26.04
* Debian 11
* Debian 12
* Debian 13

The packages are available for x86_64 and ARM64 architectures.

Percona also provides Valkey as a Docker image for x86_64 and ARM64.

## Prerequisites

1. To install Valkey from packages, you need access to the Percona repositories. Use the [`percona-release`](https://docs.percona.com/percona-software-repositories/index.html) repository management tool to enable the required repository.
2. Valkey and Redis use conflicting libraries. If Redis is already installed on the host, remove it before installing Valkey, or install Valkey on a separate host.

## Installation methods

Choose the installation method that best fits your environment:

* [Install from Percona repositories](install-package.md)
* [Install using Docker](install-docker.md)
* [Install from Package Repository](install-repo-pack.md)

## Next steps

[Connect to Valkey :material-arrow-right:](connect.md){ .md-button }
