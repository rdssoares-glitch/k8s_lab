# Kubernetes Storage Lab (Minikube)

## Checking Existing StorageClasses

```bash
kubectl get storageclass
```

output:

```text
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  5d18h
```

The default Minikube provisioner is:

```text
k8s.io/minikube-hostpath
```

This means that Minikube includes a dynamic provisioner, allowing you to practice both Dynamic Provisioning and Static Provisioning.

---

# Exercise 1 – Create a PVC (Dynamic Provisioning)

## PVC manifest

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply it:

```bash
kubectl apply -f nginx-pvc.yaml
```

Verify:

```bash
kubectl get pvc
kubectl get pv
```

output:

```text
PVC

NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
nginx-pvc   Bound    pvc-484834ff-7b1b-48ac-8511-6235935f80b8   1Gi        RWO            standard
```

```text
PV

NAME                                       STATUS   CLAIM
pvc-484834ff-7b1b-48ac-8511-6235935f80b8   Bound    default/nginx-pvc
```

Notice that the **PV was created automatically** by the StorageClass..

---

# Exercise 2 – Find Where the Volume Is Stored

Inspect the PV:

```bash
kubectl describe pv pvc-484834ff-7b1b-48ac-8511-6235935f80b8
```

Look for:

```text
Source:
  Type: HostPath
  Path: /tmp/hostpath-provisioner/default/nginx-pvc
```

Access the Minikube node:

```bash
minikube ssh
```

Check the directory:

```bash
ls -latr /tmp/hostpath-provisioner/default/
```

Output:

```text
nginx-pvc
```

### Dynamic Provisioning Flow

```text
PVC
 │
 ▼
StorageClass (standard)
 │
 ▼
Provisionador hostPath
 │
 ▼
HostPath
```

---

# Exercise 3 – Mount the PVC into a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-storage
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: meu-volume

  volumes:
  - name: meu-volume
    persistentVolumeClaim:
      claimName: nginx-pvc
```

Apply it:

```bash
kubectl apply -f pod_nginx_storage.yaml
```

Get into the Pod:

```bash
kubectl exec -it nginx-storage -- bash
```

Create a file:

```bash
echo "Rodrigo Kubernetes Lab" > /usr/share/nginx/html/index.html
```

---

# Exercise 4 – Data Persistence

Delete the Pod:

```bash
kubectl delete pod nginx-storage
```

Create it again:

```bash
kubectl apply -f pod_nginx_storage.yaml
```

Verify:

```bash
kubectl exec -it nginx-storage -- cat /usr/share/nginx/html/index.html
```

output:

```text
Rodrigo Kubernetes Lab
```

The file remains because it is stored on the **PersistentVolume**, not in the container's ephemeral filesystem.

---

# Exercise 5 – Inspect the Physical Directory

Find out which node the Pod is running on:

```bash
kubectl get pod nginx-storage -o wide
```

output:

```text
NODE
minikube-m02
```

Connect to the correct node:

```bash
minikube ssh -n minikube-m02
```

Verify:

```bash
ls -lah /tmp/hostpath-provisioner/default/nginx-pvc
```

output:

```text
index.html
```

Display its contents::

```bash
cat /tmp/hostpath-provisioner/default/nginx-pvc/index.html
```

```text
Rodrigo Kubernetes Lab
```

## What Does This Demonstrate?

**hostPath** provides storage that is local to the node.

```text
hostPath
     │
     ▼
local disk Node
```

If the Pod is scheduled on another node, the volume will also be located on that node.

This behavior differs from distributed storage solutions such as **Ceph RBD**, where storage is shared across all worker nodes in the cluster.

---

---

# Exercise 6 – Create a PersistentVolume Manually

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-lab
spec:
  capacity:
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /mydisk
```

Apply it:

```bash
kubectl apply -f pv-lab.yaml
```

---

# Exercise 7 – Create a PVC Using the Manual PV

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-lab
spec:
  accessModes:
    - ReadWriteOnce

  volumeName: pv-lab

  storageClassName: ""

  resources:
    requests:
      storage: 1Gi
```

Apply it:

```bash
kubectl apply -f pvc-lab.yaml
```

Verify:

```bash
kubectl get pv
kubectl get pvc
```

Output:

```text
PV

NAME      STATUS   CLAIM
pv-lab    Bound    default/pvc-lab
```

```text
PVC

NAME      STATUS   VOLUME
pvc-lab   Bound    pv-lab
```

## How Did the Binding Occur?

Kubernetes compared:

- StorageClass
- Capacity
- AccessModes
- VolumeName

Since all attributes were compatible, Kubernetes performed the **binding**.

```text
PVC
 │
 │ Binding
 ▼
PV
```

Using:

```yaml
storageClassName: ""
```

means:

> Do not use any StorageClass. Instead, look for a PersistentVolume that also does not have a StorageClass assigned.

---

# Exercise 8 – Simulate an Insufficient Capacity Error

PVC requesting **5 GiB**:

```yaml
resources:
  requests:
    storage: 5Gi
```

Available PV:

```text
2Gi
```

Result:

```bash
kubectl describe pvc pvc-lab-5gi
```

```text
Cannot bind to requested volume "pv-lab": requested PV is too small
```

The PVC remains in:

```text
STATUS: Pending
```

---

# Exercise 9 – Simulate an AccessMode Mismatch

PV:

```yaml
accessModes:
  - ReadOnlyMany
```

PVC:

```yaml
accessModes:
  - ReadWriteOnce
```

Result:

```text
Cannot bind to requested volume "pv-lab": incompatible accessMode
```

The PVC remains in:

```text
STATUS: Pending
```

---

# Exercise 10 – Reclaim Policy

Check the PV policy:

```bash
kubectl describe pv pv-lab
```

Output:

```text
Reclaim Policy: Retain
```

## Write a File

Mount the PVC into a Pod and run:

```bash
echo "Reclaim policy test" > /usr/share/nginx/html/test.txt
```

Find the node:

```bash
kubectl get pod nginx-storage -o wide
```

Connect to the node:

```bash
minikube ssh -n minikube-m02
```

Verify:

```bash
cat /mydisk/test.txt
```

Output:

```text
Reclaim policy test
```

---

## Delete the Pod and the PVC

```bash
kubectl delete pod nginx-storage
kubectl delete pvc pvc-lab
```

Check the PV:

```bash
kubectl get pv pv-lab
```

Output:

```text
NAME      STATUS
pv-lab    Released
```

Notice that:

- the PVC has been deleted;
- the PV still exists;
- the data remains stored.

Verify again:

```bash
cat /mydisk/test.txt
```

Output:

```text
Reclaim policy test
```

## Reclaim Policy = Retain Flow

```text
PV
 │
 ▼
PVC
 │
 ▼
Delete PVC
 │
 ▼
PV → Released
 │
 ▼
Data remains on disk
```

The purpose of the **Retain** reclaim policy is to preserve the data even after the PVC has been deleted, allowing the cluster administrator to decide when and how the PersistentVolume should be reused or removed.

---

# Exercise 11 – Expand a PVC
