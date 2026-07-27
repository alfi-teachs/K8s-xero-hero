# LAB 23 - Kubernetes Namespace

## Objective

In this lab, you will learn:

- What is a Kubernetes Namespace
- Why namespaces are used
- Create namespaces
- Deploy applications inside namespaces
- Switch between namespaces
- Manage resources by namespace

---

# What is a Namespace?

A Namespace is a logical separation inside a Kubernetes cluster.

It allows us to divide one cluster into multiple isolated environments.

Example:

One Kubernetes Cluster:

```
Kubernetes Cluster

        |

--------------------------------

Development Namespace

Frontend
Backend


--------------------------------

Testing Namespace

Frontend
Backend


--------------------------------

Production Namespace

Frontend
Backend

```

---

# Why Do We Need Namespaces?

Imagine a company has:

```
Development Team

Testing Team

Production Team
```

All teams use the same Kubernetes cluster.

Without namespaces:

```
Everyone sees everything
```

With namespaces:

```
dev namespace

test namespace

production namespace
```

Resources are separated.

---

# Default Kubernetes Namespaces

Check:

```bash
kubectl get namespaces
```

Default namespaces:

```
default

kube-system

kube-public

kube-node-lease
```

---

# Namespace Types

## default

Used when no namespace is specified.

Example:

```bash
kubectl get pods
```

means:

```bash
kubectl get pods -n default
```

---

## kube-system

Contains Kubernetes internal components.

Example:

```
coredns

scheduler

controller-manager
```

---

## kube-public

Publicly accessible cluster information.

---

# Namespace Flow

```
User

 |

kubectl command

 |

Namespace

 |

Resources

 |

Pods
Services
Deployments

```

---

# Lab Steps

# Step 1 - Check Existing Namespaces

Run:

```bash
kubectl get namespaces
```

---

# Step 2 - Create Namespace Using Command

Create:

```bash
kubectl create namespace dev
```

Expected:

```
namespace/dev created
```

---

# Step 3 - Verify Namespace

Run:

```bash
kubectl get namespaces
```

Output:

```
dev
```

---

# Step 4 - Create Namespace Using YAML

Create:

```bash
touch yaml/lab-23/namespace.yaml
```

Add:

```yaml
apiVersion: v1

kind: Namespace


metadata:

  name: production
```

---

Apply:

```bash
kubectl apply -f yaml/lab-23/namespace.yaml
```

---

Verify:

```bash
kubectl get namespaces
```

---

# Step 5 - Create Pod Inside Namespace

Create:

```bash
touch yaml/lab-23/nginx-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: nginx-production

  namespace: production


spec:

  containers:

  - name: nginx

    image: nginx

    ports:

    - containerPort: 80
```

---

Create:

```bash
kubectl apply -f yaml/lab-23/nginx-pod.yaml
```

---

# Step 6 - View Pods in Namespace

Without namespace:

```bash
kubectl get pods
```

Output:

```
No resources found
```

Because Pod is inside production namespace.

---

With namespace:

```bash
kubectl get pods -n production
```

Output:

```
NAME                 READY
nginx-production     1/1
```

---

# Step 7 - View All Namespaces

Run:

```bash
kubectl get pods --all-namespaces
```

or:

```bash
kubectl get pods -A
```

---

# Step 8 - Set Default Namespace Context

Currently:

```bash
kubectl get pods
```

uses:

```
default namespace
```

Change context:

```bash
kubectl config set-context --current --namespace=production
```

Now:

```bash
kubectl get pods
```

will show:

```
nginx-production
```

---

# Step 9 - Check Current Namespace

Run:

```bash
kubectl config view --minify | grep namespace
```

---

# Step 10 - Delete Namespace

Delete:

```bash
kubectl delete namespace dev
```

Delete production:

```bash
kubectl delete namespace production
```

All resources inside namespace are deleted.

---

# Namespace Resource Isolation

Namespaces isolate:

- Pods
- Services
- Deployments
- ConfigMaps
- Secrets

Example:

```
production

app-service


development

app-service
```

Both can have the same name because they are separated.

---

# Verification

Check namespaces:

```bash
kubectl get namespaces
```

Check resources:

```bash
kubectl get all -n production
```

Check all resources:

```bash
kubectl get all -A
```

---

# Troubleshooting

## Pod Not Found

Problem:

```bash
kubectl get pods
```

Solution:

Check namespace:

```bash
kubectl get pods -A
```

---

## Wrong Namespace

Check:

```bash
kubectl config view --minify
```

---

# Interview Questions

## 1. What is a Namespace?

A Namespace provides logical isolation of Kubernetes resources inside a cluster.

---

## 2. Why use namespaces?

For:

- Environment separation
- Team separation
- Resource management
- Security

---

## 3. Can two namespaces have the same Pod name?

Yes.

Example:

```
dev/nginx

production/nginx
```

---

## 4. What happens when a Namespace is deleted?

All resources inside that namespace are deleted.

---

## 5. Difference between Namespace and Cluster?

Cluster:

Complete Kubernetes environment.

Namespace:

Logical partition inside a cluster.

---

# Commands Learned

```bash
kubectl get namespaces

kubectl create namespace

kubectl apply -f

kubectl get pods -n

kubectl get all -A

kubectl config set-context
```

---

# Cleanup

Delete resources:

```bash
kubectl delete namespace production
```

Reset namespace:

```bash
kubectl config set-context --current --namespace=default
```

---

# Navigation

Previous:

[Lab 22 - CPU and Memory Limits](lab-22-cpu-memory-limits.md)

Next:

[Lab 24 - Metrics Server](lab-24-metrics-server.md)

---

# Lab Completed ✅

You learned:

- Kubernetes Namespaces
- Resource separation
- Namespace-based management
- Multi-environment organization