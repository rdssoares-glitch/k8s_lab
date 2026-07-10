# Kubernetes DNS Service Discovery with CoreDNS

## Overview

Kubernetes provides an internal DNS mechanism that allows Pods to discover Services using DNS names instead of using dynamic IP addresses.

In this experiment, we will demonstrate how Kubernetes DNS works internally by:

- Checking the DNS configuration inside a Pod
- Identifying the Kubernetes DNS Service
- Creating a temporary DNS testing Pod
- Resolving a Kubernetes Service name
- Capturing DNS packets using tcpdump
- Understanding the communication flow between Pod, kube-dns Service, and CoreDNS

The environment used in this experiment:

| Component | Value |
|---|---|
| Kubernetes Version | v1.34.0 |
| CNI | Kindnet |
| DNS Implementation | CoreDNS |
| DNS Service | kube-dns |
| DNS Service IP | 10.96.0.10 |

---

# 1. Kubernetes Pod Network Information

The cluster contains three nginx Pods running on different nodes:

```bash
kubectl get pod -o wide | grep nginx
nginx-57457b7d55-b8bd2     10.244.0.3   minikube
nginx-57457b7d55-kmrsj     10.244.1.4   minikube-m02
nginx-57457b7d55-qbxtt     10.244.2.2   minikube-m03

Pod:
nginx-57457b7d55-qbxtt

IP:
10.244.2.2

Node:
minikube-m03
```
# 2. Checking DNS Configuration Inside a Pod

Enter the nginx Pod:
```bash
kubectl exec -it nginx-57457b7d55-qbxtt -- bash

Check the DNS configuration:

cat /etc/resolv.conf

Output:

nameserver 10.96.0.10

search default.svc.cluster.local svc.cluster.local cluster.local nsn-intra.net Home

options ndots:5

The important information is:

nameserver 10.96.0.10
```
This means every Pod sends DNS queries to the Kubernetes internal DNS service.


# 3. Identifying the Kubernetes DNS Service

The DNS Service runs in the kube-system namespace.

Execute:
```bash
kubectl get svc -n kube-system

Output:

NAME             TYPE        CLUSTER-IP

kube-dns         ClusterIP   10.96.0.10
metrics-server   ClusterIP   10.102.80.174

The DNS Service is:

Service Name:
kube-dns

ClusterIP:
10.96.0.10
```
Although the service name is kube-dns, the DNS implementation is CoreDNS.

# 4. Creating a DNS Test Pod

The nginx container does not include the nslookup command.

Create a temporary BusyBox Pod:
```bash
kubectl run dns-test \
--image=busybox:1.36 \
-it \
--rm \
-- sh

Inside the Pod:

cat /etc/resolv.conf

Output:

nameserver 10.96.0.10

search default.svc.cluster.local svc.cluster.local cluster.local nsn-intra.net Home

options ndots:5
```
The DNS configuration is automatically injected by Kubernetes.

# 5. Kubernetes Service Information

The nginx Service is:

```bash
kubectl describe svc nginx-service

Output:

rsoar001@C-PF46ZS03:~$ kubectl describe svc nginx-service
Name:                     nginx-service
Namespace:                default
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.98.248.168
IPs:                      10.98.248.168
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31713/TCP
Endpoints:                10.244.0.3:80,10.244.1.4:80,10.244.2.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>
```

The backend Pods can change, but the Service IP remains stable.

# 6. DNS Name Resolution Test

Inside the DNS test Pod:

Execute:
```bash
nslookup nginx-service.default.svc.cluster.local

Result:

/ # nslookup nginx-service.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53


Name:   nginx-service.default.svc.cluster.local
Address: 10.98.248.168

/ #
```
The DNS server translated:

nginx-service.default.svc.cluster.local

into:

10.98.248.168

The returned IP is the Kubernetes Service ClusterIP.

# 7. Short Name Resolution

Kubernetes allows using shorter names.

Execute:
```bash
/ # nslookup nginx-service
Server:         10.96.0.10
Address:        10.96.0.10:53

** server can't find nginx-service.svc.cluster.local: NXDOMAIN

** server can't find nginx-service.cluster.local: NXDOMAIN

** server can't find nginx-service.svc.cluster.local: NXDOMAIN

** server can't find nginx-service.cluster.local: NXDOMAIN

Name:   nginx-service.default.svc.cluster.local
Address: 10.98.248.168


** server can't find nginx-service.nsn-intra.net: NXDOMAIN

** server can't find nginx-service.nsn-intra.net: NXDOMAIN

** server can't find nginx-service.Home: NXDOMAIN

** server can't find nginx-service.Home: NXDOMAIN



Because of the search configuration:

search default.svc.cluster.local
       svc.cluster.local
       cluster.local

the resolver tries:

nginx-service.default.svc.cluster.local

nginx-service.svc.cluster.local

nginx-service.cluster.local

The successful lookup is:

nginx-service.default.svc.cluster.local

Result:

10.98.248.168

```
# 8. Capturing DNS Traffic with tcpdump

The DNS test Pod is running on:

Node:
minikube-m03

Connect to the node:
```bash
minikube ssh -n minikube-m03

Start packet capture:

sudo tcpdump -i any -nn port 53
```
Now execute from another terminal:
```bash
kubectl exec -it dns-test -- nslookup nginx-service
``` 
# 9. DNS Packet Analysis

The capture shows the Pod sending DNS requests:


Example:

10.244.2.6.38418 > 10.96.0.10.53

A? nginx-service.default.svc.cluster.local

Explanation:

Source:

Pod IP:
10.244.2.6


Destination:

DNS Service:
10.96.0.10

Port:
53 UDP

The Pod does not communicate directly with CoreDNS.

It communicates with the Kubernetes DNS Service.

# 10. kube-dns Service Forwarding

The packet capture also shows:

eth0 Out:

10.244.2.6 > 10.244.0.2.53

This means:

Pod
 |
 |
 v

kube-dns Service

10.96.0.10

 |
 |
 v

CoreDNS Pod

10.244.0.2

The Service forwards the DNS request to one of the CoreDNS Pods.

# 11. DNS Response

CoreDNS responds:

Example:

10.244.0.2.53 > 10.244.2.6

A 10.98.248.168

The response contains:

nginx-service.default.svc.cluster.local

=
    
10.98.248.168

The application now knows the Service ClusterIP.

# 12. Complete DNS Resolution Flow

The complete communication path:

+--------------------------------+
| Application Pod                |
|                                |
| IP: 10.244.2.6                 |
+---------------+----------------+
                |
                |
                | DNS Query
                |
                v

+--------------------------------+
| Kubernetes DNS Service         |
|                                |
| kube-dns                       |
| ClusterIP: 10.96.0.10          |
+---------------+----------------+
                |
                |
                v

+--------------------------------+
| CoreDNS Pod                    |
|                                |
| IP: 10.244.0.2                 |
+---------------+----------------+
                |
                |
                | Service Lookup
                |
                v

+--------------------------------+
| nginx-service                  |
|                                |
| ClusterIP: 10.98.248.168       |
+--------------------------------+

# 13. DNS Resolution and Service Routing

The complete Kubernetes communication process:

Step 1:

Application requests:

nginx-service.default.svc.cluster.local


        |
        v


Step 2:

CoreDNS resolves the name:


nginx-service.default.svc.cluster.local

        |

        v


10.98.248.168

(Service ClusterIP)


        |
        v


Step 3:

Kubernetes networking redirects traffic:

Service IP

        |

        v


Backend Pod:

10.244.0.3
10.244.1.4
10.244.2.2

# 14. Important Observations
Pods automatically receive DNS configuration

Kubernetes automatically creates:

/etc/resolv.conf

inside every Pod.

Example:

nameserver 10.96.0.10
Applications do not need Pod IP addresses

The application uses:

nginx-service.default.svc.cluster.local

instead of:

10.244.0.3
10.244.1.4
10.244.2.2

Kubernetes manages the backend Pod addresses automatically.

DNS is only the first step

DNS resolution provides:

Service ClusterIP

It does not directly return Pod IPs.

The next Kubernetes networking component handles forwarding:

Service ClusterIP

        |

        v

Backend Pod Endpoint
Conclusion

This experiment demonstrated Kubernetes internal DNS service discovery.

The main components involved are:

Pod DNS configuration (/etc/resolv.conf)
kube-dns Service (10.96.0.10)
CoreDNS Pods
Kubernetes Service discovery
DNS packet flow

The final communication flow is:

Pod
 |
 |
DNS Query
 |
 v
kube-dns Service
 |
 |
v
CoreDNS
 |
 |
v
Service ClusterIP
 |
 |
v
Backend Pods

Kubernetes provides a stable service discovery mechanism where applications communicate using DNS names instead of depending on dynamic Pod IP addresses.
