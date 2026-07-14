# Helm Lab with Bitnami Repository



Verificando a versão do helm

```bash
rsoar001@C-PF46ZS03:~/k8s_lab$ helm version
```
version.BuildInfo{Version:"v3.16.1", GitCommit:"5a5449dc42be07001fd5771d56429132984ab3ab", GitTreeState:"clean", GoVersion:"go1.22.7"}


# Module 1 – Working with Helm Repositories

Objetivo

Entender de onde vêm os Charts.

1 Adicionar repositório bitnami

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo add bitnami https://charts.bitnami.com/bitnami
```text
"bitnami" has been added to your repositories

o Helm acessa todos os repositórios cadastrados e baixa a versão mais recente do index.yaml.

2 helm repo update
```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo update
```text
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
```text
Update Complete. ⎈Happy Helming!⎈
```text
rsoar001@C-PF46ZS03:~/k8s_lab$

3 Para listar os repositórios existentes:

```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo list
```text
NAME                    URL
prometheus-community    https://prometheus-community.github.io/helm-charts
bitnami                 https://charts.bitnami.com/bitnami
```text
rsoar001@C-PF46ZS03:~/k8s_lab$

4 para remover um repo
```bash
helm repo remove prometheus-community
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo remove prometheus-community
```text
"prometheus-community" has been removed from your repositories

```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo list
```text
NAME    URL
bitnami https://charts.bitnami.com/bitnami

5 Pesquisar aplicações:

```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo bitnami
```text
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


6 Pesquisar apenas nginx:
```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo nginx
```text
NAME                                    CHART VERSION   APP VERSION    
bitnami/nginx                           25.0.13         1.31.2         
bitnami/nginx-ingress-controller        12.0.7          1.13.1         
bitnami/nginx-intel                     2.1.15          0.4.9          
```text
rsoar001@C-PF46ZS03:~/k8s_lab$

7 Pesquisar todas as versões
```text
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo bitnami/nginx --versions
```text
NAME                                    CHART VERSION   APP VERSION     
bitnami/nginx                           25.0.13         1.31.2          
bitnami/nginx                           25.0.12         1.31.2          
bitnami/nginx                           25.0.11         1.31.2          
bitnami/nginx                           25.0.10         1.31.2          
bitnami/nginx                           25.0.9          1.31.2          
bitnami/nginx                           25.0.8          1.31.2          



# Module 2 – Exploring a Helm Chart


Sem instalar nada.

2.1 Mostrar informações
```bash
helm show chart bitnami/nginx
```

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
```text
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

2.2 Mostrar README
```bash
helm show readme bitnami/nginx
```

2.3 mostrar os valores

``` bash
helm show values bitnami/nginx
```

2.4 Mostrar tudo

``` bash
helm show all bitnami/nginx
```

# Module 3 -- Downloading a Chart

3.1 Baixar

``` bash
helm pull bitnami/nginx
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ helm pull bitnami/nginx
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ ls -la
```text
total 72
```text
drwxr-xr-x  2 rsoar001 rsoar001  4096 Jul 14 12:28 .
```text
drwxr-xr-x 10 rsoar001 rsoar001  4096 Jul 14 12:28 ..
```text
-rw-r--r--  1 rsoar001 rsoar001 62051 Jul 14 12:28 nginx-25.0.13.tgz

3.2 Baixar e descompactar
```bash
helm pull bitnami/nginx --untar
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ helm pull bitnami/nginx --untar
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ ls -latr
```text
total 76
```text
drwxr-xr-x 10 rsoar001 rsoar001  4096 Jul 14 12:28 ..
```text
-rw-r--r--  1 rsoar001 rsoar001 62051 Jul 14 12:28 nginx-25.0.13.tgz
```text
drwxr-xr-x  3 rsoar001 rsoar001  4096 Jul 14 12:31 .
```text
drwxr-xr-x  4 rsoar001 rsoar001  4096 Jul 14 12:31 nginx

3.3 Explorar

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm$ cd nginx/
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ ls -latr
```text
total 188
```text
-rw-r--r-- 1 rsoar001 rsoar001 104816 Jul 14 12:31 README.md
```text
-rw-r--r-- 1 rsoar001 rsoar001   1129 Jul 14 12:31 Chart.yaml
```text
-rw-r--r-- 1 rsoar001 rsoar001    513 Jul 14 12:31 .relok8s-images.yaml
```text
-rw-r--r-- 1 rsoar001 rsoar001    481 Jul 14 12:31 .helmignore
```text
drwxr-xr-x 3 rsoar001 rsoar001   4096 Jul 14 12:31 ..
```text
-rw-r--r-- 1 rsoar001 rsoar001  54863 Jul 14 12:31 values.yaml
```text
drwxr-xr-x 2 rsoar001 rsoar001   4096 Jul 14 12:31 templates
```text
drwxr-xr-x 3 rsoar001 rsoar001   4096 Jul 14 12:31 charts
```text
drwxr-xr-x 4 rsoar001 rsoar001   4096 Jul 14 12:31 .
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$


# Module 4 – Templates


4.1 Gerar os YAMLs Antes de instalar.

```bash
helm template nginx .  ou helm template nginx bitnami/nginx
```

Agora compare com

``` bash
kubectl get deployment -o yaml
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get deployment -o yaml
```text
apiVersion: v1
items: []
kind: List
metadata:
  resourceVersion: ""

Você perceberá que o Helm apenas gera YAML.


# Module 5 – First Installation


5.1 Primeira instalação

```bash
helm install nginx bitnami/nginx
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm install nginx bitnami/nginx
```text
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:36:12 2026
```text
NAMESPACE: default
STATUS: deployed
```text
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2

⚠ WARNING: Since August 28th, 2025, only a limited subset of images/charts are available for free.
    Subscribe to Bitnami Secure Images to receive continued support and security updates.
    More info at https://bitnami.com and https://github.com/bitnami/containers/issues/83267

** Please be patient while the chart is being deployed **
NGINX can be accessed through the following DNS name from within your cluster:

    nginx.default.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w nginx'

    export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services nginx)
    export SERVICE_IP=$(kubectl get svc --namespace default nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"
WARNING: Rolling tag detected (bitnami/nginx:latest), please note that it is strongly recommended to avoid using rolling tags in a production environment.
+info https://techdocs.broadcom.com/us/en/vmware-tanzu/bitnami-secure-images/bitnami-secure-images/services/bsi-doc/apps-tutorials-understand-rolling-tags-containers-index.html
WARNING: Rolling tag detected (bitnami/git:latest), please note that it is strongly recommended to avoid using rolling tags in a production environment.
+info https://techdocs.broadcom.com/us/en/vmware-tanzu/bitnami-secure-images/bitnami-secure-images/services/bsi-doc/apps-tutorials-understand-rolling-tags-containers-index.html
WARNING: Rolling tag detected (bitnami/nginx-exporter:latest), please note that it is strongly recommended to avoid using rolling tags in a production environment.
+info https://techdocs.broadcom.com/us/en/vmware-tanzu/bitnami-secure-images/bitnami-secure-images/services/bsi-doc/apps-tutorials-understand-rolling-tags-containers-index.html

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - cloneStaticSiteFromGit.gitSync.resources
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$

5.2 Ver releases

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm list
```text
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
nginx   default         1               2026-07-14 12:36:12.539157865 -0300 -03 deployed        nginx-25.0.13   1.31.2
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$

5.3 Ver recursos
```bash
kubectl get all
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get all
```text
NAME                        READY   STATUS    RESTARTS   AGE
```text
pod/nginx-86f6cf6c5-wlf8f   1/1     Running   0          4m27s

```text
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
```text
service/kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP                      6d20h
```text
service/nginx        LoadBalancer   10.108.45.84   <pending>     80:32696/TCP,443:32305/TCP   4m28s

```text
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
```text
deployment.apps/nginx   1/1     1            1           4m28s

```text
NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-86f6cf6c5   1         1         1       4m28s
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$


# Module 6 – Exploring the Installation


6.1 Ver valores utilizados
```bash
helm get values nginx
```

6.2 Ver manifests

``` bash
helm get manifest nginx
```

6.3 ver notas

``` bash
helm get notes nginx
```

6.4 ver status

``` bash
helm status nginx
```

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm status nginx
```text
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:36:12 2026
```text
NAMESPACE: default
STATUS: deployed
```text
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2


# Module 7 – Simple Customization


Alterar o número de replica de 1 para 3 e service de LoadBalancer para NodePort

replicaCount: 3

service:
  type: NodePort

  7.1 Aplicar as atualizações
  helm upgrade nginx bitnami/nginx \
    -f values.yaml
```text
  rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm upgrade nginx bitnami/nginx \
    -f values.yaml
```text
Release "nginx" has been upgraded. Happy Helming!
```text
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:48:31 2026
```text
NAMESPACE: default
STATUS: deployed
```text
REVISION: 2
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm list
```text
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
nginx   default         2               2026-07-14 12:48:31.961737483 -0300 -03 deployed        nginx-25.0.13   1.31.2

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get po
```text
NAME                    READY   STATUS    RESTARTS   AGE
nginx-86f6cf6c5-fr2cp   1/1     Running   0          52s
nginx-86f6cf6c5-wlf8f   1/1     Running   0          13m
nginx-86f6cf6c5-zkttr   1/1     Running   0          52s

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get svc
```text
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP                      6d21h
nginx        NodePort    10.108.45.84   <none>        80:32696/TCP,443:32305/TCP   13m
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$



# Module 8 – Overrides


Sem criar arquivo.
```bash
helm upgrade nginx bitnami/nginx \
```

    --set replicaCount=4

``` text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm upgrade nginx bitnami/nginx \
    --set replicaCount=4
```text
Release "nginx" has been upgraded. Happy Helming!
```text
NAME: nginx
LAST DEPLOYED: Tue Jul 14 12:50:26 2026
```text
NAMESPACE: default
STATUS: deployed
```text
REVISION: 3
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 25.0.13
APP VERSION: 1.31.2

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm list
```text
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
nginx   default         3               2026-07-14 12:50:26.625538786 -0300 -03 deployed        nginx-25.0.13   1.31.2

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get po
```text
NAME                    READY   STATUS    RESTARTS   AGE
nginx-86f6cf6c5-dnlmz   1/1     Running   0          29s
nginx-86f6cf6c5-fr2cp   1/1     Running   0          2m23s
nginx-86f6cf6c5-wlf8f   1/1     Running   0          14m
nginx-86f6cf6c5-zkttr   1/1     Running   0          2m23s
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$

Depois:

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm get values nginx
USER-SUPPLIED VALUES:
replicaCount: 4
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$



# Module 9 – Release History

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm history nginx
```text
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Tue Jul 14 12:36:12 2026        superseded      nginx-25.0.13   1.31.2          Install complete
2               Tue Jul 14 12:48:31 2026        superseded      nginx-25.0.13   1.31.2          Upgrade complete
3               Tue Jul 14 12:50:26 2026        deployed        nginx-25.0.13   1.31.2          Upgrade complete
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$



# Module 10 – Rollback


```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm rollback nginx 2
```text
Rollback was a success! Happy Helming!

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ helm history nginx
```text
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Tue Jul 14 12:36:12 2026        superseded      nginx-25.0.13   1.31.2          Install complete
2               Tue Jul 14 12:48:31 2026        superseded      nginx-25.0.13   1.31.2          Upgrade complete
3               Tue Jul 14 12:50:26 2026        superseded      nginx-25.0.13   1.31.2          Upgrade complete
4               Tue Jul 14 12:53:03 2026        deployed        nginx-25.0.13   1.31.2          Rollback to 2

```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get po
```text
NAME                    READY   STATUS    RESTARTS   AGE
nginx-86f6cf6c5-dnlmz   1/1     Running   0          2m59s
nginx-86f6cf6c5-wlf8f   1/1     Running   0          17m
nginx-86f6cf6c5-zkttr   1/1     Running   0          4m53s
```text
rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$


# Module 11 – Uninstall


```bash
helm uninstall nginx
```

\`\`\`text rsoar001@C-PF46ZS03:\~/k8s_lab/05-helm/nginx\$ helm uninstall
nginx release "nginx" uninstalled

`text rsoar001@C-PF46ZS03:~/k8s_lab/05-helm/nginx$ kubectl get all`text
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
`text service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d21h`text
rsoar001@C-PF46ZS03:\~/k8s_lab/05-helm/nginx\$
