## 📚 Course Roadmap

### Level 1 - Kubernetes Basics

| Lab | Topic | Status |
|------|-------|--------|
| 01 | # LAB 03 - Multi Container Pod

## Objective

In this lab, you will learn:

- What is a Multi Container Pod
- Why multiple containers run inside one Pod
- Shared network between containers
- Shared storage between containers
- Create a Pod with nginx and busybox containers
- Access containers inside a Pod
- Test communication between containers

---

# What You Will Learn

By the end of this lab, you will understand:

- Pod design pattern
- Sidecar containers
- Container communication
- Shared localhost networking
- Shared volumes

---

# What is a Multi Container Pod?

A Pod can contain one or more containers.

Normally:

```
Pod
 |
 +---- Application Container
```

But Kubernetes also supports:

```
Pod
 |
 +----------------+
 |                |
nginx          busybox
Container      Container
```

All containers inside the same Pod:

- Share the same network
- Share the same IP address
- Can communicate using localhost
- Can share storage volumes

---

# Why Use Multiple Containers?

A common pattern is the **Sidecar Pattern**.

Example:

Main Application:

```
Web Application
```

Sidecar Container:

```
Log Collector
```

The sidecar helps the main application.

Examples:

- Log collection
- Monitoring agents
- Configuration reloaders
- Security scanners

---

# Container Communication

Containers inside the same Pod communicate using:

```
localhost
```

Example:

```
Pod IP: 10.244.0.5


nginx container
      |
      |
 localhost:80
      |
      |
busybox container
```

They do not need a Kubernetes Service to communicate.

---

# Prerequisites

Required:

- Kubernetes running
- Minikube started
- kubectl installed

Check cluster:

```bash
kubectl get nodes
```

Expected:

```
NAME       STATUS   ROLES
minikube   Ready    control-plane
```

---

# Lab Steps

## Step 1 - Create YAML Folder

Go to project directory:

```bash
cd yaml
```

Create lab folder:

```bash
mkdir lab-03
```

Move inside:

```bash
cd lab-03
```

Create YAML file:

```bash
touch multi-container-pod.yaml
```

Open:

```bash
code multi-container-pod.yaml
```

---

# Step 2 - Create Multi Container Pod YAML

Add:

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: multi-container-pod
  labels:
    app: multi-container

spec:

  containers:

  - name: nginx-container
    image: nginx:latest
    ports:
      - containerPort: 80


  - name: busybox-container
    image: busybox
    command:
      - sleep
      - "3600"
```

Save the file.

---

# Understanding YAML

## Pod Name

```yaml
metadata:
  name: multi-container-pod
```

Creates a Pod named:

```
multi-container-pod
```

---

## First Container

```yaml
name: nginx-container
image: nginx:latest
```

Runs nginx web server.

---

## Second Container

```yaml
name: busybox-container
image: busybox
```

Busybox is a lightweight Linux container.

It is useful for:

- Testing
- Debugging
- Network checks

---

# Step 3 - Create Pod

Go back to project root:

```bash
cd ../..
```

Apply YAML:

```bash
kubectl apply -f yaml/lab-03/multi-container-pod.yaml
```

Expected:

```
pod/multi-container-pod created
```

---

# Step 4 - Check Pod

Run:

```bash
kubectl get pods
```

Expected:

```
NAME                   READY   STATUS
multi-container-pod    2/2     Running
```

Explanation:

```
2/2 means:

2 containers created

2 containers running
```

---

# Step 5 - View Pod Details

Run:

```bash
kubectl describe pod multi-container-pod
```

Observe:

- Two containers
- Container images
- Container status
- Events

---

# Step 6 - List Containers Inside Pod

Command:

```bash
kubectl get pod multi-container-pod -o jsonpath='{.spec.containers[*].name}'
```

Expected:

```
nginx-container busybox-container
```

---

# Step 7 - Access nginx Container

Enter nginx container:

```bash
kubectl exec -it multi-container-pod -c nginx-container -- /bin/bash
```

Check nginx files:

```bash
ls /usr/share/nginx/html
```

Exit:

```bash
exit
```

---

# Step 8 - Access Busybox Container

Enter busybox:

```bash
kubectl exec -it multi-container-pod -c busybox-container -- sh
```

Inside container:

Check hostname:

```bash
hostname
```

Exit:

```bash
exit
```

---

# Step 9 - Test Container Communication

Enter busybox:

```bash
kubectl exec -it multi-container-pod -c busybox-container -- sh
```

Install curl:

```bash
wget -qO- http://localhost
```

You should see nginx HTML output.

Why?

Because both containers share the same network.

Exit:

```bash
exit
```

---

# Step 10 - View Logs

nginx logs:

```bash
kubectl logs multi-container-pod -c nginx-container
```

busybox logs:

```bash
kubectl logs multi-container-pod -c busybox-container
```

---

# Step 11 - Delete Pod

Delete:

```bash
kubectl delete pod multi-container-pod
```

Verify:

```bash
kubectl get pods
```

Expected:

```
No resources found
```

---

# Verification

Create again:

```bash
kubectl apply -f yaml/lab-03/multi-container-pod.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
multi-container-pod   2/2   Running
```

---

# Troubleshooting

## Problem: Pod shows 1/2 Running

Check:

```bash
kubectl describe pod multi-container-pod
```

Look at:

```
Events
```

---

## Problem: Container not starting

Check logs:

```bash
kubectl logs multi-container-pod -c container-name
```

---

# Interview Questions

## 1. Can a Pod have multiple containers?

Yes. A Pod can contain one or more containers.

---

## 2. How do containers communicate inside the same Pod?

Containers communicate using localhost because they share the same network namespace.

---

## 3. Do containers in the same Pod get different IP addresses?

No.

All containers share the same Pod IP address.

---

## 4. What is a sidecar container?

A sidecar container is a helper container running alongside the main application container.

Examples:

- Logging
- Monitoring
- Security

---

## 5. Difference between Pod and Container?

Container:

- Runs application

Pod:

- Manages one or more containers

---

# Commands Learned

```bash
kubectl get pods

kubectl describe pod multi-container-pod

kubectl exec -it multi-container-pod -c container-name -- sh

kubectl logs multi-container-pod -c container-name

kubectl delete pod multi-container-pod

kubectl apply -f multi-container-pod.yaml
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod multi-container-pod
```

---

# Navigation

Previous:

[Lab 02 - First Kubernetes Pod](lab-02-first-pod.md)

Next:

[Lab 04 - Labels and Selectors](lab-04-labels-selectors.md)

---

# Lab Completed ✅

You have successfully learned:

- Multi container Pods
- Sidecar pattern
- Shared networking
- Container communication
- Container troubleshooting ⬜ |
| 02 | [First Kubernetes Pod](labs/lab-02-first-pod.md) | ⬜ |
| 03 | [Multi-Container Pod](labs/lab-03-multi-container-pod.md) | ⬜ |
| 04 | [Labels and Selectors](labs/lab-04-labels-selectors.md) | ⬜ |
