# LAB 08 - Kubernetes Rollback

## Objective

In this lab, you will learn:

- What is Kubernetes rollback
- Why rollback is required
- Deploy a working application version
- Deploy a bad image version
- Detect deployment failure
- Rollback to previous version
- Verify application recovery

---

# What is Rollback?

Rollback means returning an application to a previous working version.

Example:

Application Version 1:

```
nginx:1.25
```

Application Version 2:

```
nginx:wrong-version
```

If Version 2 fails:

Kubernetes can return to:

```
nginx:1.25
```

---

# Real World Example

A developer deploys a new application version.

After deployment:

- Pods are failing
- Users cannot access application
- Errors appear

Instead of manually fixing everything:

We rollback to the previous stable version.

---

# Deployment History

Kubernetes stores previous Deployment versions.

Example:

```
Revision 1
nginx:1.25

Revision 2
nginx:wrong

Revision 3
nginx:1.26
```

Rollback allows moving back.

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

## Step 1 - Create Working Deployment Version 1

Create file:

```bash
touch yaml/lab-08/nginx-v1.yaml
```

Open:

```bash
code yaml/lab-08/nginx-v1.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-rollback


spec:

  replicas: 3


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

---

# Step 2 - Deploy Version 1

Run:

```bash
kubectl apply -f yaml/lab-08/nginx-v1.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
3 Pods Running
```

---

# Step 3 - Check Rollout History

Run:

```bash
kubectl rollout history deployment/nginx-rollback
```

Expected:

```
REVISION
1
```

---

# Step 4 - Deploy Bad Image

Now simulate a failed deployment.

Run:

```bash
kubectl set image deployment/nginx-rollback nginx=nginx:badversion
```

---

# Step 5 - Watch Deployment Failure

Run:

```bash
kubectl get pods -w
```

You may see:

```
ImagePullBackOff
ErrImagePull
```

because:

```
nginx:badversion
```

does not exist.

---

# Step 6 - Check Deployment Status

Run:

```bash
kubectl rollout status deployment/nginx-rollback
```

You will see deployment is not successful.

---

# Step 7 - Check Rollout History

Run:

```bash
kubectl rollout history deployment/nginx-rollback
```

Example:

```
REVISION

1
2
```

Revision 2 contains the bad image.

---

# Step 8 - Rollback Deployment

Rollback:

```bash
kubectl rollout undo deployment/nginx-rollback
```

Expected:

```
deployment.apps/nginx-rollback rolled back
```

---

# Step 9 - Verify Recovery

Check Pods:

```bash
kubectl get pods
```

Expected:

```
3 Pods Running
```

---

Check image:

```bash
kubectl describe deployment nginx-rollback
```

Verify:

```
Image: nginx:1.25
```

---

# Step 10 - Check Rollout History

Run:

```bash
kubectl rollout history deployment/nginx-rollback
```

---

# Useful Rollback Commands

## View History

```bash
kubectl rollout history deployment/nginx-rollback
```

---

## Rollback Previous Version

```bash
kubectl rollout undo deployment/nginx-rollback
```

---

## Rollback Specific Revision

Example:

```bash
kubectl rollout undo deployment/nginx-rollback --to-revision=1
```

---

# Verification

Run:

```bash
kubectl get deployment

kubectl get pods

kubectl rollout history deployment/nginx-rollback
```

Application should be healthy.

---

# Troubleshooting

## Pods showing ImagePullBackOff

Check image:

```bash
kubectl describe pod <pod-name>
```

Look for:

```
Failed to pull image
```

---

## Rollback not working

Check:

```bash
kubectl rollout history deployment/nginx-rollback
```

---

# Interview Questions

## 1. What is Kubernetes rollback?

Rollback returns an application to a previous working version.

---

## 2. Why do we need rollback?

To quickly recover from failed deployments.

---

## 3. Command to rollback?

```bash
kubectl rollout undo deployment <name>
```

---

## 4. Where does Kubernetes store deployment history?

Deployment keeps ReplicaSet history.

---

## 5. Difference between Rolling Update and Rollback?

Rolling Update:

Deploys a new version.

Rollback:

Returns to an old version.

---

# Commands Learned

```bash
kubectl rollout history deployment/name

kubectl rollout status deployment/name

kubectl rollout undo deployment/name

kubectl rollout undo deployment/name --to-revision=1

kubectl set image deployment/name container=image
```

---

# Cleanup

Delete deployment:

```bash
kubectl delete deployment nginx-rollback
```

---

# Navigation

Previous:

[Lab 07 - Rolling Update](lab-07-rolling-update.md)

Next:

[Lab 09 - ClusterIP Service](lab-09-clusterip-service.md)

---

# Lab Completed ✅

You learned:

- Deployment history
- Failed deployments
- Rollback process
- Recovery strategy