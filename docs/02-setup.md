# 🚀 Setup do Ambiente Kubernetes com Minikube

Este documento descreve o processo de instalação do **Docker Engine**, **kubectl** e **Minikube** para criação de um cluster Kubernetes local com **3 nós**, utilizando **WSL2 (Ubuntu 24.04)** sobre **Windows 11**.

---

# 📋 Pré-requisitos

- Windows 11
- WSL2 habilitado
- Ubuntu 24.04 LTS
- Mínimo de 2 vCPUs
- Mínimo de 4 GB de memória livre
- Aproximadamente 20 GB de espaço em disco

---

# 1. Instalação do Docker Engine

Atualize os pacotes do sistema.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```

## Adicionar a chave GPG do Docker

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor \
-o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## Adicionar o repositório oficial

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Instalar o Docker

```bash
sudo apt update

sudo apt install -y \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin
```

## Permitir executar Docker sem sudo

```bash
sudo usermod -aG docker $USER

newgrp docker
```

## Validar instalação

```bash
docker version
```

---

# 2. Instalação do kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client
```

---

# 3. Instalação do Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

minikube version
```

---

# 4. Criando um cluster Kubernetes com 3 nós

```bash
minikube start --driver=docker --nodes=3
```

---

# 5. Validando a instalação

## Status do Minikube

```bash
minikube status
```

## Listar os nós do cluster

```bash
kubectl get nodes -o wide
```

## Listar todos os Pods

```bash
kubectl get pods -A
```

---

# 6. Comandos úteis

## Parar o cluster

```bash
minikube stop
```

## Iniciar o cluster

```bash
minikube start
```

## Excluir o cluster

```bash
minikube delete
```

## Abrir o Dashboard

```bash
minikube dashboard
```

## Obter o IP do cluster

```bash
minikube ip
```

## Acessar um nó via SSH

Nó principal:

```bash
minikube ssh
```

Segundo nó:

```bash
minikube ssh -n minikube-m02
```

Terceiro nó:

```bash
minikube ssh -n minikube-m03
```

---

# 7. Exemplo de ambiente criado

## Perfis do Minikube

```bash
$ minikube profile list

┌──────────┬────────┬─────────┬──────────────┬─────────┬────────┬───────┬────────────────┬────────────────────┐
│ PROFILE  │ DRIVER │ RUNTIME │      IP      │ VERSION │ STATUS │ NODES │ ACTIVE PROFILE │ ACTIVE KUBECONTEXT │
├──────────┼────────┼─────────┼──────────────┼─────────┼────────┼───────┼────────────────┼────────────────────┤
│ minikube │ docker │ docker  │ 192.168.49.2 │ v1.34.0 │ OK     │ 3     │ *              │ *                  │
└──────────┴────────┴─────────┴──────────────┴─────────┴────────┴───────┴────────────────┴────────────────────┘
```

## Nós do cluster

```bash
$ kubectl get nodes

NAME           STATUS   ROLES           AGE   VERSION
minikube       Ready    control-plane   84s   v1.34.0
minikube-m02   Ready    <none>          67s   v1.34.0
minikube-m03   Ready    <none>          43s   v1.34.0
```

---

# ✅ Resultado

Ao final deste procedimento será criado um ambiente Kubernetes local contendo:

- Docker Engine
- kubectl
- Minikube
- Cluster Kubernetes com 3 nós
- Runtime Docker
- Ambiente pronto para execução de Pods, Deployments, Services e demais recursos Kubernetes.
