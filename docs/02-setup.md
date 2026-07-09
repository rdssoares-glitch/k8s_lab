# 🚀 Kubernetes Environment Setup with Minikube

This document describes the installation process for **Docker Engine**, **kubectl**, and **Minikube** to create a local **3-node Kubernetes cluster** running on **WSL2 (Ubuntu 24.04)** on **Windows 11**.

---

# 📋 Prerequisites

- Windows 11
- WSL2 enabled
- Ubuntu 24.04 LTS
- Minimum 2 vCPUs
- Minimum 4 GB of available RAM
- Approximately 20 GB of available disk space

---

# 1. Install Docker Engine

Update the system packages.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```

## Add the Docker GPG key

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor \
-o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## Add the official Docker repository

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Install Docker

```bash
sudo apt update

sudo apt install -y \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin
```

## Allow Docker to run without sudo

```bash
sudo usermod -aG docker $USER

newgrp docker
```

## Verify the installation

```bash
docker version
```

---

# 2. Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client
```

---

# 3. Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

minikube version
```

---

# 4. Create a 3-node Kubernetes cluster

```bash
minikube start --driver=docker --nodes=3
```

---

# 5. Verify the installation

## Check the Minikube status

```bash
minikube status
```

## List the cluster nodes

```bash
kubectl get nodes -o wide
```

## List all running Pods

```bash
kubectl get pods -A
```

---

# 6. Useful commands

## Stop the cluster

```bash
minikube stop
```

## Start the cluster

```bash
minikube start
```

## Delete the cluster

```bash
minikube delete
```

## Open the Kubernetes Dashboard

```bash
minikube dashboard
```

## Get the cluster IP address

```bash
minikube ip
```

## Check the cluster status

```bash
minikube status
```

## List container images available in the cluster

```bash
minikube image ls
```

## List images using CRI

```bash
minikube ssh
crictl images
```

## Access a node via SSH

### Control plane node

```bash
minikube ssh
```

### Worker node 1

```bash
minikube ssh -n minikube-m02
```

### Worker node 2

```bash
minikube ssh -n minikube-m03
```

## Add worker nodes to an existing cluster

If Minikube is already running, additional worker nodes can be added using:

```bash
minikube node add
```

---

# 7. Example environment

## Minikube profiles

```bash
$ minikube profile list

┌──────────┬────────┬─────────┬──────────────┬─────────┬────────┬───────┬────────────────┬────────────────────┐
│ PROFILE  │ DRIVER │ RUNTIME │      IP      │ VERSION │ STATUS │ NODES │ ACTIVE PROFILE │ ACTIVE KUBECONTEXT │
├──────────┼────────┼─────────┼──────────────┼─────────┼────────┼───────┼────────────────┼────────────────────┤
│ minikube │ docker │ docker  │ 192.168.49.2 │ v1.34.0 │ OK     │ 3     │ *              │ *                  │
└──────────┴────────┴─────────┴──────────────┴─────────┴────────┴───────┴────────────────┴────────────────────┘
```

## Cluster nodes

```bash
$ kubectl get nodes

NAME           STATUS   ROLES           AGE   VERSION
minikube       Ready    control-plane   84s   v1.34.0
minikube-m02   Ready    <none>          67s   v1.34.0
minikube-m03   Ready    <none>          43s   v1.34.0
```

---

# ✅ Result

At the end of this procedure, the local Kubernetes environment will include:

- Docker Engine
- kubectl
- Minikube
- A 3-node Kubernetes cluster
- Docker container runtime
- A fully functional environment ready to run Pods, Deployments, Services, and other Kubernetes resources.
