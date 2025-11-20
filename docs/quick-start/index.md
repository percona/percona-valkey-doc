# QuickStart guide

## Prerequisites  

- Supported Linux (x86_64 or ARM64)  
- `curl` or `wget`

## Install percona-release

Fetch the percona-release package and install it:

```bash
$ wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb`
$ sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
```

Enable and update the repository:

```bash
$ sudo percona-release enable valkey experimental
$ sudo apt update
```

## Install Valkey

```bash
$ sudo apt install valkey
```

## Start the server

## Connect to Valkey

In the valkey-cli, pass the following command:

```bash
$ valkey-cli ping
PONG
```

## 1. Introduction  
**Audience**  
- **Beginner**: What Valkey is, why Percona supports it  
- **Developer**: High-level architecture, supported data types  
- **Manager**: Business use-cases, ROI  
- **Leader**: Strategic value, community backing  

### 1.1 What Is Valkey?  
### 1.2 Why Percona?  
### 1.3 Quick Links  
- Upstream Quick Start: https://valkey.io/topics/quickstart/  
- Percona docs repo: https://github.com/percona/percona-valkey-doc  
- Valkey upstream docs: https://github.com/valkey-io/valkey-doc  

## 2. Prerequisites  
**Audience**  
- **Beginner**: local machine requirements  
- **Developer**: build/runtime deps  
- **Manager**: licensing/support snapshot  
- **Leader**: compliance, SLAs  

### 2.1 System Requirements  
### 2.2 Getting Percona Support  

## 3. Installation  
**Audience**  
- **Beginner**: one-line install with Docker  
- **Developer**: RPM/DEB or source-build  
- **Manager**: version matrix, upgrade policy  
- **Leader**: enterprise add-ons  

### 3.1 Docker (Quickest)  
```bash
docker pull valkey/valkey:latest
docker run --rm -p 6379:6379 valkey/valkey:latest
