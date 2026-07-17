# Kubernetes kubeconfig Overview

## What is kubeconfig?

The **kubeconfig** file is the configuration file used by **kubectl** and other Kubernetes tools (such as Helm, Kustomize, and Terraform) to connect to a Kubernetes cluster.

Its primary purpose is to answer three questions:

1. Which cluster should I connect to?
2. Who am I?
3. Which context should I use?

The default location is:

```text
~/.kube/config
or
Run:
kubectl config view
kubectl config get-contexts
```

---

# Main Components

A kubeconfig file is composed of four main sections:

```text
Config
│
├── clusters
├── users
├── contexts
└── current-context
```

Each section has a specific responsibility.

## 1. Cluster

The **cluster** section contains the connection information for the Kubernetes API Server.

Example:

```yaml
clusters:
- name: minikube
  cluster:
    server: https://127.0.0.1:32771
    certificate-authority: ~/.minikube/ca.crt
```

### server

The API Server endpoint that `kubectl` connects to.

```text
https://127.0.0.1:32771
```

### certificate-authority

The Certificate Authority (CA) used to verify the identity of the API Server during the TLS handshake.

This prevents clients from connecting to an untrusted or malicious server.

---

## 2. User

The **user** section defines the credentials used to authenticate to the cluster.

Example:

```yaml
users:
- name: minikube
  user:
    client-certificate: ~/.minikube/profiles/minikube/client.crt
    client-key: ~/.minikube/profiles/minikube/client.key
```

### client-certificate

The client's X.509 certificate.

### client-key

The private key associated with the client certificate.

Together, these files prove the client's identity to the Kubernetes API Server.

> **Important:** Never share your private key (`client.key`).

---

## 3. Context

A **context** combines:

- Cluster
- User
- Namespace

Example:

```yaml
contexts:
- name: minikube
  context:
    cluster: minikube
    user: minikube
    namespace: default
```

Instead of specifying these values for every command, `kubectl` simply uses the active context.

---

## 4. Current Context

The active context is defined by:

```yaml
current-context: minikube
```

You can check it with:

```bash
kubectl config current-context
```

Switch contexts using:

```bash
kubectl config use-context <context-name>
```

---

# Authentication Flow

When you run a command such as:

```bash
kubectl get pods
```

the following process occurs:

```text
kubectl
    │
    ▼
Reads ~/.kube/config
    │
    ├── API Server
    ├── CA Certificate
    ├── Client Certificate
    ├── Client Private Key
    └── Current Context
    │
    ▼
Establishes an HTTPS connection
    │
    ▼
TLS Handshake
    │
    ▼
API Server validates:
    • Server certificate
    • Client certificate
    │
    ▼
User authenticated
    │
    ▼
RBAC Authorization
    │
    ▼
Response returned
```

---

# Authentication vs Authorization

The kubeconfig file **does not define permissions**.

Its only responsibility is to authenticate the client.

Permissions are determined later by **RBAC (Role-Based Access Control)**.

```text
kubeconfig
      │
      ▼
Authentication
      │
      ▼
RBAC Authorization
      │
      ▼
Request Allowed or Denied
```

A user can be successfully authenticated but still receive:

```text
Error from server (Forbidden)
```

This means the authentication succeeded, but RBAC denied the requested operation.

---

# File-Based References

In Minikube, the kubeconfig references certificate files stored on disk:

```yaml
certificate-authority: ~/.minikube/ca.crt
client-certificate: ~/.minikube/profiles/minikube/client.crt
client-key: ~/.minikube/profiles/minikube/client.key
```

In other Kubernetes environments, these certificates may be embedded directly in the kubeconfig using Base64-encoded fields such as:

- `certificate-authority-data`
- `client-certificate-data`
- `client-key-data`

Both approaches are fully supported by Kubernetes.

---

# Summary

The **kubeconfig** file is the client configuration used to access a Kubernetes cluster. It defines:

- Which cluster to connect to
- How to authenticate
- Which context to use

It does **not** grant permissions. After the client is authenticated, Kubernetes uses **RBAC** to determine whether the requested operation is allowed.

# Understanding RBAC in Minikube

In the previous section, we learned that the **kubeconfig** file authenticates the client using an X.509 certificate.

The next step is **authorization**, which is performed by **RBAC (Role-Based Access Control)**.

The following examples use a Minikube cluster to demonstrate how this process works.

---

# Step 1 - Verify the Current User Permissions

The following command checks whether the current authenticated user can perform any action on any resource across all namespaces.

```bash
kubectl auth can-i '*' '*' --all-namespaces
```

Output:

```text
yes
```

This confirms that the current user has full administrative privileges.

However, **why** does Kubernetes allow every operation?

To answer that question, we need to inspect the client certificate.

---

# Step 2 - Identify the Authenticated User

The Minikube kubeconfig uses X.509 certificates for client authentication.

Display the certificate subject:

```bash
openssl x509 \
-in ~/.minikube/profiles/minikube/client.crt \
-noout -subject
```

Output:

```text
subject=O = system:masters, CN = minikube-user
```

The certificate contains two important attributes.

## Common Name (CN)

```text
CN = minikube-user
```

The **Common Name (CN)** identifies the authenticated user.

During authentication, the Kubernetes API Server recognizes the request as coming from:

```text
User: minikube-user
```

---

## Organization (O)

```text
O = system:masters
```

The **Organization (O)** field is interpreted by Kubernetes as a **Group**.

Therefore, the authenticated identity becomes:

```text
User:   minikube-user
Group:  system:masters
```

Authentication is now complete.

At this point, Kubernetes knows **who** is making the request, but it has **not yet decided** whether the request should be allowed.

---

# Step 3 - RBAC Authorization

Once authentication succeeds, the API Server evaluates the RBAC configuration.

The authorization process looks like this:

```text
Authenticated User
        │
        ▼
User: minikube-user
Group: system:masters
        │
        ▼
Search for RoleBindings
        │
        ▼
Search for ClusterRoleBindings
        │
        ▼
Apply matching Roles or ClusterRoles
        │
        ▼
Allow or Deny the request
```

---

# Step 4 - Inspect the ClusterRoleBinding

List the available ClusterRoleBindings:

```bash
kubectl get clusterrolebinding | grep -i admin
```

Output:

```text
cluster-admin
kubeadm:cluster-admins
kubernetes-dashboard
minikube-rbac
```

Now inspect the default `cluster-admin` binding:

```bash
kubectl describe clusterrolebinding cluster-admin
```

Output:

```text
Role:
  Kind: ClusterRole
  Name: cluster-admin

Subjects:
  Kind    Name
  ----    ----
  Group   system:masters
```

This is the critical relationship.

It tells Kubernetes:

> Every authenticated user that belongs to the **system:masters** group is granted the **cluster-admin** ClusterRole.

Graphically:

```text
minikube-user
        │
        ▼
Group
system:masters
        │
        ▼
ClusterRoleBinding
cluster-admin
        │
        ▼
ClusterRole
cluster-admin
        │
        ▼
Full Cluster Access
```

---

# Step 5 - Inspect the ClusterRole

Now inspect the permissions granted by the `cluster-admin` ClusterRole.

```bash
kubectl describe clusterrole cluster-admin
```

Output:

```text
PolicyRule:

Resources          Verbs
---------          -----
*.*                [*]
[*]                [*]
```

This rule means:

- All API Groups (`*`)
- All Resources (`*`)
- All Verbs (`*`)

In other words, this role grants unrestricted access to the Kubernetes cluster.

Equivalent representation:

```text
API Groups : *
Resources  : *
Verbs       : *
```

Which means:

- create
- get
- list
- watch
- update
- patch
- delete

on every Kubernetes resource.

---

# Why does `kubectl auth can-i` return "yes"?

The complete authorization flow is:

```text
kubectl
    │
    ▼
Client Certificate
    │
    ▼
Authentication
    │
    ├── User: minikube-user
    └── Group: system:masters
            │
            ▼
ClusterRoleBinding
            │
            ▼
cluster-admin
            │
            ▼
ClusterRole
            │
            ▼
All Resources
All API Groups
All Verbs
            │
            ▼
Authorization: ALLOWED
```

Therefore, the command:

```bash
kubectl auth can-i '*' '*' --all-namespaces
```

returns:

```text
yes
```

because the authenticated user belongs to the **system:masters** group, which is bound to the **cluster-admin** ClusterRole.

---

# Key Takeaways

- **Authentication** identifies the user using the X.509 client certificate.
- The certificate **CN (Common Name)** becomes the Kubernetes username.
- The certificate **Organization (O)** becomes the Kubernetes group.
- RBAC evaluates **RoleBindings** and **ClusterRoleBindings** associated with that user or group.
- In Minikube, the **system:masters** group is bound to the **cluster-admin** ClusterRole.
- The **cluster-admin** role grants unrestricted access to every resource in the cluster.
- Authentication answers **"Who are you?"**
- RBAC answers **"What are you allowed to do?"**

- # Role vs ClusterRole

Both **Role** and **ClusterRole** define a set of permissions in Kubernetes. The key difference is their **scope**.

## Role

A **Role** is **namespace-scoped**.

It grants permissions only within a single namespace.

Example:

```yaml
kind: Role
metadata:
  name: pod-reader
  namespace: development
```

A user or ServiceAccount bound to this Role can access resources **only** in the `development` namespace.

---

## ClusterRole

A **ClusterRole** is **cluster-scoped**.

It can grant permissions:

- Across all namespaces
- To cluster-wide resources (such as Nodes, Namespaces, PersistentVolumes, and StorageClasses)

Example:

```yaml
kind: ClusterRole
metadata:
  name: pod-reader
```

Unlike a Role, a ClusterRole is not associated with any namespace.

---

## Comparison

| Feature | Role | ClusterRole |
|---------|------|-------------|
| Scope | Single namespace | Entire cluster |
| Namespace-scoped resources | ✅ | ✅ |
| Cluster-scoped resources (Nodes, PVs, Namespaces, etc.) | ❌ | ✅ |
| Can be bound with RoleBinding | ✅ | ✅ |
| Can be bound with ClusterRoleBinding | ❌ | ✅ |

---

## When to Use

Use a **Role** when permissions should apply only within a specific namespace.

Use a **ClusterRole** when permissions should:

- Apply across multiple namespaces
- Access cluster-wide resources
- Be reused across different namespaces

---

## Key Takeaway

- **Role** = Namespace-level permissions
- **ClusterRole** = Cluster-level permissions

A **ClusterRole** can also be bound to a single namespace using a **RoleBinding**, making it reusable without granting cluster-wide access.


# Lab 02 - Creating a Role

## Objective

In this lab, we will create a **ServiceAccount** and a **Role**, then demonstrate that **creating a Role alone does not grant permissions**.

A Role only defines **what actions are allowed**. Until it is associated with a user or ServiceAccount through a **RoleBinding**, the permissions are never applied.

---

# Step 1 - Create a ServiceAccount

Create a ServiceAccount named `developer`.

```bash
kubectl create serviceaccount developer
```

Expected output:

```text
serviceaccount/developer created
```

Verify that it exists.

```bash
kubectl get serviceaccounts
```

Example:

```text
NAME        SECRETS   AGE
default     0         10d
developer   0         5s
```

Describe the ServiceAccount.

```bash
kubectl describe serviceaccount developer
```

Example:

```text
Name:                developer
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Events:              <none>
```

At this point, the ServiceAccount exists but has **no permissions**.

Verify this using:

```bash
kubectl auth can-i list pods \
--as system:serviceaccount:default:developer
```

Expected output:

```text
no
```

---

# Step 2 - Verify Existing Roles

Before creating a Role, verify that no Roles exist in the current namespace.

```bash
kubectl get roles
```

Expected output:

```text
No resources found in default namespace.
```

---

# Step 3 - Create a Role

Create the following file.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default

rules:
- apiGroups: [""]
  resources:
  - pods
  verbs:
  - get
  - list
```

Apply the Role.

```bash
kubectl apply -f role-pod-reader.yaml
```

Expected output:

```text
role.rbac.authorization.k8s.io/pod-reader created
```

---

# Step 4 - Verify the Role

```bash
kubectl get roles
```

Example:

```text
NAME         CREATED AT
pod-reader   2026-07-17T20:03:51Z
```

Describe the Role.

```bash
kubectl describe role pod-reader
```

Example:

```text
PolicyRule:
  Resources  Verbs
  ---------  -----
  pods       [get list]
```

The Role now defines the following permissions:

- get Pods
- list Pods

within the `default` namespace.

---

# Step 5 - Test the ServiceAccount Again

Test the ServiceAccount one more time.

```bash
kubectl auth can-i list pods \
--as system:serviceaccount:default:developer
```

Expected output:

```text
no
```

Although the Role exists, the ServiceAccount still cannot list Pods.

---

# Why?

The authorization process is:

```text
ServiceAccount
        │
        ▼
Authentication
        │
        ▼
Authenticated
        │
        ▼
Role exists?
        │
       YES
        │
        ▼
RoleBinding exists?
        │
       NO
        │
        ▼
Authorization
        │
        ▼
Forbidden
```

The `pod-reader` Role defines permissions, but they are not assigned to any identity.

Without a **RoleBinding**, Kubernetes has no way to associate the `developer` ServiceAccount with the `pod-reader` Role.

---

# Key Takeaways

This lab demonstrated that:

- A **ServiceAccount** provides an identity.
- A **Role** defines a set of permissions.
- Creating a Role does **not** automatically grant permissions.
- A **RoleBinding** is required to associate a Role with a user, group, or ServiceAccount.
- Without a RoleBinding, every authorization request is denied.

In the next lab, we will create a **RoleBinding** and observe how the ServiceAccount immediately gains the permissions defined by the Role.


# Lab 03 - Creating a RoleBinding

## Objective

In this lab, we will create a **RoleBinding** to associate the `developer` ServiceAccount with the `pod-reader` Role.

This demonstrates the final step of the Kubernetes RBAC authorization model.

---

# Current RBAC State

At this point:

- The `developer` ServiceAccount exists.
- The `pod-reader` Role exists.
- No RoleBinding has been created.

The authorization flow is currently:

```text
ServiceAccount (developer)
            │
            ▼
Authentication
            │
            ▼
Authenticated
            │
            ▼
RoleBinding
            │
            ▼
Not Found
            │
            ▼
Authorization
            │
            ▼
Forbidden
```

Verify the current permissions.

```bash
kubectl auth can-i list pods \
--as system:serviceaccount:default:developer
```

Expected output:

```text
no
```

---

# Step 1 - Create a RoleBinding

Create the following file.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-pod-reader
  namespace: default

subjects:
- kind: ServiceAccount
  name: developer
  namespace: default

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
```

Apply the manifest.

```bash
kubectl apply -f rolebinding-pod-reader.yaml
```

Expected output:

```text
rolebinding.rbac.authorization.k8s.io/developer-pod-reader created
```

---

# Step 2 - Verify the RoleBinding

List the RoleBindings.

```bash
kubectl get rolebindings
```

Example:

```text
NAME                   ROLE                AGE
developer-pod-reader   Role/pod-reader     5s
```

Describe the RoleBinding.

```bash
kubectl describe rolebinding developer-pod-reader
```

Example:

```text
Name:         developer-pod-reader
Namespace:    default

Role:
  Kind:  Role
  Name:  pod-reader

Subjects:
  Kind            Name
  ----            ----
  ServiceAccount  developer
```

Notice that the RoleBinding connects two objects:

- ServiceAccount: `developer`
- Role: `pod-reader`

---

# Step 3 - Test RBAC Again

Run the same authorization test.

```bash
kubectl auth can-i list pods \
--as system:serviceaccount:default:developer
```

Expected output:

```text
yes
```

The ServiceAccount can now list Pods.

Test another permitted operation.

```bash
kubectl auth can-i get pods \
--as system:serviceaccount:default:developer
```

Expected output:

```text
yes
```

Now test an operation that was **not** granted.

```bash
kubectl auth can-i delete pods \
--as system:serviceaccount:default:developer
```

Expected output:

```text
no
```

Likewise:

```bash
kubectl auth can-i create deployments \
--as system:serviceaccount:default:developer
```

Expected output:

```text
no
```

---

# Why?

The authorization flow has now changed.

```text
ServiceAccount
(developer)
        │
        ▼
Authentication
        │
        ▼
Authenticated
        │
        ▼
RoleBinding Found
        │
        ▼
Role
(pod-reader)
        │
        ▼
Rules

Pods
get
list
        │
        ▼
Authorization
        │
        ├── list pods   → ALLOWED
        ├── get pods    → ALLOWED
        ├── delete pods → DENIED
        └── create deployments → DENIED
```

---

# Visualizing the Relationship

The complete RBAC relationship is now:

```text
                  developer
              ServiceAccount
                     │
                     ▼
          developer-pod-reader
             RoleBinding
                     │
                     ▼
               pod-reader
                  Role
                     │
                     ▼
        get pods
        list pods
```

The RoleBinding is the object that assigns the Role's permissions to the ServiceAccount.

---

# Key Takeaways

This lab demonstrated that:

- A Role defines permissions.
- A ServiceAccount provides an identity.
- A RoleBinding associates the identity with the permissions.
- Permissions are granted immediately after the RoleBinding is created.
- RBAC follows the principle of least privilege: only the explicitly granted operations are allowed.

# Authorization Modes in Minikube

The Kubernetes API Server supports multiple authorization modules.

The enabled modules can be verified by inspecting the API Server manifest.

```bash
minikube ssh -n minikube

sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Relevant configuration:

```text
--authorization-mode=Node,RBAC
```

This means the API Server evaluates every request using two authorization modules:

1. **Node Authorizer**
2. **RBAC Authorizer**

---

# Authorization Flow

```text
             Authentication
                    │
                    ▼
          User / Groups Identified
                    │
                    ▼
          Authorization Pipeline
                    │
      ┌─────────────┴─────────────┐
      ▼                           ▼
Node Authorizer            RBAC Authorizer
      │                           │
      └─────────────┬─────────────┘
                    ▼
          Allow or Deny Request
```

Each authorization module is evaluated in the configured order.

---

# Node Authorizer

The **Node Authorizer** is designed specifically for Kubernetes worker nodes.

Its purpose is to ensure that a kubelet can only access resources related to the node it manages.

For example, a kubelet is allowed to:

- Read Pods scheduled on its node
- Update its own Node object
- Read Secrets and ConfigMaps required by its Pods

The Node Authorizer does **not** evaluate requests from regular users or ServiceAccounts.

---

# RBAC Authorizer

The **RBAC Authorizer** is responsible for authorizing:

- Users
- Groups
- ServiceAccounts

It evaluates Kubernetes RBAC objects such as:

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

For example, when executing:

```bash
kubectl auth can-i list pods \
--as system:serviceaccount:default:developer
```

the request is evaluated by the RBAC Authorizer.

The authorizer searches for matching RoleBindings and ClusterRoleBindings to determine whether the requested operation is allowed.

---

# Why Both Modules?

Different identities require different authorization mechanisms.

| Identity | Authorization Module |
|----------|----------------------|
| kubelet | Node Authorizer |
| User | RBAC |
| Group | RBAC |
| ServiceAccount | RBAC |

This separation improves security by ensuring that kubelets receive only the permissions required to manage their own workloads, while users and applications are governed by RBAC policies.

---

# Key Takeaway

The Minikube cluster is configured with:

```text
--authorization-mode=Node,RBAC
```

This means:

- **Node Authorizer** protects communications between kubelets and the API Server.
- **RBAC Authorizer** controls access for users, groups, and ServiceAccounts.
- **User** = identity for people and external systems.
- **ServiceAccount** = identity for applications running inside Kubernetes.
- Humans usually authenticate through external identity providers.
- Applications should use ServiceAccounts with the minimum RBAC permissions required.

The Kubernetes security model follows the principle of:

> **Least Privilege: grant only the permissions required for each identity.**
> Throughout this RBAC lab, every authorization decision involving the `developer` ServiceAccount is handled by the **RBAC Authorizer**.
