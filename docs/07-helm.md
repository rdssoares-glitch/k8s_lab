# Helm Lab with Bitnami Repository

checking helm version

```bash
rsoar001@C-PF46ZS03:~/k8s_lab$ helm version
```
output:

```text
version.BuildInfo{Version:"v3.16.1", GitCommit:"5a5449dc42be07001fd5771d56429132984ab3ab", GitTreeState:"clean", GoVersion:"go1.22.7"}
```

# Module 1 – Working with Helm Repositories

Objective

Understand where Charts come from.

## 1 Add bitnami repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```
output:

```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```
## 2 helm repo update
The helm repo update command accesses all registered repositories and downloads the latest version of index.yaml.

```bash
helm repo update
```

```bash
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```


## 3 To list existing repositories:

```bash
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo list
NAME                    URL
prometheus-community    https://prometheus-community.github.io/helm-charts
bitnami                 https://charts.bitnami.com/bitnami
```


## 4 To remove a repo:

```bash
helm repo remove prometheus-community
```
output:

``` text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo remove prometheus-community
"prometheus-community" has been removed from your repositories
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo list

NAME    URL
bitnami https://charts.bitnami.com/bitnami
```

## 5 Search application:

```bash
helm search repo bitnami
```
output:
``` text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo bitnami

NAME                                            CHART VERSION   APP VERSION  
bitnami/airflow                                 25.0.2          3.0.5        
bitnami/apache                                  11.4.29         2.4.65       
bitnami/apisix                                  6.0.0           3.13.0       
bitnami/appsmith                                7.0.3           1.85.0       
bitnami/argo-cd                                 11.0.0          3.1.1        
bitnami/argo-workflows                          13.0.6          3.7.1        
bitnami/aspnet-core                             9.4.9           10.0.9       
bitnami/cadvisor                                0.1.13          0.53.0       
bitnami/cassandra                               12.3.11         5.0.5        
bitnami/cert-manager                            1.5.14          1.18.2       
bitnami/chainloop                               4.0.75          1.43.1       
bitnami/cilium                                  3.1.9           1.18.1       
bitnami/clickhouse                              9.4.4           25.7.5       
bitnami/clickhouse-operator                     0.2.33          0.25.3       
bitnami/cloudnative-pg                          1.0.11          1.26.1       
bitnami/common                                  2.41.0          2.41.0       
bitnami/concourse                               5.1.45          7.13.2       
```

## 6 Search for nginx:
```bash
helm search repo nginx
```
output: 

```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo nginx
NAME                                    CHART VERSION   APP VERSION    
bitnami/nginx                           25.0.13         1.31.2         
bitnami/nginx-ingress-controller        12.0.7          1.13.1         
bitnami/nginx-intel                     2.1.15          0.4.9          
```

## 7 search for available versions:
```bash
helm search repo bitnami/nginx --versions
```
output: 
```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo bitnami/nginx --versions
NAME                                    CHART VERSION   APP VERSION     
bitnami/nginx                           25.0.13         1.31.2          
bitnami/nginx                           25.0.12         1.31.2          
bitnami/nginx                           25.0.11         1.31.2          
bitnami/nginx                           25.0.10         1.31.2          
bitnami/nginx                           25.0.9          1.31.2          
bitnami/nginx                           25.0.8          1.31.2          
```
# Module 2 – Exploring a Helm Chart

2.1 Show charts information:
```bash
helm show chart bitnami/nginx
```
Output:

``` text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm show chart bitnami/nginx
annotations:
  fips: "true"
  images: |
    - name: git
      version: 2.55.0
      image: registry-1.docker.io/bitnami/git:latest
    - name: nginx
      version: 1.31.2
      image: registry-1.docker.io/bitnami/nginx:latest
    - name: nginx-exporter
      version: 1.5.1
      image: registry-1.docker.io/bitnami/nginx-exporter:latest
  licenses: Apache-2.0
  tanzuCategory: clusterUtility
apiVersion: v2
appVersion: 1.31.2
dependencies:
- name: common
  repository: oci://registry-1.docker.io/bitnamicharts
  tags:
  - bitnami-common
  version: 2.41.0
description: NGINX Open Source is a web server that can be also used as a reverse
  proxy, load balancer, and HTTP cache. Recommended for high-demanding sites due to
  its ability to provide faster content.
home: https://bitnami.com
icon: https://dyltqmyl993wv.cloudfront.net/assets/stacks/nginx/img/nginx-stack-220x234.png
keywords:
- nginx
- http
- web
- www
- reverse proxy
maintainers:
- name: Broadcom, Inc. All Rights Reserved.
  url: https://github.com/bitnami/charts
name: nginx
sources:
- https://github.com/bitnami/charts/tree/main/bitnami/nginx
version: 25.0.13
```
## 2.2 Show README
```bash
helm show readme bitnami/nginx
```

## 2.3 Show values

``` bash
helm show values bitnami/nginx
```

## 2.4 show all details

``` bash
helm show all bitnami/nginx
```

# Module 3 -- Downloading a Chart

## 3.1 Download

``` bash
helm pull bitnami/nginx
```
output: 
``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ helm pull bitnami/nginx
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ ls -la
total 72
drwxr-xr-x  2 rsoar001 rsoar001  4096 Jul 14 12:28 .
drwxr-xr-x 10 rsoar001 rsoar001  4096 Jul 14 12:28 ..
-rw-r--r--  1 rsoar001 rsoar001 62051 Jul 14 12:28 nginx-25.0.13.tgz
``` 

## 3.2 Download and unpack

```bash
helm pull bitnami/nginx --untar
```
Output:

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ helm pull bitnami/nginx --untar
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ ls -latr
total 76
drwxr-xr-x 10 rsoar001 rsoar001  4096 Jul 14 12:28 ..
-rw-r--r--  1 rsoar001 rsoar001 62051 Jul 14 12:28 nginx-25.0.13.tgz
drwxr-xr-x  3 rsoar001 rsoar001  4096 Jul 14 12:31 .
drwxr-xr-x  4 rsoar001 rsoar001  4096 Jul 14 12:31 nginx
```

## 3.3 Explore directory structure

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ cd nginx/
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ ls -latr
total 188
-rw-r--r-- 1 rsoar001 rsoar001 104816 Jul 14 12:31 README.md
-rw-r--r-- 1 rsoar001 rsoar001   1129 Jul 14 12:31 Chart.yaml
-rw-r--r-- 1 rsoar001 rsoar001    513 Jul 14 12:31 .relok8s-images.yaml
-rw-r--r-- 1 rsoar001 rsoar001    481 Jul 14 12:31 .helmignore
drwxr-xr-x 3 rsoar001 rsoar001   4096 Jul 14 12:31 ..
-rw-r--r-- 1 rsoar001 rsoar001  54863 Jul 14 12:31 values.yaml
drwxr-xr-x 2 rsoar001 rsoar001   4096 Jul 14 12:31 templates
drwxr-xr-x 3 rsoar001 rsoar001   4096 Jul 14 12:31 charts
drwxr-xr-x 4 rsoar001 rsoar001   4096 Jul 14 12:31 .
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$
```

# Module 4 – Templates


## 4.1 Generate YAMLs before install.

```bash
helm template nginx .  ou helm template nginx bitnami/nginx
```

Now compare with:

``` bash
kubectl get deployment -o yaml
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get deployment -o yaml
apiVersion: v1
items: []
kind: List
metadata:
  resourceVersion: ""
```
You will notice that Helm only generates YAML files locally and doesn't apply them yet.

# Module 5 – First Installation

## 5.1 Nginx installation

```bash
helm install nginx bitnami/nginx
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm install nginx bitnami/nginx
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:36:12 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2
```

## 5.2 Show releases

```bash
helm list
```
output: 

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
nginx   default         1               2026-07-14 12:36:12.539157865 -0300 -03 deployed        nginx-25.0.13   1.31.2
```
## 5.3 Show resources
```bash
kubectl get all
```
output: 
``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-86f6cf6c5-wlf8f   1/1     Running   0          4m27s
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP                      6d20h
service/nginx        LoadBalancer   10.108.45.84   <pending>     80:32696/TCP,443:32305/TCP   4m28s
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           4m28s
NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-86f6cf6c5   1         1         1       4m28s
```
# Module 6 – Exploring the Installation


## 6.1 View user-supplied values
```bash
helm get values nginx
```

## 6.2 view maifests

``` bash
helm get manifest nginx
```

## 6.3 view notes

``` bash
helm get notes nginx
```

## 6.4 view status

``` bash
helm status nginx
```
output:

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm status nginx
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:36:12 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2
```

# Module 7 – Simple Customization

Change the number of replicas from 1 to 3 and service type from LoadBalancer to NodePort:

## 7.1 Apply changes
 ```bash 
helm upgrade nginx bitnami/nginx -f values.yaml
```
output:

```text
  rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm upgrade nginx bitnami/nginx -f values.yaml
Release "nginx" has been upgraded. Happy Helming!
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:48:31 2026
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2
```
output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
nginx   default         2               2026-07-14 12:48:31.961737483 -0300 -03 deployed        nginx-25.0.13   1.31.2

rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
nginx-86f6cf6c5-fr2cp   1/1     Running   0          52s
nginx-86f6cf6c5-wlf8f   1/1     Running   0          13m
nginx-86f6cf6c5-zkttr   1/1     Running   0          52s

rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get svc

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP                      6d21h
nginx        NodePort    10.108.45.84   <none>        80:32696/TCP,443:32305/TCP   13m
```

# Module 8 – Overrides

overrides without files.
```bash
helm upgrade nginx bitnami/nginx --set replicaCount=4
```
output:

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm upgrade nginx bitnami/nginx --set replicaCount=4
Release "nginx" has been upgraded. Happy Helming!
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:50:26 2026
NAMESPACE: default
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2

rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
nginx   default         3               2026-07-14 12:50:26.625538786 -0300 -03 deployed        nginx-25.0.13   1.31.2
```
Checking the number of replicas after the Helm upgrade:

```bash
kubectl get po
```
output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get po

NAME                    READY   STATUS    RESTARTS   AGE
nginx-86f6cf6c5-dnlmz   1/1     Running   0          29s
nginx-86f6cf6c5-fr2cp   1/1     Running   0          2m23s
nginx-86f6cf6c5-wlf8f   1/1     Running   0          14m
nginx-86f6cf6c5-zkttr   1/1     Running   0          2m23s
```

After:

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm get values nginx
USER-SUPPLIED VALUES:
replicaCount: 4
```
# Module 9 – Release History

Checking Helm release history:
```bash
helm history nginx
```
output: 
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm history nginx
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Tue Jul 14 12:36:12 2026        superseded      nginx-25.0.13   1.31.2          Install complete
2               Tue Jul 14 12:48:31 2026        superseded      nginx-25.0.13   1.31.2          Upgrade complete
3               Tue Jul 14 12:50:26 2026        deployed        nginx-25.0.13   1.31.2          Upgrade complete
```
# Module 10 – Rollback

To perform a rollback to a previous Helm release revision:

```bash
helm rollback nginx 2
```
output: 

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm rollback nginx 2
Rollback was a success! Happy Helming!

rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm history nginx

REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Tue Jul 14 12:36:12 2026        superseded      nginx-25.0.13   1.31.2          Install complete
2               Tue Jul 14 12:48:31 2026        superseded      nginx-25.0.13   1.31.2          Upgrade complete
3               Tue Jul 14 12:50:26 2026        superseded      nginx-25.0.13   1.31.2          Upgrade complete
4               Tue Jul 14 12:53:03 2026        deployed        nginx-25.0.13   1.31.2          Rollback to 2
```
Checking the updated replica count:

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
nginx-86f6cf6c5-dnlmz   1/1     Running   0          2m59s
nginx-86f6cf6c5-wlf8f   1/1     Running   0          17m
nginx-86f6cf6c5-zkttr   1/1     Running   0          4m53s
```

# Module 11 – Uninstall


```bash
helm uninstall nginx
```
output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx\$ helm uninstall
nginx release "nginx" uninstalled

rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get all`text
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d21h`text
rsoar001@C-PF46ZS03:\~/k8s_lab/05-helm/nginx\$
```
