
rsoar001@C-PF46ZS03:~/k8s_lab/02-grafana$ kubectl get storageclass
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  5d18h

k8s.io/minikube-hostpath 
Isso significa que o Minikube possui um provisionador dinâmico, então você pode praticar tanto provisionamento dinâmico quanto estático.

Exercício 1 - Criar um PVC (Provisionamento Dinâmico)
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

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl apply -f nginx-pvc.yaml
persistentvolumeclaim/nginx-pvc created

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
nginx-pvc     Bound    pvc-484834ff-7b1b-48ac-8511-6235935f80b8   1Gi        RWO            standard       <unset>                 6s

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-484834ff-7b1b-48ac-8511-6235935f80b8   1Gi        RWO            Delete           Bound    default/nginx-pvc     standard       <unset>                          24s

Exercício 2 - Descobrir onde o volume está

Procure por hostPath

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl describe pv pvc-484834ff-7b1b-48ac-8511-6235935f80b8
Name:            pvc-484834ff-7b1b-48ac-8511-6235935f80b8
Labels:          <none>
Annotations:     hostPathProvisionerIdentity: f5994e37-7da1-401e-a4b6-0ea8e518a608
                 pv.kubernetes.io/provisioned-by: k8s.io/minikube-hostpath
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Bound
Claim:           default/nginx-pvc
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/hostpath-provisioner/default/nginx-pvc 
    HostPathType:
Events:            <none>

Agora entre no Minikube:

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ minikube ssh
docker@minikube:~$
docker@minikube:/home$ ls -latr /tmp/hostpath-provisioner/default/
total 16
drwxrwxrwx 2 root root 4096 Jul 13 13:21 nginx-pvc
drwxr-xr-x 4 root root 4096 Jul 13 13:21 .
docker@minikube:/home$

Você verá o diretório criado automaticamente.

PVC
     │
     ▼
StorageClass
     │
     ▼
Provisionador
     │
     ▼
HostPath

Exercício 3 - Montar o PVC em um Pod

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
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl apply -f pod_nginx_storage.yaml
pod/nginx-storage created

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl get po
NAME                            READY   STATUS    RESTARTS      AGE
dns-test                        1/1     Running   1 (36m ago)   2d18h
grafana-5f6cdf4b4f-77cwb        1/1     Running   3 (37m ago)   5d
nginx-57457b7d55-b8bd2          1/1     Running   4 (37m ago)   5d18h
nginx-57457b7d55-kmrsj          1/1     Running   4 (37m ago)   5d16h
nginx-57457b7d55-qbxtt          1/1     Running   4 (36m ago)   5d16h
nginx-storage                   1/1     Running   0             6s
prometheus-78d4b7db4d-j8vww     1/1     Running   0             22m
teste-static-pod-minikube-m02   1/1     Running   0             16m
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$

Entre no Pod:

kubectl exec -it nginx-storage -- bash

Crie um arquivo:

echo "Rodrigo Kubernetes Lab" > /usr/share/nginx/html/index.html

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl exec -it nginx-storage -- bash
root@nginx-storage:/# echo "Rodrigo Kubernetes Lab" > /usr/share/nginx/html/index.html
root@nginx-storage:/#

Exercício 4 - Apagar o Pod

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl delete pod nginx-storage
pod "nginx-storage" deleted from default namespace
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl get po
NAME                            READY   STATUS    RESTARTS      AGE
dns-test                        1/1     Running   1 (39m ago)   2d18h
grafana-5f6cdf4b4f-77cwb        1/1     Running   3 (40m ago)   5d
nginx-57457b7d55-b8bd2          1/1     Running   4 (40m ago)   5d18h
nginx-57457b7d55-kmrsj          1/1     Running   4 (40m ago)   5d16h
nginx-57457b7d55-qbxtt          1/1     Running   4 (39m ago)   5d16h
prometheus-78d4b7db4d-j8vww     1/1     Running   0             25m
teste-static-pod-minikube-m02   1/1     Running   0             19m
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$

Crie novamente exatamente o mesmo Pod.

Entre nele:
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl apply -f pod_nginx_storage.yaml
pod/nginx-storage created
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl get po
NAME                            READY   STATUS    RESTARTS      AGE
dns-test                        1/1     Running   1 (40m ago)   2d18h
grafana-5f6cdf4b4f-77cwb        1/1     Running   3 (41m ago)   5d
nginx-57457b7d55-b8bd2          1/1     Running   4 (41m ago)   5d18h
nginx-57457b7d55-kmrsj          1/1     Running   4 (41m ago)   5d16h
nginx-57457b7d55-qbxtt          1/1     Running   4 (40m ago)   5d16h
nginx-storage                   1/1     Running   0             2s
prometheus-78d4b7db4d-j8vww     1/1     Running   0             26m
teste-static-pod-minikube-m02   1/1     Running   0             20m
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$


rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl exec -it nginx-storage -- bash
root@nginx-storage:/# cat /usr/share/nginx/html/index.html
Rodrigo Kubernetes Lab
root@nginx-storage:/#
O arquivo ainda estará lá.

Esse é o momento em que fica claro o papel do PVC.

Exercício 5 - Verificar o diretório no node que o pod foi assigned:

rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ kubectl get po nginx-storage -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
nginx-storage   1/1     Running   0          15m   10.244.1.9   minikube-m02   <none>           <none>
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$

Pod está em execução no node minikube-m02
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ minikube ssh -n minikube-m02
docker@minikube-m02:~$  ls -lah /tmp/hostpath-provisioner/default/nginx-pvc
total 12K
drwxr-xr-x 2 root root 4.0K Jul 13 13:37 .
drwxr-xr-x 4 root root 4.0K Jul 13 13:36 ..
-rw-r--r-- 1 root root   23 Jul 13 13:37 index.html
docker@minikube-m02:~$ cd /tmp/hostpath-provisioner/default/nginx-pvc
docker@minikube-m02:/tmp/hostpath-provisioner/default/nginx-pvc$ cat index.html
Rodrigo Kubernetes Lab


O que isso ensina sobre HostPath

Esse comportamento evidencia uma característica importante do hostPath:

hostPath
      │
      ▼
Armazenamento LOCAL ao nó

Ou seja, o armazenamento não é compartilhado entre os nós.

Se o Pod estiver no minikube-m02, os dados ficam no disco do minikube-m02.

Se o Pod estiver no minikube-m03, os dados ficam no disco do minikube-m03.
Esse é justamente o motivo de existirem soluções como Ceph,Nesse caso, 
o volume não pertence ao disco local de um único nó. Ele fica distribuído no cluster Ceph
e pode ser montado por qualquer worker autorizado, permitindo que o Pod seja reagendado sem perder acesso aos dados.
Essa diferença entre hostPath (local ao nó) e um armazenamento distribuído como Ceph RBD é um dos conceitos mais importantes quando se passa de um laboratório Minikube para um ambiente de produção. Seu experimento acabou ilustrando isso de forma muito clara.

Exercício 6 - Criar um PV manualmente
rsoar001@C-PF46ZS03:~/k8s_lab/01-nginx$ minikube ssh
docker@minikube:~$ sudo mkdir /mydisk
