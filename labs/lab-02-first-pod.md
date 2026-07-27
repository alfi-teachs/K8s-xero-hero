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
```bash
kind: Pod
metadata:
  name: nginx-pod
```
Then apply:
```bash
kubectl apply -f pod.yaml
```
Kubernetes creates the required resource.

#### Prerequisites

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

Lab Steps
### Step 1 - Check Existing Pods

Command:
```bash
kubectl get pods
```
Expected:

No resources found in default namespace.
### Step 2 - Create Pod YAML File

Go to YAML folder:
```bash
cd yaml/lab-02
```
Create file:
```bash
touch pod.yaml
```
Open:

code pod.yaml

Add:
```bash
apiVersion: v1

kind: Pod

metadata:
  name: nginx-pod
  labels:
    app: nginx

spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```
Save the file.


### Step 3 - Create Pod

Move back:

cd ../..

Apply YAML:
```bash
kubectl apply -f yaml/lab-02/pod.yaml
```
Expected:

pod/nginx-pod created
### Step 4 - Verify Pod

Check:
```bash
kubectl get pods
```
Expected:

NAME        READY   STATUS
nginx-pod   1/1     Running
### Step 5 - Watch Pod Creation

Command:
```bash
kubectl get pods -w
```
You will see:

- Pending
- ContainerCreating
- Running
```bash
CTRL + C
```
to exit.

### Step 6 - Describe Pod

Command:
```bash
kubectl describe pod nginx-pod
```
Check:
```bash
Pod name
Labels
Node
IP address
Container image
Events
```
### Step 7 - View Pod YAML

Command:
```bash
kubectl get pod nginx-pod -o yaml
```
This shows the complete Kubernetes object.

### Step 8 - View Pod Logs

Command:
```bash
kubectl logs nginx-pod
```
### Step 9 - Access Inside Container

Enter the container:
```bash
kubectl exec -it nginx-pod -- /bin/bash
```
If bash is unavailable:
```bash
kubectl exec -it nginx-pod -- /bin/sh
```
Inside container:

Check hostname:
```bash
hostname
```
Check files:
```bash
ls
```
Check running processes:
```bash
ps
```
Exit:

exit
### Step 10 - Delete Pod

Delete:
```bash
kubectl delete pod nginx-pod
```
Verify:
```bash
kubectl get pods
```
Expected:

No resources found
### Step 11 - Recreate Pod

Create again:
```bash
kubectl apply -f yaml/lab-02/pod.yaml
```
Verify:
```bash
kubectl get pods
```
## Verification

Run:
```bash
kubectl get pods
```
Expected:

nginx-pod   1/1   Running

Check details:
```bash
kubectl describe pod nginx-pod
```
Check logs:
```bash
kubectl logs nginx-pod
```
Access container:
```bash
kubectl exec -it nginx-pod -- /bin/bash
```
## Troubleshooting
Pod is stuck in Pending

Check:
```bash
kubectl describe pod nginx-pod
```
Look at:

Events:
- ImagePullBackOff Error
- Check image name:
- image: nginx:latest

View:
```bash
kubectl describe pod nginx-pod
```
Pod is not starting

Check logs:
```bash
kubectl logs nginx-pod
```
## Interview Questions
1. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes that contains one or more containers.

2. Why does Kubernetes use Pods instead of containers directly?

Pods provide:

Networking
Storage sharing
Lifecycle management
3. Difference between Pod and Container?

Container runs the application.

Pod manages one or more containers.

4. What is kubectl apply?

kubectl apply creates or updates Kubernetes resources from YAML files.

5. What is the difference between imperative and declarative approach?

Imperative:

Tell Kubernetes what to do

Example:
```bash
kubectl run nginx
```
### Declarative:

Describe desired state

Example:
```bash
kubectl apply -f pod.yaml
```
## Commands Learned
```bash
kubectl get pods

kubectl apply -f pod.yaml

kubectl describe pod nginx-pod

kubectl logs nginx-pod

kubectl exec -it nginx-pod -- /bin/bash

kubectl get pod nginx-pod -o yaml

kubectl delete pod nginx-pod
```
### Cleanup

Delete Pod:
```bash
kubectl delete pod nginx-pod
```
## navigation
# LAB 03 - Multi Container Pod

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
- Container troubleshooting

