# LAB 06 - Kubernetes Deployment

## Objective

In this lab, you will learn:

- What is a Kubernetes Deployment
- Relationship between Deployment and ReplicaSet
- Create a Deployment
- Scale applications
- Update applications
- Check rollout history
- Understand self-healing

---

# What is Deployment?

A Deployment is a Kubernetes object used to manage applications.

Deployment provides:

- Replica management
- Rolling updates
- Rollbacks
- Self-healing

Example:

You create:

```yaml
replicas: 3
```

Deployment creates:

```
Deployment

      |
      |
 ReplicaSet

      |
      |
+-----+-----+-----+

Pod   Pod   Pod
```

---

# Deployment vs ReplicaSet

## ReplicaSet

Responsible for:

- Maintaining Pod count
- Self-healing


## Deployment

Responsible for:

- Creating ReplicaSets
- Updating applications
- Rollbacks
- Version control


Real-world usage:

Normally we create Deployments, not ReplicaSets directly.

---

# Deployment Architecture

```
             Deployment

                  |

                  |

             ReplicaSet

                  |

        +---------+---------+

        |         |         |

       Pod       Pod       Pod
```

---

# Prerequisites

Check Kubernetes:

```bash
kubectl get nodes
```

Expected:

```
minikube   Ready
```

---

# Lab Steps

## Step 1 - Create Deployment YAML

Create file:

```bash
touch yaml/lab-06/deployment.yaml
```

Open:

```bash
code yaml/lab-06/deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-deployment


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

      - name: nginx-container
        image: nginx:1.25
        ports:
        - containerPort: 80
```

Save.

---

# Understanding YAML

## Kind

```yaml
kind: Deployment
```

Creates Deployment object.

---

## Replicas

```yaml
replicas: 3
```

Maintains three Pods.

---

## Selector

```yaml
matchLabels:
  app: nginx
```

Deployment manages Pods having:

```
app=nginx
```

---

## Container Image

```yaml
image: nginx:1.25
```

Runs nginx version 1.25.

---

# Step 2 - Create Deployment

Run:

```bash
kubectl apply -f yaml/lab-06/deployment.yaml
```

Expected:

```
deployment.apps/nginx-deployment created
```

---

# Step 3 - Check Deployment

```bash
kubectl get deployments
```

Expected:

```
NAME                READY   UP-TO-DATE

nginx-deployment    3/3     3
```

---

# Step 4 - Check ReplicaSet

Run:

```bash
kubectl get replicaset
```

Example:

```
nginx-deployment-xxxxx
```

Deployment automatically created ReplicaSet.

---

# Step 5 - Check Pods

```bash
kubectl get pods
```

Expected:

```
nginx-deployment-xxxxx   Running
nginx-deployment-xxxxx   Running
nginx-deployment-xxxxx   Running
```

---

# Step 6 - Scale Deployment

Current:

```
replicas: 3
```

Scale to 5:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check:

```bash
kubectl get pods
```

Now 5 Pods should run.

---

# Step 7 - Scale Down

Scale to 2:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Verify:

```bash
kubectl get pods
```

---

# Step 8 - Delete Pod and Test Self Healing

Find Pods:

```bash
kubectl get pods
```

Delete one:

```bash
kubectl delete pod <pod-name>
```

Watch:

```bash
kubectl get pods -w
```

Deployment creates a replacement Pod.

---

# Step 9 - View Deployment Details

```bash
kubectl describe deployment nginx-deployment
```

Observe:

- ReplicaSet
- Labels
- Events
- Image

---

# Step 10 - Check Rollout Status

```bash
kubectl rollout status deployment nginx-deployment
```

Expected:

```
successfully rolled out
```

---

# Step 11 - View Rollout History

```bash
kubectl rollout history deployment nginx-deployment
```

Example:

```
REVISION  CHANGE-CAUSE

1         <none>
```

---

# Verification

Run:

```bash
kubectl get deployment
kubectl get replicaset
kubectl get pods
```

All should be running.

---

# Troubleshooting

## Deployment not creating Pods

Check:

```bash
kubectl describe deployment nginx-deployment
```

Look at:

```
Events
```

---

## Pod not starting

Check:

```bash
kubectl describe pod <pod-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

---

# Interview Questions

## 1. What is Deployment in Kubernetes?

Deployment manages application lifecycle including scaling, updates, and rollback.

---

## 2. Deployment creates which object?

Deployment creates ReplicaSet.

---

## 3. Difference between Deployment and ReplicaSet?

ReplicaSet maintains Pods.

Deployment manages ReplicaSets and provides updates.

---

## 4. How do you check Deployment status?

Command:

```bash
kubectl rollout status deployment <name>
```

---

# Commands Learned

```bash
kubectl apply -f deployment.yaml

kubectl get deployment

kubectl get replicaset

kubectl get pods

kubectl scale deployment nginx-deployment --replicas=5

kubectl describe deployment nginx-deployment

kubectl rollout status deployment nginx-deployment

kubectl rollout history deployment nginx-deployment
```

---

# Cleanup

Delete Deployment:

```bash
kubectl delete deployment nginx-deployment
```

---

# Navigation

Previous:

[Lab 05 - ReplicaSet](lab-05-replicaset.md)

Next:

[Lab 07 - Rolling Update](lab-07-rolling-update.md)

---

# Lab Completed ✅

You learned:

- Deployment
- ReplicaSet relationship
- Scaling
- Self-healing
- Rollout management