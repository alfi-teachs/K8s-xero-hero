# LAB 05 - ReplicaSet

## Objective

In this lab, you will learn:

- What is a ReplicaSet
- Why ReplicaSets are used
- Desired state in Kubernetes
- Self-healing capability
- Create ReplicaSet using YAML
- Scale replicas
- Delete Pods and observe recreation

---

# What You Will Learn

By the end of this lab, you will understand:

- ReplicaSet
- Replica management
- Labels and selectors
- Self-healing
- Scaling applications

---

# What is a ReplicaSet?

A ReplicaSet ensures that a specific number of Pod replicas are running at all times.

Example:

You want:

```
3 nginx Pods
```

ReplicaSet maintains:

```
Desired Pods = 3

Running Pods = 3
```

If one Pod fails:

```
Before:

Pod 1
Pod 2
Pod 3


Pod 2 crashes


After:

Pod 1
Pod 3
New Pod
```

ReplicaSet automatically creates a replacement.

---

# Desired State

Kubernetes works on the desired state model.

Example:

You declare:

```yaml
replicas: 3
```

You are telling Kubernetes:

"I always want 3 Pods running."

Kubernetes continuously checks and maintains this state.

---

# ReplicaSet Architecture

```
              ReplicaSet

                   |

        +----------+----------+

        |          |          |

      Pod        Pod        Pod
```

ReplicaSet manages Pods using Labels.

---

# ReplicaSet vs Pod

Pod:

- Runs containers
- Can fail permanently

ReplicaSet:

- Creates Pods
- Maintains Pod count
- Provides self-healing

---

# Prerequisites

Required:

- Kubernetes running
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

## Step 1 - Create ReplicaSet YAML

Create file:

```bash
touch yaml/lab-05/replicaset.yaml
```

Open:

```bash
code yaml/lab-05/replicaset.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: ReplicaSet

metadata:
  name: nginx-replicaset

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
        image: nginx:latest
        ports:
        - containerPort: 80
```

Save.

---

# Understanding YAML

## Kind

```yaml
kind: ReplicaSet
```

Creates a ReplicaSet object.

---

## Replicas

```yaml
replicas: 3
```

Maintains three Pods.

---

## Selector

```yaml
selector:
  matchLabels:
    app: nginx
```

ReplicaSet manages Pods with:

```
app=nginx
```

---

## Template

Template defines the Pod that ReplicaSet creates.

---

# Step 2 - Create ReplicaSet

Run:

```bash
kubectl apply -f yaml/lab-05/replicaset.yaml
```

Expected:

```
replicaset.apps/nginx-replicaset created
```

---

# Step 3 - Check ReplicaSet

Run:

```bash
kubectl get replicaset
```

Expected:

```
NAME               DESIRED   CURRENT   READY

nginx-replicaset   3         3         3
```

---

# Step 4 - Check Pods

Run:

```bash
kubectl get pods
```

Expected:

```
nginx-replicaset-xxxxx   Running
nginx-replicaset-xxxxx   Running
nginx-replicaset-xxxxx   Running
```

---

# Step 5 - Delete One Pod

Find Pods:

```bash
kubectl get pods
```

Delete one:

```bash
kubectl delete pod <pod-name>
```

Example:

```bash
kubectl delete pod nginx-replicaset-abc12
```

---

# Step 6 - Watch Self Healing

Run:

```bash
kubectl get pods -w
```

You will see:

```
Terminating

New Pod Creating

Running
```

ReplicaSet automatically created a replacement.

---

# Step 7 - Scale ReplicaSet

Current:

```
replicas: 3
```

Scale to 5:

```bash
kubectl scale replicaset nginx-replicaset --replicas=5
```

Check:

```bash
kubectl get pods
```

Now:

```
5 Pods Running
```

---

# Step 8 - Scale Down

Reduce to 2:

```bash
kubectl scale replicaset nginx-replicaset --replicas=2
```

Check:

```bash
kubectl get pods
```

Now:

```
2 Pods Running
```

---

# Step 9 - Describe ReplicaSet

Run:

```bash
kubectl describe replicaset nginx-replicaset
```

Observe:

- Labels
- Selector
- Events
- Pod creation

---

# Verification

Run:

```bash
kubectl get replicaset
```

Expected:

```
nginx-replicaset   2   2   2
```

Check Pods:

```bash
kubectl get pods
```

---

# Troubleshooting

## ReplicaSet not creating Pods

Check:

```bash
kubectl describe replicaset nginx-replicaset
```

Look at:

```
Events
```

---

## Pods not matching ReplicaSet

Check labels:

```bash
kubectl get pods --show-labels
```

Selector and labels must match.

Example:

ReplicaSet:

```
app=nginx
```

Pod:

```
app=nginx
```

---

# Interview Questions

## 1. What is a ReplicaSet?

ReplicaSet maintains a stable number of Pod replicas.

---

## 2. What happens if a Pod managed by ReplicaSet fails?

ReplicaSet creates a new Pod automatically.

---

## 3. How does ReplicaSet identify Pods?

Using labels and selectors.

---

## 4. Difference between Pod and ReplicaSet?

Pod runs containers.

ReplicaSet manages Pods.

---

## 5. Why do we use Deployments instead of ReplicaSets directly?

Deployments provide:

- Rolling updates
- Rollbacks
- Version management

---

# Commands Learned

```bash
kubectl apply -f replicaset.yaml

kubectl get replicaset

kubectl get pods

kubectl delete pod pod-name

kubectl scale replicaset nginx-replicaset --replicas=5

kubectl describe replicaset nginx-replicaset
```

---

# Cleanup

Delete ReplicaSet:

```bash
kubectl delete replicaset nginx-replicaset
```

---

# Navigation

Previous:

[Lab 04 - Labels and Selectors](lab-04-labels-selectors.md)

Next:

[Lab 06 - Deployment](lab-06-deployment.md)

---

# Lab Completed ✅

You have successfully learned:

- ReplicaSet
- Desired state
- Self-healing
- Scaling Pods
- Labels and selectors