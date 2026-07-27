# LAB 19 - Kubernetes Node Selector

## Objective

In this lab, you will learn:

- What is Node Selector
- What are Node Labels
- Add labels to Nodes
- Schedule Pods on specific Nodes
- Control Pod placement

---

# What is Node Selector?

Node Selector is a simple Kubernetes scheduling method that allows Pods to run only on Nodes with specific labels.

Example:

You have:

```
Node 1
Label:
environment=production


Node 2
Label:
environment=testing
```

You want your application to run only on production nodes.

Node Selector helps.

---

# How Node Selector Works

Flow:

```
Node

   |

Node Label

   |

Pod Specification

   |

nodeSelector

   |

Scheduler places Pod
```

---

# What is a Node Label?

A label is a key-value pair attached to Kubernetes objects.

Example:

```
environment=production
```

Labels help Kubernetes identify resources.

---

# Real World Example

Company has:

```
Worker Node 1
CPU: Normal


Worker Node 2
CPU: High Performance
Label:
type=gpu
```

Machine Learning application:

```
nodeSelector:

type=gpu
```

Pod runs only on GPU node.

---

# Node Selector vs Taints

| Node Selector | Taints |
|-|-|
| Selects where Pod should run | Prevents Pods from running |
| Uses labels | Uses restrictions |
| Pod chooses Node | Node rejects Pods |

---

# Prerequisites

Check Nodes:

```bash
kubectl get nodes
```

---

# Lab Steps

# Step 1 - Check Existing Nodes

Run:

```bash
kubectl get nodes
```

Example:

```
NAME
minikube
```

---

# Step 2 - View Node Labels

Run:

```bash
kubectl get nodes --show-labels
```

You will see existing labels.

---

# Step 3 - Add Label to Node

Add label:

```bash
kubectl label nodes minikube environment=production
```

Expected:

```
node/minikube labeled
```

---

# Step 4 - Verify Label

Run:

```bash
kubectl get nodes --show-labels
```

Look for:

```
environment=production
```

---

# Step 5 - Create Pod YAML With Node Selector

Create:

```bash
touch yaml/lab-19/node-selector-pod.yaml
```

Open:

```bash
code yaml/lab-19/node-selector-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: node-selector-demo


spec:

  nodeSelector:

    environment: production


  containers:

  - name: nginx

    image: nginx

    ports:

    - containerPort: 80
```

---

# Step 6 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-19/node-selector-pod.yaml
```

Expected:

```
pod/node-selector-demo created
```

---

# Step 7 - Verify Pod Location

Run:

```bash
kubectl get pod node-selector-demo -o wide
```

Example:

```
NAME                  NODE
node-selector-demo    minikube
```

The Pod is running on the Node with:

```
environment=production
```

---

# Step 8 - Describe Pod

Run:

```bash
kubectl describe pod node-selector-demo
```

Look for:

```
Node:
minikube
```

---

# Step 9 - Test Scheduling Failure

Remove label:

```bash
kubectl label nodes minikube environment-
```

Delete Pod:

```bash
kubectl delete pod node-selector-demo
```

Create again:

```bash
kubectl apply -f yaml/lab-19/node-selector-pod.yaml
```

Check:

```bash
kubectl get pods
```

You will see:

```
Pending
```

Reason:

No Node has matching label.

---

# Step 10 - Add Label Back

Run:

```bash
kubectl label nodes minikube environment=production
```

Delete and recreate:

```bash
kubectl delete pod node-selector-demo
```

```bash
kubectl apply -f yaml/lab-19/node-selector-pod.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
Running
```

---

# Verification

Check labels:

```bash
kubectl get nodes --show-labels
```

Check Pod:

```bash
kubectl get pods -o wide
```

Check details:

```bash
kubectl describe pod node-selector-demo
```

---

# Troubleshooting

## Pod stuck in Pending

Check:

```bash
kubectl describe pod node-selector-demo
```

Look for:

```
didn't match node selector
```

---

## Wrong Node Label

Check:

```bash
kubectl get nodes --show-labels
```

---

# Interview Questions

## 1. What is Node Selector?

Node Selector allows Pods to run only on Nodes with matching labels.

---

## 2. How do you add labels to Nodes?

Command:

```bash
kubectl label nodes NODE_NAME key=value
```

---

## 3. What happens if no Node matches?

Pod remains in Pending state.

---

## 4. Difference between Node Selector and Node Affinity?

Node Selector:

- Simple matching

Node Affinity:

- Advanced rules
- More flexibility

---

# Commands Learned

```bash
kubectl label nodes

kubectl get nodes --show-labels

kubectl get pods -o wide

kubectl describe pod

kubectl apply -f
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod node-selector-demo
```

Remove label:

```bash
kubectl label nodes minikube environment-
```

---

# Navigation

Previous:

[Lab 18 - Taints and Tolerations](lab-18-taints-tolerations.md)

Next:

[Lab 20 - Node Affinity](lab-20-node-affinity.md)

---

# Lab Completed ✅

You learned:

- Node labels
- Node Selector
- Pod scheduling
- Controlling workload placement