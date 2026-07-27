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
Commands Learned
```bash
kubectl get pods

kubectl apply -f pod.yaml

kubectl describe pod nginx-pod

kubectl logs nginx-pod

kubectl exec -it nginx-pod -- /bin/bash

kubectl get pod nginx-pod -o yaml

kubectl delete pod nginx-pod
Cleanup

Delete Pod:

kubectl delete pod nginx-pod
```
## Navigation

Previous:

Lab 01 - Install Kubernetes

Next:

Lab 03 - Multi Container Pod

