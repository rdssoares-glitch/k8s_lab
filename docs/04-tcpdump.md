# 04 - Capturing Pod-to-Pod Traffic with tcpdump

This section demonstrates how to capture network traffic exchanged between two Pods running on different Kubernetes nodes. The objective is to observe how Kindnet forwards packets across nodes and verify that Pod IP addresses remain unchanged throughout the communication.

## Test Environment

List the running NGINX Pods:

```bash
kubectl get po -o wide | grep nginx
```

Output:

```text
nginx-57457b7d55-b8bd2   1/1 Running   10.244.0.3   minikube
nginx-57457b7d55-kmrsj   1/1 Running   10.244.1.4   minikube-m02
nginx-57457b7d55-qbxtt   1/1 Running   10.244.2.2   minikube-m03
```

For this experiment:

| Role | Pod | IP Address | Node |
|------|------|------------|------|
| Source | nginx-57457b7d55-qbxtt | **10.244.2.2** | **minikube-m03** |
| Destination | nginx-57457b7d55-kmrsj | **10.244.1.4** | **minikube-m02** |

Traffic flow:

```text
Source:
Pod 10.244.2.2
Node minikube-m03

        │
        ▼

Destination:
Pod 10.244.1.4
Node minikube-m02
```

---

# Generate HTTP Traffic

From the source Pod, generate multiple HTTP requests toward the destination Pod.

```bash
kubectl exec -it nginx-57457b7d55-qbxtt -- bash
```

Inside the container:

```bash
for i in {1..10}; do
    curl http://10.244.1.4 > /dev/null
done
```

---

# Identify the Host-side veth Interface

SSH into the destination node:

```bash
minikube ssh -n minikube-m02
```

Identify the veth interface associated with the destination Pod:

```bash
ip r | grep 10.244.1.4
```

Output:

```text
10.244.1.4 dev veth9c608741 scope host
```

This route shows that packets destined for Pod **10.244.1.4** are delivered through the host-side interface:

```text
veth9c608741
```

---

# Capture Traffic on the Node Interface (eth0)

Start a packet capture on the destination node:

```bash
sudo tcpdump -i eth0 -nn host 10.244.2.2
```

Example output:

```text
IP 10.244.2.2.54186 > 10.244.1.4.80: Flags [S]
IP 10.244.1.4.80 > 10.244.2.2.54186: Flags [S.]
IP 10.244.2.2.54186 > 10.244.1.4.80: Flags [.]
IP 10.244.2.2.54186 > 10.244.1.4.80: HTTP: GET /
IP 10.244.1.4.80 > 10.244.2.2.54186: HTTP/1.1 200 OK
...
```

The capture clearly shows the TCP three-way handshake:

```text
SYN
SYN-ACK
ACK
```

followed by the HTTP request and response.

---

# What Does This Tell Us About Kindnet?

The packet capture demonstrates that **Kindnet relies on the Linux kernel routing table** to forward packets between Kubernetes nodes.

Notice that the IP header remains unchanged during the entire communication:

```text
Source IP:      10.244.2.2
Destination IP: 10.244.1.4
```

Although the traffic traverses two different nodes, **no Source NAT (SNAT)** or **Destination NAT (DNAT)** is performed.

The packet arriving at the destination node still contains the original Pod IP addresses.

This behavior is one of the fundamental networking principles of Kubernetes:

- Every Pod has its own routable IP address.
- Pods communicate directly using Pod IPs.
- The CNI plugin is responsible for ensuring end-to-end connectivity.

---

# Capture Traffic on the veth Interface

Now capture traffic on the host-side veth interface connected to the destination Pod.

```bash
sudo tcpdump -i veth9c608741 -nn
```

Example output:

```text
IP 10.244.2.2.40764 > 10.244.1.4.80: Flags [S]
IP 10.244.1.4.80 > 10.244.2.2.40764: Flags [S.]
IP 10.244.2.2.40764 > 10.244.1.4.80: Flags [.]
IP 10.244.2.2.40764 > 10.244.1.4.80: HTTP: GET /
IP 10.244.1.4.80 > 10.244.2.2.40764: HTTP/1.1 200 OK
...
```

The captured packets are virtually identical to those observed on the node's `eth0` interface.

This is expected because the Linux kernel simply routes the packet from the node interface to the veth interface without modifying the IP header.

---

# Packet Path

The complete packet path is illustrated below:

```text
Pod (10.244.2.2)
        │
        ▼
eth0 (Pod)
        │
veth
        │
eth0 (Node minikube-m03)
═══════════════════════════════════════
        Network between Nodes
═══════════════════════════════════════
eth0 (Node minikube-m02)
        │
        ▼
veth9c608741
        │
eth0 (Pod)
        │
Pod (10.244.1.4)
```

---

# Conclusion

This experiment demonstrates several important Kubernetes networking concepts:

- Pod-to-Pod communication works transparently across different nodes.
- Kindnet relies on the Linux routing table instead of performing address translation.
- Pod IP addresses remain unchanged from source to destination.
- The destination node forwards packets from its physical interface (`eth0`) to the corresponding Pod through the host-side `veth` interface.
- Capturing traffic on both `eth0` and the corresponding `veth` interface shows essentially the same IP packets, confirming that packet forwarding occurs through native Linux routing.

This experiment provides a practical demonstration of how Kubernetes networking operates internally and how Kindnet integrates with the Linux networking stack.
