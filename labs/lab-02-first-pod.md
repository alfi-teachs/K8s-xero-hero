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
