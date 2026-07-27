# LAB 02 - First Kubernetes Pod

## Objective

In this lab, you will learn:

- What is a Kubernetes Pod
- Create a Pod using YAML
- Understand Kubernetes YAML structure
- Use imperative and declarative approaches
- Apply Kubernetes configuration
- Inspect Pod details
- View Pod logs
- Access container shell
- Delete a Pod

---

# What You Will Learn

By the end of this lab, you will understand:

- Pod lifecycle
- Kubernetes YAML structure
- apiVersion
- kind
- metadata
- spec
- containers
- kubectl apply
- kubectl describe
- kubectl logs
- kubectl exec

---

# What is a Kubernetes Pod?

A Pod is the smallest deployable unit in Kubernetes.

A Pod contains one or more containers.

Example:

A Pod provides:

- Network identity
- Storage sharing
- Container management

---

# Pod vs Container

Container:

- Runs application
- Example: nginx container

Pod:

- Manages one or more containers
- Provides networking and storage

Example:

```bash
Pod
|
+-- nginx container
|
+-- sidecar container
```

---

# Two Ways to Create Kubernetes Resources

Kubernetes supports:

## 1. Imperative Approach

We directly tell Kubernetes what to create.

Example:

```bash
kubectl run nginx-pod --image=nginx
```
Kubernetes immediately creates the Pod.
---------------------
# 2. Declarative Approach

We describe the desired state in a YAML file.

Example:

kind: Pod
metadata:
  name: nginx-pod

Then apply:

kubectl apply -f pod.yaml

Kubernetes creates the required resource.

Prerequisites

Required:
```bash
Docker Desktop
Minikube
kubectl
Git
```
Check cluster:
```bash
kubectl get nodes
```
Expected:

NAME       STATUS   ROLES
minikube   Ready    control-plane
