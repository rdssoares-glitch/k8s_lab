# 1- Kubernetes Lab - Introduction to Custom Resource Definitions (CRDs)

This lab demonstrates how to extend the Kubernetes API by creating your own custom resources.
Throughout the exercises, you will create a new Kubernetes resource called Database and understand how Operators use Custom Resource Definitions (CRDs).

1- Check the cluster status:

```bash
kubectl cluster-info
````
```bash
kubectl get nodes
```
output:
```text
rsoar001@C-PF46ZS03:~$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:32771
CoreDNS is running at https://127.0.0.1:32771/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

rsoar001@C-PF46ZS03:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
minikube       Ready    control-plane   7d20h   v1.34.0
minikube-m02   Ready    <none>          7d20h   v1.34.0
minikube-m03   Ready    <none>          7d20h   v1.34.0
```
So far in this series, you've worked with several Kubernetes resources.
For example:
```text
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get configmaps
kubectl get pvc
kubectl get pv
```
Each command retrieves a different type of resource from the Kubernetes API.
But have you ever wondered...
Can I create my own Kubernetes resource?
The answer is:
Yes!
And that's exactly what you'll learn in this lab.

## Exercise 1 - Exploring the Kubernetes API

Let's begin by discovering which resource types already exist.

Run:
```bash
kubectl api-resources
```
The output will be long.
You'll see entries similar to:

```text
NAME                     SHORTNAMES
pods                     po
services                 svc
deployments              deploy
replicasets              rs
daemonsets               ds
jobs
cronjobs
configmaps               cm
persistentvolumes        pv
persistentvolumeclaims   pvc
...
```
These are the resource types currently known by the Kubernetes API.

Challenge

Try to answer the following questions.

Question 1

Can you find a resource named:

Database

website

Application

The answer to all of them should be:

No.

These resources do not exist in a default Kubernetes installation.

Did You Know?

When you create a Pod, Kubernetes already knows what a Pod is.

When you create a Deployment, Kubernetes already knows what a Deployment is.

But for a resource like:

Database

The Kubernetes API has no idea what a Database resource is, someone must first teach Kubernetes that this new resource exists.

That's exactly what a Custom Resource Definition (CRD) does.

# 2 - Extending the Kubernetes API
So how can we teach Kubernetes about a completely new resource type?

The answer is by creating a Custom Resource Definition (CRD).

A CRD extends the Kubernetes API by registering a new resource type. Once installed, the new resource behaves like any native Kubernetes resource and can be managed using **kubectl**.

## Exercise 2.1 - Looking for Existing CRDs

Before creating our own CRD, let's see whether the cluster already contains any.

Run:
```bash
kubectl get crd
```
output:

On a fresh Minikube installation, you may see:
```text
rsoar001@C-PF46ZS03:~$ kubectl get crd
No resources found
```
O Minikube come with a oficial addon named OLM (Operator Lifecycle Manager). It is the kubernetes default tool to install, manager and list operators.
To check if OLM addon is enable or not, run:
```bash
minikube addons list | grep olm
```
output:
```text
rsoar001@C-PF46ZS03:~$ minikube addons list | grep olm
│ olm                         │ minikube │ disabled │ 3rd party (Operator Framework)         │
```

Since it is disabled, we need to enable it on minikube first:
Run:

```bash
minikube addons enable olm
```
output 
```text
rsoar001@C-PF46ZS03:~$ minikube addons enable olm
❗  olm is a 3rd party addon and is not maintained or verified by minikube maintainers, enable at your own risk.
❗  olm does not currently have an associated maintainer.
❗  The OLM addon has stopped working, for more details visit: https://github.com/operator-framework/operator-lifecycle-manager/issues/2534
    ▪ Using image quay.io/operator-framework/olm
    ▪ Using image quay.io/operatorhubio/catalog
🌟  The 'olm' addon is enabled
```
Now if we run **kubectl get crd** again:

output:
```text
rsoar001@C-PF46ZS03:~$ kubectl get crd
NAME                                          CREATED AT
catalogsources.operators.coreos.com           2026-07-15T15:35:35Z
clusterserviceversions.operators.coreos.com   2026-07-15T15:35:35Z
installplans.operators.coreos.com             2026-07-15T15:35:35Z
operatorconditions.operators.coreos.com       2026-07-15T15:35:35Z
operatorgroups.operators.coreos.com           2026-07-15T15:35:35Z
operators.operators.coreos.com                2026-07-15T15:35:35Z
subscriptions.operators.coreos.com            2026-07-15T15:35:35Z
```
The addon olm run on a specific namespace olm,

Run:
 ```bash
kubectl get pods -n olm
```
Output:
```text
rsoar001@C-PF46ZS03:~$ kubectl get pods -n olm
NAME                               READY   STATUS    RESTARTS   AGE
catalog-operator-dccb7b8b4-ffk9d   1/1     Running   0          44m
olm-operator-777f7658b5-mhsqr      1/1     Running   0          44m
operatorhubio-catalog-57vgb        1/1     Running   0          11m
packageserver-696c67f8ff-fgqxz     1/1     Running   0          44m
packageserver-696c67f8ff-ww72j     1/1     Running   0          44m
rsoar001@C-PF46ZS03:~$
```

## Exercise 2.2 - Creating Our First CRD

Now it's time to teach Kubernetes about a brand-new resource named Database.

Create a file named:

database-crd.yaml

with the following content:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: databases.labs.example.com

spec:
  group: labs.example.com

  scope: Namespaced

  names:
    plural: databases
    singular: database
    kind: Database
    shortNames:
      - db

  versions:
    - name: v1
      served: true
      storage: true

      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                version:
                  type: string
                storage:
                  type: string
```

For now, simply apply it:
```bash
kubectl apply -f database-crd.yaml
```
output:

```tet
customresourcedefinition.apiextensions.k8s.io/databases.labs.example.com created
```

## Exercise 2.3 - Did the API Change?

Let's ask Kubernetes again which resources it knows.

Run:
```bash
kubectl api-resources | grep database
```
Expected output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/06-CRD$ kubectl api-resources | grep database
databases                           db           labs.example.com/v1                true         Database
```

run:
```bash
kubectl get crd | grep database
```
output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/06-CRD$ kubectl get crd | grep database
databases.labs.example.com                    2026-07-15T17:51:45Z
```

Or inspect the CRD in detail:
run:
```bash
kubectl describe crd databases.labs.example.com
```
These define how Kubernetes understands your new resource.

The Kubernetes API now recognizes a resource named Database:

```bash
kubectl get database
kubectl get db
```
Output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/06-CRD$ kubectl get database
No resources found in default namespace.

rsoar001@C-PF46ZS03:~/k8s_lab/06-CRD$ kubectl get db
No resources found in default namespace.
```
No actual databases have been created yet.

We created a new resource type, not a new application.

## Exercise 2.4: Creating a customr Resource

Create a file named:

postgres.yaml

with the following content:
```yaml
apiVersion: labs.example.com/v1
kind: Database

metadata:
  name: postgres-prod

spec:
  engine: postgres
  version: "17"
  storage: 20Gi
```

Apply the resource:

```bash
kubectl apply -f postgres.yaml
```
Output:
```text
database.labs.example.com/postgres-prod created
```

Listing Database Resources

Now list all Database objects.
```bash
kubectl get databases
```
Expected output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/06-CRD$ kubectl get databases
NAME            AGE
postgres-prod   63s
```

The short name also works.
```bash
kubectl get db
```
Expected output:
```text
rsoar001@C-PF46ZS03:~/k8s_lab/06-CRD$ kubectl get db
NAME            AGE
postgres-prod   77s
```

Now Kubernetes understands that word and knows how to store objects of that type. However...

It still doesn't know what to do with a Database.
f Kubernetes already knows what a Database is, who is supposed to create the PostgreSQL Pod?

# Chapter 3 - Introducing Controllers
A Controller continuously watches the Kubernetes API looking for specific resources.

For example:

Database

Whenever a new Database appears, the Controller receives an event.
```text
New Database created!

↓

Let's create a Deployment.

↓

Let's create a Service.

↓

Let's create a PVC.

↓

Done!
```
Our lab currently has:

Database

But there is no Controller watching it.

Therefore:
```text
Database

        │

        ▼

Nothing
```
The resource simply sits in the Kubernetes API.

How Operators Work

Now imagine adding a Controller.
```text
                Kubernetes API

                        │

                        ▼

            Database Custom Resource

                        │

                        ▼

             Database Controller

             ├──────────────┐
             │              │
             ▼              ▼

        Deployment      Service

             │

             ▼

        PostgreSQL Pod
```

Suddenly, the Database object has meaning.

The Controller transforms your desired state into real Kubernetes resources.

The CRD defines what users can create.

The Controller defines what should happen when those resources exist.

Together, they form an Operator.

# chapter 4 - What Is an Operator?
An Operator is simply a Kubernetes application that continuously watches one or more Custom Resources.

Whenever a Custom Resource is created, modified, or deleted, the Operator reacts and performs the required actions.
The Operator acts like an automation engine.

It compares:

**Desired State**

with

**Current State**

and continuously works to make both match.

The desired state is:

I want three PostgreSQL replicas.

Imagine only two Pods are currently running.

Desired State : 3 Pods

Current State : 2 Pods

The Operator detects the difference and creates one additional Pod.

After reconciliation:

Desired State : 3 Pods

Current State : 3 Pods

This continuous comparison is known as the reconciliation loop.
The Reconciliation Loop runs continuously for as long as the Operator is running.
