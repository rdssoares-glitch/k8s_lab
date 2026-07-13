# Laboratório de Armazenamento no Kubernetes (Minikube)

## Verificando as StorageClasses existentes

```bash
kubectl get storageclass
```

Saída:

```text
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  5d18h
```

O provisionador padrão do Minikube é:

```text
k8s.io/minikube-hostpath
```

Isso significa que o Minikube possui um **provisionador dinâmico**, permitindo praticar tanto **Provisionamento Dinâmico** quanto **Provisionamento Estático**.

---

# Exercício 1 – Criar um PVC (Provisionamento Dinâmico)

## Manifesto do PVC

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

Aplicação:

```bash
kubectl apply -f nginx-pvc.yaml
```

Verificando:

```bash
kubectl get pvc
kubectl get pv
```

Resultado:

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

Observe que **o PV foi criado automaticamente** pela StorageClass.

---

# Exercício 2 – Descobrir onde o volume está

Verifique o PV:

```bash
kubectl describe pv pvc-484834ff-7b1b-48ac-8511-6235935f80b8
```

Procure por:

```text
Source:
  Type: HostPath
  Path: /tmp/hostpath-provisioner/default/nginx-pvc
```

Entre no Minikube:

```bash
minikube ssh
```

Verifique o diretório:

```bash
ls -latr /tmp/hostpath-provisioner/default/
```

Resultado:

```text
nginx-pvc
```

### Fluxo do Provisionamento Dinâmico

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

# Exercício 3 – Montar o PVC em um Pod

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

Aplicação:

```bash
kubectl apply -f pod_nginx_storage.yaml
```

Entre no Pod:

```bash
kubectl exec -it nginx-storage -- bash
```

Crie um arquivo:

```bash
echo "Rodrigo Kubernetes Lab" > /usr/share/nginx/html/index.html
```

---

# Exercício 4 – Persistência dos dados

Remova o Pod:

```bash
kubectl delete pod nginx-storage
```

Crie novamente:

```bash
kubectl apply -f pod_nginx_storage.yaml
```

Verifique:

```bash
kubectl exec -it nginx-storage -- cat /usr/share/nginx/html/index.html
```

Resultado:

```text
Rodrigo Kubernetes Lab
```

O arquivo permaneceu porque está armazenado no **PersistentVolume**, não no sistema de arquivos efêmero do container.

---

# Exercício 5 – Verificar o diretório físico

Descubra em qual nó o Pod foi executado:

```bash
kubectl get pod nginx-storage -o wide
```

Resultado:

```text
NODE
minikube-m02
```

Entre no nó correto:

```bash
minikube ssh -n minikube-m02
```

Verifique:

```bash
ls -lah /tmp/hostpath-provisioner/default/nginx-pvc
```

Resultado:

```text
index.html
```

Conteúdo:

```bash
cat /tmp/hostpath-provisioner/default/nginx-pvc/index.html
```

```text
Rodrigo Kubernetes Lab
```

## O que isso demonstra?

O **hostPath** é um armazenamento **local ao nó**.

```text
hostPath
     │
     ▼
Disco local do Node
```

Se o Pod for executado em outro nó, o volume também estará nesse outro nó.

Esse comportamento é diferente de soluções distribuídas como **Ceph RBD**, onde o armazenamento é compartilhado entre todos os workers do cluster.

---

# Exercício 6 – Criar um PV manualmente

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

Aplicação:

```bash
kubectl apply -f pv-lab.yaml
```

---

# Exercício 7 – Criar um PVC utilizando o PV manual

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

Aplicação:

```bash
kubectl apply -f pvc-lab.yaml
```

Verificando:

```bash
kubectl get pv
kubectl get pvc
```

Resultado:

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

## Como ocorreu o Binding?

O Kubernetes comparou:

- StorageClass
- Capacidade
- AccessModes
- VolumeName

Como todos eram compatíveis, realizou o **Binding**.

```text
PVC
 │
 │ Binding
 ▼
PV
```

O uso de:

```yaml
storageClassName: ""
```

significa:

> Não utilize nenhuma StorageClass; procure um PV que também não possua StorageClass.

---

# Exercício 8 – Simular erro de capacidade

PVC solicitando **5 GiB**:

```yaml
resources:
  requests:
    storage: 5Gi
```

PV disponível:

```text
2Gi
```

Resultado:

```bash
kubectl describe pvc pvc-lab-5gi
```

```text
Cannot bind to requested volume "pv-lab": requested PV is too small
```

O PVC permanece:

```text
STATUS: Pending
```

---

# Exercício 9 – Simular incompatibilidade de AccessMode

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

Resultado:

```text
Cannot bind to requested volume "pv-lab": incompatible accessMode
```

O PVC permanece:

```text
STATUS: Pending
```

---

# Exercício 10 – Reclaim Policy

Verifique a política do PV:

```bash
kubectl describe pv pv-lab
```

Resultado:

```text
Reclaim Policy: Retain
```

## Grave um arquivo

Monte o PVC em um Pod e execute:

```bash
echo "Reclaim policy test" > /usr/share/nginx/html/test.txt
```

Descubra o nó:

```bash
kubectl get pod nginx-storage -o wide
```

Entre no nó:

```bash
minikube ssh -n minikube-m02
```

Verifique:

```bash
cat /mydisk/test.txt
```

Resultado:

```text
Reclaim policy test
```

---

## Delete o Pod e o PVC

```bash
kubectl delete pod nginx-storage
kubectl delete pvc pvc-lab
```

Verifique o PV:

```bash
kubectl get pv pv-lab
```

Resultado:

```text
NAME      STATUS
pv-lab    Released
```

Observe que:

- o PVC foi removido;
- o PV permanece;
- os dados continuam armazenados.

Verifique novamente:

```bash
cat /mydisk/test.txt
```

Resultado:

```text
Reclaim policy test
```

## Fluxo do Reclaim Policy = Retain

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
Dados permanecem no disco
```

O objetivo da política **Retain** é preservar os dados mesmo após a remoção do PVC, permitindo que o administrador decida quando e como reutilizar ou remover esse volume.
