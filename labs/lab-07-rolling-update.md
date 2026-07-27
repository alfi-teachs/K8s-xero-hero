# LAB 07 - Kubernetes Rolling Update

## Objective

In this lab, you will learn:

- What is Rolling Update
- How Kubernetes updates applications
- Deploy version 1 of an application
- Update application image
- Monitor rollout process
- Verify zero downtime deployment

---

# What is Rolling Update?

Rolling Update is the default Kubernetes deployment strategy.

It updates Pods gradually instead of deleting all Pods at once.

Example:

Current version:

```
nginx:v1

Pod 1
Pod 2
Pod 3
```

Update to:

```
nginx:v2
```

Kubernetes does:

```
Create new Pod v2

Remove old Pod v1

Create another Pod v2

Remove old Pod v1
```

Until all Pods run the new version.

---

# Why Rolling Updates?

Without rolling update:

```
Stop old application

       ↓

Deploy new application

       ↓

Downtime
```

With rolling update:

```
Old Pods running

       ↓

New Pods created

       ↓

Old Pods removed
```

No downtime.

---

# Deployment Strategy

Deployment supports:

```yaml
strategy:
  type: RollingUpdate
```

Default strategy:

```
RollingUpdate
```

---

# Prerequisites

Check cluster:

```bash
kubectl get nodes
```

Expected:

```
minikube   Ready
```

---

# Lab Steps

## Step 1 - Create Version 1 Deployment

Create:

```bash
touch yaml/lab-07/nginx-v1.yaml
```

Open:

```bash
code yaml/lab-07/nginx-v1.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-rollout


spec:

  replicas: 3


  strategy:
    type: RollingUpdate


  selector:
    matchLabels:
      app: nginx


  template:

    metadata:
      labels:
        app: nginx


    spec:

      containers:

      - name: nginx
        image: nginx:1.25

        ports:
        - containerPort: 80
```

Save.

---

# Step 2 - Deploy Version 1

Run:

```bash
kubectl apply -f yaml/lab-07/nginx-v1.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
3 nginx Pods running
```

---

# Step 3 - Check Deployment

```bash
kubectl get deployment
```

Output:

```
NAME             READY

nginx-rollout    3/3
```

---

# Step 4 - Check Current Image

Run:

```bash
kubectl describe deployment nginx-rollout
```

Look for:

```
Image: nginx:1.25
```

---

# Step 5 - Update Image Version

Update nginx:

```bash
kubectl set image deployment/nginx-rollout nginx=nginx:1.26
```

Expected:

```
deployment.apps/nginx-rollout image updated
```

---

# Step 6 - Watch Rolling Update

Run:

```bash
kubectl get pods -w
```

You will see:

```
nginx:v1 Terminating

nginx:v2 Creating

nginx:v2 Running
```

Kubernetes replaces Pods gradually.

---

# Step 7 - Check Rollout Status

Run:

```bash
kubectl rollout status deployment/nginx-rollout
```

Expected:

```
successfully rolled out
```

---

# Step 8 - Check New Image

Run:

```bash
kubectl describe deployment nginx-rollout
```

Verify:

```
Image: nginx:1.26
```

---

# Step 9 - View Rollout History

Run:

```bash
kubectl rollout history deployment/nginx-rollout
```

Example:

```
REVISION

1
2
```

Revision 1:

```
nginx:1.25
```

Revision 2:

```
nginx:1.26
```

---

# Step 10 - Check ReplicaSets

Run:

```bash
kubectl get replicaset
```

You will see:

```
nginx-rollout-old-version

nginx-rollout-new-version
```

Old ReplicaSet keeps history.

New ReplicaSet runs current version.

---

# Verification

Run:

```bash
kubectl get deployment

kubectl get pods

kubectl get replicaset

kubectl rollout history deployment/nginx-rollout
```

Everything should be healthy.

---

# Troubleshooting

## Update stuck

Check:

```bash
kubectl rollout status deployment/nginx-rollout
```

---

## Check Events

```bash
kubectl describe deployment nginx-rollout
```

---

## Check Pod Errors

```bash
kubectl describe pod <pod-name>
```

---

# Interview Questions

## 1. What is Rolling Update?

A deployment strategy where old Pods are replaced gradually with new Pods.

---

## 2. What is the default Deployment strategy?

RollingUpdate.

---

## 3. How do you update an image?

Command:

```bash
kubectl set image deployment/<name> container=image
```

---

## 4. How do you check rollout status?

Command:

```bash
kubectl rollout status deployment/<name>
```

---

## 5. What happens during Rolling Update?

New Pods are created before old Pods are removed.

---

# Commands Learned

```bash
kubectl apply -f nginx-v1.yaml

kubectl set image deployment/nginx-rollout nginx=nginx:1.26

kubectl get pods -w

kubectl rollout status deployment/nginx-rollout

kubectl rollout history deployment/nginx-rollout

kubectl get replicaset
```

---

# Cleanup

Delete deployment:

```bash
kubectl delete deployment nginx-rollout
```

---

# Navigation

Previous:

[Lab 06 - Deployment](lab-06-deployment.md)

Next:

[Lab 08 - Rollback](lab-08-rollback.md)

---

# Lab Completed ✅

You learned:

- Rolling Update
- Deployment strategy
- Image updates
- Zero downtime deployment
- Rollout history