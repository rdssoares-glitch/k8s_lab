## Cluster Network Configuration

The cluster networking configuration can be obtained from the `kubeadm-config` ConfigMap.

### Command

```bash
kubectl -n kube-system get cm kubeadm-config -o yaml
```

### Relevant Output

```yaml
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
```

### Network Summary

| Parameter | Value | Description |
|-----------|-------|-------------|
| Kubernetes Version | v1.34.0 | Cluster version |
| DNS Domain | `cluster.local` | Internal Kubernetes DNS domain |
| Pod CIDR | `10.244.0.0/16` | Address range assigned to Pods |
| Service CIDR | `10.96.0.0/12` | Address range assigned to ClusterIP Services |

### Pod CIDR Allocation per Node

Command:

```bash
kubectl get nodes \
-o custom-columns=NODE:.metadata.name,POD_CIDR:.spec.podCIDR
```

Output:

```text
NODE           POD_CIDR
minikube       10.244.0.0/24
minikube-m02   10.244.1.0/24
minikube-m03   10.244.2.0/24
```

| Node | Pod CIDR |
|------|----------|
| minikube | `10.244.0.0/24` |
| minikube-m02 | `10.244.1.0/24` |
| minikube-m03 | `10.244.2.0/24` |

### Addressing Overview

| Network | CIDR | Purpose |
|----------|------|---------|
| Docker Network | `192.168.49.0/24` | Minikube node network |
| Pod Network | `10.244.0.0/16` | Network assigned to Pods |
| Service Network | `10.96.0.0/12` | ClusterIP Services |
| DNS Domain | `cluster.local` | Internal Kubernetes DNS |


## Docker Network (Node Network)

The Minikube nodes are connected through a dedicated Docker bridge network.

### Command

```bash
docker network inspect minikube
```

### Relevant Output

```json
"IPAM": {
  "Config": [
    {
      "Subnet": "192.168.49.0/24",
      "Gateway": "192.168.49.1"
    }
  ]
}
```

### Node IPs

| Node | IP Address |
|-------|------------|
| minikube | 192.168.49.2 |
| minikube-m02 | 192.168.49.3 |
| minikube-m03 | 192.168.49.4 |
