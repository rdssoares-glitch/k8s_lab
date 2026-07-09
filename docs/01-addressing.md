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

# Network Investigation

This section demonstrates how to identify the network namespace and the IP address of a Pod by starting from the Kubernetes API and progressively inspecting the container runtime.

## Step 1 - Locate the Pod

```bash
kubectl get pods -o wide
```

Output:

```text
NAME                        IP           NODE
nginx-57457b7d55-qbxtt      10.244.2.3   minikube-m03
```

The Pod is running on **minikube-m03**.

---

## Step 2 - List the containers on the node

Connect to the node:

```bash
minikube ssh -n minikube-m03
```

List the containers:

```bash
crictl ps
```

Output:

```text
CONTAINER ID      NAME
6568fec4d101b     nginx
```

---

## Step 3 - Identify the container PID

Inspect the container:

```bash
crictl inspect 6568fec4d101b | grep pid
```

Output:

```text
"pid": 1827
```

The nginx container is running as process **1827**.

---

## Step 4 - Enter the Pod network namespace

Use the process PID to enter the network namespace:

```bash
sudo nsenter -t 1827 -n ip a
```

Output:

```text
docker@minikube-m03:~$ sudo nsenter -t 1827 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 4e:b4:ae:08:f5:30 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.2.3/24 brd 10.244.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::4cb4:aeff:fe08:f530/64 scope link
       valid_lft forever preferred_lft forever
docker@minikube-m03:~$

```

The Pod network namespace contains:

- loopback interface (`lo`)
- one Ethernet interface (`eth0`)
- Pod IP: **10.244.2.3**

---

## Step 5 - Host-side veth

Back on the node:

```bash
ip -br link
```

Output:

```text
docker@minikube-m03:~$ ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0@if8         UP             06:a5:ea:98:5d:44 <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          DOWN           42:c3:51:07:a7:09 <NO-CARRIER,BROADCAST,MULTICAST,UP>
veth8c6c99e4@if2 UP             de:51:cd:2e:bb:19 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth5b461568@if2 UP             d2:6e:da:c8:a9:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
docker@minikube-m03:~$
```

Inspect the interface:

```bash
ip link
```

Output:


```text
docker@minikube-m03:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 06:a5:ea:98:5d:44 brd ff:ff:ff:ff:ff:ff link-netnsid 0
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 42:c3:51:07:a7:09 brd ff:ff:ff:ff:ff:ff
4: veth8c6c99e4@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether de:51:cd:2e:bb:19 brd ff:ff:ff:ff:ff:ff link-netnsid 1
5: veth5b461568@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether d2:6e:da:c8:a9:4b brd ff:ff:ff:ff:ff:ff link-netnsid 2

```
``` bash
ip -d link show veth5b461568
```
Output:

```text
docker@minikube-m03:~$ ip -d link show veth5b461568
5: veth5b461568@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether d2:6e:da:c8:a9:4b brd ff:ff:ff:ff:ff:ff link-netnsid 2 promiscuity 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 12 numrxqueues 12 gso_max_size 65536 gso_max_segs 65535
docker@minikube-m03:~$

````
Observe the interface index (**ifindex**) on the host:

```text
5: veth5b461568@if2
^
|
ifindex 5
```

The interface index is used to map the Pod network interface to the host-side `veth` interface.

Pod network namespace:

```text
eth0@if5
IP: 10.244.2.3
```
O campo **`link-netnsid 2`** indica a associação da interface `veth` com um Network Namespace.
```

This is the host-side interface connected to the Pod namespace.

---
# Address Translation

| Layer | Address |
|--------|---------|
| Node Network | 192.168.49.4 |
| Pod CIDR | 10.244.2.0/24 |
| Pod IP | 10.244.2.3 |
| Interface inside Pod | eth0@if5 |
| Interface on Host | veth5b461568@if2 |



