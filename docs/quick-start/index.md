# Quickstart guide

**Valkey** is an open-source high-performance in-memory data store, using official packages, containers, or by building from source. This guide explains how to install Valkey on supported Linux distributions and containerized environments using Percona-recommended methods.

Valkey is fully compatible with Redis clients and protocols, making it a drop-in replacement for most Redis-based use cases.

Valkey supports scenarios such as:

- Caching frequently accessed data
- Session management
- Real-time analytics
- Pub/sub systems and background queueing

!!! note
    These instructions are intended for Linux-based systems. For other platforms, consult the [Valkey community documentation](https://valkey.io/docs/).

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
