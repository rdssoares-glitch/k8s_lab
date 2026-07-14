
Minha recomendação para quem está começando

O Bitnami é provavelmente o melhor repositório para aprender Helm.

Ele possui dezenas de aplicações populares:

nginx
apache
mariadb
mysql
postgresql
mongodb
redis
rabbitmq
kafka
keycloak
wordpress
jenkins
minio

Verificando a versão do helm
rsoar001@C-PF46ZS03:~/k8s_lab$ helm version
version.BuildInfo{Version:"v3.16.1", GitCommit:"5a5449dc42be07001fd5771d56429132984ab3ab", GitTreeState:"clean", GoVersion:"go1.22.7"}

Módulo 1 – Conhecendo o repositório
Objetivo

Entender de onde vêm os Charts.

1 Adicionar repositório bitnami

helm repo add bitnami https://charts.bitnami.com/bitnami
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

o Helm acessa todos os repositórios cadastrados e baixa a versão mais recente do index.yaml.

2 helm repo update
rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
rsoar001@C-PF46ZS03:~/k8s_lab$

3 Para listar os repositórios existentes:

rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo list
NAME                    URL
prometheus-community    https://prometheus-community.github.io/helm-charts
bitnami                 https://charts.bitnami.com/bitnami
rsoar001@C-PF46ZS03:~/k8s_lab$

4 para remover um repo 
helm repo remove prometheus-community

rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo remove prometheus-community
"prometheus-community" has been removed from your repositories

rsoar001@C-PF46ZS03:~/k8s_lab$ helm repo list
NAME    URL
bitnami https://charts.bitnami.com/bitnami

5 Pesquisar aplicações:

rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo bitnami
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/airflow                                 25.0.2          3.0.5           Apache Airflow is a tool to express and execute...
bitnami/apache                                  11.4.29         2.4.65          Apache HTTP Server is an open-source HTTP serve...
bitnami/apisix                                  6.0.0           3.13.0          Apache APISIX is high-performance, real-time AP...
bitnami/appsmith                                7.0.3           1.85.0          Appsmith is an open source platform for buildin...
bitnami/argo-cd                                 11.0.0          3.1.1           Argo CD is a continuous delivery tool for Kuber...
bitnami/argo-workflows                          13.0.6          3.7.1           Argo Workflows is meant to orchestrate Kubernet...
bitnami/aspnet-core                             9.4.9           10.0.9          ASP.NET Core is an open-source framework for we...
bitnami/cadvisor                                0.1.13          0.53.0          cAdvisor (Container Advisor) is an open-source ...
bitnami/cassandra                               12.3.11         5.0.5           Apache Cassandra is an open source distributed ...
bitnami/cert-manager                            1.5.14          1.18.2          cert-manager is a Kubernetes add-on to automate...
bitnami/chainloop                               4.0.75          1.43.1          Chainloop is an open-source Software Supply Cha...
bitnami/cilium                                  3.1.9           1.18.1          Cilium is an eBPF-based networking, observabili...
bitnami/clickhouse                              9.4.4           25.7.5          ClickHouse is an open-source column-oriented OL...
bitnami/clickhouse-operator                     0.2.33          0.25.3          ClickHouse Operator is a production-ready opera...
bitnami/cloudnative-pg                          1.0.11          1.26.1          CloudNativePG is an open-source tool for managi...
bitnami/common                                  2.41.0          2.41.0          A Library Helm Chart for grouping common logic ...
bitnami/concourse                               5.1.45          7.13.2          Concourse is an automation system written in Go...


6 Pesquisar apenas nginx:
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo nginx
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/nginx                           25.0.13         1.31.2          NGINX Open Source is a web server that can be a...
bitnami/nginx-ingress-controller        12.0.7          1.13.1          NGINX Ingress Controller is an Ingress controll...
bitnami/nginx-intel                     2.1.15          0.4.9           DEPRECATED NGINX Open Source for Intel is a lig...
rsoar001@C-PF46ZS03:~/k8s_lab$

7 Pesquisar todas as versões
rsoar001@C-PF46ZS03:~/k8s_lab$ helm search repo bitnami/nginx --versions
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/nginx                           25.0.13         1.31.2          NGINX Open Source is a web server that can be a...
bitnami/nginx                           25.0.12         1.31.2          NGINX Open Source is a web server that can be a...
bitnami/nginx                           25.0.11         1.31.2          NGINX Open Source is a web server that can be a...
bitnami/nginx                           25.0.10         1.31.2          NGINX Open Source is a web server that can be a...
bitnami/nginx                           25.0.9          1.31.2          NGINX Open Source is a web server that can be a...
bitnami/nginx                           25.0.8          1.31.2          NGINX Open Source is a web server that can be a...


Módulo 2 – Conhecendo um Chart

Sem instalar nada.

2.1 Mostrar informações
helm show chart bitnami/nginx

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

2.2 Mostrar README
helm show readme bitnami/nginx

2.3 mostrar os valores 

helm show values bitnami/nginx

2.4 Mostrar tudo
helm show all bitnami/nginx

Módulo 3 – Baixando o Chart
Baixar

