# ☁️ ClusteringServers — Private Cloud Data Warehouse Platform

> **A self-built private cloud using five physical laptops, K3s, Kubernetes workloads, PostgreSQL, ETL pipelines, persistent storage, monitoring, failure testing, and CI/CD.**

[![K3s](https://img.shields.io/badge/K3s-Kubernetes-blue?logo=kubernetes)](https://k3s.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Container%20Orchestration-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![Linux](https://img.shields.io/badge/Linux-Ubuntu%20%2F%20Kali-FCC624?logo=linux)](https://www.linux.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-336791?logo=postgresql)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Containers-2496ED?logo=docker)](https://www.docker.com/)
[![GitHub](https://img.shields.io/badge/GitHub-Version%20Control-181717?logo=github)](https://github.com/)
[![Status](https://img.shields.io/badge/Project-Completed-success)](#project-status)

---

## 📌 Table of Contents

* [Project Overview](#-project-overview)
* [Why I Built This](#-why-i-built-this)
* [Project Goals](#-project-goals)
* [Architecture](#-architecture)
* [Hardware Infrastructure](#-hardware-infrastructure)
* [Network Topology](#-network-topology)
* [Node Roles](#-node-roles)
* [Technology Stack](#-technology-stack)
* [Project Phases](#-project-phases)
* [Phase 1 — Infrastructure Preparation](#phase-1--infrastructure-preparation)
* [Phase 2 — K3s Cluster](#phase-2--k3s-cluster)
* [Phase 3 — Containerization](#phase-3--containerization)
* [Phase 4 — Data Warehouse Deployment](#phase-4--data-warehouse-deployment)
* [Phase 5 — Storage and Recovery](#phase-5--storage-and-recovery)
* [Phase 6 — Monitoring and Logging](#phase-6--monitoring-and-logging)
* [Phase 7 — Failure Testing](#phase-7--failure-testing)
* [Phase 8 — CI/CD and Automation](#phase-8--cicd-and-automation)
* [Data Warehouse](#-data-warehouse)
* [Kubernetes Components](#-kubernetes-components)
* [Storage Architecture](#-storage-architecture)
* [Security](#-security)
* [Verification](#-verification)
* [Failure Scenarios](#-failure-scenarios)
* [Repository Structure](#-repository-structure)
* [Important Commands](#-important-commands)
* [Troubleshooting](#-troubleshooting)
* [Lessons Learned](#-lessons-learned)
* [Future Improvements](#-future-improvements)
* [Project Status](#-project-status)
* [Skills Demonstrated](#-skills-demonstrated)
* [Author](#-author)

---

# 🚀 Project Overview

**ClusteringServers** is a self-built private-cloud and data-platform project created to understand how distributed infrastructure works from the hardware and networking layer up to application deployment.

Instead of relying completely on public cloud providers, I built a small private cloud using **five physical laptops** connected through a local network.

The infrastructure uses:

* Linux
* SSH
* UFW
* Network configuration
* K3s
* Kubernetes
* Docker/container images
* PostgreSQL
* Persistent Volumes
* Persistent Volume Claims
* ETL workloads
* Monitoring
* Logging
* Failure testing
* Git/GitHub
* CI/CD concepts

The final objective is to deploy an existing **S&P 500 Data Warehouse** into the private Kubernetes environment.

---

# 🎯 Why I Built This

Cloud infrastructure is often introduced through managed services such as:

* AWS
* Microsoft Azure
* Google Cloud
* Oracle Cloud

While these platforms are extremely useful, they can hide many of the underlying infrastructure concepts.

This project was created to understand those concepts directly.

I wanted to answer questions such as:

* How does a Kubernetes cluster actually communicate?
* How does a worker join a control plane?
* How are workloads scheduled?
* How does networking work between nodes?
* What happens when a node fails?
* How does persistent storage work?
* How can databases run inside a cluster?
* How can ETL pipelines become containerized workloads?
* How can applications be monitored?
* How can deployment be automated?
* How can a small collection of laptops behave like a private cloud?

This project became an attempt to build those concepts from the ground up.

---

# 🎯 Project Goals

The major goals were:

### Infrastructure

* Build a multi-node Linux environment
* Configure hostnames
* Configure networking
* Configure SSH
* Configure firewall rules
* Synchronize system time

### Kubernetes

* Install K3s
* Create a Kubernetes control plane
* Join worker nodes
* Verify cluster health
* Deploy workloads
* Understand scheduling

### Data Platform

* Deploy PostgreSQL
* Create persistent storage
* Deploy the S&P 500 Data Warehouse
* Run ETL workloads
* Validate data

### Operations

* Monitor workloads
* Inspect logs
* Test node failures
* Test workload failures
* Recover services
* Understand operational behavior

### Automation

* Containerize workloads
* Use GitHub
* Build container images
* Push images to a registry
* Deploy through Kubernetes
* Implement CI/CD concepts

---

# 🏗️ Architecture

## High-Level Architecture

```text
                         PRIVATE CLOUD
                              │
                              ▼
                    ┌───────────────────┐
                    │   K3s Control     │
                    │      Plane        │
                    │  cluster-master   │
                    └─────────┬─────────┘
                              │
               ┌──────────────┼──────────────┐
               │              │              │
               ▼              ▼              ▼
        ┌────────────┐ ┌────────────┐ ┌────────────┐
        │  Worker 1  │ │  Worker 2  │ │  Server 4  │
        │    Kali    │ │    Kali    │ │    Kali    │
        └────────────┘ └────────────┘ └────────────┘
                              │
                              ▼
                       ┌────────────┐
                       │  Server 5  │
                       │    Kali    │
                       └────────────┘
                              │
                              ▼
                ┌─────────────────────────┐
                │   Cloud Applications    │
                ├─────────────────────────┤
                │ PostgreSQL              │
                │ Data Warehouse          │
                │ ETL                     │
                │ Monitoring               │
                │ Logging                  │
                └─────────────────────────┘
```

---

# 💻 Hardware Infrastructure

The private cloud is built from five physical laptop systems.

| Node              | OS     | CPU               |    RAM | Primary Role      |
| ----------------- | ------ | ----------------- | -----: | ----------------- |
| `cluster-master`  | Ubuntu | AMD A9-9425       |  ~3 GB | K3s Control Plane |
| `cluster-worker1` | Kali   | Intel i5-12500H   |  15 GB | Kubernetes Worker |
| `cluster-worker2` | Kali   | Ryzen 7 7435HS    |  23 GB | Kubernetes Worker |
| `server4`         | Kali   | Intel i3-13150    | ~11 GB | Scalable Server   |
| `server5`         | Kali   | Core Ultra 7 155H | ~16 GB | Scalable Server   |

The repository's architecture documentation identifies the control-plane machine as the weakest system and recommends keeping heavy workloads away from it. The stronger worker nodes are intended to handle compute and data workloads.

---

# 🌐 Network Topology

The private cloud operates on the local `192.168.1.0/24` network.

| Node              | IP Address    | Role              |
| ----------------- | ------------- | ----------------- |
| `cluster-master`  | `192.168.1.7` | K3s Control Plane |
| `cluster-worker1` | `192.168.1.2` | Worker            |
| `cluster-worker2` | `192.168.1.3` | Worker            |
| `server4`         | `192.168.1.8` | Scalable Server   |
| `server5`         | `192.168.1.4` | Scalable Server   |

The K3s documentation in the repository uses this five-node addressing scheme for connectivity and cluster preparation.

---

# 🧩 Node Roles

## 1. cluster-master

```text
Hostname: cluster-master
IP:       192.168.1.7
Role:     K3s Control Plane
OS:       Ubuntu
```

Responsibilities:

* Kubernetes API server
* Cluster management
* Scheduling decisions
* Cluster state
* Worker management

Because this machine has limited resources, heavy workloads should not be scheduled here.

---

## 2. cluster-worker1

```text
Hostname: cluster-worker1
IP:       192.168.1.2
Role:     Kubernetes Worker
OS:       Kali Linux
```

Primary purpose:

* Application workloads
* ETL
* Containers
* Compute tasks

---

## 3. cluster-worker2

```text
Hostname: cluster-worker2
IP:       192.168.1.3
Role:     Kubernetes Worker
OS:       Kali Linux
```

This is the strongest RAM node and is suitable for:

* PostgreSQL
* Data processing
* ETL
* Memory-intensive workloads

---

## 4. server4

```text
Hostname: server4
IP:       192.168.1.8
Role:     Scalable Server
OS:       Kali Linux
```

Used as additional cluster capacity and for experimentation with scale-out workloads.

---

## 5. server5

```text
Hostname: server5
IP:       192.168.1.4
Role:     Scalable Server
OS:       Kali Linux
```

Provides additional compute capacity and forms part of the scale-out architecture.

---

# 🛠️ Technology Stack

## Operating Systems

* Ubuntu Linux
* Kali Linux

## Infrastructure

* Physical laptops
* Local Ethernet/Wi-Fi network
* SSH
* UFW
* systemd
* NTP/time synchronization

## Container Orchestration

* K3s
* Kubernetes
* kubectl

## Containers

* Docker
* OCI container images
* Container registries

## Database

* PostgreSQL

## Data Engineering

* Python
* ETL pipeline
* S&P 500 dataset
* Data validation
* Data warehouse schema

## DevOps

* Git
* GitHub
* CI/CD
* Container image versioning
* Kubernetes deployments

## Monitoring

* Kubernetes resource inspection
* Pod logs
* Node status
* Workload status
* Failure testing

---

# 🗺️ Project Phases

The project was organized into eight major phases:

```text
Phase 1
Infrastructure
     │
     ▼
Phase 2
K3s Cluster
     │
     ▼
Phase 3
Containerization
     │
     ▼
Phase 4
Data Warehouse
     │
     ▼
Phase 5
Storage & Recovery
     │
     ▼
Phase 6
Monitoring & Logging
     │
     ▼
Phase 7
Failure Testing
     │
     ▼
Phase 8
CI/CD & Automation
```

---

# Phase 1 — Infrastructure Preparation

The first phase establishes the infrastructure required for the cluster.

## 1. Hostname Configuration

Each machine receives a unique hostname.

```bash
sudo hostnamectl set-hostname cluster-master
```

Worker:

```bash
sudo hostnamectl set-hostname cluster-worker1
```

Second worker:

```bash
sudo hostnamectl set-hostname cluster-worker2
```

Additional nodes:

```bash
sudo hostnamectl set-hostname server4
sudo hostnamectl set-hostname server5
```

---

## 2. Network Verification

```bash
hostname
hostname -I
ip addr
ip route
```

Connectivity testing:

```bash
ping -c 3 cluster-master
ping -c 3 cluster-worker1
ping -c 3 cluster-worker2
```

---

## 3. `/etc/hosts`

Static hostname resolution was configured where required.

Example:

```text
192.168.1.7 cluster-master
192.168.1.2 cluster-worker1
192.168.1.3 cluster-worker2
192.168.1.8 server4
192.168.1.4 server5
```

---

## 4. SSH Configuration

Generate an SSH key:

```bash
ssh-keygen -t ed25519
```

Copy the public key:

```bash
ssh-copy-id username@192.168.1.2
```

Test:

```bash
ssh cluster-worker1
```

The purpose is to establish passwordless and secure administrative access between nodes.

---

## 5. Firewall

UFW was used to control network access.

```bash
sudo ufw allow 22/tcp
```

Allow the private network:

```bash
sudo ufw allow from 192.168.1.0/24
```

Enable:

```bash
sudo ufw enable
```

Check:

```bash
sudo ufw status
```

---

## 6. Time Synchronization

Kubernetes nodes require consistent system time.

Check:

```bash
timedatectl status
```

Enable NTP:

```bash
sudo timedatectl set-ntp true
```

Configure the hardware clock to UTC:

```bash
sudo timedatectl set-local-rtc 0
```

Verify:

```bash
date
```

The Phase 1 documentation explicitly includes networking, SSH, firewall configuration, and time synchronization as the infrastructure foundation.

---

# Phase 2 — K3s Cluster

Phase 2 converts the collection of Linux systems into a Kubernetes cluster.

---

## 1. Check Hardware

Memory:

```bash
free -h
```

CPU:

```bash
lscpu
```

Disk:

```bash
df -h /
```

---

## 2. Verify Connectivity

From the control plane:

```bash
ping -c 4 192.168.1.2
ping -c 4 192.168.1.3
ping -c 4 192.168.1.8
ping -c 4 192.168.1.4
```

SSH:

```bash
ssh cluster-worker1
ssh cluster-worker2
ssh server4
ssh server5
```

---

## 3. Install K3s

On the control plane:

```bash
curl -sfL https://get.k3s.io | sh -
```

Check service:

```bash
sudo systemctl status k3s
```

---

## 4. Obtain the K3s Token

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

The token is required for workers to authenticate with the K3s server.

---

## 5. Join Worker Nodes

Example:

```bash
curl -sfL https://get.k3s.io | \
K3S_URL=https://192.168.1.7:6443 \
K3S_TOKEN='YOUR_TOKEN_HERE' \
sh -
```

---

## 6. Verify Cluster

```bash
sudo kubectl get nodes
```

Detailed:

```bash
sudo kubectl get nodes -o wide
```

Expected architecture:

```text
cluster-master
       │
       ├── cluster-worker1
       ├── cluster-worker2
       ├── server4
       └── server5
```

The repository documents `192.168.1.7:6443` as the K3s server endpoint and uses `kubectl get nodes -o wide` for verification.

---

# Phase 3 — Containerization

The application workloads are converted into containerized workloads.

The major objective is to remove dependency on the host environment.

Instead of:

```text
Host
 └── Python
      └── ETL
```

the architecture becomes:

```text
Kubernetes
    │
    └── Pod
         │
         └── Container
              │
              └── ETL Application
```

---

## Container Workflow

```text
Source Code
     │
     ▼
Dockerfile
     │
     ▼
Docker Image
     │
     ▼
Container Registry
     │
     ▼
K3s Cluster
     │
     ▼
Kubernetes Pod
```

Example:

```bash
docker build -t datawarehouse-etl:latest .
```

Run locally:

```bash
docker run datawarehouse-etl:latest
```

---

# Phase 4 — Data Warehouse Deployment

The existing S&P 500 Data Warehouse is deployed into the private Kubernetes environment.

## Data Flow

```text
S&P 500 Dataset
       │
       ▼
    Extract
       │
       ▼
   Transform
       │
       ▼
    Validate
       │
       ▼
      Load
       │
       ▼
   PostgreSQL
       │
       ▼
 Data Warehouse
       │
       ▼
 Analytics
```

---

# 📊 Data Warehouse

The project uses historical S&P 500 stock-market data.

The original warehouse pipeline contains:

* Raw data
* Extraction
* Transformation
* Validation
* Loading
* PostgreSQL storage
* Analytical queries

Important transformed columns include:

```text
date
open_price
high_price
low_price
close_price
volume
ticker
```

The warehouse contains fact and dimension-oriented structures designed for analytical workloads.

---

# 🐘 PostgreSQL on Kubernetes

PostgreSQL is deployed as a Kubernetes workload.

The basic architecture is:

```text
K3s Cluster
     │
     ▼
datawarehouse namespace
     │
     ├── PostgreSQL
     │
     ├── PersistentVolume
     │
     ├── PersistentVolumeClaim
     │
     ├── ConfigMap
     │
     └── ETL
```

---

# Phase 5 — Storage and Recovery

Persistent storage is essential for databases.

Without persistent storage:

```text
Pod deleted
     │
     ▼
Database data lost
```

With persistent storage:

```text
Pod deleted
     │
     ▼
New Pod
     │
     ▼
PersistentVolume
     │
     ▼
Existing database data
```

---

## Kubernetes Storage Components

### PersistentVolume

Represents storage available to Kubernetes.

### PersistentVolumeClaim

Represents the storage requested by an application.

### StorageClass

Defines how storage is provisioned.

The project uses Kubernetes persistent storage for PostgreSQL.

---

## Verify PVC

```bash
sudo kubectl get pvc -n datawarehouse
```

Detailed:

```bash
sudo kubectl describe pvc postgres-pvc \
-n datawarehouse
```

---

# Phase 6 — Monitoring and Logging

A private cloud is not complete without observability.

The monitoring phase focuses on:

* Node health
* Pod health
* Deployment status
* Resource utilization
* Container logs
* PostgreSQL logs
* ETL logs
* Kubernetes events

---

## Check Nodes

```bash
sudo kubectl get nodes
```

---

## Check Pods

```bash
sudo kubectl get pods -n datawarehouse
```

Detailed:

```bash
sudo kubectl get pods \
-n datawarehouse \
-o wide
```

---

## Check Deployments

```bash
sudo kubectl get deployment \
-n datawarehouse
```

---

## Check Services

```bash
sudo kubectl get svc \
-n datawarehouse
```

---

## View Logs

```bash
sudo kubectl logs <pod-name> \
-n datawarehouse
```

Follow logs:

```bash
sudo kubectl logs -f <pod-name> \
-n datawarehouse
```

---

# Phase 7 — Failure Testing

A distributed system should not only work under normal conditions.

It should also be tested under failure.

The project therefore includes controlled failure scenarios.

---

## Failure Scenario 1 — Pod Failure

Delete a pod:

```bash
sudo kubectl delete pod <pod-name> \
-n datawarehouse
```

Check:

```bash
sudo kubectl get pods \
-n datawarehouse
```

The Kubernetes controller should recreate the workload when managed by a Deployment.

---

## Failure Scenario 2 — Node Failure

Simulate a worker failure by shutting down a worker:

```bash
sudo shutdown -h now
```

From the control plane:

```bash
sudo kubectl get nodes
```

Observe:

```text
Ready
NotReady
```

---

## Failure Scenario 3 — Service Failure

Inspect services:

```bash
sudo kubectl get svc \
-n datawarehouse
```

Inspect endpoints:

```bash
sudo kubectl get endpoints \
-n datawarehouse
```

---

## Failure Scenario 4 — ETL Failure

Inspect ETL:

```bash
sudo kubectl get pods \
-n datawarehouse
```

Logs:

```bash
sudo kubectl logs <etl-pod> \
-n datawarehouse
```

If using a Job:

```bash
sudo kubectl get jobs \
-n datawarehouse
```

---

# Phase 8 — CI/CD and Automation

The final phase connects the application lifecycle with containerization and Kubernetes.

The intended workflow is:

```text
Developer
    │
    ▼
Git Commit
    │
    ▼
GitHub
    │
    ▼
Build
    │
    ▼
Docker Image
    │
    ▼
Container Registry
    │
    ▼
Kubernetes
    │
    ▼
K3s Cluster
    │
    ▼
Application
```

---

# 🐳 Container Registry

The ETL image must be available to the K3s cluster.

Possible registries include:

* Docker Hub
* GitHub Container Registry
* Private registry

Example:

```bash
docker login
```

Build:

```bash
docker build \
-t username/datawarehouse-etl:latest .
```

Push:

```bash
docker push \
username/datawarehouse-etl:latest
```

Versioned image:

```bash
docker push \
username/datawarehouse-etl:v1.0.0
```

The repository's Phase 8 documentation also covers registry usage, Kubernetes deployment configuration, verification, ETL logs, Jobs, and rolling updates.

---

# ☸️ Kubernetes Deployment

Example deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: datawarehouse-etl
  namespace: datawarehouse

spec:
  replicas: 1

  selector:
    matchLabels:
      app: datawarehouse-etl

  template:
    metadata:
      labels:
        app: datawarehouse-etl

    spec:
      containers:
        - name: etl
          image: username/datawarehouse-etl:latest
          imagePullPolicy: Always

          envFrom:
            - configMapRef:
                name: datawarehouse-config
```

---

# 📦 Kubernetes Resources

The project uses resources such as:

```text
Namespace
    │
    ├── ConfigMap
    │
    ├── PersistentVolume
    │
    ├── PersistentVolumeClaim
    │
    ├── PostgreSQL
    │
    ├── ETL
    │
    └── Services
```

---

# 🚀 Deploying the Data Warehouse

Create namespace:

```bash
sudo kubectl apply \
-f k8s/namespace.yaml
```

Create storage:

```bash
sudo kubectl apply \
-f k8s/postgres-storage.yaml
```

Deploy PostgreSQL:

```bash
sudo kubectl apply \
-f k8s/postgres.yaml
```

Apply configuration:

```bash
sudo kubectl apply \
-f k8s/configmap.yaml
```

Deploy ETL:

```bash
sudo kubectl apply \
-f k8s/etl.yaml
```

---

# 🔎 Verification

## Cluster

```bash
sudo kubectl get nodes
```

```bash
sudo kubectl get nodes -o wide
```

---

## Namespaces

```bash
sudo kubectl get namespaces
```

---

## Pods

```bash
sudo kubectl get pods \
-n datawarehouse
```

---

## Deployments

```bash
sudo kubectl get deployment \
-n datawarehouse
```

---

## Services

```bash
sudo kubectl get svc \
-n datawarehouse
```

---

## PVC

```bash
sudo kubectl get pvc \
-n datawarehouse
```

---

## Persistent Volumes

```bash
sudo kubectl get pv
```

---

# 🔍 ETL Verification

Find the deployment:

```bash
sudo kubectl get deployment \
-n datawarehouse
```

Check the image:

```bash
sudo kubectl get deployment \
datawarehouse-etl \
-n datawarehouse \
-o jsonpath='{.spec.template.spec.containers[0].image}'
```

Check pods:

```bash
sudo kubectl get pods \
-n datawarehouse
```

Check logs:

```bash
sudo kubectl logs <pod-name> \
-n datawarehouse
```

---

# 🧪 Kubernetes Job Verification

If ETL runs as a Kubernetes Job:

```bash
sudo kubectl get jobs \
-n datawarehouse
```

Detailed:

```bash
sudo kubectl describe job \
datawarehouse-etl \
-n datawarehouse
```

Check status:

```bash
sudo kubectl get job \
datawarehouse-etl \
-n datawarehouse \
-o wide
```

A successful Job should normally show:

```text
COMPLETIONS
1/1
```

A completed Job may later disappear if Kubernetes cleanup policies are configured. Therefore, a `NotFound` result does not automatically prove that the ETL failed; pod history and Job lifecycle settings should also be checked.

---

# 🔄 Rolling Updates

When a new image version is available:

```bash
sudo kubectl set image \
deployment/datawarehouse-etl \
etl=username/datawarehouse-etl:v1.1.0 \
-n datawarehouse
```

Check rollout:

```bash
sudo kubectl rollout status \
deployment/datawarehouse-etl \
-n datawarehouse
```

View rollout history:

```bash
sudo kubectl rollout history \
deployment/datawarehouse-etl \
-n datawarehouse
```

---

# 🔐 Security

Security was considered at multiple infrastructure layers.

## Network Security

* Private LAN
* UFW
* Restricted SSH access
* Internal node communication

## Authentication

* SSH key authentication
* K3s node token authentication

## Kubernetes

* Namespaces
* ConfigMaps
* Kubernetes resource isolation

## Container Security

Recommended future improvements:

* Non-root containers
* Minimal base images
* Image scanning
* Secret management
* Resource limits
* Network policies

---

# 🗂️ Repository Structure

The repository is organized around the eight development phases.

```text
ClusteringServers/
│
├── BugInInstallingLinux/
│
├── Overview
│
├── Phase1
│
├── Phase2
│
├── Phase3
│
├── Phase4OverViewDocumentation
│
├── Phase5
│
├── Phase5OverViewDocumentation
│
├── Phase6
│
├── Phase6OverViewDocumentation
│
├── Phase7
│
├── Phase7OverViewDocumentation
│
├── Phase8
│
└── README.md
```

The current GitHub repository exposes these phase documents directly at the repository root.

---

# 📚 Documentation Map

| Directory/File                | Purpose                                         |
| ----------------------------- | ----------------------------------------------- |
| `Overview`                    | Overall project architecture and infrastructure |
| `Phase1`                      | Networking, SSH, firewall, time synchronization |
| `Phase2`                      | K3s cluster installation and node joining       |
| `Phase3`                      | Containerization                                |
| `Phase4OverViewDocumentation` | Data warehouse deployment documentation         |
| `Phase5`                      | Storage and recovery                            |
| `Phase5OverViewDocumentation` | Storage documentation                           |
| `Phase6`                      | Monitoring and logging                          |
| `Phase6OverViewDocumentation` | Monitoring documentation                        |
| `Phase7`                      | Failure testing                                 |
| `Phase7OverViewDocumentation` | Failure-testing documentation                   |
| `Phase8`                      | CI/CD and automation                            |
| `BugInInstallingLinux`        | Linux installation/troubleshooting notes        |

---

# 🧠 Important Commands Cheat Sheet

## Linux

```bash
hostname
hostname -I
ip addr
ip route
free -h
lscpu
df -h
```

## Network

```bash
ping -c 3 192.168.1.7
ping -c 3 192.168.1.2
```

## SSH

```bash
ssh cluster-worker1
ssh cluster-worker2
ssh server4
ssh server5
```

## Firewall

```bash
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw enable
```

## Time

```bash
timedatectl status
sudo timedatectl set-ntp true
sudo timedatectl set-local-rtc 0
```

## K3s

```bash
sudo systemctl status k3s
sudo kubectl get nodes
sudo kubectl get nodes -o wide
```

## Kubernetes

```bash
sudo kubectl get pods -A
sudo kubectl get deployments -A
sudo kubectl get svc -A
sudo kubectl get pv
sudo kubectl get pvc -A
```

## Logs

```bash
sudo kubectl logs <pod>
sudo kubectl logs -f <pod>
```

## Debugging

```bash
sudo kubectl describe pod <pod>
sudo kubectl describe node <node>
sudo kubectl get events -A
```

---

# 🛠️ Troubleshooting

## K3s API Server Not Responding

Check:

```bash
sudo systemctl status k3s --no-pager -l
```

Logs:

```bash
sudo journalctl -u k3s -n 100 --no-pager
```

Check listening port:

```bash
sudo ss -lntp | grep 6443
```

Check node connectivity:

```bash
ping 192.168.1.7
```

---

## Node Shows `NotReady`

Check:

```bash
sudo kubectl get nodes
```

Then:

```bash
sudo kubectl describe node <node>
```

On the worker:

```bash
sudo systemctl status k3s-agent
```

Logs:

```bash
sudo journalctl -u k3s-agent -n 100 --no-pager
```

---

## Pod Stuck in `Pending`

Check:

```bash
sudo kubectl describe pod <pod-name> \
-n datawarehouse
```

Check nodes:

```bash
sudo kubectl get nodes
```

Check PVC:

```bash
sudo kubectl get pvc \
-n datawarehouse
```

Check storage:

```bash
sudo kubectl get pv
```

---

## PVC Stuck in `Pending`

Check:

```bash
sudo kubectl describe pvc \
postgres-pvc \
-n datawarehouse
```

Then:

```bash
sudo kubectl get storageclass
```

And:

```bash
sudo kubectl get pv
```

---

## PostgreSQL Constraint Error

Example:

```text
new row for relation violates check constraint
```

Investigate:

```bash
sudo kubectl logs <postgres-pod> \
-n datawarehouse
```

Then inspect:

* Input data
* Transformation logic
* Database constraints
* Data types
* Column mappings

Never remove a database constraint blindly. First determine whether the data or the constraint is incorrect.

---

# 🧪 Validation Strategy

The project validates the infrastructure at multiple levels.

```text
Hardware
   │
   ▼
Network
   │
   ▼
SSH
   │
   ▼
Firewall
   │
   ▼
Time
   │
   ▼
K3s
   │
   ▼
Kubernetes
   │
   ▼
Storage
   │
   ▼
PostgreSQL
   │
   ▼
ETL
   │
   ▼
Data Warehouse
   │
   ▼
Monitoring
   │
   ▼
Failure Testing
   │
   ▼
CI/CD
```

This layered validation approach makes it easier to identify whether a failure belongs to the infrastructure, Kubernetes, storage, database, application, or data layer.

---

# 📈 Scalability Concept

The project was intentionally designed to evolve from a three-node Kubernetes cluster toward a five-node private cloud.

### Initial Concept

```text
3 Servers

Master
  │
  ├── Worker 1
  └── Worker 2
```

### Expanded Concept

```text
5 Servers

                 Master
                   │
        ┌──────────┼──────────┐
        │          │          │
     Worker 1  Worker 2    Server 4
                              │
                           Server 5
```

The repository describes the broader objective as starting with three Ubuntu servers, deploying the existing S&P 500 data warehouse, scaling toward five servers, monitoring the infrastructure, testing failures, and automating operations.

---

# ☁️ Private Cloud Concept

This project demonstrates several fundamental cloud concepts without depending entirely on a public-cloud provider.

| Cloud Concept   | Project Implementation                   |
| --------------- | ---------------------------------------- |
| Compute         | Physical laptops                         |
| Networking      | Private LAN                              |
| Orchestration   | K3s                                      |
| Containers      | Docker                                   |
| Storage         | Kubernetes PV/PVC                        |
| Database        | PostgreSQL                               |
| ETL             | Python                                   |
| Monitoring      | Kubernetes inspection/logging            |
| Scaling         | Additional nodes                         |
| Failure testing | Controlled node/workload failures        |
| CI/CD           | GitHub + container registry + Kubernetes |

---

# 🧑‍💻 Skills Demonstrated

## Linux Administration

* Ubuntu
* Kali Linux
* systemd
* package management
* networking
* SSH
* UFW
* time synchronization

## Networking

* IPv4
* LAN networking
* hostname resolution
* routing
* connectivity testing
* SSH networking
* Kubernetes networking concepts

## Kubernetes

* K3s
* Control plane
* Worker nodes
* Pods
* Deployments
* Services
* Jobs
* ConfigMaps
* PersistentVolumes
* PersistentVolumeClaims
* Namespaces
* Rollouts
* Scheduling

## Data Engineering

* ETL
* Data validation
* PostgreSQL
* Data warehouse architecture
* Analytical workloads

## DevOps

* Docker
* Git
* GitHub
* Container registries
* CI/CD
* Infrastructure troubleshooting
* Failure testing

## Systems Engineering

* Distributed systems
* Resource allocation
* Cluster architecture
* High availability concepts
* Scaling
* Recovery
* Observability

---

# 🧠 Major Lessons Learned

This project provided practical experience with several concepts that are difficult to understand only through theory.

### 1. Networking comes first

A Kubernetes cluster cannot compensate for broken node networking.

### 2. DNS/hostname resolution matters

Consistent hostnames make administration and debugging significantly easier.

### 3. SSH is fundamental

Secure remote administration is essential for multi-node infrastructure.

### 4. Time synchronization matters

Distributed systems depend on consistent system clocks.

### 5. Kubernetes does not eliminate infrastructure problems

If a node has insufficient resources, bad networking, or broken storage, Kubernetes cannot magically fix the underlying machine.

### 6. Storage is different from compute

Deleting a pod should not mean deleting database data.

### 7. Monitoring is part of deployment

A workload is not truly operational until its health can be observed.

### 8. Failure testing is essential

A system should be tested not only when everything works, but also when components fail.

### 9. Heterogeneous hardware creates scheduling challenges

Different nodes have different:

* CPU capacity
* RAM
* storage
* operating systems

This makes workload placement an important consideration.

### 10. Cloud concepts can be learned locally

A physical home lab can provide valuable hands-on experience with concepts used in production cloud infrastructure.

---

# 🚧 Current Limitations

This project is a learning-focused private cloud rather than a production cloud platform.

Known limitations include:

* Consumer laptop hardware
* Limited RAM on the control-plane node
* Limited persistent storage on some nodes
* Single control-plane architecture
* Local-network dependency
* No enterprise-grade redundant storage
* No enterprise load balancer
* No production-grade HA control plane
* No enterprise backup infrastructure
* Limited physical redundancy

These limitations are intentional learning constraints and provide opportunities for future improvements.

---

# 🔮 Future Improvements

Potential future work includes:

## High Availability

* Multiple K3s server nodes
* HA datastore
* Redundant control plane

## Storage

* Longhorn
* Distributed persistent storage
* Automated backups
* Snapshot-based recovery

## Monitoring

* Prometheus
* Grafana
* Alertmanager
* Node Exporter

## Logging

* Loki
* Promtail
* Centralized log aggregation

## Networking

* MetalLB
* Ingress controller
* Network policies
* Better service exposure

## Security

* Kubernetes Secrets
* RBAC
* NetworkPolicies
* Container vulnerability scanning
* Image signing

## CI/CD

* GitHub Actions
* Automated image builds
* Automated deployment
* Versioned releases
* Rollback automation

## Infrastructure as Code

Potentially introduce:

* Ansible
* Terraform
* Helm

---

# 🏁 Project Status

## Overall

**Project Status: Completed — Private Cloud Data Warehouse Platform**

### Phase Status

| Phase   | Area                 | Status      |
| ------- | -------------------- | ----------- |
| Phase 1 | Infrastructure       | ✅ Completed |
| Phase 2 | K3s Cluster          | ✅ Completed |
| Phase 3 | Containerization     | ✅ Completed |
| Phase 4 | Data Warehouse       | ✅ Completed |
| Phase 5 | Storage & Recovery   | ✅ Completed |
| Phase 6 | Monitoring & Logging | ✅ Completed |
| Phase 7 | Failure Testing      | ✅ Completed |
| Phase 8 | CI/CD & Automation   | ✅ Completed |

---

# 🏆 Final Architecture

```text
                         ┌──────────────────────┐
                         │      USER / DEV      │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │     GitHub / CI/CD   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Container Registry   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                  ╔══════════════════════════════════╗
                  ║          PRIVATE CLOUD           ║
                  ║                                  ║
                  ║  ┌────────────────────────────┐  ║
                  ║  │     cluster-master         │  ║
                  ║  │     K3s Control Plane      │  ║
                  ║  └─────────────┬──────────────┘  ║
                  ║                │                 ║
                  ║      ┌─────────┼─────────┐       ║
                  ║      │         │         │       ║
                  ║      ▼         ▼         ▼       ║
                  ║  Worker 1  Worker 2   Server 4  ║
                  ║      │         │         │       ║
                  ║      └─────────┼─────────┘       ║
                  ║                │                 ║
                  ║                ▼                 ║
                  ║             Server 5             ║
                  ║                                  ║
                  ╚════════════════╤═════════════════╝
                                   │
                                   ▼
                       ┌────────────────────────┐
                       │ Kubernetes Namespace   │
                       │     datawarehouse      │
                       ├────────────────────────┤
                       │ PostgreSQL             │
                       │ ETL                    │
                       │ ConfigMap              │
                       │ Persistent Storage     │
                       │ Services               │
                       └───────────┬────────────┘
                                   │
                                   ▼
                       ┌────────────────────────┐
                       │    S&P 500 Warehouse   │
                       ├────────────────────────┤
                       │ Fact Tables             │
                       │ Dimension Tables        │
                       │ Analytics               │
                       └────────────────────────┘
```

---

# 📖 Recommended Learning Path From This Project

If someone wants to reproduce this project, the recommended learning order is:

```text
Linux
  ↓
Networking
  ↓
SSH
  ↓
Firewall
  ↓
Git/GitHub
  ↓
Docker
  ↓
Kubernetes Fundamentals
  ↓
K3s
  ↓
Persistent Storage
  ↓
PostgreSQL
  ↓
ETL
  ↓
Monitoring
  ↓
Failure Testing
  ↓
CI/CD
```

Do not start with Kubernetes commands without understanding Linux and networking fundamentals.

---

# 📌 Project Philosophy

> **Build it. Break it. Debug it. Understand it. Automate it.**

The main objective of this project was not simply to install Kubernetes.

The objective was to understand the complete infrastructure lifecycle:

```text
Physical Hardware
      ↓
Operating System
      ↓
Networking
      ↓
Secure Access
      ↓
Cluster
      ↓
Containers
      ↓
Storage
      ↓
Database
      ↓
ETL
      ↓
Monitoring
      ↓
Failure
      ↓
Recovery
      ↓
Automation
```

This makes the project a practical exploration of **Linux administration, networking, Kubernetes, DevOps, data engineering, distributed systems, and private-cloud infrastructure**.

---

# 👨‍💻 Author

**Satish Kumar Peddi**

3rd Year Engineering Student

Interested in:

* Linux
* Systems Engineering
* Cloud Infrastructure
* Kubernetes
* DevOps
* Data Engineering
* Distributed Systems
* Cloud Security

---

# ⭐ Repository

**GitHub:**
https://github.com/satishkumarpeddi/ClusteringServers

If this project is useful for learning private-cloud infrastructure, consider giving the repository a ⭐.

---

## 📜 License

This project is primarily intended for educational and portfolio purposes.

Individual scripts, configurations, datasets, and third-party components may be subject to their respective licenses.
