# LAB 04 - Labels and Selectors

## Objective

In this lab, you will learn:

- What Kubernetes Labels are
- Why Labels are important
- How to add Labels to Pods
- How Selectors work
- Filter resources using Labels
- Understand how Services use Selectors

---

# What You Will Learn

By the end of this lab, you will understand:

- metadata.labels
- label key-value pairs
- kubectl label command
- Label selectors
- Matching Pods using labels

---

# What are Kubernetes Labels?

Labels are key-value pairs attached to Kubernetes objects.

They help Kubernetes identify and organize resources.

Example:

```yaml
labels:
  app: nginx
  environment: development
```

Here:

```
Key              Value

app              nginx

environment      development
```

---

# Why Do We Need Labels?

Imagine you have 100 Pods running.

Without labels:

```
pod1
pod2
pod3
pod4
```

It becomes difficult to manage.

With labels:

```
Frontend Pods

app=frontend


Backend Pods

app=backend
```

Now Kubernetes can easily identify them.

---

# What are Selectors?

Selectors are used to find objects with specific labels.

Example:

Label:

```yaml
app: nginx
```

Selector:

```yaml
selector:
  app: nginx
```

Kubernetes matches objects where:

```
label = selector
```

---

# Real World Example

A Service needs to know which Pods receive traffic.

Service:

```
selector:
   app: nginx
```

Pods:

```
Pod 1
label:
 app: nginx


Pod 2
label:
 app: nginx


Pod 3
label:
 app: database
```

Traffic goes only to:

```
Pod 1
Pod 2
```

---

# Prerequisites

Required:

- Kubernetes cluster running
- Minikube started
- kubectl installed

Check:

```bash
kubectl get nodes
```

Expected:

```
minikube   Ready
```

---

# Lab Steps

## Step 1 - Create Pod YAML

Create file:

```bash
touch yaml/lab-04/labels-pod.yaml
```

Open:

```bash
code yaml/lab-04/labels-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: nginx-label-pod
  labels:
    app: nginx
    environment: dev

spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Save.

---

# Step 2 - Create Pod

Apply:

```bash
kubectl apply -f yaml/lab-04/labels-pod.yaml
```

Expected:

```
pod/nginx-label-pod created
```

---

# Step 3 - View Labels

Command:

```bash
kubectl get pods --show-labels
```

Output:

```
NAME              READY   STATUS    LABELS

nginx-label-pod   1/1     Running   app=nginx,environment=dev
```

---

# Step 4 - Filter Pods Using Labels

Command:

```bash
kubectl get pods -l app=nginx
```

Output:

```
nginx-label-pod
```

---

# Step 5 - Filter Environment Label

Command:

```bash
kubectl get pods -l environment=dev
```

Output:

```
nginx-label-pod
```

---

# Step 6 - Add Label Using Command

Add another label:

```bash
kubectl label pod nginx-label-pod tier=frontend
```

Check:

```bash
kubectl get pods --show-labels
```

You should see:

```
tier=frontend
```

---

# Step 7 - Remove Label

Remove label:

```bash
kubectl label pod nginx-label-pod tier-
```

Check:

```bash
kubectl get pods --show-labels
```

---

# Step 8 - Describe Pod Labels

Command:

```bash
kubectl describe pod nginx-label-pod
```

Look for:

```
Labels:
 app=nginx
 environment=dev
```

---

# Step 9 - Delete Pod

Delete:

```bash
kubectl delete pod nginx-label-pod
```

Verify:

```bash
kubectl get pods
```

---

# Verification

Create again:

```bash
kubectl apply -f yaml/lab-04/labels-pod.yaml
```

Check:

```bash
kubectl get pods --show-labels
```

Filter:

```bash
kubectl get pods -l app=nginx
```

---

# Troubleshooting

## Label selector returns no pods

Check labels:

```bash
kubectl get pods --show-labels
```

Make sure selector matches:

Example:

Pod:

```
app=nginx
```

Selector:

```
app=nginx
```

---

# Interview Questions

## 1. What are Kubernetes Labels?

Labels are key-value pairs used to organize and identify Kubernetes resources.

---

## 2. Why are Labels important?

Labels help Kubernetes:

- Group resources
- Select Pods
- Connect Services
- Manage deployments

---

## 3. What is a Selector?

A Selector searches Kubernetes objects based on labels.

---

## 4. How does a Service find Pods?

A Service uses selectors to match Pod labels.

---

# Commands Learned

```bash
kubectl get pods --show-labels

kubectl get pods -l app=nginx

kubectl label pod nginx-label-pod tier=frontend

kubectl label pod nginx-label-pod tier-

kubectl describe pod nginx-label-pod

kubectl apply -f labels-pod.yaml

kubectl delete pod nginx-label-pod
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod nginx-label-pod
```

---

# Navigation

Previous:

[Lab 03 - Multi Container Pod](lab-03-multi-container-pod.md)

Next:

[Lab 05 - ReplicaSet](lab-05-replicaset.md)

---

# Lab Completed ✅

You have successfully learned:

- Kubernetes Labels
- Label selectors
- Filtering Pods
- Adding and removing labels
- How Kubernetes connects resources using labels